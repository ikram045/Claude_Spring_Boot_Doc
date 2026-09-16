[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

---

# 50. Complete Spring Security / JWT Flow

```
 ══════════════════ PHASE 1: LOGIN (once) ══════════════════

 POST /api/v1/auth/login  { "username": "raj", "password": "secret" }
        │
        ▼
 AuthController → authenticationManager.authenticate(token)
        │
        ▼
 ProviderManager → DaoAuthenticationProvider
        │
        ├─► userDetailsService.loadUserByUsername("raj")
        │       SELECT u.*, r.* FROM users u JOIN roles r ...
        │       → UserDetails(username, BCRYPT HASH, authorities)
        │
        └─► passwordEncoder.matches(raw, storedHash)
                BCrypt re-hashes the input with the stored salt
                and compares. The hash is NEVER decrypted.
        │
   ┌────┴─────┐
 match      no match
   │            │
   ▼            ▼
 Authenticated   BadCredentialsException
 Authentication      → AuthenticationEntryPoint → 401
   │                  (generic message — no user enumeration)
   ▼
 jwtService.generateAccessToken(userDetails)
     header  { alg: HS256, typ: JWT }
     payload { sub, roles, iat, exp(+15m), iss, jti }
     signature = HMACSHA256(header.payload, SECRET)
   │
   ▼
 refreshTokenService.create()  → row saved in DB (revocable)
   │
   ▼
 200 { accessToken, refreshToken, tokenType: "Bearer", expiresIn: 900 }


 ═════════ PHASE 2: EVERY SUBSEQUENT REQUEST ═════════

 GET /api/v1/orders    Authorization: Bearer eyJhbGc...
        │
        ▼
 JwtAuthenticationFilter (OncePerRequestFilter)
        │
        ├─ no "Bearer " header → continue UNAUTHENTICATED
        │     (public endpoints must still work)
        │
        └─ token present:
              1. verifyWith(key).parseSignedClaims(token)
                   ★ signature verified with the PINNED algorithm ★
                   invalid/forged → exception → 401
              2. check exp → expired → 401
              3. extract subject
              4. load UserDetails
              5. build UsernamePasswordAuthenticationToken
              6. SecurityContextHolder.getContext()
                          .setAuthentication(auth)   ← ThreadLocal
        │
        ▼
 AuthorizationFilter
        - URL rules (hasRole/permitAll/anyRequest().authenticated())
        - not permitted → AccessDeniedException
              → AccessDeniedHandler → 403
        │
        ▼
 DispatcherServlet → Controller
        │
        ▼
 Service  @PreAuthorize("#customerId == authentication.name or hasRole('ADMIN')")
        - fine-grained, SpEL over method arguments
        - fails → AccessDeniedException → 403
        │
        ▼
 Business logic → response


 ═══════════════ PHASE 3: TOKEN REFRESH ═══════════════

 Access token expires after 15 minutes
        │
        ▼
 POST /api/v1/auth/refresh  { refreshToken }
        │
        ▼
 Look up the refresh token IN THE DATABASE   ← deliberately STATEFUL
        - not found / revoked / expired → 401, force re-login
        - found → issue a NEW access token
                → ROTATE the refresh token (invalidate the old one)
        │
        ▼
 Reuse of an already-rotated refresh token
        → signals theft → revoke the whole family → force re-login


 ═══════════════ LOGOUT ═══════════════
 Delete the refresh token row.
 ⚠ The current access token stays valid for up to 15 more minutes.
   Immediate revocation requires a jti blacklist in Redis — which
   trades away some statelessness. That is a deliberate decision.
```

**The two sentences that show you understand JWT:**
> "The payload is Base64-encoded, not encrypted — anyone can read the claims, so nothing sensitive goes in them; the signature provides integrity, not confidentiality. And a JWT can't be revoked before it expires, which is why the access token is short-lived and paired with a database-backed refresh token that *can* be revoked."

---

[← 49. Complete JPA/Hibernate Flow](49-complete-jpa-hibernate-flow.md) | [Contents](../README.md) | [51. Complete Exception Handling Flow →](51-complete-exception-handling-flow.md)
