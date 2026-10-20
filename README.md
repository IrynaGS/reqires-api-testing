# ReqRes API Testing Portfolio

API testing portfolio project built around the ReqRes demo API.

The project demonstrates API test design in Postman, positive and negative testing, security-oriented input checks, response validation, JSON Schema validation, and documentation of observed API behavior.

## Scope

The project covers CRUD operations for the `/api/users` endpoint:

- GET single user
- POST create user
- PUT update user
- DELETE user

Testing includes positive scenarios, functional negative scenarios, payload validation, path parameter validation, malformed JSON handling, Content-Type behavior, and security-oriented inputs.

## Tools

- Postman
- JavaScript assertions
- JSON Schema validation
- GitHub
- Markdown

## Test Coverage

### GET

Tests include:

- Existing user
- Non-existing and invalid user IDs
- Response fields and data types
- Response Content-Type
- Special-character path values
- SQL-like and suspicious path inputs
- Upstream security-layer behavior

### POST

Tests include:

- Successful user creation
- Response data validation
- JSON Schema validation
- Missing fields
- Empty and null values
- Unexpected data types
- Additional properties
- Nested objects
- Repeated identical requests
- Incorrect Content-Type
- Malformed JSON

### PUT

Tests include:

- Successful user update
- Response data and schema validation
- Empty and unusual payloads
- Missing fields
- Additional properties
- Non-existing user ID
- Invalid path parameters
- Security-oriented path inputs
- Malformed JSON

### DELETE

Tests include:

- Existing user
- Non-existing user
- Invalid path parameters
- Security-oriented path inputs
- HTTP 204 behavior
- Empty response validation

## Repository Structure

```text
reqres-api-testing/
├── README.md
├── docs/
│   └── test-cases.md
└── postman/
    └── ReqRes_API_Testing_Portfolio.postman_collection.json
```

## Environment Variables

The Postman collection uses environment variables including:

- `base_url`
- `api_key`
- `positive_user_id`
- `non_existing_user_id`
- `invalid_user_id`
- `security_user_id`

The API key is not stored in this repository.

## Test Results

Latest full Postman collection run:

- 81 tests executed
- 81 passed
- 0 failed
- 0 errors

## Key Findings

- Valid GET requests return user data for existing IDs, while ordinary invalid and non-existing IDs return HTTP 404.
- POST and PUT endpoints demonstrate limited payload validation and accept several missing, unusual, or unexpected values.
- A POST request may return HTTP 201 even when submitted fields are not processed if an incorrect Content-Type is used.
- PUT may return HTTP 200 for non-existing or invalid user identifiers.
- DELETE may return HTTP 204 for existing, non-existing, and ordinary invalid identifiers.
- Malformed JSON is rejected with HTTP 400 and a structured `invalid_json` response.
- Security-oriented path inputs may be intercepted by the upstream Cloudflare security layer and return HTTP 403.
- Upstream blocking of suspicious inputs does not prove that the underlying application itself is immune to injection vulnerabilities.

## Documentation

Detailed test cases, test data, expected results, observed results, and API behavior notes are documented in:

`docs/test-cases.md`

## Next Steps

- Add Playwright API automation
- Convert repeatable scenarios into parameterized automated tests
- Add CI/CD execution
- Add automated test reporting
