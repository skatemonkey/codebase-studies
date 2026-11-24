# Spring Security Integration

[All studies](../../README.md) · [Backend Engineering](../README.md) · [mall overview](README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Authentication](#1-authentication)
> - [2. Authorization](#2-authorization)

## 0. Overview

> Spring Security in mall has two main responsibilities: authentication establishes who is making a request, and authorization decides whether that identity may access the requested API.

1. [Authentication](#1-authentication)
   - `mall-admin` and `mall-portal` verify passwords at login and issue signed JWTs.
   - Later requests present the JWT so Spring Security can rebuild the current identity without a server-side session.
2. [Authorization](#2-authorization)
   - `mall-portal` uses a login-only rule for protected APIs.
   - `mall-admin` additionally matches each API path against the administrator's database-defined resources.

The implementation boundary is:

- `mall-security` provides the shared filter chain, JWT handling, password encoder, security responses, and optional dynamic authorization machinery.
- `mall-admin` and `mall-portal` provide their own `UserDetailsService` and application-specific access rules.
- `mall-demo` has a separate Spring Security configuration that permits every request.
- `mall-search` does not depend on `mall-security` and has no application security configuration.

## 1. Authentication

> Authentication proves the identity of an administrator or member. A password establishes the identity at login; a JWT re-establishes it on later requests.

### 1.1 Shared configuration

- Password storage and comparison use `BCryptPasswordEncoder`.
- HTTP sessions are disabled through `SessionCreationPolicy.STATELESS`.
- CSRF protection is disabled because protected requests authenticate with a token rather than a server-side login session.
- `JwtAuthenticationTokenFilter` runs once per request before Spring Security's username-password filter.
- Both applications read the token from `Authorization: Bearer <token>`.
- Admin and portal tokens use separate signing secrets but the same seven-day configured lifetime.

### 1.2 Login flow

1. The login controller sends the submitted username and password to its account service.
2. The application's `UserDetailsService` loads the account.
   - Admin authentication loads an `UmsAdmin` and its authorized resource list into `AdminUserDetails`.
   - Portal authentication loads an `UmsMember` into `MemberDetails`.
   - These account reads use the [Redis-backed caches](01-redis-integration.md), with MySQL as the source on a cache miss.
3. `BCryptPasswordEncoder` compares the submitted password with the stored password hash.
4. The service creates a `UsernamePasswordAuthenticationToken` and places it in `SecurityContextHolder` for the current request.
5. `JwtTokenUtil` returns a signed token containing the username, creation time, and expiration time.

| Application | Login endpoint | Identity implementation | Additional login behavior |
| --- | --- | --- | --- |
| `mall-admin` | `POST /admin/login` | `AdminUserDetails` | Rejects disabled accounts and records a login log. |
| `mall-portal` | `POST /sso/login` | `MemberDetails` | Does not explicitly check `isEnabled()` before issuing the token; later request validation does. |

### 1.3 Authenticated-request flow

1. `JwtAuthenticationTokenFilter` reads the configured HTTP header and removes the `Bearer ` prefix.
2. `JwtTokenUtil` verifies the signature and extracts the username.
3. The filter reloads the current `UserDetails` through the application's account service.
4. Token validation checks that the username still matches, the expiration time has not passed, and the account is enabled.
5. The filter places the identity and its authorities in `SecurityContextHolder` for the rest of the request.

The JWT is therefore not the complete current user record. It identifies the username, while each request reloads the account and, for administrators, the current resource authorities.

### 1.4 Token lifecycle

- A non-expired token can be refreshed. Refreshing within 30 minutes of its creation returns the same token; refreshing later issues one with a new creation and expiration time.
- Redis does not store authenticated sessions or a token blacklist.
- Admin logout clears the cached administrator and resource list. It does not revoke the signed JWT, so the client must discard the token; a still-valid token can authenticate again if the account remains enabled.
- `mall-portal` does not define a logout endpoint.

## 2. Authorization

> Authorization evaluates the authenticated identity against the access rule for the requested path.

### 2.1 Shared request boundary

The shared filter chain first permits configured public paths and all CORS preflight `OPTIONS` requests. It then applies the application's rule to every remaining request.

| Application | Public business paths at the filter layer | Rule for remaining requests |
| --- | --- | --- |
| `mall-admin` | `/admin/login`, `/admin/register`, `/admin/info`, `/admin/logout`, `/minio/upload`, and `/aliyun/oss/policy` | The administrator must hold a resource authority matching the requested path. |
| `mall-portal` | `/sso/**`, `/home/**`, `/product/**`, `/brand/**`, and `/alipay/**` | The member must be authenticated. |

Both applications also permit configured documentation, static-resource, monitoring, and database-monitoring paths.

The portal's entire `/sso/**` group is public at the filter layer. Endpoints in that group that need an identity, such as `/sso/info`, inspect the `Principal` inside the controller.

### 2.2 Portal authorization

`mall-portal` does not provide a `DynamicSecurityService`, so the shared configuration does not create the dynamic authorization manager. Every non-public request therefore uses `AuthenticatedAuthorizationManager.authenticated()`.

The result is one authorization level: a protected portal API is available to any authenticated, enabled member. `MemberDetails` exposes a placeholder `TEST` authority, but the configured portal access rule does not use it.

### 2.3 Admin authorization

`mall-admin` adds resource-level authorization on top of authentication:

1. `DynamicSecurityService` loads all `UmsResource` rows and builds an in-memory mapping from each Ant-style URL pattern to an authority named `<resourceId>:<resourceName>`.
2. `AdminUserDetails` converts the current administrator's resource list into authorities using the same naming rule.
3. `DynamicSecurityMetadataSource` finds every configured URL pattern that matches the request path.
4. `DynamicAuthorizationManager` permits the request when at least one required authority is present in the administrator's authorities.
5. A protected path with no matching resource configuration is denied because it produces no authority match.

The resource-to-path map is loaded at startup. Creating, updating, or deleting an admin resource clears it so the next authorization check reloads the current rows.

### 2.4 Rejected requests

- Missing, invalid, or expired authentication is written as a `CommonResult` JSON body with application code `401`.
- Insufficient authorization is written as a `CommonResult` JSON body with application code `403`.
- The custom handlers set the JSON body but do not explicitly set the HTTP response status.
