# Code Improvement Patterns

> This document catalogs common code issues, anti-patterns, and their recommended solutions. Agents should reference this when reviewing code.

## Performance Issues

### N+1 Query Problem

**What**: Querying in a loop, causing unnecessary database requests

**Example (BAD)**:
```javascript
const users = await getUsers(); // 1 query
for (const user of users) {
  user.profile = await getUserProfile(user.id); // N queries
}
// Total: 1 + N queries
```

**Solution**:
```javascript
const users = await getUsers();
const profiles = await getUserProfileBatch(users.map(u => u.id)); // 1 batch query
const userMap = new Map(profiles.map(p => [p.userId, p]));
users.forEach(u => u.profile = userMap.get(u.id));
```

**Prevention**: Use batch/bulk query methods for related data

---

### Unnecessary Re-renders (React)

**What**: Components re-render when props/state haven't changed

**Example (BAD)**:
```javascript
function UserList({ users, onSelect }) {
  return users.map(user => 
    <UserItem 
      user={user} 
      onSelect={onSelect}  // New function on every render!
    />
  );
}
```

**Solution**:
```javascript
const UserItem = React.memo(({ user, onSelect }) => (
  <div onClick={() => onSelect(user.id)}>
    {user.name}
  </div>
));

function UserList({ users, onSelect }) {
  const handleSelect = useCallback(onSelect, [onSelect]);
  return users.map(user => 
    <UserItem 
      key={user.id}
      user={user} 
      onSelect={handleSelect}
    />
  );
}
```

**Prevention**: Use `React.memo()`, `useCallback()`, and `useMemo()`

---

### Synchronous Long-Running Operations

**What**: Blocking operations in the main thread

**Example (BAD)**:
```javascript
app.get('/process', (req, res) => {
  const result = expensiveSync(); // Blocks all other requests!
  res.json(result);
});
```

**Solution**:
```javascript
app.get('/process', async (req, res) => {
  try {
    const result = await expensiveAsync();
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

**Prevention**: Always use async/await for I/O; move heavy computation to workers

---

### Memory Leaks

**What**: References keeping objects in memory when they should be garbage collected

**Example (BAD)**:
```javascript
const timers = [];

function startMonitoring(user) {
  const timer = setInterval(() => {
    console.log(user); // Closure keeps user in memory
  }, 1000);
  timers.push(timer);
  // Timers never cleared = memory leak
}
```

**Solution**:
```javascript
const timers = new Map();

function startMonitoring(userId) {
  const timer = setInterval(() => {
    updateStatus(userId); // Just the ID, not the whole object
  }, 1000);
  timers.set(userId, timer);
}

function stopMonitoring(userId) {
  const timer = timers.get(userId);
  if (timer) clearInterval(timer);
  timers.delete(userId);
}
```

**Prevention**: Always clean up timers, event listeners, and subscriptions

---

## Security Issues

### SQL Injection

**What**: Concatenating user input into SQL queries

**Example (BAD)**:
```javascript
const query = `SELECT * FROM users WHERE email = '${email}'`;
db.query(query); // SQL injection risk!
```

**Solution**:
```javascript
const query = 'SELECT * FROM users WHERE email = ?';
db.query(query, [email]); // Parameterized query
```

**Prevention**: Always use parameterized queries or ORM

---

### Hardcoded Secrets

**What**: API keys, passwords, or tokens in source code

**Example (BAD)**:
```javascript
const API_KEY = 'sk-123456789abcdef'; // Never commit this!

fetch('https://api.example.com/data', {
  headers: { 'Authorization': `Bearer ${API_KEY}` }
});
```

**Solution**:
```javascript
const API_KEY = process.env.EXTERNAL_API_KEY;
if (!API_KEY) {
  throw new Error('EXTERNAL_API_KEY environment variable is required');
}

fetch('https://api.example.com/data', {
  headers: { 'Authorization': `Bearer ${API_KEY}` }
});
```

**Prevention**: Use environment variables; add `.env*` to gitignore

---

### Missing Input Validation

**What**: Accepting and using user input without validation

**Example (BAD)**:
```javascript
app.post('/users', (req, res) => {
  const user = {
    email: req.body.email,      // Could be anything!
    age: req.body.age,          // Could be negative!
    admin: req.body.admin       // User could set this!
  };
  saveUser(user);
});
```

**Solution**:
```javascript
app.post('/users', (req, res) => {
  const { email, age } = req.body;
  
  if (!email || !email.includes('@')) {
    return res.status(400).json({ error: 'Invalid email' });
  }
  
  if (typeof age !== 'number' || age < 0 || age > 150) {
    return res.status(400).json({ error: 'Invalid age' });
  }
  
  const user = {
    email,
    age,
    admin: false  // Never trust client for this!
  };
  saveUser(user);
});
```

**Prevention**: Validate all inputs; use schema validation libraries (Zod, Yup, Joi)

---

### Missing Permission Checks

**What**: Not verifying user has permission before operation

**Example (BAD)**:
```javascript
app.delete('/posts/:id', (req, res) => {
  const post = Post.findById(req.params.id);
  post.delete(); // Any authenticated user can delete any post!
});
```

**Solution**:
```javascript
app.delete('/posts/:id', (req, res) => {
  const post = Post.findById(req.params.id);
  
  if (post.authorId !== req.user.id) {
    return res.status(403).json({ error: 'Unauthorized' });
  }
  
  post.delete();
  res.json({ success: true });
});
```

**Prevention**: Always check permissions before sensitive operations

---

## Code Organization Issues

### God Object Anti-pattern

**What**: One class/function doing too much

**Example (BAD)**:
```javascript
class User {
  // User properties
  
  // Database operations
  save() { ... }
  delete() { ... }
  
  // Email operations
  sendWelcomeEmail() { ... }
  
  // Authentication
  hashPassword() { ... }
  verifyPassword() { ... }
  
  // Validation
  validate() { ... }
  
  // Business logic
  calculateScore() { ... }
}
```

**Solution**:
```javascript
// Separate concerns
class User { /* properties and basic methods */ }
class UserRepository { /* database operations */ }
class UserNotification { /* email operations */ }
class PasswordService { /* password operations */ }
class UserValidator { /* validation */ }
class UserScoringService { /* business logic */ }
```

**Prevention**: Apply Single Responsibility Principle; each class has one reason to change

---

### Callback Hell / Pyramid of Doom

**What**: Deeply nested callbacks making code hard to read

**Example (BAD)**:
```javascript
function processUser(userId, callback) {
  getUser(userId, (err, user) => {
    if (err) return callback(err);
    getProfile(user.id, (err, profile) => {
      if (err) return callback(err);
      updateProfile(profile, (err, updated) => {
        if (err) return callback(err);
        saveToDatabase(updated, (err, result) => {
          if (err) return callback(err);
          callback(null, result);
        });
      });
    });
  });
}
```

**Solution**:
```javascript
async function processUser(userId) {
  const user = await getUser(userId);
  const profile = await getProfile(user.id);
  const updated = await updateProfile(profile);
  return await saveToDatabase(updated);
}
```

**Prevention**: Use async/await instead of callbacks; use Promises

---

### Magic Numbers and Strings

**What**: Unexplained numeric or string constants in code

**Example (BAD)**:
```javascript
if (user.age > 18 && user.status === 'active' && user.accountBalance > 100) {
  // Why 18? Why 100? What do these mean?
}
```

**Solution**:
```javascript
const LEGAL_AGE = 18;
const MIN_ACCOUNT_BALANCE = 100;
const ACTIVE_STATUS = 'active';

if (user.age > LEGAL_AGE && 
    user.status === ACTIVE_STATUS && 
    user.accountBalance > MIN_ACCOUNT_BALANCE) {
  // Now it's clear!
}
```

**Prevention**: Extract to named constants; add comments explaining "why"

---

### Incomplete Error Handling

**What**: Not handling all error cases or catching errors too broadly

**Example (BAD)**:
```javascript
try {
  const user = await getUser(userId);
  const profile = await getProfile(user.id);
  // What if getUser fails? What if getProfile fails?
  // We don't know which operation failed
  return { user, profile };
} catch (error) {
  console.log('Error'); // No context!
}
```

**Solution**:
```javascript
async function getUserWithProfile(userId) {
  try {
    const user = await getUser(userId);
    if (!user) {
      throw new NotFoundError(`User ${userId} not found`);
    }
    
    let profile = null;
    try {
      profile = await getProfile(user.id);
    } catch (error) {
      // Profile is optional, log but don't fail
      console.warn(`Could not load profile for user ${userId}:`, error.message);
    }
    
    return { user, profile };
  } catch (error) {
    if (error instanceof NotFoundError) {
      throw error; // Re-throw user not found
    }
    throw new InternalError('Failed to load user data', error);
  }
}
```

**Prevention**: Use specific error types; handle different errors differently; log context

---

## Code Quality Issues

### Duplicated Code

**What**: Same logic repeated in multiple places

**Example (BAD)**:
```javascript
// In UserController
if (!email.includes('@')) {
  throw new Error('Invalid email');
}

// In ProfileController  
if (!email.includes('@')) {
  throw new Error('Invalid email');
}

// In AdminController
if (!email.includes('@')) {
  throw new Error('Invalid email');
}
```

**Solution**:
```javascript
// In validators.ts
function validateEmail(email) {
  if (!email.includes('@')) {
    throw new ValidationError('Invalid email format');
  }
  return email;
}

// Use everywhere
validateEmail(email);
```

**Prevention**: Extract to utility functions; apply DRY principle

---

### Overly Complex Functions

**What**: Functions that do too much or have high cyclomatic complexity

**Example (BAD)**:
```javascript
function processOrder(order) {
  if (order.status === 'pending') {
    if (order.items.length > 0) {
      let total = 0;
      for (let i = 0; i < order.items.length; i++) {
        if (order.items[i].inStock) {
          total += order.items[i].price * order.items[i].quantity;
        } else {
          // Handle out of stock
        }
      }
      if (total > 0) {
        // Process payment
        // Update inventory
        // Send confirmation email
      }
    }
  }
  // ... more conditions
}
```

**Solution**:
```javascript
async function processOrder(order) {
  if (!isOrderValid(order)) throw new ValidationError('Invalid order');
  const total = calculateOrderTotal(order);
  if (total === 0) throw new ValidationError('Order total is zero');
  
  await processPayment(order, total);
  await updateInventory(order);
  await sendConfirmationEmail(order);
}

function calculateOrderTotal(order) {
  return order.items
    .filter(item => item.inStock)
    .reduce((sum, item) => sum + (item.price * item.quantity), 0);
}
```

**Prevention**: Break into smaller functions; extract helper functions; use early returns

---

### Unclear Variable Names

**What**: Variables with ambiguous or non-descriptive names

**Example (BAD)**:
```javascript
const u = users.filter(x => x.a > 18 && x.s === 'active');
const d = new Date().getTime() - u.lastLogin;
if (d > 2592000000) { // What's this number?
  u.status = 'inactive';
}
```

**Solution**:
```javascript
const THIRTY_DAYS_MS = 30 * 24 * 60 * 60 * 1000;

const activeAdultUsers = users.filter(user => 
  user.age > LEGAL_AGE && user.status === 'active'
);

for (const user of activeAdultUsers) {
  const daysSinceLogin = (new Date().getTime() - user.lastLogin) / (1000 * 60 * 60 * 24);
  if (daysSinceLogin > 30) {
    user.status = 'inactive';
  }
}
```

**Prevention**: Use descriptive names; extract magic numbers to constants

---

## Testing Issues

### Missing Edge Case Tests

**What**: Not testing boundary conditions and special cases

**Example (BAD)**:
```javascript
function divide(a, b) {
  return a / b;
}

// Only tests happy path
test('divides two numbers', () => {
  expect(divide(10, 2)).toBe(5);
});
```

**Solution**:
```javascript
function divide(a, b) {
  if (b === 0) throw new Error('Cannot divide by zero');
  return a / b;
}

test('divide_dividesPositiveNumbers', () => {
  expect(divide(10, 2)).toBe(5);
});

test('divide_throwsOnDivideByZero', () => {
  expect(() => divide(10, 0)).toThrow();
});

test('divide_handlesNegativeNumbers', () => {
  expect(divide(-10, 2)).toBe(-5);
});

test('divide_handlesDecimals', () => {
  expect(divide(10, 3)).toBeCloseTo(3.333);
});
```

**Prevention**: Test edge cases: null, undefined, empty, zero, negative, boundary values

---

### No Error Path Testing

**What**: Tests only cover success cases

**Example (BAD)**:
```javascript
test('getUser_returnsUser', async () => {
  const user = await getUser('123');
  expect(user.id).toBe('123');
});
// No test for user not found, database error, etc.
```

**Solution**:
```javascript
test('getUser_returnsUserWhenFound', async () => {
  const user = await getUser('123');
  expect(user.id).toBe('123');
});

test('getUser_throwsNotFoundWhenUserMissing', async () => {
  await expect(getUser('nonexistent')).rejects.toThrow(NotFoundError);
});

test('getUser_throwsDatabaseErrorOnQueryFailure', async () => {
  // Mock database failure
  await expect(getUser('123')).rejects.toThrow(DatabaseError);
});
```

**Prevention**: Test all code paths; test error scenarios explicitly

---

## When Agents Find These Issues

1. **Reference this document** in your review feedback
2. **Show the bad pattern** from the code
3. **Explain the solution** with clear example
4. **Suggest specific changes** to make
5. **Link to quality-standards.md** if applicable

Example feedback:
> Found improvement pattern: N+1 Query Problem  
> Location: `services/userService.ts:45-50`  
> Issue: Loading user profiles in a loop causes multiple database queries  
> Solution: Use batch query method  
> See: `improvement-patterns.md#n1-query-problem` and `quality-standards.md#performance`
