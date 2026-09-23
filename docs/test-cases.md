# GET Single User – Test Cases

Endpoint:

`GET /api/users/{id}`

## Scope

The purpose of these tests is to validate:

- successful retrieval of an existing user
- handling of invalid and non-existing user IDs
- response structure and data types
- behavior for suspicious or injection-like path values
- interaction with the upstream security layer

## Test Cases

| ID | Scenario | Test Data | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|
| GET-USER-001 | Retrieve existing user | `2` | HTTP 200; JSON response; returned user ID matches requested ID | HTTP 200; JSON response; user ID = 2 | Pass |
| GET-USER-002 | Validate required user fields | `2` | Response contains `id`, `email`, `first_name`, `last_name`, `avatar` | All required fields returned | Pass |
| GET-USER-003 | Validate user field data types | `2` | `id` is number; remaining user fields are strings | All field types matched expectations | Pass |
| GET-USER-004 | Negative user ID | `-1` | Invalid/non-existing user should not return user data or server error | HTTP 404 Not Found | Observed / Acceptable |
| GET-USER-005 | Very large user ID | `1000` | Non-existing user should not return user data or server error | HTTP 404 Not Found | Observed / Acceptable |
| GET-USER-006 | Alphabetic user ID | `abc` | Invalid identifier should be handled safely | HTTP 404 Not Found | Observed / Acceptable |
| GET-USER-007 | Special-character user ID | special-character value | Invalid identifier should be handled safely | HTTP 404 Not Found | Observed / Acceptable |
| GET-USER-008 | SQL-like input in user ID | SQL-like payload | Payload must not be executed or expose unintended data | HTTP 403 Forbidden; Cloudflare HTML block page | Security control observed |
| GET-USER-009 | Encoded/suspicious input | value containing `%` | Suspicious input should be rejected safely | HTTP 403 Forbidden; Cloudflare HTML block page | Security control observed |
| GET-USER-010 | Boolean/injection-like expression | logical payload | Input must not affect query behavior or return unintended data | HTTP 403 Forbidden; Cloudflare HTML block page | Security control observed |

## Observations

- Existing valid user IDs return HTTP 200 with JSON user data.
- Invalid or non-existing identifiers such as negative, large, and alphabetic values return HTTP 404.
- SQL-like and certain suspicious inputs are intercepted by Cloudflare and return HTTP 403 with an HTML block page.
- The HTTP 403 result indicates that the upstream security layer blocked the request before the normal API response was returned.
- This behavior does not prove that the underlying application itself is immune to injection vulnerabilities; it only confirms that these specific requests were blocked by the upstream protection layer.
