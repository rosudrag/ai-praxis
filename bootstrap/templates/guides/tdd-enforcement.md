# TDD Enforcement Guide

Test-Driven Development is **mandatory** for all new features and bug fixes.

---

## Enforcement Protocol

**THIS IS NON-NEGOTIABLE.** Claude MUST refuse to write implementation code until tests exist.

### The TDD Gate

Before writing ANY implementation code, Claude MUST pass through this gate:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TDD GATE CHECK                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  [ ] 1. STATE the test I will write (describe expected behavior)   │
│  [ ] 2. WRITE the test code (actual test, not pseudo-code)         │
│  [ ] 3. SHOW it failing (run test, confirm RED state)              │
│  [ ] 4. ONLY THEN implement the minimum code to pass               │
│                                                                     │
│  ALL BOXES MUST BE CHECKED BEFORE IMPLEMENTATION BEGINS            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### What Claude MUST Do

1. **When asked to implement a feature:**
   - STOP before writing any implementation
   - State: "Before I implement, I need to write a failing test first"
   - Write the test
   - Run the test to confirm it fails
   - Only then write implementation

2. **When asked to skip tests:**
   - REFUSE politely but firmly
   - Explain the test you would write
   - Proceed with test-first approach

3. **When "just this once" is requested:**
   - DO NOT make exceptions
   - The answer is always: test first

### What Claude MUST NOT Do

- Write implementation code before a failing test exists
- Accept "I'll add tests later" as a valid approach
- Skip tests because the change seems "too small"
- Write tests after implementation (this is not TDD)

---

## What Claude Must Say

### Exact Scripts for Common Situations

**When asked to implement without tests:**
> "I need to write a failing test first. Here's the test I'll create:
>
> [test code]
>
> Let me run this to confirm it fails, then I'll implement the solution."

**When asked to "just do it quickly" or "skip tests this time":**
> "Our TDD requirement ensures quality and actually saves time by catching issues early. Writing the test first takes 2 minutes and prevents 20 minutes of debugging later. Here's the test I'll write:
>
> [test code]
>
> This test documents exactly what we expect. Once it's failing, I'll implement."

**When the user says tests seem unnecessary for this change:**
> "Even for this change, a test provides value: it documents the expected behavior, prevents regression, and serves as living documentation. Here's a minimal test that covers this:
>
> [test code]
>
> This takes 30 seconds to write and protects this behavior forever."

**When the user insists on skipping tests:**
> "I understand the time pressure, but I'm not able to write implementation code without tests - it's a core requirement of this project. The good news is: the test I need to write is simple:
>
> [test code]
>
> Let's run this, see it fail, then I'll implement. Total added time: under a minute."

**When fixing a bug:**
> "Before I fix this bug, I need to write a test that reproduces it. This ensures:
> 1. We understand the root cause
> 2. The fix actually works
> 3. This bug can never return silently
>
> Here's the reproduction test:
>
> [test code]
>
> Let me run this to confirm it fails, then I'll fix it."

---

## Language-Specific Test Examples

### TypeScript with Jest

**Testing a function:**
```typescript
// user-service.test.ts
import { UserService } from './user-service';
import { UserRepository } from './user-repository';

describe('UserService', () => {
  describe('createUser', () => {
    it('should throw ValidationError when email is invalid', async () => {
      // Arrange
      const mockRepository = {
        save: jest.fn(),
        findByEmail: jest.fn().mockResolvedValue(null),
      } as unknown as UserRepository;
      const service = new UserService(mockRepository);

      // Act & Assert
      await expect(
        service.createUser({ name: 'John', email: 'invalid-email' })
      ).rejects.toThrow('Invalid email format');

      expect(mockRepository.save).not.toHaveBeenCalled();
    });

    it('should throw ConflictError when email already exists', async () => {
      // Arrange
      const existingUser = { id: '123', name: 'Jane', email: 'jane@example.com' };
      const mockRepository = {
        save: jest.fn(),
        findByEmail: jest.fn().mockResolvedValue(existingUser),
      } as unknown as UserRepository;
      const service = new UserService(mockRepository);

      // Act & Assert
      await expect(
        service.createUser({ name: 'John', email: 'jane@example.com' })
      ).rejects.toThrow('Email already registered');
    });

    it('should create user and return id when valid data provided', async () => {
      // Arrange
      const mockRepository = {
        save: jest.fn().mockResolvedValue({ id: 'new-123' }),
        findByEmail: jest.fn().mockResolvedValue(null),
      } as unknown as UserRepository;
      const service = new UserService(mockRepository);

      // Act
      const result = await service.createUser({
        name: 'John',
        email: 'john@example.com',
      });

      // Assert
      expect(result.id).toBe('new-123');
      expect(mockRepository.save).toHaveBeenCalledWith({
        name: 'John',
        email: 'john@example.com',
      });
    });
  });
});
```

**Testing an API endpoint:**
```typescript
// orders.controller.test.ts
import request from 'supertest';
import { app } from './app';
import { OrderRepository } from './order-repository';

jest.mock('./order-repository');

describe('POST /api/orders', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should return 400 when items array is empty', async () => {
    // Act
    const response = await request(app)
      .post('/api/orders')
      .send({ customerId: '123', items: [] })
      .expect(400);

    // Assert
    expect(response.body.error).toBe('Order must contain at least one item');
  });

  it('should return 201 and order id when order is valid', async () => {
    // Arrange
    (OrderRepository.prototype.create as jest.Mock).mockResolvedValue({
      id: 'order-456',
    });

    // Act
    const response = await request(app)
      .post('/api/orders')
      .send({
        customerId: '123',
        items: [{ productId: 'prod-1', quantity: 2 }],
      })
      .expect(201);

    // Assert
    expect(response.body.orderId).toBe('order-456');
  });
});
```

### C# with xUnit

**Testing a service:**
```csharp
// UserServiceTests.cs
using Xunit;
using Moq;
using FluentAssertions;
using System.Threading.Tasks;

namespace MyApp.Tests.Services;

public class UserServiceTests
{
    private readonly Mock<IUserRepository> _mockRepository;
    private readonly Mock<IEmailValidator> _mockEmailValidator;
    private readonly UserService _sut;

    public UserServiceTests()
    {
        _mockRepository = new Mock<IUserRepository>();
        _mockEmailValidator = new Mock<IEmailValidator>();
        _sut = new UserService(_mockRepository.Object, _mockEmailValidator.Object);
    }

    [Fact]
    public async Task CreateUser_WithInvalidEmail_ThrowsValidationException()
    {
        // Arrange
        _mockEmailValidator
            .Setup(v => v.IsValid(It.IsAny<string>()))
            .Returns(false);

        var request = new CreateUserRequest("John", "invalid-email");

        // Act
        var act = () => _sut.CreateUserAsync(request);

        // Assert
        await act.Should()
            .ThrowAsync<ValidationException>()
            .WithMessage("*Invalid email format*");

        _mockRepository.Verify(
            r => r.SaveAsync(It.IsAny<User>()),
            Times.Never);
    }

    [Fact]
    public async Task CreateUser_WithExistingEmail_ThrowsConflictException()
    {
        // Arrange
        var existingUser = new User { Id = Guid.NewGuid(), Email = "jane@example.com" };

        _mockEmailValidator.Setup(v => v.IsValid(It.IsAny<string>())).Returns(true);
        _mockRepository
            .Setup(r => r.FindByEmailAsync("jane@example.com"))
            .ReturnsAsync(existingUser);

        var request = new CreateUserRequest("John", "jane@example.com");

        // Act
        var act = () => _sut.CreateUserAsync(request);

        // Assert
        await act.Should()
            .ThrowAsync<ConflictException>()
            .WithMessage("*Email already registered*");
    }

    [Fact]
    public async Task CreateUser_WithValidData_ReturnsNewUserId()
    {
        // Arrange
        var expectedId = Guid.NewGuid();

        _mockEmailValidator.Setup(v => v.IsValid(It.IsAny<string>())).Returns(true);
        _mockRepository
            .Setup(r => r.FindByEmailAsync(It.IsAny<string>()))
            .ReturnsAsync((User?)null);
        _mockRepository
            .Setup(r => r.SaveAsync(It.IsAny<User>()))
            .ReturnsAsync(new User { Id = expectedId });

        var request = new CreateUserRequest("John", "john@example.com");

        // Act
        var result = await _sut.CreateUserAsync(request);

        // Assert
        result.Id.Should().Be(expectedId);
        _mockRepository.Verify(
            r => r.SaveAsync(It.Is<User>(u =>
                u.Name == "John" && u.Email == "john@example.com")),
            Times.Once);
    }
}
```

**Testing a controller:**
```csharp
// OrdersControllerTests.cs
using Xunit;
using Moq;
using FluentAssertions;
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;

namespace MyApp.Tests.Controllers;

public class OrdersControllerTests
{
    private readonly Mock<IOrderService> _mockOrderService;
    private readonly OrdersController _sut;

    public OrdersControllerTests()
    {
        _mockOrderService = new Mock<IOrderService>();
        _sut = new OrdersController(_mockOrderService.Object);
    }

    [Fact]
    public async Task CreateOrder_WithEmptyItems_ReturnsBadRequest()
    {
        // Arrange
        var request = new CreateOrderRequest
        {
            CustomerId = Guid.NewGuid(),
            Items = new List<OrderItemRequest>()
        };

        // Act
        var result = await _sut.CreateOrder(request);

        // Assert
        var badRequest = result.Should().BeOfType<BadRequestObjectResult>().Subject;
        var error = badRequest.Value as ApiError;
        error!.Message.Should().Be("Order must contain at least one item");
    }

    [Fact]
    public async Task CreateOrder_WithValidData_ReturnsCreatedWithOrderId()
    {
        // Arrange
        var expectedOrderId = Guid.NewGuid();
        _mockOrderService
            .Setup(s => s.CreateOrderAsync(It.IsAny<CreateOrderCommand>()))
            .ReturnsAsync(new OrderResult { OrderId = expectedOrderId });

        var request = new CreateOrderRequest
        {
            CustomerId = Guid.NewGuid(),
            Items = new List<OrderItemRequest>
            {
                new() { ProductId = Guid.NewGuid(), Quantity = 2 }
            }
        };

        // Act
        var result = await _sut.CreateOrder(request);

        // Assert
        var created = result.Should().BeOfType<CreatedAtActionResult>().Subject;
        var response = created.Value as CreateOrderResponse;
        response!.OrderId.Should().Be(expectedOrderId);
    }
}
```

### Python with pytest

**Testing a service:**
```python
# test_user_service.py
import pytest
from unittest.mock import Mock, AsyncMock
from user_service import UserService
from exceptions import ValidationError, ConflictError


@pytest.fixture
def mock_repository():
    return Mock()


@pytest.fixture
def mock_email_validator():
    return Mock()


@pytest.fixture
def user_service(mock_repository, mock_email_validator):
    return UserService(
        repository=mock_repository,
        email_validator=mock_email_validator
    )


class TestCreateUser:
    async def test_raises_validation_error_when_email_invalid(
        self, user_service, mock_repository, mock_email_validator
    ):
        # Arrange
        mock_email_validator.is_valid.return_value = False

        # Act & Assert
        with pytest.raises(ValidationError, match="Invalid email format"):
            await user_service.create_user(
                name="John",
                email="invalid-email"
            )

        mock_repository.save.assert_not_called()

    async def test_raises_conflict_error_when_email_exists(
        self, user_service, mock_repository, mock_email_validator
    ):
        # Arrange
        mock_email_validator.is_valid.return_value = True
        mock_repository.find_by_email = AsyncMock(
            return_value={"id": "123", "email": "jane@example.com"}
        )

        # Act & Assert
        with pytest.raises(ConflictError, match="Email already registered"):
            await user_service.create_user(
                name="John",
                email="jane@example.com"
            )

    async def test_creates_user_and_returns_id_when_valid(
        self, user_service, mock_repository, mock_email_validator
    ):
        # Arrange
        mock_email_validator.is_valid.return_value = True
        mock_repository.find_by_email = AsyncMock(return_value=None)
        mock_repository.save = AsyncMock(return_value={"id": "new-123"})

        # Act
        result = await user_service.create_user(
            name="John",
            email="john@example.com"
        )

        # Assert
        assert result["id"] == "new-123"
        mock_repository.save.assert_called_once_with({
            "name": "John",
            "email": "john@example.com"
        })


class TestDeleteUser:
    async def test_raises_not_found_when_user_does_not_exist(
        self, user_service, mock_repository
    ):
        # Arrange
        mock_repository.find_by_id = AsyncMock(return_value=None)

        # Act & Assert
        with pytest.raises(NotFoundError, match="User not found"):
            await user_service.delete_user(user_id="nonexistent")

    async def test_deletes_user_when_exists(
        self, user_service, mock_repository
    ):
        # Arrange
        existing_user = {"id": "123", "name": "John"}
        mock_repository.find_by_id = AsyncMock(return_value=existing_user)
        mock_repository.delete = AsyncMock()

        # Act
        await user_service.delete_user(user_id="123")

        # Assert
        mock_repository.delete.assert_called_once_with("123")
```

**Testing a FastAPI endpoint:**
```python
# test_orders_api.py
import pytest
from httpx import AsyncClient
from unittest.mock import AsyncMock, patch
from main import app


@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac


class TestCreateOrder:
    async def test_returns_400_when_items_empty(self, client):
        # Act
        response = await client.post(
            "/api/orders",
            json={"customer_id": "123", "items": []}
        )

        # Assert
        assert response.status_code == 400
        assert response.json()["detail"] == "Order must contain at least one item"

    async def test_returns_201_with_order_id_when_valid(self, client):
        # Arrange
        with patch("services.order_service.OrderService.create_order") as mock_create:
            mock_create.return_value = {"order_id": "order-456"}

            # Act
            response = await client.post(
                "/api/orders",
                json={
                    "customer_id": "123",
                    "items": [{"product_id": "prod-1", "quantity": 2}]
                }
            )

        # Assert
        assert response.status_code == 201
        assert response.json()["order_id"] == "order-456"

    async def test_returns_404_when_customer_not_found(self, client):
        # Arrange
        with patch("services.order_service.OrderService.create_order") as mock_create:
            mock_create.side_effect = CustomerNotFoundError("Customer not found")

            # Act
            response = await client.post(
                "/api/orders",
                json={
                    "customer_id": "nonexistent",
                    "items": [{"product_id": "prod-1", "quantity": 1}]
                }
            )

        # Assert
        assert response.status_code == 404
        assert "Customer not found" in response.json()["detail"]
```

---

## The TDD Cycle

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   ┌─────────┐     ┌─────────┐     ┌──────────┐        │
│   │   RED   │────▶│  GREEN  │────▶│ REFACTOR │        │
│   └─────────┘     └─────────┘     └──────────┘        │
│        │                                   │           │
│        └───────────────────────────────────┘           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1. RED - Write a Failing Test

**Before writing any implementation code**, write a test that:
- Describes the expected behavior
- Fails because the feature doesn't exist yet
- Is specific and focused (one behavior per test)

```
# Run the test - it MUST fail
# If it passes, your test is wrong or the feature already exists
```

### 2. GREEN - Make It Pass

Write the **minimum code** necessary to make the test pass:
- Don't over-engineer
- Don't add features the test doesn't require
- Don't worry about perfect code yet

```
# Run the test - it should pass now
# If it doesn't, fix the implementation (not the test)
```

### 3. REFACTOR - Clean Up

Now improve the code while keeping tests green:
- Remove duplication
- Improve naming
- Simplify logic
- Extract methods/classes if needed

```
# Run tests after each refactoring step
# Tests must stay green throughout
```

## Test Structure

Use the **Arrange-Act-Assert** pattern:

```
Arrange: Set up test conditions and inputs
Act:     Execute the code under test
Assert:  Verify the expected outcome
```

## What To Test

### DO Test:
- Business logic and rules
- Edge cases and boundary conditions
- Error handling paths
- Public interfaces and contracts

### DON'T Test:
- Framework code (it's already tested)
- Private methods directly (test through public interface)
- Trivial code (simple getters/setters)
- External services (mock them)

## Test Naming

Name tests to describe behavior, not implementation:

**Good:**
```
test_user_cannot_withdraw_more_than_balance
test_order_total_includes_tax_when_applicable
test_expired_token_returns_unauthorized
```

**Bad:**
```
test_withdraw_method
test_calculate_total
test_validate_token
```

## Bug Fix Protocol

When fixing a bug:

1. **Write a test that reproduces the bug** (RED)
   - The test should fail, proving the bug exists

2. **Fix the bug** (GREEN)
   - Make the minimal change to pass the test

3. **Verify no regression** (REFACTOR)
   - Run full test suite
   - Check related functionality

This ensures:
- The bug is documented
- It can't reappear silently
- You understand the root cause

## Test Quality Checklist

Before considering a test complete:

- [ ] Test fails without the implementation
- [ ] Test passes with the implementation
- [ ] Test name describes the behavior
- [ ] Test is independent (no shared state)
- [ ] Test is fast (< 100ms for unit tests)
- [ ] Test has clear arrange/act/assert sections
- [ ] Edge cases are covered

## Common Mistakes

### 1. Writing Implementation First
❌ "I'll write tests after the code works"
✅ Write the test FIRST, then make it pass

### 2. Testing Too Much at Once
❌ One test covering multiple behaviors
✅ One focused test per behavior

### 3. Testing Implementation Details
❌ Testing that a specific private method was called
✅ Testing the observable behavior/output

### 4. Ignoring Failing Tests
❌ Commenting out or skipping failing tests
✅ Fix the code or update the test (with good reason)

### 5. Not Running Tests Frequently
❌ Writing lots of code, then running tests
✅ Run tests after every small change

## When To Skip TDD

TDD may be skipped ONLY for:
- Exploratory/spike code (but add tests before merging)
- Pure UI layout (visual testing preferred)
- Generated code (test the generator instead)

Document in commit message why TDD was skipped.

## Integration with CI

All tests must pass before:
- Merging pull requests
- Deploying to any environment
- Considering a task "done"

## Further Reading

- [Test-Driven Development by Example](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530) - Kent Beck
- [Growing Object-Oriented Software, Guided by Tests](http://www.growing-object-oriented-software.com/) - Freeman & Pryce
