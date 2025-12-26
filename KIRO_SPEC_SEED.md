Kiro Spec Seed
# Kiro Spec Seed — Request Validation API (TypeScript + AWS CDK)

## 1) Objective
Build a small HTTP API that validates and normalizes JSON input using a **strict, spec-first contract**.  
The system must produce deterministic, machine-readable validation errors and a normalized payload on success.

This project prioritizes correctness, contract clarity, and maintainability over feature breadth.

## 2) Technology Constraints (Explicit)
### 2.1 Implementation Language
- All application code **must be written in TypeScript**.
- This includes:
  - AWS Lambda handler
  - Validation logic
  - Normalization logic
  - Types and contracts
  - Tests

### 2.2 Infrastructure as Code
- All infrastructure **must be defined using AWS CDK in TypeScript**.
- No manual console configuration.
- No CloudFormation templates written by hand.
- CDK synth output is considered a build artifact, not a source of truth.

### 2.3 Runtime Environment
- AWS Lambda runtime:
  - Node.js (LTS version supported by AWS at time of implementation)
- API exposed via Amazon API Gateway.

## 3) Scope
### In Scope
- One endpoint: `POST /validate`
- JSON request/response
- Strict schema validation + rule-based validation
- Deterministic error aggregation
- Input normalization on success
- Automated tests for core logic

### Out of Scope
- Authentication / authorization
- Databases or persistence
- UI / frontend
- Multiple endpoints
- Async workflows
- Advanced observability (beyond basic logging)

## 4) Public API Contract
### Endpoint
- `POST /validate`
- Request `Content-Type`: `application/json`
- Response `Content-Type`: `application/json`

### Success Response (HTTP 200)
```json
{
  "valid": true,
  "normalized": {
    "email": "user@example.com",
    "age": 21,
    "country": "US"
  }
}
```
### Error Response (HTTP 400)
```json
{
  "valid": false,
  "errors": [
    {
      "field": "age",
      "message": "Must be at least 18",
      "code": "MIN_VALUE"
    }
  ]
}
```
### Error Object Requirements
- field: the field name or "__root__" for payload-wide errors.
- message: human-readable explanation.
- code: stable error identifier.

### Allowed Error Codes
- REQUIRED
- TYPE_INVALID
- FORMAT_INVALID
- MIN_VALUE
- UNKNOWN_FIELD
- LENGTH_INVALID

## 5) Input Contract
### Accepted Fields
- email (required, string)
- age (required, integer)
- country (optional, string)

### Strictness Rules
- Reject unknown fields.
- Reject missing required fields.
- Reject incorrect types.
- Do not coerce types implicitly.

## 6) Validation Rules
- email
     - Must be non-empty after trimming.
  - Must be a valid email format.
  - Must be lowercased.

age
- Must be an integer.
- Must be >= 18.

country (optional)
- If present:
  - Must be exactly 2 characters.
  - Must be uppercased.
  - Must contain only alphabetic characters (A–Z).

## 7) Normalization Rules
### On successful validation:
- email: trimmed and lowercased.
- age: preserved as integer.
- country: trimmed and uppercased (if present).
- Normalization must not be applied to invalid input.

## 8) Error Semantics
### Aggregation
- All validation errors must be returned in a single response.

### Deterministic Ordering
- Errors must be ordered deterministically:
  - __root__
  - email
  - age
  - country
- Within a field, errors are ordered by a fixed error-code priority.

### Security
- Error responses must not expose stack traces or internal exceptions.

## 9) Implementation Structure (Guidance)
### Application Code (TypeScript)
- handler/
  - Lambda handler adapting API Gateway events
- validation/
  - Input validation rules and error aggregation
- normalization/
  - Post-validation transformations
- types/
  - Request/response and error type definitions
- tests/
  - Unit tests for validation and normalization

## Infrastructure Code (CDK, TypeScript)
Define:
- Lambda function
- API Gateway (REST or HTTP)
- IAM roles with least privilege
Export:
- Deployed API URL as a CloudFormation output

## 10) Test Requirements
### Unit tests (TypeScript) must cover:
- Valid payload normalization
- Missing required fields
- Unknown fields
- Type mismatches
- Invalid formats
- Minimum age violation
- Invalid JSON payloads

## 11) Acceptance Criteria
- All code and infrastructure is written in TypeScript.
- The POST /validate endpoint conforms exactly to the contract.
- Validation behavior is deterministic and repeatable.
- Normalization only occurs on valid input.
- Tests pass locally and in CI.
- CDK deployment produces a working public endpoint.
A README document containing:
- Local development steps
- Test execution
- Deployment instructions
- Comprehensive "Testing Your Deployed API" section with:
  - 18 different test scenarios covering all validation cases
  - Valid requests: Complete, minimal, and normalization examples
  - Invalid requests: Email, age, country, and schema violation tests
  - Request format errors: Malformed JSON, wrong content-type, wrong HTTP method
  - Ready-to-use curl commands that users can copy and paste
  - Expected response examples for both success and error cases
