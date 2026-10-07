# GET Single User – Test Cases

Endpoint:

`GET /api/users/{id}`

Postman requests:

- `GET Single User Positive`
- `GET Single User Negative`
- `GET Single User Security Inputs`

Environment variables:

- `positive_user_id`
- `negative_user_id`
- `security_user_id`

## Positive Tests

| ID | Scenario | Test Data | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|
| GET-USER-001 | Retrieve existing user | `positive_user_id = 2` | HTTP 200; JSON response; requested user returned | HTTP 200; JSON response; user ID = 2 | Pass |
| GET-USER-002 | Validate required fields | `positive_user_id = 2` | Response contains `id`, `email`, `first_name`, `last_name`, `avatar` | All required fields returned | Pass |
| GET-USER-003 | Validate field data types | `positive_user_id = 2` | `id` is number; remaining fields are strings | Types matched expectations | Pass |
| GET-USER-004 | Validate requested user ID | `positive_user_id = 2` | Response `data.id` matches requested ID | IDs match | Pass |
| GET-USER-005 | Validate response content type | `positive_user_id = 2` | Response Content-Type is JSON | JSON returned | Pass |

## Functional Negative Tests

Postman request:

`GET Single User Negative`

| ID | Scenario | Test Data | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|
| GET-USER-006 | Negative user ID | `-1` | Invalid/non-existing user handled safely | HTTP 404 | Observed / Acceptable |
| GET-USER-007 | Very large user ID | `1000` | Non-existing user handled safely | HTTP 404 | Observed / Acceptable |
| GET-USER-008 | Alphabetic user ID | `abc` | Invalid identifier handled safely | HTTP 404 | Observed / Acceptable |
| GET-USER-009 | Special-character user ID | `#` | Invalid identifier handled safely | HTTP 404 | Observed / Acceptable |

## Security-Oriented Input Tests

Postman request:

`GET Single User Security Inputs`

Request:

`GET {{base_url}}/api/users/{{security_user_id}}`

| ID | Scenario | Test Data | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|
| GET-USER-010 | SQL-like input | `2' OR '1'='1` | Input must not execute or expose unintended data | HTTP 403 Forbidden; Cloudflare HTML block page | Security control observed |
| GET-USER-011 | Encoded / suspicious input | `2%` | Suspicious input should be rejected safely | HTTP 403 Forbidden; Cloudflare HTML block page | Security control observed |
| GET-USER-012 | Boolean / injection-like input | `1=1` | Input must not alter query behavior or return unintended data | HTTP 403 Forbidden; Cloudflare HTML block page | Security control observed |

### Security Test Data

```text
2' OR '1'='1
2%
1=1
```

### GET Observations

- Valid existing user IDs return HTTP 200 with JSON user data.
- Invalid and non-existing identifiers return HTTP 404.
- SQL-like and suspicious inputs are intercepted by the upstream Cloudflare security layer and return HTTP 403.
- The HTTP 403 response confirms that these specific requests were blocked by the upstream security layer.
- This does not prove that the underlying application itself is immune to injection vulnerabilities.

---

# POST Create User

Endpoint:

`POST /api/users`

Postman requests:

- `POST Create User Positive`
- `POST Create User - Payload Validation`
- `POST Create User - Content Type Validation`
- `POST Create User - Malformed JSON`

---

## POST Create User Positive

Request:

`POST {{base_url}}/api/users`

Content-Type:

`application/json`

Example request body:

```json
{
  "name": "Ivan",
  "job": "QA Engineer"
}
```

| ID | Scenario | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| POST-USER-001 | Create user with valid request body | HTTP 201 Created | HTTP 201 Created | Pass |
| POST-USER-002 | Validate response Content-Type | Response is JSON | `application/json` returned | Pass |
| POST-USER-003 | Validate returned user data | Response `name` and `job` match request body | Values match request | Pass |
| POST-USER-004 | Validate generated user ID | Response contains non-empty `id` | Generated ID returned | Pass |
| POST-USER-005 | Validate creation timestamp | Response contains valid `createdAt` | Valid timestamp returned | Pass |
| POST-USER-006 | Validate response schema | Required fields and data types match expected schema | Schema validation passed | Pass |

### Expected Response Structure

```text
name      -> string
job       -> string
id        -> string
createdAt -> string / date-time
```

---

## POST Create User - Payload Validation

Purpose:

Validate how the API handles missing, invalid, unexpected, and unusual JSON payload values.

Content-Type:

`application/json`

| ID | Scenario | Test Data | Observed Result |
|---|---|---|---|
| POST-NEG-001 | Empty body | `{}` | HTTP 201; generated `id` and `createdAt` |
| POST-NEG-002 | Missing `name` | `{"job":"Developer"}` | HTTP 201; `job` returned; no `name` |
| POST-NEG-003 | Missing `job` | `{"name":"Igor"}` | HTTP 201; `name` returned; no `job` |
| POST-NEG-004 | Empty `name` | `{"name":"","job":"QA Engineer"}` | HTTP 201; value accepted |
| POST-NEG-005 | Whitespace-only `name` | `{"name":"   ","job":"QA Engineer"}` | HTTP 201; value accepted |
| POST-NEG-006 | Null value | `{"name":null,"job":"QA Engineer"}` | HTTP 201; value accepted |
| POST-NEG-007 | Numeric `name` | `{"name":5,"job":"QA Engineer"}` | HTTP 201; numeric value accepted |
| POST-NEG-008 | Boolean `job` | `{"name":"Igor","job":true}` | HTTP 201; boolean value accepted |
| POST-NEG-009 | Unexpected property | `{"name":"Igor","job":"QA Engineer","role":"admin"}` | HTTP 201; extra field accepted and returned |
| POST-NEG-010 | Nested object instead of string | `{"name":{"first":"Igor"},"job":"QA Engineer"}` | HTTP 201; object accepted |
| POST-NEG-011 | Repeated identical request | Same valid body submitted twice | HTTP 201 each time; different generated IDs |

### Payload Validation Observations

- The demo endpoint does not enforce `name` or `job` as required fields.
- The endpoint accepts unexpected data types for `name` and `job`.
- Null, empty, and whitespace-only values are accepted.
- Unexpected properties such as `role` are accepted and returned.
- Nested objects are accepted where a string might normally be expected.
- Repeated identical POST requests create new resources with different IDs.
- No uniqueness constraint for `name` and `job` was observed.

---

## POST Create User - Content Type Validation

Purpose:

Validate behavior when JSON-looking content is sent with an incorrect request Content-Type.

Request Content-Type:

`text/plain`

Example request body:

```text
{
  "name": "Igor",
  "job": "QA Engineer",
  "role": "admin"
}
```

| ID | Scenario | Expected / Observed Behavior | Status |
|---|---|---|---|
| POST-CTYPE-001 | Send JSON-looking body as `text/plain` | API returns HTTP 201 | Observed |
| POST-CTYPE-002 | Validate generated ID | Response contains `id` | Pass |
| POST-CTYPE-003 | Validate creation timestamp | Response contains valid `createdAt` | Pass |
| POST-CTYPE-004 | Validate submitted fields are processed | `name`, `job`, and `role` are not parsed into the response | Observed |
| POST-CTYPE-005 | Validate request Content-Type | Request Content-Type is not `application/json` | Pass |

### Content-Type Observation

When the request body contains JSON-looking text but the request Content-Type is `text/plain`, the endpoint still returns HTTP 201 and generates `id` and `createdAt`.

However, the submitted user fields are not parsed and are not returned in the created-user response.

Therefore, HTTP 201 alone does not confirm that the submitted user data was successfully processed.

---

## POST Create User - Malformed JSON

Purpose:

Validate API behavior when the request declares JSON content but the JSON syntax is invalid.

Content-Type:

`application/json`

Example malformed body:

```text
{
  "name": "Igor",
  "job": "QA Engineer",
```

| ID | Scenario | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| POST-JSON-001 | Send malformed JSON | HTTP 400 Bad Request | HTTP 400 Bad Request | Pass |
| POST-JSON-002 | Validate error response type | Response is JSON | JSON returned | Pass |
| POST-JSON-003 | Validate error code | Response contains `error = invalid_json` | `invalid_json` returned | Pass |
| POST-JSON-004 | Validate error message | Response contains non-empty error message | Error message returned | Pass |
| POST-JSON-005 | Validate no user is created | Response does not contain `id` or `createdAt` | No generated user fields returned | Pass |

### Malformed JSON Observation

Malformed JSON submitted with `Content-Type: application/json` is rejected with HTTP 400 Bad Request.

The API returns a structured JSON error and does not generate a user ID or creation timestamp.

---

# Overall Observations

- The GET endpoint distinguishes between ordinary invalid identifiers and suspicious inputs intercepted by the upstream security layer.
- The POST demo endpoint performs very limited payload validation.
- Missing fields, unexpected data types, null values, nested objects, and additional properties are accepted.
- Incorrect request Content-Type may still result in HTTP 201 even though submitted user fields are not processed.
- Malformed JSON is correctly rejected with HTTP 400.
- These behaviors describe the observed ReqRes demo API and should not automatically be treated as production API requirements.
