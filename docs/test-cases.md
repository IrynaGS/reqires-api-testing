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

- Malformed JSON submitted with `Content-Type: application/json` is rejected with HTTP 400 Bad Request.
- The API returns a structured JSON error and does not generate a user ID or creation timestamp.


## POST Observations

- The POST demo endpoint performs very limited payload validation.
- Missing fields, unexpected data types, null values, nested objects, and additional properties are accepted.
- Incorrect request Content-Type may still result in HTTP 201 even though submitted user fields are not processed.
- Malformed JSON is rejected with HTTP 400.
  
---

# PUT Update User

Endpoint:

`PUT /api/users/{id}`

Postman requests:

- `PUT Update User Positive`
- `PUT Update User - Payload Validation`
- `PUT Update User - Non-existing User`
- `PUT Update User - Invalid Path Parameter`
- `PUT Update User - Security Inputs`
- `PUT Update User - Malformed JSON`

---

## PUT Update User Positive

Postman request:

`PUT Update User Positive`

Request:

`PUT {{base_url}}/api/users/{{positive_user_id}}`

Default test data:

`positive_user_id = 2`

Content-Type:

`application/json`

Example request body:

```json
{
  "name": "Igor Updated",
  "job": "Senior QA Engineer"
}
```

| ID | Scenario | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| PUT-USER-001 | Update existing user | HTTP 200 OK | HTTP 200 OK | Pass |
| PUT-USER-002 | Validate response Content-Type | Response is JSON | `application/json` returned | Pass |
| PUT-USER-003 | Validate returned user data | `name` and `job` match request body | Values match request | Pass |
| PUT-USER-004 | Validate update timestamp | Response contains `updatedAt` | `updatedAt` returned | Pass |
| PUT-USER-005 | Validate update timestamp format | `updatedAt` is a valid date-time | Valid timestamp returned | Pass |
| PUT-USER-006 | Validate response schema | Required fields and data types match schema | Schema validation passed | Pass |

### Expected Response Structure

```text
name      -> string
job       -> string
updatedAt -> string / date-time
```

---

## PUT Update User - Payload Validation

Purpose:

Validate how the endpoint handles missing, empty, invalid, unexpected, and unusual JSON values during an update.

Request:

`PUT {{base_url}}/api/users/{{positive_user_id}}`

Content-Type:

`application/json`

| ID | Scenario | Test Data | Observed Result |
|---|---|---|---|
| PUT-PAYLOAD-001 | Empty JSON body | `{}` | HTTP 200; only `updatedAt` generated |
| PUT-PAYLOAD-002 | Missing `name` | `{"job":"Senior QA Engineer"}` | HTTP 200; supplied field accepted |
| PUT-PAYLOAD-003 | Missing `job` | `{"name":"Igor"}` | HTTP 200; supplied field accepted |
| PUT-PAYLOAD-004 | Empty string values | `{"name":"","job":""}` | HTTP 200; empty values accepted |
| PUT-PAYLOAD-005 | Null value | `{"name":"Igor","job":"QA","nose":null}` | HTTP 200; null value accepted |
| PUT-PAYLOAD-006 | Additional property | `{"name":"Igor","job":"QA","role":"admin"}` | HTTP 200; additional property accepted and returned |
| PUT-PAYLOAD-007 | Multiple unusual values | `{"name":"","job":"","role":"admin","nose":null}` | HTTP 200; supplied values accepted and returned |

### Payload Validation Observations

- The endpoint accepts an empty JSON object and still returns HTTP 200.
- No required-field validation for `name` or `job` was observed.
- Empty values and null values are accepted.
- Additional properties are accepted and reflected in the response.
- The demo endpoint behaves permissively when processing update payloads.

---

## PUT Update User - Non-existing User

Postman request:

`PUT Update User - Non-existing User`

Request:

`PUT {{base_url}}/api/users/{{non_existing_user_id}}`

Test data:

`non_existing_user_id = 1000`

| ID | Scenario | Expected / Observed Result | Status |
|---|---|---|---|
| PUT-NONEXIST-001 | Update non-existing user ID `1000` | HTTP 200 returned | Observed |
| PUT-NONEXIST-002 | Validate response format | JSON response returned | Pass |
| PUT-NONEXIST-003 | Validate submitted fields | Submitted fields are reflected in response | Pass |
| PUT-NONEXIST-004 | Validate update timestamp | `updatedAt` returned | Pass |
| PUT-NONEXIST-005 | Validate timestamp format | `updatedAt` is valid date-time | Pass |

### Non-existing User Observation

A PUT request to user ID `1000`, which is not an existing user in the demo dataset, returns HTTP 200 and generates an `updatedAt` value.

No resource-existence validation was observed.

This behavior is documented as observed behavior rather than classified as a defect because no production API requirement is available requiring HTTP 404 for this demo endpoint.

---

## PUT Update User - Invalid Path Parameter

Postman request:

`PUT Update User - Invalid Path Parameter`

Request:

`PUT {{base_url}}/api/users/{{invalid_user_id}}`

Example test data:

```text
-1
abc
#
```

| ID | Scenario | Test Data | Observed Result |
|---|---|---|---|
| PUT-PATH-001 | Negative ID | `-1` | HTTP 200; update response returned |
| PUT-PATH-002 | Alphabetic ID | `abc` | HTTP 200; update response returned |
| PUT-PATH-003 | Special-character ID | `#` | Invalid path value handled without application server error |

### Invalid Path Parameter Observations

- PUT processing is more permissive than GET processing for invalid identifiers.
- Invalid path parameter values may still receive a successful update-style response.
- The endpoint does not appear to validate the identifier against an existing user before returning an update result.
- These results represent observed ReqRes demo behavior and are not assumed to be the expected contract of a production API.

---

## PUT Update User - Security Inputs

Postman request:

`PUT Update User - Security Inputs`

Request:

`PUT {{base_url}}/api/users/{{security_user_id}}`

Security test data:

```text
2' OR '1'='1
2%
1=1
```

| ID | Scenario | Test Data | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|
| PUT-SEC-001 | SQL-like path input | `2' OR '1'='1` | Input must not cause unintended application behavior | HTTP 403; Cloudflare HTML response | Security control observed |
| PUT-SEC-002 | Encoded / suspicious input | `2%` | Suspicious input should be handled safely | HTTP 403; Cloudflare HTML response | Security control observed |
| PUT-SEC-003 | Boolean / injection-like input | `1=1` | Input must not alter update behavior | HTTP 403; Cloudflare HTML response | Security control observed |

### Security Input Observations

- Security-oriented path values are intercepted by the upstream Cloudflare security layer.
- The response is HTTP 403 Forbidden with `text/html` rather than the normal application JSON response.
- No `updatedAt` value or normal application update result is returned.
- This confirms that these specific requests were blocked upstream.
- It does not prove that the underlying application itself is immune to injection vulnerabilities.

---

## PUT Update User - Malformed JSON

Postman request:

`PUT Update User - Malformed JSON`

Request:

`PUT {{base_url}}/api/users/{{positive_user_id}}`

Content-Type:

`application/json`

Example malformed request body:

```text
{
  "name": "Igor"
  "job": "QA Engineer",
}
```

| ID | Scenario | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| PUT-JSON-001 | Send malformed JSON | HTTP 400 Bad Request | HTTP 400 Bad Request | Pass |
| PUT-JSON-002 | Validate error response type | Response is JSON | JSON returned | Pass |
| PUT-JSON-003 | Validate error code | Response contains `error = invalid_json` | `invalid_json` returned | Pass |
| PUT-JSON-004 | Validate error message | Response contains non-empty error message | Error message returned | Pass |
| PUT-JSON-005 | Validate no update result | Response does not contain `updatedAt` | No update result returned | Pass |

### Malformed JSON Observation

Malformed JSON submitted with `Content-Type: application/json` is rejected with HTTP 400 Bad Request.

The API returns a structured `invalid_json` response and does not return the normal PUT update result.

---

## PUT Observations

- Valid PUT requests return HTTP 200 with the supplied fields and an `updatedAt` timestamp.
- The demo endpoint performs very limited payload validation.
- Empty bodies, missing fields, null values, empty values, and additional properties may still be accepted.
- A non-existing but syntactically valid user ID can still receive HTTP 200.
- Invalid path parameters may also be processed without resource validation.
- Security-oriented path inputs are intercepted by the upstream Cloudflare layer and return HTTP 403.
- Malformed JSON is rejected with HTTP 400 and a structured `invalid_json` error.
- These findings describe observed behavior of the ReqRes demo API and should not automatically be interpreted as production API requirements or vulnerabilities.

---

# DELETE User

Endpoint:

`DELETE /api/users/{id}`

Postman requests:

- `DELETE User Positive`
- `DELETE User - Non-existing User`
- `DELETE User - Invalid Path Parameter`
- `DELETE User - Security Inputs`

---

## DELETE User Positive

Postman request:

`DELETE User Positive`

Request:

`DELETE {{base_url}}/api/users/{{positive_user_id}}`

Default test data:

`positive_user_id = 2`

Request body:

`none`

| ID | Scenario | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| DELETE-USER-001 | Delete existing user | HTTP 204 No Content | HTTP 204 No Content | Pass |
| DELETE-USER-002 | Validate response body | Response body is empty | Empty response body returned | Pass |
| DELETE-USER-003 | Validate response time | Response time is below 2000 ms | Response time within limit | Pass |

### Positive DELETE Observation

A DELETE request for an existing user returns HTTP 204 No Content with an empty response body.

---

## DELETE User - Non-existing User

Postman request:

`DELETE User - Non-existing User`

Request:

`DELETE {{base_url}}/api/users/{{non_existing_user_id}}`

Test data:

`non_existing_user_id = 1000`

Request body:

`none`

| ID | Scenario | Expected / Observed Result | Status |
|---|---|---|---|
| DELETE-NONEXIST-001 | Delete non-existing user ID `1000` | HTTP 204 No Content returned | Observed |
| DELETE-NONEXIST-002 | Validate response body | Empty response body returned | Pass |
| DELETE-NONEXIST-003 | Validate server handling | Request is handled without server error | Pass |

### Non-existing User Observation

A DELETE request to user ID `1000`, which is not an existing user in the demo dataset, still returns HTTP 204 No Content.

No resource-existence validation was observed.

This behavior is documented as observed behavior rather than classified as a defect because no production API requirement is available requiring HTTP 404 for this demo endpoint.

---

## DELETE User - Invalid Path Parameter

Postman request:

`DELETE User - Invalid Path Parameter`

Request:

`DELETE {{base_url}}/api/users/{{invalid_user_id}}`

Example test data:

```text
-1
abc
%23
```

`%23` is the URL-encoded representation of `#`.

Request body:

`none`

| ID | Scenario | Test Data | Observed Result |
|---|---|---|---|
| DELETE-PATH-001 | Negative user ID | `-1` | HTTP 204 No Content |
| DELETE-PATH-002 | Alphabetic user ID | `abc` | HTTP 204 No Content |
| DELETE-PATH-003 | Special-character user ID | `%23` | HTTP 204 No Content |

### Invalid Path Parameter Observations

- Invalid user identifiers still receive HTTP 204 No Content.
- The DELETE endpoint does not appear to validate whether the supplied identifier is a valid existing user before returning a success-style response.
- The response body remains empty.
- These results represent observed ReqRes demo behavior and are not assumed to be the expected contract of a production API.

---

## DELETE User - Security Inputs

Postman request:

`DELETE User - Security Inputs`

Request:

`DELETE {{base_url}}/api/users/{{security_user_id}}`

Security test data:

```text
2' OR '1'='1
2%
1=1
```

Request body:

`none`

| ID | Scenario | Test Data | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|
| DELETE-SEC-001 | SQL-like path input | `2' OR '1'='1` | Input must not cause unintended application behavior | HTTP 403 Forbidden; Cloudflare HTML response | Security control observed |
| DELETE-SEC-002 | Encoded / suspicious input | `2%` | Suspicious input should be handled safely | HTTP 403 Forbidden; Cloudflare HTML response | Security control observed |
| DELETE-SEC-003 | Boolean / injection-like input | `1=1` | Input must not alter delete behavior | HTTP 403 Forbidden; Cloudflare HTML response | Security control observed |

### Security Input Observations

- Security-oriented path values are intercepted by the upstream Cloudflare security layer.
- The response is HTTP 403 Forbidden with `text/html` instead of the normal DELETE response.
- No HTTP 204 application response is returned.
- This confirms that these specific requests were blocked upstream.
- It does not prove that the underlying application itself is immune to injection vulnerabilities.

---

## DELETE Overall Observations

- A valid existing user ID returns HTTP 204 No Content.
- A non-existing user ID also returns HTTP 204 No Content.
- Invalid path parameter values such as negative IDs, alphabetic values, and encoded special characters may also return HTTP 204.
- The endpoint does not appear to validate resource existence or identifier format before returning the normal success-style DELETE response.
- Security-oriented path inputs are intercepted by the upstream Cloudflare layer and return HTTP 403.
- These findings describe observed behavior of the ReqRes demo API and should not automatically be interpreted as production API requirements or vulnerabilities.

---

# Overall Observations

- GET requests return user data for valid IDs and HTTP 404 for ordinary invalid or non-existing identifiers.
- Security-oriented GET and PUT path inputs may be intercepted by the upstream Cloudflare security layer and return HTTP 403.
- POST and PUT demo endpoints perform limited payload validation and accept missing fields, unexpected data types, null values, and additional properties.
- POST requests with an incorrect Content-Type may return HTTP 201 even when submitted user fields are not processed.
- PUT requests may return HTTP 200 for non-existing and invalid user identifiers.
- Malformed JSON is rejected with HTTP 400 and a structured `invalid_json` response.
- These findings describe observed behavior of the ReqRes demo API and should not automatically be interpreted as production requirements or vulnerabilities.
