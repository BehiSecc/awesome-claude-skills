---
name: pict-test-designer
description: Design comprehensive test cases using PICT (Pairwise Independent Combinatorial Testing) for any piece of requirements or code. Analyzes inputs, generates PICT models with parameters, values, and constraints for valid scenarios using pairwise testing. Outputs the PICT model, markdown table of test cases, and expected results.
---

# PICT Test Designer

This skill enables systematic test case design using PICT (Pairwise Independent Combinatorial Testing). Given requirements or code, it analyzes the system to identify test parameters, generates a PICT model with appropriate constraints, executes the model to generate pairwise test cases, and formats the results with expected outputs.

## When to Use This Skill

Use this skill when:
- Designing test cases for a feature, function, or system with multiple input parameters
- Creating test suites for configurations with many combinations
- Needing comprehensive coverage with minimal test cases
- Analyzing requirements to identify test scenarios
- Working with code that has multiple conditional paths
- Building test matrices for API endpoints, web forms, or system configurations

## Workflow

Follow this process for test design:

### 1. Analyze Requirements or Code

From the user's requirements or code, identify:
- **Parameters**: Input variables, configuration options, environmental factors
- **Values**: Possible values for each parameter (using equivalence partitioning)
- **Constraints**: Business rules, technical limitations, dependencies between parameters
- **Expected Outcomes**: What should happen for different combinations

**Example Analysis:**

For a login function with requirements:
- Users can login with username/password
- Supports 2FA (on/off)
- Remembers login on trusted devices
- Rate limits after 3 failed attempts

Identified parameters:
- Credentials: Valid, Invalid
- TwoFactorAuth: Enabled, Disabled
- RememberMe: Checked, Unchecked
- PreviousFailures: 0, 1, 2, 3, 4

### 2. Generate PICT Model

Create a PICT model with:
- Clear parameter names
- Well-defined value sets (using equivalence partitioning and boundary values)
- Constraints for invalid combinations
- Comments explaining business rules

**Model Structure:**
```
# Parameter definitions
ParameterName: Value1, Value2, Value3

# Constraints (if any)
IF [Parameter1] = "Value" THEN [Parameter2] <> "OtherValue";
```

**Refer to references/pict_syntax.md for:**
- Complete syntax reference
- Constraint grammar and operators
- Advanced features (sub-models, aliasing, negative testing)
- Command-line options
- Detailed constraint patterns

**Refer to references/examples.md for:**
- Complete real-world examples by domain
- Software function testing examples
- Web application, API, and mobile testing examples
- Database and configuration testing patterns
- Common patterns for authentication, resource access, error handling

### 3. Execute PICT Model

Generate the PICT model text and format it for the user. You can use Python code directly to work with the model:

```python
# Define parameters and constraints
parameters = {
    "OS": ["Windows", "Linux", "MacOS"],
    "Browser": ["Chrome", "Firefox", "Safari"],
    "Memory": ["4GB", "8GB", "16GB"]
}

constraints = [
    'IF [OS] = "MacOS" THEN [Browser] IN {Safari, Chrome}',
    'IF [Memory] = "4GB" THEN [OS] <> "MacOS"'
]

# Generate model text
model_lines = []
for param_name, values in parameters.items():
    values_str = ", ".join(values)
    model_lines.append(f"{param_name}: {values_str}")

if constraints:
    model_lines.append("")
    for constraint in constraints:
        if not constraint.endswith(';'):
            constraint += ';'
        model_lines.append(constraint)

model_text = "\n".join(model_lines)
print(model_text)
```

**Using the helper script (optional):**
The `scripts/pict_helper.py` script provides utilities for model generation and output formatting:

```bash
# Generate model from JSON config
python scripts/pict_helper.py generate config.json

# Format PICT tool output as markdown table
python scripts/pict_helper.py format output.txt

# Parse PICT output to JSON
python scripts/pict_helper.py parse output.txt
```

**To generate actual test cases**, the user can:
1. Save the PICT model to a file (e.g., `model.txt`)
2. Use online PICT tools like:
   - https://pairwise.yuuniworks.com/
   - https://pairwise.teremokgames.com/
3. Or install PICT locally (see references/pict_syntax.md)

### 4. Determine Expected Outputs

For each generated test case, determine the expected outcome based on:
- Business requirements
- Code logic
- Valid/invalid combinations

Create a list of expected outputs corresponding to each test case.

### 5. Format Complete Test Suite

Provide the user with:
1. **PICT Model** - The complete model with parameters and constraints
2. **Markdown Table** - Test cases in table format with test numbers
3. **Expected Outputs** - Expected result for each test case

## Output Format

Present results in this structure:

````markdown
## PICT Model

```
# Parameters
Parameter1: Value1, Value2, Value3
Parameter2: ValueA, ValueB

# Constraints
IF [Parameter1] = "Value1" THEN [Parameter2] = "ValueA";
```

## Generated Test Cases

| Test # | Parameter1 | Parameter2 | Expected Output |
| --- | --- | --- | --- |
| 1 | Value1 | ValueA | Success |
| 2 | Value2 | ValueB | Success |
| 3 | Value1 | ValueB | Error: Invalid combination |
...

## Test Case Summary

- Total test cases: N
- Coverage: Pairwise (all 2-way combinations)
- Constraints applied: N
````

## Best Practices

### Parameter Identification

**Good:**
- Use descriptive names: `AuthMethod`, `UserRole`, `PaymentType`
- Apply equivalence partitioning: `FileSize: Small, Medium, Large` instead of `FileSize: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10`
- Include boundary values: `Age: 0, 17, 18, 65, 66`
- Add negative values for error testing: `Amount: ~-1, 0, 100, ~999999`

**Avoid:**
- Generic names: `Param1`, `Value1`, `V1`
- Too many values without partitioning
- Missing edge cases

### Constraint Writing

**Good:**
- Document rationale: `# Safari only available on MacOS`
- Start simple, add incrementally
- Test constraints work as expected

**Avoid:**
- Over-constraining (eliminates too many valid combinations)
- Under-constraining (generates invalid test cases)
- Complex nested logic without clear documentation

### Expected Output Definition

**Be specific:**
- "Login succeeds, user redirected to dashboard"
- "HTTP 400: Invalid credentials error"
- "2FA prompt displayed"

**Not vague:**
- "Works"
- "Error"
- "Success"

### Scalability

For large parameter sets:
- Use sub-models to group related parameters with different orders
- Consider separate test suites for unrelated features
- Start with order 2 (pairwise), increase for critical combinations
- Typical pairwise testing reduces test cases by 80-90% vs exhaustive

## Common Patterns

### Web Form Testing

```python
parameters = {
    "Name": ["Valid", "Empty", "TooLong"],
    "Email": ["Valid", "Invalid", "Empty"],
    "Password": ["Strong", "Weak", "Empty"],
    "Terms": ["Accepted", "NotAccepted"]
}

constraints = [
    'IF [Terms] = "NotAccepted" THEN [Name] = "Valid"',  # Test validation even if terms not accepted
]
```

### API Endpoint Testing

```python
parameters = {
    "HTTPMethod": ["GET", "POST", "PUT", "DELETE"],
    "Authentication": ["Valid", "Invalid", "Missing"],
    "ContentType": ["JSON", "XML", "FormData"],
    "PayloadSize": ["Empty", "Small", "Large"]
}

constraints = [
    'IF [HTTPMethod] = "GET" THEN [PayloadSize] = "Empty"',
    'IF [Authentication] = "Missing" THEN [HTTPMethod] IN {GET, POST}'
]
```

### Configuration Testing

```python
parameters = {
    "Environment": ["Dev", "Staging", "Production"],
    "CacheEnabled": ["True", "False"],
    "LogLevel": ["Debug", "Info", "Error"],
    "Database": ["SQLite", "PostgreSQL", "MySQL"]
}

constraints = [
    'IF [Environment] = "Production" THEN [LogLevel] <> "Debug"',
    'IF [Database] = "SQLite" THEN [Environment] = "Dev"'
]
```

## Troubleshooting

### No Test Cases Generated

- Check constraints aren't over-restrictive
- Verify constraint syntax (must end with `;`)
- Ensure parameter names in constraints match definitions (use `[ParameterName]`)

### Too Many Test Cases

- Verify using order 2 (pairwise) not higher order
- Consider breaking into sub-models
- Check if parameters can be separated into independent test suites

### Invalid Combinations in Output

- Add missing constraints
- Verify constraint logic is correct
- Check if you need to use `NOT` or `<>` operators

### Script Errors

- Ensure pypict is installed: `pip install pypict --break-system-packages`
- Check Python version (3.7+)
- Verify model syntax is valid

## References

- **references/pict_syntax.md** - Complete PICT syntax reference with grammar and operators
- **references/examples.md** - Comprehensive real-world examples across different domains
- **scripts/pict_helper.py** - Python utilities for model generation and output formatting
- [PICT GitHub Repository](https://github.com/microsoft/pict) - Official PICT documentation
- [pypict Documentation](https://github.com/kmaehashi/pypict) - Python binding documentation
- [Online PICT Tools](https://pairwise.yuuniworks.com/) - Web-based PICT generator

## Examples

### Example 1: Simple Function Testing

**User Request:** "Design tests for a divide function that takes two numbers and returns the result."

**Analysis:**
- Parameters: dividend (number), divisor (number)
- Values: Using equivalence partitioning and boundaries
  - Numbers: negative, zero, positive, large values
- Constraints: Division by zero is invalid
- Expected outputs: Result or error

**PICT Model:**
```
Dividend: -10, 0, 10, 1000
Divisor: ~0, -5, 1, 5, 100

IF [Divisor] = "0" THEN [Dividend] = "10";
```

**Test Cases:**

| Test # | Dividend | Divisor | Expected Output |
| --- | --- | --- | --- |
| 1 | 10 | 0 | Error: Division by zero |
| 2 | -10 | 1 | -10.0 |
| 3 | 0 | -5 | 0.0 |
| 4 | 1000 | 5 | 200.0 |
| 5 | 10 | 100 | 0.1 |

### Example 2: E-commerce Checkout

**User Request:** "Design tests for checkout flow with payment methods, shipping options, and user types."

**Analysis:**
- Payment: Credit Card, PayPal, Bank Transfer (limited by user type)
- Shipping: Standard, Express, Overnight
- User: Guest, Registered, Premium
- Constraints: Guests can't use Bank Transfer, Premium users get free Express

**PICT Model:**
```
PaymentMethod: CreditCard, PayPal, BankTransfer
ShippingMethod: Standard, Express, Overnight
UserType: Guest, Registered, Premium

IF [UserType] = "Guest" THEN [PaymentMethod] <> "BankTransfer";
IF [UserType] = "Premium" AND [ShippingMethod] = "Express" THEN [PaymentMethod] IN {CreditCard, PayPal};
```

**Output:** 12-15 test cases covering all valid payment/shipping/user combinations with expected costs and outcomes.
