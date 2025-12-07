# TDD Enforcement Guide

Test-Driven Development is **mandatory** for all new features and bug fixes.

---

## The True Nature of TDD: Hypothesis-Driven Development

TDD is not about testing. **TDD is about thinking clearly before coding.**

A test is a hypothesis expressed in code. When you write a test first, you're saying:

> "I hypothesize that when [condition], the system should [behavior]."

The test is your experiment. The implementation is your attempt to make the hypothesis true.

### Why This Framing Matters

Most developers learn TDD mechanically: "write test, make it pass, refactor." This misses the point. The power of TDD comes from **forcing clear thinking before coding**:

```
WITHOUT TDD (implementation-first):
  "I'll write some code and see if it works"
  → Vague understanding
  → Code reflects confusion
  → Tests written to match buggy code

WITH TDD (hypothesis-first):
  "I hypothesize the system should behave like THIS"
  → Clear expectation documented as test
  → Implementation guided by clear goal
  → Test proves hypothesis correct
```

### The Hypothesis Protocol

Before writing any test, state your hypothesis explicitly:

```
HYPOTHESIS: When [specific input/condition],
            the system should [specific observable behavior].

FALSIFICATION: The test fails if [specific failure condition].

EXPERIMENT: [The test code that proves or disproves this]
```

**Example:**

```
HYPOTHESIS: When a user submits an order with no items,
            the system should reject it with a 400 error.

FALSIFICATION: The test fails if the API returns any status
               other than 400, or if the error message doesn't
               mention the empty items.

EXPERIMENT:
  POST /api/orders with { customerId: "123", items: [] }
  Assert: status === 400
  Assert: response.error contains "at least one item"
```

This explicit framing prevents the common mistake of writing tests that don't actually test anything meaningful.

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
│  [ ] 1. STATE THE HYPOTHESIS                                       │
│        "I hypothesize that when [X], the system should [Y]"        │
│                                                                     │
│  [ ] 2. WRITE THE EXPERIMENT (the test)                            │
│        Actual test code that proves/disproves the hypothesis       │
│                                                                     │
│  [ ] 3. RUN THE EXPERIMENT (confirm RED state)                     │
│        Test must fail - proving hypothesis is not yet true         │
│                                                                     │
│  [ ] 4. MAKE THE HYPOTHESIS TRUE (implement)                       │
│        Write minimum code to make the test pass                    │
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
> "Before I implement, I need to form and test a hypothesis. Here's my hypothesis:
>
> **HYPOTHESIS:** When [specific condition], the system should [specific behavior].
>
> Here's the experiment (test) that will prove this hypothesis:
>
> [test code]
>
> Let me run this to confirm it fails (hypothesis not yet true), then I'll implement."

**When asked to "just do it quickly" or "skip tests this time":**
> "The test isn't extra work - it's how I clarify what needs to be built. Without stating the hypothesis first, I might build the wrong thing. Here's what I hypothesize you need:
>
> **HYPOTHESIS:** [clear statement of expected behavior]
>
> If this hypothesis is correct, here's the test that proves it:
>
> [test code]
>
> This takes 30 seconds and ensures we're building the right thing."

**When the user says tests seem unnecessary for this change:**
> "Even simple changes benefit from stating the hypothesis explicitly. Watch:
>
> **HYPOTHESIS:** When [condition], the system should [behavior].
>
> Is this hypothesis correct? If yes, this test documents it forever:
>
> [test code]
>
> If the hypothesis is wrong, we just saved ourselves from building the wrong thing."

**When the user insists on skipping tests:**
> "I understand the pressure, but writing the test IS thinking through the problem. Without it, I'm guessing. Here's my hypothesis for what you need:
>
> **HYPOTHESIS:** [clear statement]
>
> Does this match your expectation? If yes:
>
> [test code]
>
> Once this fails, I'll implement. Total time: under a minute, and we'll know it's right."

**When fixing a bug:**
> "Before I fix this bug, I need to understand it precisely. Here's my hypothesis:
>
> **HYPOTHESIS:** The bug occurs when [specific condition], causing [specific wrong behavior]. The correct behavior should be [expected behavior].
>
> This test will prove the hypothesis (reproduce the bug):
>
> [test code]
>
> Once this fails, I'll know exactly what to fix. Then the same test proves the fix works."

**When the hypothesis might be wrong:**
> "I'm going to state my hypothesis, but I may be wrong. Please correct me:
>
> **HYPOTHESIS:** When [condition], you expect [behavior].
>
> If this is correct, I'll write the test. If not, let's refine the hypothesis before I write any code."

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

## The TDD Cycle: Hypothesis → Experiment → Proof

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐             │
│   │  HYPOTHESIS │────▶│  EXPERIMENT │────▶│    PROOF    │             │
│   │    (RED)    │     │   (GREEN)   │     │ (REFACTOR)  │             │
│   └─────────────┘     └─────────────┘     └─────────────┘             │
│         │                                         │                    │
│         └─────────────────────────────────────────┘                    │
│                                                                         │
│   The cycle continues: each new behavior needs a new hypothesis        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1. HYPOTHESIS (RED) - State What Should Be True

**Before writing any implementation code:**

1. **State the hypothesis explicitly:**
   > "I hypothesize that when [X], the system should [Y]"

2. **Write the experiment (test) that would prove it:**
   - One hypothesis per test
   - Specific, observable, measurable outcome
   - Clear failure condition

3. **Run the experiment - it MUST fail:**
   - Failure proves the hypothesis is not yet true
   - If it passes, either the hypothesis is already true (feature exists) or your experiment is flawed

```
# The failing test is EVIDENCE that you understand the problem
# If you can't write a failing test, you don't understand what you're building
```

### 2. EXPERIMENT (GREEN) - Make the Hypothesis True

Write the **minimum code** necessary to make the test pass:
- Don't over-engineer - you're proving ONE hypothesis
- Don't add features the test doesn't require
- Don't worry about perfect code yet

```
# Run the test - it should pass now
# If it doesn't, your implementation doesn't satisfy the hypothesis
# Fix the implementation, not the test (the test IS the hypothesis)
```

**Key insight:** The test defines what "correct" means. If your implementation passes the test, the hypothesis is proven. If you think the test is wrong, you need to refine your hypothesis, not hack the test.

### 3. PROOF (REFACTOR) - Strengthen the Evidence

Now improve the code while keeping the hypothesis proven (tests green):
- Remove duplication
- Improve naming
- Simplify logic
- Extract methods/classes if needed

```
# Run tests after each refactoring step
# Tests must stay green - the hypothesis must remain proven
# If tests fail, you've broken the proof - revert and try again
```

### The Scientific Method Applied

| Scientific Method | TDD Equivalent |
|-------------------|----------------|
| Observe phenomenon | Understand the requirement |
| Form hypothesis | "When X, the system should Y" |
| Design experiment | Write the test |
| Run experiment | Run the test (should fail) |
| Analyze results | Failure confirms hypothesis is testable |
| Implement theory | Write code to pass the test |
| Verify reproducibility | Run test suite, refactor, tests stay green |

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

## Test Naming = Hypothesis Naming

Test names should read as hypotheses about system behavior:

**Good (reads as hypothesis):**
```
test_user_cannot_withdraw_more_than_balance
  → Hypothesis: A user cannot withdraw more than their balance

test_order_total_includes_tax_when_applicable
  → Hypothesis: Order totals include tax when tax is applicable

test_expired_token_returns_unauthorized
  → Hypothesis: An expired token causes an unauthorized response
```

**Bad (describes implementation, not hypothesis):**
```
test_withdraw_method      → What about it? What's the hypothesis?
test_calculate_total      → Calculate total what? When? What result?
test_validate_token       → Validation does what, exactly?
```

**The test name IS the hypothesis.** If you can't state the hypothesis clearly in the name, you don't understand what you're testing.

## Bug Fix Protocol: Hypothesis About What's Broken

When fixing a bug, you must form a hypothesis about the bug before fixing it:

### 1. Form the Bug Hypothesis

```
BUG HYPOTHESIS:
  OBSERVED: [What actually happens - the symptom]
  EXPECTED: [What should happen instead]
  CAUSE: I hypothesize this happens because [specific cause]
  PROOF: This test will prove the bug exists:
```

### 2. Write the Reproduction Test (RED)

Write a test that:
- Fails because of the bug (proving the bug exists)
- Will pass once the bug is fixed
- Documents exactly what the correct behavior should be

```
# If this test passes before you fix anything, your hypothesis is wrong
# Either the bug doesn't exist or you misunderstood it
```

### 3. Fix the Bug (GREEN)

Make the **minimal change** to pass the test:
- Don't fix things the test doesn't cover
- Don't refactor while fixing
- Focus only on proving your hypothesis correct

### 4. Verify the Hypothesis (REFACTOR)

- Run full test suite - no regressions
- Consider: does this fix reveal other related bugs?
- If yes, form new hypotheses for those bugs

**Why this matters:**
- The bug hypothesis forces you to understand root cause
- The test proves you understood correctly
- The test prevents the bug from ever returning silently

### Example

```
BUG HYPOTHESIS:
  OBSERVED: Users can place orders with negative quantities
  EXPECTED: Orders with negative quantities should be rejected
  CAUSE: The quantity validation only checks for zero, not negative
  PROOF: test_order_rejects_negative_quantity will fail

TEST:
  def test_order_rejects_negative_quantity():
      order = Order(item="widget", quantity=-5)
      with pytest.raises(ValidationError, match="positive"):
          order.validate()

FIX:
  - if quantity == 0:
  + if quantity <= 0:
      raise ValidationError("Quantity must be positive")
```

---

## Multiple Hypotheses: When Requirements Are Complex

For complex features, you'll have multiple hypotheses to test. State them all upfront:

```
FEATURE: User authentication

HYPOTHESIS 1: Valid credentials return an auth token
HYPOTHESIS 2: Invalid password returns 401 Unauthorized
HYPOTHESIS 3: Unknown user returns 401 Unauthorized (not 404, to prevent enumeration)
HYPOTHESIS 4: Token expires after configured timeout
HYPOTHESIS 5: Expired token returns 401 Unauthorized
HYPOTHESIS 6: Malformed token returns 400 Bad Request
```

Then work through them one at a time:
1. Write test for Hypothesis 1
2. Run test (fails)
3. Implement until it passes
4. Write test for Hypothesis 2
5. Run test (fails)
6. Implement until it passes
7. ...continue...

**Never implement multiple hypotheses at once.** Each hypothesis gets its own RED-GREEN-REFACTOR cycle.

This prevents the common mistake of writing all the code first and then writing tests to match what you built (which isn't TDD - it's testing-after).

## Test Quality Checklist

Before considering a test complete:

- [ ] **Hypothesis is clear** - Can state "When X, system should Y"
- [ ] **Test fails without implementation** - Hypothesis not yet proven
- [ ] **Test passes with implementation** - Hypothesis proven
- [ ] **Test name states the hypothesis** - Readable as a claim
- [ ] **Test is independent** - No shared state with other tests
- [ ] **Test is fast** - < 100ms for unit tests
- [ ] **Arrange/Act/Assert is clear** - Setup, execution, verification
- [ ] **Edge cases covered** - Additional hypotheses for boundaries

## Common Mistakes (Hypothesis Framing)

### 1. Coding Without a Hypothesis
❌ "I'll write some code and see if it works"
✅ "I hypothesize that when X happens, Y should result. Let me write a test to prove it."

### 2. Testing Implementation, Not Hypothesis
❌ "Test that processOrder calls saveToDatabase"
✅ "Test that a valid order is persisted and retrievable"

The first tests implementation details. The second tests observable behavior.

### 3. Multiple Hypotheses Per Test
❌ One test that checks login, validates token, AND verifies permissions
✅ Three separate tests, each proving one hypothesis

### 4. Vague Hypotheses
❌ "Test that the system works correctly"
✅ "Test that users with expired subscriptions cannot access premium content"

### 5. Writing Tests to Match Existing Code
❌ Code first, then write tests that pass against current behavior
✅ Hypothesis first, then test that fails, then code that passes

The first approach encodes bugs as expected behavior. The second proves correctness.

### 6. Abandoning Failing Tests
❌ "This test fails, let me comment it out"
✅ "This test fails, so either my hypothesis is wrong or my implementation is wrong. Let me figure out which."

A failing test is information. A skipped test is ignorance.

### 7. Not Running Tests During Development
❌ Write 200 lines of code, then run tests
✅ Write test, run (fails), write 5 lines, run (passes), refactor, run (still passes)

Each test run is an experiment. Run experiments frequently.

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

## Quick Reference: The Hypothesis-Driven TDD Protocol

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     TDD = HYPOTHESIS-DRIVEN DEVELOPMENT                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. STATE THE HYPOTHESIS                                               │
│     "I hypothesize that when [condition], the system should [behavior]"│
│                                                                         │
│  2. DESIGN THE EXPERIMENT                                              │
│     Write a test that proves or disproves the hypothesis               │
│                                                                         │
│  3. RUN THE EXPERIMENT                                                 │
│     Test must FAIL (hypothesis not yet true)                           │
│                                                                         │
│  4. MAKE HYPOTHESIS TRUE                                               │
│     Write minimum code to pass the test                                │
│                                                                         │
│  5. VERIFY & STRENGTHEN                                                │
│     Refactor while keeping tests green                                 │
│                                                                         │
│  6. REPEAT                                                             │
│     Next hypothesis, next test, next implementation                    │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│  REMEMBER: The test IS the hypothesis. The code IS the proof.          │
│            If you can't write a failing test, you don't understand     │
│            what you're building.                                       │
└─────────────────────────────────────────────────────────────────────────┘
```

## Related Guides

- [Iterative Problem Solving](iterative-problem-solving.md) - Systematic hypothesis-based debugging
- [Multi-Approach Validation](multi-approach-validation.md) - Testing multiple implementation approaches

## Further Reading

- [Test-Driven Development by Example](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530) - Kent Beck
- [Growing Object-Oriented Software, Guided by Tests](http://www.growing-object-oriented-software.com/) - Freeman & Pryce
- [The Pragmatic Programmer](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/) - Hunt & Thomas (on tracer bullets and prototypes)
