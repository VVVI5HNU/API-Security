# 🔐 API Security Testing – Academic Teaching Guide

This repository is designed for structured cybersecurity education focused on API security testing concepts.

It covers:

- Endpoint discovery
- Hidden functionality
- Mass assignment
- Server-side parameter pollution (SSPP)
- JSON injection risks
- Backend parsing behavior
---

# 1️⃣ API Endpoint Discovery

During API testing, attempt to identify exposed documentation endpoints:

```
/api
/swagger/index.html
/openapi.json
/swagger.json
/docs
```

These may reveal:

- Available routes
- Request structure
- Authentication methods
- Hidden parameters
- Versioning

---

# 2️⃣ Finding and Testing Unused API Endpoints

If you observe:

```
PUT /api/user/update
```

Consider whether related endpoints may exist:

```
/api/user/delete
/api/user/add
/api/user/create
/api/user/admin
```

Test different HTTP methods:

- GET
- POST
- PUT
- PATCH
- OPTIONS

Observe:

- Allowed methods
- Required headers (Content-Type: application/json)
- Missing JSON parameters
- Informative error messages

Example:
If response shows `"price parameter is missing"`, it indicates expected backend structure.

---

# 3️⃣ Mass Assignment (Auto-Binding)

Mass assignment occurs when frameworks automatically bind request parameters to internal object fields.

Example:

```
PATCH /api/users/
{
    "username": "wiener",
    "email": "wiener@example.com"
}
```

GET request:

```
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": false
}
```

Hidden fields like:

- id
- isAdmin

may be internally bound.

Testers should compare GET and PATCH responses to identify hidden object properties.

---

# 4️⃣ Server-Side Parameter Pollution (SSPP)

## Basic Concept

Server-side parameter pollution occurs when user input is concatenated into internal API requests without proper sanitization.

---

## Example: Query Parameter Injection

Client request:

```
GET /userSearch?name=peter&back=/home
```

Internal request:

```
GET /users/search?name=peter&publicProfile=true
```

Testing with encoded characters:

```
name=peter%26foo=xyz
```

Internally becomes:

```
name=peter&foo=xyz&publicProfile=true
```

Purpose of testing:

- Determine if backend accepts extra parameters
- Check if parameters can override existing ones
- Analyze parsing behavior

---

## Duplicate Parameter Behavior

Example:

```
GET /userSearch?name=peter%26name=carlos&back=/home
```

Internal:

```
GET /users/search?name=peter&name=carlos&publicProfile=true
```

Framework differences:

| Technology | Behavior |
|------------|----------|
| PHP        | Last parameter wins |
| ASP.NET    | Parameters merged |
| Node.js    | First parameter wins |

Understanding parsing behavior is critical.

---

# 5️⃣ Server-Side Parameter Pollution – Forgot Password Scenario

This scenario demonstrates backend query manipulation concepts.

---

## Step 1: Observe Normal Behavior

Capture forgot password request.

Example request body:

```
username=admin
```

Try entering wrong username and observe:

- "Invalid user" message
- Error behavior

---

## Step 2: Attempt to Add Second Parameter

Attempt adding URL-encoded &:

```
username=admin%26x=y
```

If output gives "invalid parameter", it suggests:

- `%26x=y` is not treated as part of username
- Backend may be parsing injected parameter

---

## Step 3: Attempt Query Truncation

Use URL-encoded `#`:

```
username=administrator%23
```

If response returns:

```
Field not specified
```

This suggests:

- Server-side query may include additional parameter like `field`
- `#` may truncate internal query

---

## Step 4: Add Field Parameter

```
username=administrator%26field=x%23
```

If response shows:

```
Invalid field
```

This suggests:

- Backend recognizes `field` parameter
- Injected parameter is being parsed

---

## Step 5: Identify Valid Field Value

Through testing, suppose `email` is recognized.

```
username=administrator%26field=email%23
```

---

## Step 6: Analyze JavaScript Files

Check HTTP history for:

```
/static/js/forgotPassword.js
```

Look for endpoints like:

```
/forgot-password?reset_token=${resetToken}
```

---

## Step 7: Parameter Mapping Observation

If `email` is changed to:

```
reset_token%23
```

Observe backend behavior carefully.

---

## Step 8: Password Reset Endpoint Structure

Example endpoint:

```
/forgot-password?reset_token=123456789
```

Testers should analyze:

- Token handling
- Parameter mapping
- Backend trust assumptions
- Query construction logic

---

# 6️⃣ JSON Injection Scenario

Consider:

Client-side request:

```
POST /myaccount
{"name": "peter"}
```

Internal request:

```
PATCH /users/7312/update
{"name":"peter"}
```

If user input is concatenated improperly into backend JSON, injected attributes may be parsed.

Example modified input:

```
{"name": "peter\",\"access_level\":\"administrator"}
```

If decoded and inserted without proper encoding, server-side may interpret:

```
{"name":"peter","access_level":"administrator"}
```

This demonstrates risks of:

- Improper JSON serialization
- Missing schema validation
- Automatic object binding

---

# 7️⃣ Key Academic Teaching Points

Testers must understand:

- How internal API requests are constructed
- How URL encoding affects backend logic
- How different frameworks parse duplicate parameters
- How truncation characters (`#`) alter query logic
- How JSON concatenation can alter internal objects
- Why schema validation is critical

---

# 8️⃣ Defensive Measures

## Preventing SSPP

- Use parameterized internal API calls
- Never concatenate user input into query strings
- Validate allowed parameters strictly
- Reject unknown parameters

## Preventing Mass Assignment

- Explicit allowlists
- Reject unexpected JSON fields
- Use strict object mappers

## Preventing JSON Injection

- Proper serialization
- Escape user input
- Enforce JSON schema validation

## General API Hardening

- Disable Swagger in production
- Remove debug endpoints
- Enforce RBAC
- Avoid verbose errors

---

# 📚 OWASP Mapping

- OWASP API Top 10 – Broken Object Level Authorization
- OWASP API Top 10 – Mass Assignment
- CWE-915 – Improperly Controlled Modification of Dynamically-Determined Object Attributes
- CWE-235 – Improper Handling of Extra Parameters
- CWE-444 – Improper HTTP Parsing

---

# 🎓 Learning Outcomes

After studying this material, Testers should be able to:

- Analyze API attack surfaces
- Identify backend parsing weaknesses
- Understand framework parameter handling differences
- Recognize mass assignment risks
- Design secure API architectures

---

# 📜 License

For academic teaching and authorized lab environments only.

---

## End of API Security Academic Guide
