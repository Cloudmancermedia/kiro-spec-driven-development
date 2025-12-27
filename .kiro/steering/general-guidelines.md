---
inclusion: always
---

# General Development Guidelines

## Code Style

- Use 2 spaces for indentation
- Use single quotes for strings
- Add trailing commas in objects and arrays
- Use descriptive variable and function names
- Prefer const over let, avoid var

## TypeScript

- Use explicit types for all function parameters and return values
- Avoid `any` type - use `unknown` if type is truly unknown
- Use type inference for simple variable declarations

## Import Style

- Always use destructured imports instead of namespace imports
- Example: `import { Stack } from 'aws-cdk-lib'` not `import * as cdk from 'aws-cdk-lib'`
- Group imports: external packages, then internal modules, then relative imports

## AWS

### CDK

- Any time I want to build something that uses AWS infrastructure, always assume it will use the AWS CDK for Typescript to deploy that infrastructure, unless I explicitly say I want to use something else
- Always use destructured imports for CDK constructs
- Prefer L2 constructs over L1 when available
- Tag all resources with appropriate metadata
- Use meaningful construct IDs that reflect their purpose
- Follow least privilege principle for IAM policies
- Always search the AWS knowledge base and CDK docs before building an architecture to ensure it is being done with best practices and the most up to date patterns, libraries, and tools.

### NodeJs Lambda Packaging & Modules 

#### General

- When using Node as a runtime, always use the most current version

#### Bundling Standard
- All Node.js/TypeScript Lambdas **MUST** be built as **esbuild bundles**  
  (one artifact per Lambda).
- Bundles **SHOULD** enable:
  - Tree-shaking
  - Minification for production environments
- Source maps:
  - **SHOULD** be enabled for non-production environments
  - **MAY** be enabled in production if logging and observability tooling supports them safely

#### Module System Standard

- The default module system for Lambda functions is **CommonJS**.
- **ESM is allowed only** when:
  - Required by third-party dependencies, **or**
  - Explicitly approved as an architecture decision
- Mixing **ESM** and **CommonJS** across Lambdas **SHOULD be avoided**.  
  If adopted, the choice **MUST** be consistent within a repository.

#### Dependencies & Shared Code

- Shared code **MUST**:
  - Live in versioned internal packages (workspace libraries), **or**
  - Be bundled directly into the Lambda function package
- Shared code **MUST NOT** primarily rely on Lambda Layers.
- Lambda Layers **MAY** be used for:
  - Native binaries or heavy dependencies (e.g., image processing libraries)
  - Shared runtime components that change infrequently
- Raw `node_modules` **MUST NOT** be shipped with a Lambda bundle unless:
  - Bundling is not feasible, **and**
  - The exception is documented with justification

#### Build Reproducibility

- Repositories **MUST**:
  - Commit a lockfile
  - Use deterministic dependency installation in CI
- Lambda bundles **MUST**:
  - Be generated in CI
  - Be built from a clean checkout
  - Avoid local-only or developer-specific artifacts

#### Operational Constraints

- Each Lambda function **SHOULD** meet defined bundle size budgets.
- Exceptions **MUST**:
  - Include justification
  - Document the dependency causing the increase
  - Describe the operational impact
- Bundles **MUST** include:
  - Only what the handler requires
  - No unused dependencies or “kitchen sink” packages

## Documentation

- Add comments only for complex business logic
- Document public functions with JSDoc
- Avoid obvious comments that repeat the code
- Keep README files up to date with setup instructions

## Security

- Never log sensitive data (passwords, tokens, PII)
- Always validate and sanitize user input
- Use environment variables for secrets and configuration
- Follow OWASP best practices

## Assistant Behavior

- Only suggest code changes necessary to solve the stated problem
- Avoid refactoring or improvements unless explicitly requested
- Respect existing patterns and conventions in the codebase
- Never fabricate features, APIs, or configuration options
- Explicitly state uncertainty rather than guessing
- Provide official documentation links when discussing third-party tools
- Say "I don't have verified information" when uncertain
- Never invent API parameters, CLI flags, or claim unverified features exist
- Be concise and direct in responses
- Focus on actionable solutions

## Error Handling

- Handle errors gracefully with meaningful messages
- Use try-catch blocks for async operations
- Log errors with sufficient context for debugging
- Provide user-friendly error messages in UI

## Performance

- Avoid premature optimization
- Profile before optimizing
- Consider lazy loading for large components or data
- Use appropriate data structures for the task

## Context7 MCP Usage

- Always use Context7 MCP tools when code generation, setup, configuration, or library/API documentation is needed
- Automatically invoke `resolve-library-id` to find the correct library before using `get-library-docs`
- Use Context7 proactively without waiting for explicit requests
- Leverage Context7 for up-to-date documentation on frameworks, libraries, and APIs
