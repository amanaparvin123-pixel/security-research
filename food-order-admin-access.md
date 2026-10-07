# Unauthenticated Administrative Access in Food Order Management System

## Summary

A broken access control vulnerability was identified in the Food Order Management System. Multiple administrative endpoints do not enforce server-side authentication or authorization checks, allowing unauthenticated users to directly access administrative functionality.

Affected endpoints include:

- `admin/add_admin.php`
- `admin/add_category.php`
- `admin/add_food.php`
- `admin/update_category.php`
- `admin/update_food.php`
- `admin/delete.php`
- `admin/delete_category.php`
- `admin/change_pwd.php`
- `admin/edit_admin.php`

The application's primary `admin.php` page attempts to verify the authenticated session, but the redirect is performed without terminating script execution. More importantly, the individual administrative endpoints listed above do not consistently perform authentication checks themselves.

## Impact

An unauthenticated attacker may be able to directly invoke administrative functionality, potentially allowing them to:

- Create additional administrator accounts
- Modify food and category information
- Change prices or other application content
- Modify administrator accounts
- Delete application data or images
- Perform other administrative operations exposed by the affected endpoints

The practical impact depends on the permissions and functionality implemented by each endpoint.

## Affected Component

Food Order Management System administrative interface.

## Vulnerability Type

Broken Access Control / Missing Authorization

## CWE

CWE-862 — Missing Authorization

## Technical Details

Administrative functionality relies on session-based authentication, but several administrative PHP endpoints can be requested directly without first verifying that the requesting user has an authenticated administrative session.

For example, endpoints such as `add_admin.php`, `add_food.php`, `change_pwd.php`, and `delete.php` process requests without a consistent server-side authorization check.

The application therefore assumes that users will reach these endpoints only through the authenticated administration interface. An attacker can bypass this assumption by directly requesting the affected endpoint.

Additionally, the authentication check in `admin.php` sends a redirect response when the session is missing but does not terminate execution with `exit`, allowing subsequent server-side processing to continue.

## Reproduction

1. Start the affected Food Order Management System.
2. Ensure that no authenticated administrator session exists.
3. Send a direct request to an administrative endpoint such as:

   `admin/add_admin.php`

4. Observe that the endpoint is reachable without first establishing an authenticated administrative session.
5. Similar behavior can be tested against the other affected administrative endpoints listed above.

The exact impact depends on the parameters and functionality exposed by each endpoint.

## Expected Behavior

Every administrative endpoint should independently verify that the requester has a valid authenticated administrative session and sufficient authorization before processing the request.

## Observed Behavior

Multiple administrative endpoints process requests without enforcing authentication/authorization.

## Recommended Remediation

Implement a centralized server-side authorization mechanism and require it at the beginning of every administrative endpoint.

For example:

- Require an authenticated administrative session before processing requests.
- Terminate execution immediately after an unauthorized redirect.
- Enforce authorization on the server rather than relying on the visibility of links or the administration interface.
- Apply the same authorization mechanism consistently to every administrative endpoint.
- Use deny-by-default authorization for administrative functionality.
- Regenerate the session identifier after successful authentication.

## Suggested Fix

A shared authentication function can be used by all administrative endpoints. Unauthorized requests should be terminated immediately rather than merely sending a redirect response.

Example logic:

```php
session_start();

if (!isset($_SESSION['user'])) {
    header("Location: login.php");
    exit;
}
```

The actual authorization implementation should be adapted to the application's authentication architecture.

## Disclosure Status

This advisory documents a security issue identified during authorized security testing.

## Researcher

[V M Fathimath Amana Parvin]


## Product

Food Order Management System
