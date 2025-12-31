# Testing

- Tests MUST cover the functionality being implemented
- NEVER ignore the output of the system or the tests - logs and messages often contain CRITICAL information
- Test output must be pristine to pass
- If the logs are supposed to contain errors, capture and test it
- NO EXCEPTIONS POLICY: Every project MUST have unit tests, integration tests, AND end-to-end tests. If you believe a test type doesn't apply, you need explicit authorization: "I AUTHORIZE YOU TO SKIP WRITING TESTS THIS TIME"

## We Practice TDD

- Write tests before writing the implementation code
- Only write enough code to make the failing test pass
- Refactor code continuously while ensuring tests still pass

## TDD Implementation Process

1. Write a failing test that defines a desired function or improvement
2. Run the test to confirm it fails as expected
3. Write minimal code to make the test pass
4. Run the test to confirm success
5. Refactor code to improve design while keeping tests green
6. Repeat the cycle for each new feature or bugfix
