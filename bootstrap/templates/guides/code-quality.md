# Code Quality Principles

Guidelines for writing clean, maintainable code.

## Core Philosophy

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler

## Self-Documenting Code

### Names Tell the Story

Code should read like prose. If you need a comment to explain what code does, the code needs better names.

**Variables:**
```
❌ d = 7          # days until expiry
✅ daysUntilExpiry = 7

❌ tmp = users.filter(u => u.a)
✅ activeUsers = users.filter(user => user.isActive)
```

**Functions:**
```
❌ process(data)
✅ validateAndStoreUserSubmission(formData)

❌ calc(a, b)
✅ calculateOrderTotalWithTax(subtotal, taxRate)
```

**Classes:**
```
❌ Manager, Handler, Processor  (too vague)
✅ PaymentGateway, OrderValidator, EmailSender
```

### Functions Should Do One Thing

A function should:
- Fit on one screen (roughly 20-30 lines max)
- Have one reason to change
- Be testable in isolation

If you're using "and" to describe a function, split it.

### Comments: Why, Not What

**Bad comments** (explain what):
```
// Loop through users
for (user in users) {
    // Check if active
    if (user.isActive) {
        // Add to list
        result.add(user)
    }
}
```

**Good comments** (explain why):
```
// Active users are processed first per SLA requirements (ADR-012)
for (user in activeUsers) {
    processUser(user)
}

// Legacy systems expect null, not empty string (known issue #4521)
return value ?? null
```

## Complexity Management

### Avoid Deep Nesting

Maximum nesting depth: 3 levels

```
❌ Deep nesting:
if (user) {
    if (user.isActive) {
        if (user.hasPermission) {
            if (order.isValid) {
                // actual logic buried here
            }
        }
    }
}

✅ Early returns:
if (!user) return
if (!user.isActive) return
if (!user.hasPermission) return
if (!order.isValid) return

// actual logic at top level
```

### Guard Clauses

Handle edge cases first, then the main logic:

```
function processOrder(order) {
    // Guards first
    if (!order) throw new Error("Order required")
    if (order.items.isEmpty()) return EmptyResult
    if (order.isPaid) return AlreadyPaidResult

    // Main logic - no nesting needed
    const total = calculateTotal(order)
    const payment = processPayment(total)
    return createReceipt(order, payment)
}
```

### Extract Till You Drop

When code gets complex, extract smaller functions:

```
❌ One big function:
function processOrder(order) {
    // 50 lines of validation
    // 30 lines of calculation
    // 40 lines of payment processing
    // 20 lines of notification
}

✅ Composed functions:
function processOrder(order) {
    validateOrder(order)
    const total = calculateOrderTotal(order)
    const payment = processPayment(order, total)
    notifyCustomer(order, payment)
    return createOrderReceipt(order, payment)
}
```

## Code Organization

### File Structure

One concept per file. If a file has multiple unrelated things, split it.

### Import Organization

Group imports logically:
1. Standard library
2. Third-party packages
3. Internal modules
4. Relative imports

### Consistent Patterns

Within a codebase, similar things should look similar:
- If one controller uses async/await, all should
- If one service uses dependency injection, all should
- If one test uses describe/it, all should

## Error Handling

### Be Specific

```
❌ catch (e) { throw new Error("Failed") }
✅ catch (e) { throw new PaymentProcessingError(`Payment failed: ${e.message}`, e) }
```

### Fail Fast

Check preconditions early and fail immediately:

```
function transferFunds(from, to, amount) {
    if (amount <= 0) throw new InvalidAmountError(amount)
    if (!from.hasBalance(amount)) throw new InsufficientFundsError(from, amount)
    if (from.isFrozen) throw new AccountFrozenError(from)

    // Now proceed with confidence
    from.withdraw(amount)
    to.deposit(amount)
}
```

### Don't Swallow Errors

```
❌ catch (e) { /* ignore */ }
❌ catch (e) { console.log(e) }  // log and continue

✅ catch (e) {
    logger.error("Payment failed", { error: e, orderId: order.id })
    throw new PaymentError("Could not process payment", e)
}
```

## Magic Numbers and Strings

Extract constants with meaningful names:

```
❌ if (user.age >= 18)
❌ if (retries > 3)
❌ if (status === "ACTIVE")

✅ const LEGAL_ADULT_AGE = 18
✅ const MAX_RETRY_ATTEMPTS = 3
✅ const UserStatus = { ACTIVE: "ACTIVE", SUSPENDED: "SUSPENDED" }

✅ if (user.age >= LEGAL_ADULT_AGE)
✅ if (retries > MAX_RETRY_ATTEMPTS)
✅ if (status === UserStatus.ACTIVE)
```

## Dependencies

### Depend on Abstractions

```
❌ class OrderService {
    constructor() {
        this.db = new PostgresDatabase()  // Concrete dependency
    }
}

✅ class OrderService {
    constructor(repository: OrderRepository) {  // Abstract dependency
        this.repository = repository
    }
}
```

### Keep Dependencies Explicit

Don't hide dependencies:

```
❌ function sendEmail(to, subject) {
    const client = EmailService.getInstance()  // Hidden dependency
    client.send(to, subject)
}

✅ function sendEmail(emailClient, to, subject) {
    emailClient.send(to, subject)  // Explicit, testable
}
```

## Code Smells to Watch For

| Smell | Symptom | Fix |
|-------|---------|-----|
| Long Method | > 30 lines | Extract methods |
| Long Parameter List | > 4 parameters | Use parameter object |
| Duplicate Code | Same code in multiple places | Extract shared function |
| Feature Envy | Method uses other class's data more than its own | Move method |
| Data Clumps | Same fields always appear together | Create a class |
| Primitive Obsession | Using primitives instead of small objects | Create value objects |
| Shotgun Surgery | One change requires many file edits | Consolidate logic |

## Quality Checklist

Before considering code complete:

- [ ] Names are clear and descriptive
- [ ] Functions do one thing
- [ ] No deep nesting (max 3 levels)
- [ ] No magic numbers/strings
- [ ] Errors are handled explicitly
- [ ] Dependencies are explicit
- [ ] Tests are passing
- [ ] Code is formatted consistently

## Language-Specific Patterns

### TypeScript

#### Proper Type Narrowing vs Type Assertions

Type assertions (`as`) bypass the type system. Type narrowing proves to the compiler that a type is what you expect.

```typescript
// Bad: Type assertion bypasses safety
function processUser(data: unknown) {
    const user = data as User;  // No runtime check - will crash if data isn't User
    console.log(user.email.toLowerCase());  // Runtime error if email is undefined
}

// Also bad: Double assertion to force types
const id = (response as any as number);  // "Trust me bro" typing
```

```typescript
// Good: Type narrowing with runtime checks
function isUser(data: unknown): data is User {
    return (
        typeof data === 'object' &&
        data !== null &&
        'email' in data &&
        typeof (data as Record<string, unknown>).email === 'string'
    );
}

function processUser(data: unknown) {
    if (!isUser(data)) {
        throw new InvalidDataError('Expected User object');
    }
    // TypeScript now knows data is User
    console.log(data.email.toLowerCase());  // Safe
}
```

**When to use:** Always prefer narrowing for data from external sources (APIs, user input, file reads). Use assertions only when you have information the compiler cannot infer (e.g., after a DOM query you know succeeds).

#### Async/Await Error Handling

```typescript
// Bad: Unhandled promise rejection
async function fetchUserData(userId: string) {
    const response = await fetch(`/api/users/${userId}`);
    const data = await response.json();  // Crashes if response is not JSON
    return data;
}

// Also bad: Generic catch that loses error context
async function saveOrder(order: Order) {
    try {
        await database.save(order);
        await emailService.sendConfirmation(order);
        await analyticsService.track('order_created', order);
    } catch (e) {
        console.log('Something failed');  // Which step? What error?
        throw e;
    }
}
```

```typescript
// Good: Explicit error handling with context
async function fetchUserData(userId: string): Promise<Result<User, FetchError>> {
    let response: Response;
    try {
        response = await fetch(`/api/users/${userId}`);
    } catch (error) {
        return { ok: false, error: new NetworkError('Failed to reach API', { cause: error }) };
    }

    if (!response.ok) {
        return { ok: false, error: new ApiError(`API returned ${response.status}`, response) };
    }

    try {
        const data = await response.json();
        if (!isUser(data)) {
            return { ok: false, error: new ValidationError('Invalid user data shape') };
        }
        return { ok: true, value: data };
    } catch (error) {
        return { ok: false, error: new ParseError('Response was not valid JSON', { cause: error }) };
    }
}

// Good: Separate try-catch blocks for different failure modes
async function saveOrder(order: Order): Promise<void> {
    await database.save(order);  // Let critical errors propagate

    // Non-critical operations - log but don't fail the order
    try {
        await emailService.sendConfirmation(order);
    } catch (error) {
        logger.warn('Failed to send confirmation email', { orderId: order.id, error });
        await alerting.notify('email_failure', { orderId: order.id });
    }

    try {
        await analyticsService.track('order_created', order);
    } catch (error) {
        logger.warn('Failed to track analytics', { orderId: order.id, error });
    }
}
```

**When to use:** Use Result types for operations that can fail in expected ways. Use separate try-catch blocks when different failures need different handling. Let truly unexpected errors propagate.

#### Null Checking with Strict Mode

```typescript
// Bad: Optional chaining everywhere hides bugs
function getOrderTotal(user?: User): number {
    return user?.orders?.reduce((sum, o) => sum + (o?.total ?? 0), 0) ?? 0;
    // If user is unexpectedly undefined, we return 0 silently - hiding a bug
}

// Also bad: Non-null assertion operator
function processOrder(order: Order | null) {
    const items = order!.items;  // Crashes at runtime if null
    items!.forEach(item => {     // Compounds the problem
        updateInventory(item!.productId);
    });
}
```

```typescript
// Good: Explicit null checks with clear failure modes
function getOrderTotal(user: User): number {
    // Caller must provide a user - null check is caller's responsibility
    return user.orders.reduce((sum, order) => sum + order.total, 0);
}

// Good: Handle null at boundaries, then work with non-null types
function processOrder(order: Order | null): ProcessResult {
    if (order === null) {
        return { success: false, error: 'No order provided' };
    }

    // From here, order is definitely Order
    for (const item of order.items) {
        updateInventory(item.productId);
    }
    return { success: true };
}

// Good: Use optional chaining for truly optional data
interface UserProfile {
    name: string;
    preferences?: {
        theme?: 'light' | 'dark';
        notifications?: boolean;
    };
}

function getTheme(profile: UserProfile): 'light' | 'dark' {
    return profile.preferences?.theme ?? 'light';  // Appropriate - preferences are optional
}
```

**When to use:** Use optional chaining for genuinely optional data (user preferences, optional API fields). Use explicit null checks at system boundaries. Avoid `!` except when you can prove to humans (via comments) why it's safe.

#### Interface vs Type Usage

```typescript
// Bad: Using type for object shapes that may be extended
type ApiResponse = {
    status: number;
    data: unknown;
};

type UserApiResponse = ApiResponse & {
    data: User;
};  // Works but less clear intent

// Bad: Using interface for unions/computed types
interface Status = 'pending' | 'active' | 'closed';  // Syntax error
interface StringOrNumber = string | number;  // Syntax error
```

```typescript
// Good: Use interface for object shapes - enables declaration merging and clear extension
interface ApiResponse {
    status: number;
    data: unknown;
}

interface UserApiResponse extends ApiResponse {
    data: User;
}

// Interfaces can be extended by third-party code if needed
declare module './api' {
    interface ApiResponse {
        requestId: string;  // Added by logging middleware
    }
}

// Good: Use type for unions, intersections, and computed types
type OrderStatus = 'pending' | 'processing' | 'shipped' | 'delivered';
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };
type Nullable<T> = T | null;
type UserKeys = keyof User;
type PartialUser = Partial<User>;

// Good: Use type for function signatures
type EventHandler<T> = (event: T) => void;
type AsyncOperation<T> = () => Promise<T>;
```

**When to use:** Use `interface` for object shapes that represent contracts (DTOs, props, service interfaces). Use `type` for unions, mapped types, conditional types, and function signatures. Be consistent within a codebase.

---

### C#

#### Nullable Reference Types Usage

```csharp
// Bad: Ignoring nullable warnings
public class UserService
{
    public string GetUserEmail(int userId)
    {
        var user = _repository.Find(userId);  // Returns User?
        return user.Email;  // CS8602: Dereference of possibly null reference
    }

    public void ProcessUser(User? user)
    {
        var name = user!.Name;  // Null-forgiving operator hides the bug
        Console.WriteLine(name.ToUpper());  // Crashes if user was null
    }
}
```

```csharp
// Good: Explicit null handling
public class UserService
{
    public string? GetUserEmail(int userId)
    {
        var user = _repository.Find(userId);
        return user?.Email;  // Caller knows to handle null
    }

    public string GetUserEmailOrThrow(int userId)
    {
        var user = _repository.Find(userId)
            ?? throw new UserNotFoundException(userId);
        return user.Email;  // Safe - we've proven user is not null
    }

    public void ProcessUser(User? user)
    {
        if (user is null)
        {
            _logger.LogWarning("Attempted to process null user");
            return;
        }

        // Pattern matching has narrowed the type
        Console.WriteLine(user.Name.ToUpper());
    }

    // Good: Using required members for non-null properties
    public record CreateUserRequest
    {
        public required string Email { get; init; }
        public required string Name { get; init; }
        public string? PhoneNumber { get; init; }  // Explicitly optional
    }
}
```

**When to use:** Enable nullable reference types project-wide (`<Nullable>enable</Nullable>`). Make null-safety explicit in signatures. Use `required` for properties that must be set. Avoid `!` except when interfacing with legacy code.

#### Async/Await with ConfigureAwait

```csharp
// Bad: Not considering synchronization context in libraries
public class DataService
{
    public async Task<User> GetUserAsync(int id)
    {
        var data = await _httpClient.GetStringAsync($"/users/{id}");
        var user = JsonSerializer.Deserialize<User>(data);  // May deadlock in UI apps
        return await _cache.GetOrAddAsync(id, user);         // Context switches add overhead
    }
}

// Also bad: Using .Result or .Wait() to block on async
public User GetUser(int id)
{
    return GetUserAsync(id).Result;  // Deadlock waiting to happen
}
```

```csharp
// Good: ConfigureAwait(false) in library code
public class DataService
{
    public async Task<User> GetUserAsync(int id, CancellationToken ct = default)
    {
        var data = await _httpClient.GetStringAsync($"/users/{id}", ct)
            .ConfigureAwait(false);

        var user = JsonSerializer.Deserialize<User>(data);

        return await _cache.GetOrAddAsync(id, user, ct)
            .ConfigureAwait(false);
    }
}

// Good: If you must have sync-over-async (rare), be explicit about it
public User GetUserBlocking(int id)
{
    // Only use this when you absolutely cannot use async
    // Document why this is necessary
    return Task.Run(() => GetUserAsync(id)).GetAwaiter().GetResult();
}

// Good: Async all the way in application code (no ConfigureAwait needed)
public class UserController : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id, CancellationToken ct)
    {
        var user = await _userService.GetUserAsync(id, ct);  // Context preserved for MVC
        return Ok(user);
    }
}
```

**When to use:** Use `ConfigureAwait(false)` in library code and non-UI application layers. Omit it in UI event handlers and ASP.NET controller actions where you need the context. Always pass CancellationToken through async chains.

#### LINQ Best Practices

```csharp
// Bad: Multiple enumerations of IEnumerable
public void ProcessOrders(IEnumerable<Order> orders)
{
    if (!orders.Any()) return;  // First enumeration

    Console.WriteLine($"Processing {orders.Count()} orders");  // Second enumeration

    foreach (var order in orders)  // Third enumeration
    {
        Process(order);
    }
}

// Bad: Inefficient LINQ chains
var result = users
    .Where(u => u.IsActive)
    .ToList()  // Unnecessary materialization
    .Select(u => u.Email)
    .ToList()  // Another unnecessary materialization
    .Where(e => e.Contains("@company.com"))
    .ToList();
```

```csharp
// Good: Materialize once, use many times
public void ProcessOrders(IEnumerable<Order> orders)
{
    var orderList = orders.ToList();  // Single enumeration

    if (orderList.Count == 0) return;

    Console.WriteLine($"Processing {orderList.Count} orders");

    foreach (var order in orderList)
    {
        Process(order);
    }
}

// Good: Efficient LINQ - filter and project in one pass
var companyEmails = users
    .Where(u => u.IsActive)
    .Select(u => u.Email)
    .Where(e => e.Contains("@company.com"))
    .ToList();  // Single materialization at the end

// Good: Use appropriate method for the job
var hasActiveUsers = users.Any(u => u.IsActive);  // Stops at first match
var firstAdmin = users.FirstOrDefault(u => u.Role == Role.Admin);  // Returns one
var adminCount = users.Count(u => u.Role == Role.Admin);  // Full scan but single pass

// Good: Consider performance for large collections
var userLookup = users.ToDictionary(u => u.Id);  // O(1) lookups after O(n) build
var groupedByDept = users.ToLookup(u => u.Department);  // One-to-many grouping
```

**When to use:** Materialize `IEnumerable` to a `List` or array when you need to iterate multiple times. Chain LINQ operators without intermediate `ToList()` calls. Use `Any()` instead of `Count() > 0`. Consider memory and performance for large data sets.

#### Dispose Pattern

```csharp
// Bad: Not disposing resources
public class ReportGenerator
{
    public void GenerateReport(string path)
    {
        var stream = new FileStream(path, FileMode.Create);
        var writer = new StreamWriter(stream);
        writer.WriteLine("Report content");
        // File handle leaked - file may remain locked
    }
}

// Bad: Only implementing IDisposable without the full pattern
public class DatabaseConnection : IDisposable
{
    private SqlConnection _connection;

    public void Dispose()
    {
        _connection.Close();  // What if called twice? What about finalizer?
    }
}
```

```csharp
// Good: Using statement for simple cases
public class ReportGenerator
{
    public void GenerateReport(string path)
    {
        using var stream = new FileStream(path, FileMode.Create);
        using var writer = new StreamWriter(stream);
        writer.WriteLine("Report content");
        // Both disposed automatically, even if exception occurs
    }

    public async Task GenerateReportAsync(string path, CancellationToken ct)
    {
        await using var stream = new FileStream(path, FileMode.Create);
        await using var writer = new StreamWriter(stream);
        await writer.WriteLineAsync("Report content");
    }
}

// Good: Full dispose pattern when you have unmanaged resources
public class DatabaseConnection : IDisposable
{
    private SqlConnection? _connection;
    private bool _disposed;

    public void ExecuteQuery(string sql)
    {
        ObjectDisposedException.ThrowIf(_disposed, this);
        // Use _connection
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;

        if (disposing)
        {
            // Dispose managed resources
            _connection?.Dispose();
            _connection = null;
        }

        // Free unmanaged resources here if any
        _disposed = true;
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    ~DatabaseConnection()
    {
        Dispose(false);
    }
}
```

**When to use:** Use `using` statements for any `IDisposable` object. Implement the full dispose pattern when your class holds unmanaged resources or wraps disposables. For async disposal, implement `IAsyncDisposable`.

---

### Python

#### Type Hints Usage

```python
# Bad: No type hints - unclear contracts
def process_order(order, discount=None):
    if discount:
        order['total'] = order['total'] * (1 - discount)
    return order

def fetch_users(ids):
    return [get_user(id) for id in ids]  # What type is returned?
```

```python
# Good: Clear type hints with proper Optional usage
from typing import Optional
from decimal import Decimal

@dataclass
class Order:
    id: str
    total: Decimal
    items: list[OrderItem]

def process_order(order: Order, discount: Optional[Decimal] = None) -> Order:
    """Apply optional discount to order total."""
    if discount is not None:
        order.total = order.total * (1 - discount)
    return order

def fetch_users(user_ids: list[int]) -> list[User]:
    """Fetch users by their IDs. Returns empty list for unknown IDs."""
    return [user for id in user_ids if (user := get_user(id)) is not None]

# Good: TypedDict for dictionary structures (when dataclass isn't appropriate)
from typing import TypedDict, NotRequired

class UserDict(TypedDict):
    id: int
    email: str
    name: str
    phone: NotRequired[str]  # Optional field

def create_user(data: UserDict) -> User:
    return User(**data)

# Good: Generic types for reusable functions
from typing import TypeVar, Callable

T = TypeVar('T')
E = TypeVar('E', bound=Exception)

def retry(
    operation: Callable[[], T],
    max_attempts: int = 3,
    exceptions: tuple[type[E], ...] = (Exception,)
) -> T:
    """Retry an operation up to max_attempts times."""
    last_error: Optional[E] = None
    for attempt in range(max_attempts):
        try:
            return operation()
        except exceptions as e:
            last_error = e
            logger.warning(f"Attempt {attempt + 1} failed: {e}")
    raise last_error
```

**When to use:** Add type hints to all public function signatures. Use `Optional[X]` instead of `X | None` for Python 3.9 compatibility. Use dataclasses or TypedDict for structured data. Run mypy in CI.

#### Context Managers

```python
# Bad: Manual resource management
def read_config(path):
    f = open(path)
    try:
        data = json.load(f)
    finally:
        f.close()  # Easy to forget, especially with early returns
    return data

# Also bad: Not handling cleanup in classes
class DatabasePool:
    def __init__(self, connection_string):
        self.pool = create_pool(connection_string)

    def close(self):
        self.pool.close()  # Caller must remember to call this
```

```python
# Good: Use with statement for file operations
def read_config(path: Path) -> dict[str, Any]:
    with open(path) as f:
        return json.load(f)

# Good: Implement context manager for resource classes
class DatabasePool:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self._pool: Optional[Pool] = None

    def __enter__(self) -> 'DatabasePool':
        self._pool = create_pool(self.connection_string)
        return self

    def __exit__(
        self,
        exc_type: Optional[type[BaseException]],
        exc_val: Optional[BaseException],
        exc_tb: Optional[TracebackType]
    ) -> None:
        if self._pool:
            self._pool.close()

# Usage
with DatabasePool(conn_string) as pool:
    pool.execute("SELECT * FROM users")
# Pool automatically closed

# Good: Use contextmanager decorator for simple cases
from contextlib import contextmanager

@contextmanager
def timed_operation(name: str) -> Generator[None, None, None]:
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        logger.info(f"{name} took {elapsed:.2f}s")

# Usage
with timed_operation("database_query"):
    results = db.execute(query)

# Good: Async context managers
class AsyncDatabasePool:
    async def __aenter__(self) -> 'AsyncDatabasePool':
        self._pool = await create_async_pool(self.connection_string)
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb) -> None:
        if self._pool:
            await self._pool.close()
```

**When to use:** Use `with` for any resource that needs cleanup (files, connections, locks). Implement `__enter__`/`__exit__` for classes that manage resources. Use `@contextmanager` for simple setup/teardown patterns.

#### List Comprehension vs Loops

```python
# Bad: Using loops where comprehensions are clearer
def get_active_emails(users):
    result = []
    for user in users:
        if user.is_active:
            result.append(user.email)
    return result

# Also bad: Overly complex comprehensions
emails = [
    user.email.lower().strip()
    for dept in departments
    for team in dept.teams
    for user in team.members
    if user.is_active and user.email and '@' in user.email
    and not user.email.endswith('.test')
]
```

```python
# Good: Simple, readable comprehension
def get_active_emails(users: list[User]) -> list[str]:
    return [user.email for user in users if user.is_active]

# Good: Use generator expression for large data or single iteration
def count_valid_orders(orders: Iterable[Order]) -> int:
    return sum(1 for order in orders if order.is_valid)

def find_first_match(items: Iterable[Item], predicate: Callable[[Item], bool]) -> Optional[Item]:
    return next((item for item in items if predicate(item)), None)

# Good: Break complex comprehensions into steps or use loops
def get_active_team_emails(departments: list[Department]) -> list[str]:
    """Get emails of active users from all teams in all departments."""
    emails = []
    for dept in departments:
        for team in dept.teams:
            for user in team.members:
                if _is_valid_active_user(user):
                    emails.append(user.email.lower().strip())
    return emails

def _is_valid_active_user(user: User) -> bool:
    """Check if user is active with a valid, non-test email."""
    return (
        user.is_active
        and user.email
        and '@' in user.email
        and not user.email.endswith('.test')
    )

# Good: Dict and set comprehensions where appropriate
user_by_id = {user.id: user for user in users}
unique_domains = {email.split('@')[1] for email in emails}
```

**When to use:** Use comprehensions for simple transformations and filters (one condition, one transformation). Use explicit loops when logic is complex or has side effects. Use generator expressions for large data sets or when you only need one pass.

#### Exception Handling Patterns

```python
# Bad: Bare except or catching Exception
def parse_config(path):
    try:
        with open(path) as f:
            return json.load(f)
    except:  # Catches KeyboardInterrupt, SystemExit, etc.
        return {}

def process_data(data):
    try:
        # lots of code here
        result = transform(data)
        save(result)
        notify(result)
    except Exception as e:
        print(f"Error: {e}")  # Which operation failed? Lost context
```

```python
# Good: Specific exceptions with appropriate handling
class ConfigError(Exception):
    """Raised when configuration cannot be loaded."""
    pass

def parse_config(path: Path) -> dict[str, Any]:
    """Load configuration from JSON file.

    Raises:
        ConfigError: If file cannot be read or contains invalid JSON.
    """
    try:
        with open(path) as f:
            return json.load(f)
    except FileNotFoundError:
        raise ConfigError(f"Configuration file not found: {path}")
    except json.JSONDecodeError as e:
        raise ConfigError(f"Invalid JSON in {path}: {e.msg} at line {e.lineno}")
    except PermissionError:
        raise ConfigError(f"Permission denied reading {path}")

# Good: Separate handling for different failure modes
def process_data(data: Data) -> ProcessResult:
    try:
        result = transform(data)
    except ValidationError as e:
        logger.warning(f"Invalid data: {e}")
        return ProcessResult(success=False, error="validation_failed")

    try:
        save(result)
    except DatabaseError as e:
        logger.error(f"Failed to save result: {e}")
        raise  # Re-raise - this is a critical failure

    try:
        notify(result)
    except NotificationError as e:
        # Non-critical - log but don't fail
        logger.warning(f"Failed to send notification: {e}")

    return ProcessResult(success=True, data=result)

# Good: Use exception chaining to preserve context
def load_user_preferences(user_id: int) -> Preferences:
    try:
        raw_data = fetch_from_api(user_id)
        return parse_preferences(raw_data)
    except APIError as e:
        raise PreferenceLoadError(f"Could not load preferences for user {user_id}") from e
```

**When to use:** Catch specific exceptions you can handle. Use exception chaining (`from e`) to preserve context. Let unexpected exceptions propagate. Create custom exception classes for your domain errors.

---

## Code Quality Gates

These patterns should be checked and enforced by Claude when completing any code task.

### Pre-Completion Checklist

Before marking any code task as complete, Claude must verify:

**Type Safety:**
- [ ] No type assertions (`as`, `!`, casts) without documented justification
- [ ] Null/undefined/None handled explicitly at system boundaries
- [ ] Return types are specific (not `any`, `object`, `dynamic`)

**Error Handling:**
- [ ] Async operations have explicit error handling
- [ ] Errors include context (what operation failed, with what inputs)
- [ ] Critical vs non-critical failures are distinguished
- [ ] Resources are properly disposed/cleaned up in all paths

**Code Clarity:**
- [ ] No deeply nested conditionals (max 3 levels)
- [ ] Complex comprehensions/LINQ broken into named steps
- [ ] Magic numbers/strings extracted to named constants
- [ ] Functions do one thing and are testable in isolation

### Proactive Refactoring

When Claude encounters these patterns in existing code being modified, it should:

1. **Flag the issue** in the response
2. **Propose the fix** with explanation
3. **Ask permission** if the fix is outside the immediate task scope

**Example response pattern:**

> I noticed this method uses a type assertion that could cause runtime errors:
>
> ```typescript
> const user = data as User;  // Unsafe - no runtime validation
> ```
>
> I recommend adding a type guard:
>
> ```typescript
> if (!isUser(data)) {
>     throw new ValidationError('Invalid user data');
> }
> ```
>
> Should I include this safety improvement in my changes?

### Improvement Documentation

When fixing a code quality issue, Claude should explain:

1. **What** was changed
2. **Why** the original pattern was problematic
3. **How** the new pattern prevents the issue
4. **When** the new pattern should be used going forward

This creates a learning opportunity and helps maintain consistency.
