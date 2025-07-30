# PHP Coding Standards

All code must adhere to these standards, which are enforced through PHPCS (PHP_CodeSniffer) with custom rulesets.

## Table of Contents
- [Base Standards](#base-standards)
- [File Structure and Organization](#file-structure-and-organization)
- [Type Hints and Declarations](#type-hints-and-declarations)
- [Arrays and Data Structures](#arrays-and-data-structures)
- [Namespaces and Imports](#namespaces-and-imports)
- [Functions and Methods](#functions-and-methods)
- [Classes and Object-Oriented Code](#classes-and-object-oriented-code)
- [Control Structures](#control-structures)
- [Operators and Expressions](#operators-and-expressions)
- [Comments and Documentation](#comments-and-documentation)
- [Formatting and Whitespace](#formatting-and-whitespace)
- [PHP Language Features](#php-language-features)

## Base Standards

### PSR-12 Compliance
All code must follow the **PSR-12: Extended Coding Style** standard as the foundation. This includes:
- 4 spaces for indentation (no tabs)
- Opening braces for classes and methods go on the next line
- Opening braces for control structures go on the same line
- Closing braces go on their own line

### Strict Types Declaration
**REQUIRED**: Every PHP file must start with a strict types declaration:
```php
<?php

declare(strict_types=1);
```

### File Endings
- All files must end with a single newline character
- No trailing whitespace is allowed anywhere in the file
- PHP files must NOT have a closing `?>` tag

## File Structure and Organization

### Line Length Limits
- **Soft limit**: 140 characters per line
- **Hard limit**: 160 characters per line (absolute maximum)
- When lines exceed the soft limit, consider breaking them for readability

### File Structure
Files should be organized in this order:
1. Opening PHP tag and strict types declaration
2. File-level docblock (if needed)
3. Namespace declaration
4. Use statements (imports)
5. Class declaration and content

## Type Hints and Declarations

### Return Type Hints
**REQUIRED**: All functions and methods must have explicit return type hints or `@return` annotations.

```php
// ✅ Good - explicit return type
public function getName(): string
{
    return $this->name;
}

// ✅ Good - void return type for methods with no return
public function setName(string $name): void
{
    $this->name = $name;
}

// ❌ Bad - missing return type hint
public function getName()
{
    return $this->name;
}
```

### Parameter Type Hints
**REQUIRED**: All function and method parameters must have type hints or `@param` annotations.

```php
// ✅ Good - all parameters have type hints
public function processUser(int $userId, string $email, bool $isActive = false): User
{
    // Implementation
}

// ❌ Bad - missing parameter type hints
public function processUser($userId, $email, $isActive = false)
{
    // Implementation
}
```

### Property Type Hints
**REQUIRED**: All class properties must have type hints.

```php
// ✅ Good - property type hints
class User
{
    private string $name;
    private int $age;
    private ?string $email = null;
}

// ❌ Bad - missing property type hints
class User
{
    private $name;
    private $age;
    private $email;
}
```

### Nullable Types
- When a parameter has a `null` default value, it must use nullable type syntax
- Null type hints must be positioned last in union types

```php
// ✅ Good - nullable type for null default
public function setEmail(?string $email = null): void
{
    $this->email = $email;
}

// ✅ Good - null positioned last in union type
public function process(string|int|null $value): void
{
    // Implementation
}
```

### Long Type Hints
Use short type hints instead of long ones:
- Use `int` instead of `integer`
- Use `bool` instead of `boolean`

## Arrays and Data Structures

### Array Syntax
**REQUIRED**: Use short array syntax `[]` instead of `array()`.

```php
// ✅ Good - short array syntax
$users = [
    'john' => 'John Doe',
    'jane' => 'Jane Smith',
];

// ❌ Bad - long array syntax
$users = array(
    'john' => 'John Doe',
    'jane' => 'Jane Smith',
);
```

### Trailing Commas
**REQUIRED**: Multi-line arrays must have trailing commas after the last element.

```php
// ✅ Good - trailing comma in multi-line array
$config = [
    'database' => 'mysql',
    'host' => 'localhost',
    'port' => 3306, // ← Required trailing comma
];

// ❌ Bad - missing trailing comma
$config = [
    'database' => 'mysql',
    'host' => 'localhost',
    'port' => 3306
];
```

## Namespaces and Imports

### Use Statements
- Use statements must be sorted alphabetically
- No group use statements allowed
- No multiple uses per line
- Remove unused use statements

```php
// ✅ Good - alphabetically sorted, one per line
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;

// ❌ Bad - not alphabetically sorted
use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\DB;

// ❌ Bad - group use statement
use Illuminate\Support\{Facades\DB, Collection};

// ❌ Bad - multiple uses per line
use App\Models\User, App\Models\Post;
```

### Fully Qualified Names
**REQUIRED**: Global functions must be referenced with fully qualified names.

```php
// ✅ Good - fully qualified function names
$count = \count($array);
$json = \json_encode($data);
$hash = \password_hash($password, \PASSWORD_DEFAULT);

// ❌ Bad - unqualified function names
$count = count($array);
$json = json_encode($data);
$hash = password_hash($password, PASSWORD_DEFAULT);
```

### Namespace Requirements
- Each file must contain exactly one namespace
- Namespace must match the directory structure

## Functions and Methods

### Function Declarations
- Arrow functions must follow proper declaration format
- Functions with multiple parameters should have trailing commas in their declarations

```php
// ✅ Good - arrow function formatting
$numbers = array_map(fn(int $x): int => $x * 2, $array);

// ✅ Good - trailing comma in function declaration
public function createUser(
    string $name,
    string $email,
    bool $isActive,
): User {
    // Implementation
}
```

### Unused Parameters
**FORBIDDEN**: Do not leave unused parameters in function signatures.

```php
// ❌ Bad - unused parameter $config
public function processUser(User $user, array $config): void
{
    // Only $user is used, $config is unused
    $user->activate();
}

// ✅ Good - remove unused parameter
public function processUser(User $user): void
{
    $user->activate();
}
```

### Parameter Default Values
Avoid useless parameter default values that will never be used.

```php
// ❌ Bad - useless default value
public function multiply(int $a, int $b = 1): int
{
    return $a * $b; // Default of 1 makes this meaningless
}

// ✅ Good - meaningful default or no default
public function multiply(int $a, int $b): int
{
    return $a * $b;
}
```

### Closure Variables
Do not pass unused inherited variables to closures.

```php
// ❌ Bad - $unusedVar is not used in closure
$callback = function($item) use ($config, $unusedVar) {
    return $config->process($item);
};

// ✅ Good - only pass used variables
$callback = function($item) use ($config) {
    return $config->process($item);
};
```

## Classes and Object-Oriented Code

### Class Structure
Classes must follow a specific ordering of elements:
1. Constants
2. Properties
3. Constructor
4. Methods (grouped by visibility: public, protected, private)

### Class Member Spacing
Proper spacing must be maintained between class members.

### Class Constants
**REQUIRED**: Class constants must have explicit visibility declarations.

```php
// ✅ Good - explicit visibility
class Config
{
    public const DEFAULT_TIMEOUT = 30;
    private const SECRET_KEY = 'secret';
}

// ❌ Bad - missing visibility
class Config
{
    const DEFAULT_TIMEOUT = 30;
}
```

### Class Name References
Use modern class name references with `::class`.

```php
// ✅ Good - modern class reference
$reflection = new ReflectionClass(User::class);

// ❌ Bad - string class reference
$reflection = new ReflectionClass('User');
```

### Object Instantiation
**REQUIRED**: Always use parentheses when instantiating objects, even without parameters.

```php
// ✅ Good - parentheses included
$user = new User();
$config = new Config();

// ❌ Bad - missing parentheses
$user = new User;
$config = new Config;
```

## Control Structures

### Null Coalescing Operators
**REQUIRED**: Use null coalescing operators instead of ternary operators for null checks.

```php
// ✅ Good - null coalescing operator
$name = $user->getName() ?? 'Default Name';
$config = $options['config'] ??= new Config();

// ❌ Bad - ternary for null check
$name = $user->getName() !== null ? $user->getName() : 'Default Name';
```

### Useless Conditions
Avoid useless if conditions with return statements.

```php
// ❌ Bad - useless if condition
public function isActive(): bool
{
    if ($this->status === 'active') {
        return true;
    } else {
        return false;
    }
}

// ✅ Good - direct return
public function isActive(): bool
{
    return $this->status === 'active';
}
```

### Switch Statements
Do not use `continue` without an integer operand in switch statements.

```php
// ❌ Bad - continue without operand in switch
switch ($value) {
    case 1:
        continue; // This is confusing
    case 2:
        break;
}

// ✅ Good - use break or continue with operand
switch ($value) {
    case 1:
        break;
    case 2:
        continue 2; // Clear intention
}
```

## Operators and Expressions

### Equality Operators
**REQUIRED**: Use strict comparison operators (`===`, `!==`) instead of loose ones (`==`, `!=`).

```php
// ✅ Good - strict comparison
if ($value === 0) {
    // Handle zero
}

if ($user !== null) {
    // Handle user
}

// ❌ Bad - loose comparison
if ($value == 0) {
    // This matches 0, false, '', null, etc.
}
```

### Combined Assignment Operators
**REQUIRED**: Use combined assignment operators when possible.

```php
// ✅ Good - combined assignment
$total += $amount;
$count *= 2;
$message .= ' suffix';

// ❌ Bad - separate assignment
$total = $total + $amount;
$count = $count * 2;
$message = $message . ' suffix';
```

### Operator Spacing
Proper spacing must be maintained around operators.

```php
// ✅ Good - proper spacing
$result = $a + $b * $c;
$isValid = $user !== null && $user->isActive();

// ❌ Bad - improper spacing
$result=$a+$b*$c;
$isValid=$user!==null&&$user->isActive();
```

### Type Casting
Use proper type casting with spaces.

```php
// ✅ Good - space after cast
$string = (string) $number;
$array = (array) $object;

// ❌ Bad - no space after cast
$string = (string)$number;
$array = (array)$object;
```

## Comments and Documentation

### Forbidden Comments
Avoid useless comments that don't add value:
- Comments that just repeat the code
- Empty comments
- Commented-out code blocks

```php
// ❌ Bad - useless comments
public function getName(): string
{
    // Get the name
    return $this->name; // Return name
}

// ✅ Good - meaningful comment when needed
public function getName(): string
{
    // Return normalized name for display purposes
    return trim($this->name);
}
```

### Forbidden Annotations
Remove useless `@inheritDoc` comments and similar redundant annotations.

### Function Documentation
Remove useless function docblocks that don't add information beyond type hints.

```php
// ❌ Bad - useless docblock (type hints provide same info)
/**
 * @param string $name
 * @param int $age
 * @return User
 */
public function createUser(string $name, int $age): User
{
    return new User($name, $age);
}

// ✅ Good - no docblock needed (or meaningful docblock)
public function createUser(string $name, int $age): User
{
    return new User($name, $age);
}

// ✅ Good - meaningful docblock
/**
 * Creates a user with validated input and sends welcome email.
 * 
 * @throws ValidationException When name or age is invalid
 * @throws EmailException When welcome email fails to send
 */
public function createUser(string $name, int $age): User
{
    // Implementation with validation and email logic
}
```

## Formatting and Whitespace

### Whitespace Rules
- No trailing whitespace anywhere in files
- No superfluous whitespace between tokens
- Proper spacing around casts: `(string) $value`
- Maintain consistent indentation (4 spaces)

### Line Endings
- Use Unix line endings (LF, not CRLF)
- Each file must end with a newline character

## PHP Language Features

### PHP Tags
- **REQUIRED**: Use full PHP opening tags `<?php`
- **FORBIDDEN**: Short PHP tags `<?`
- **FORBIDDEN**: PHP closing tags `?>` at end of files

### Modern PHP Features
- Use modern language constructs when available
- Prefer newer syntax over legacy alternatives
- Use arrow functions for simple callbacks

## PHPCS Configuration

### Checked Directories
The following directories are checked for compliance:
- `app/`
- `config/`
- `routes/`
- `tests/`

### Excluded Patterns
The following are excluded from checking:
- `app/Http/Livewire/*` (Livewire components)
- `vendor/*` (Third-party packages)
- `public/*` (Public assets)
- `node_modules/*` (Node.js dependencies)
- `sniffs/*` (Custom sniffs)

### Running PHPCS
To check your code against these standards:

```bash
# Check all configured files
./vendor/bin/phpcs

# Check specific file
./vendor/bin/phpcs app/Models/User.php

# Auto-fix what can be automatically corrected
./vendor/bin/phpcbf

# Check specific directory
./vendor/bin/phpcs app/Http/Controllers/
```

## Quick Reference Checklist

Before committing code, ensure:

- [ ] `declare(strict_types=1);` is present
- [ ] All functions/methods have return type hints
- [ ] All parameters have type hints
- [ ] All properties have type hints
- [ ] Short array syntax `[]` is used
- [ ] Multi-line arrays have trailing commas
- [ ] Use statements are alphabetically sorted
- [ ] Global functions are fully qualified
- [ ] Strict comparison operators are used
- [ ] Combined assignment operators are used where applicable
- [ ] No unused parameters or use statements
- [ ] Proper spacing and formatting
- [ ] No trailing whitespace
- [ ] File ends with newline

## Error Severity

All rules in this standard are configured as **errors** (severity 10), meaning they must be fixed before code can be committed. There are no warnings that can be ignored.

Following these standards ensures consistent, maintainable, and high-quality PHP code across the Beginly Health codebase.