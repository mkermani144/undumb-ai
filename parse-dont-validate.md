## Parse, Don't Validate

When handling data in your application, parse untrusted inputs at the boundaries rather than repeatedly validating them throughout your codebase.

**Core principle**: Create domain-specific types that can only be constructed with valid data, then use these types within your business logic instead of primitive types that require validation.

### ❌ Anti-pattern

The code repeatedly validates data that has already passed into the application:

```ts
// Accepts primitive string, requires validation at each use
const processEmail = (email: string) => {
  if (!email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
    return new Error("Invalid email format");
  }
  
  // Now we can finally use the email...
  return sendWelcomeMessage(email);
}

const createUser = (email: string) => {
  // Have to validate again!
  if (!email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
    return new Error("Invalid email format");
  }
  
  // Now we can use it...
  return saveToDatabase(email);
}
```

### ✅ Best practice

Create a domain type that can only be constructed with valid data:

```ts
import { Ok, Err, Result } from "ts-results-es";

class Email {
  private constructor(private readonly value: string) {}
  
  static from(input: string): Result<Email, string> {
    if (!input.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
      return Err("Invalid email format");
    }
    return Ok(new Email(input));
  }
  
  toString(): string {
    return this.value;
  }
}

// Type safety guarantees valid data
const processEmail = (email: Email) => {
  // No validation needed - guaranteed to be valid
  return sendWelcomeMessage(email.toString());
}

const createUser = (email: Email) => {
  // No validation needed - guaranteed to be valid
  return saveToDatabase(email.toString());
}

// Validation happens only at the boundary
const handleUserRegistration = (rawEmail: string) => {
  const emailResult = Email.from(rawEmail);
  
  if (emailResult.isErr()) {
    return Err(emailResult.error);
  }
  
  const email = emailResult.value;
  // Now we can use the validated email throughout the application
  createUser(email);
  processEmail(email);
  return Ok("User registered successfully");
}
```

By following this pattern, you eliminate duplication of validation logic and make your code more robust by ensuring that invalid data cannot propagate through your system.
