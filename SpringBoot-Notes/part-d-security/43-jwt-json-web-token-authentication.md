[← Back to Contents](../README.md) · Part D — Security

---

# 43. JWT (JSON Web Token) Authentication

## 43.1 What is it?

**Simple words:**
A JWT is a **self-contained, digitally signed token** the server gives you after login. You send it with every request, and the server can verify it **without looking anything up** — the token itself carries who you are and what you can do, plus a signature proving it hasn't been tampered with.

**Technical wording:**
A JWT (RFC 7519) is a compact, URL-safe means of representing claims between two parties. It comprises a Base64URL-encoded header, payload and signature, enabling stateless authentication through cryptographic verification.

## 43.2 Structure

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJyYWoiLCJyb2xlcyI6WyJVU0VSIl19.SflKxwRJ...
└────── HEADER ─────┘ └────────── PAYLOAD ──────────────┘ └─ SIGNATURE ─┘
```

```json
// HEADER — algorithm and token type
{ "alg": "HS256", "typ": "JWT" }

// PAYLOAD — the claims
{
  "sub": "raj@company.com",      // subject (who)
  "roles": ["USER", "ADMIN"],    // custom claim
  "iat": 1705315800,             // issued at
  "exp": 1705319400,             // expiry  ← the most important claim
  "iss": "order-service",        // issuer
  "jti": "a1b2c3"                // unique token ID (for revocation lists)
}

// SIGNATURE
HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), SECRET_KEY )
```

### ★ THE MOST IMPORTANT SECURITY FACT ★

**The payload is Base64-ENCODED, not encrypted.** Anyone can decode it — paste any JWT into jwt.io and read every claim.

```
Base64 is ENCODING (reversible, no key) — NOT encryption.
```

**Therefore: NEVER put sensitive data in a JWT payload.** No passwords, no card numbers, no personal identifiers beyond what's necessary. The signature guarantees **integrity** (it hasn't been modified), not **confidentiality**.

**This is the number one JWT interview question, and a real production mistake.** If you need confidentiality, use JWE (encrypted JWT) or keep the data server-side.

## 43.3 Why JWT for microservices

```
SESSION-BASED                          JWT-BASED
─────────────                          ─────────
Client sends session cookie            Client sends JWT in Authorization header
       │                                       │
       ▼                                       ▼
Server looks up the session            Server VERIFIES THE SIGNATURE
  in memory or Redis                     using its secret key
       │                                       │
   NEEDS SHARED STORAGE                  NO LOOKUP, NO STORAGE
   or sticky sessions                    any instance can verify
```

| | Session | JWT |
|---|---|---|
| Server state | **Stored server-side** | **None** |
| Scaling | Needs shared store / sticky sessions | **Any instance handles any request** |
| Revocation | **Easy — delete the session** | **Hard — valid until expiry** |
| Size | Small cookie | Larger (sent on every request) |
| Cross-domain / mobile | Awkward | **Natural** |
| Best for | Traditional server-rendered apps | **Microservices, SPAs, mobile** |

**The honest trade-off to state to a TL:** *"JWT buys us statelessness and easy horizontal scaling, but we give up instant revocation. A stolen token stays valid until it expires — which is exactly why access tokens must be short-lived."*

## 43.4 The complete JWT flow

```
 ═══════════════ LOGIN (once) ═══════════════
 1. POST /api/v1/auth/login  { username, password }
              │
 2. AuthenticationManager → UserDetailsService → BCrypt verify
              │
 3. Valid → generate:
       accessToken  (15 min, contains roles)
       refreshToken (7 days, stored server-side, minimal claims)
              │
 4. Response: { accessToken, refreshToken, expiresIn }
              │
 5. Client stores them

 ═══════════ EVERY SUBSEQUENT REQUEST ═══════════
 6. GET /api/v1/orders
    Authorization: Bearer eyJhbGc...
              │
              ▼
 7. JwtAuthenticationFilter:
       a. read the Authorization header
       b. is it "Bearer ..."?          no → continue unauthenticated
       c. verify the SIGNATURE         invalid → 401
       d. check expiry (exp)           expired → 401
       e. extract the username
       f. load UserDetails (or trust claims)
       g. build an Authentication and put it in SecurityContextHolder
              │
              ▼
 8. AuthorizationFilter checks roles → 403 if insufficient
              │
              ▼
 9. Controller executes

 ═══════════════ TOKEN REFRESH ═══════════════
 10. Access token expires (15 min)
 11. POST /api/v1/auth/refresh { refreshToken }
 12. Server validates it AGAINST THE DATABASE (it is stateful by design)
 13. Issues a new access token (and rotates the refresh token)
```

## 43.5 Implementation

### Dependency and configuration

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>
<!-- jjwt-impl and jjwt-jackson at runtime scope -->
```

```yaml
app:
  jwt:
    secret: ${JWT_SECRET}                  # from environment — NEVER in the repo
    access-token-expiration-ms: 900000     # 15 minutes
    refresh-token-expiration-ms: 604800000 # 7 days
    issuer: order-service
```

### The token service

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class JwtService {

    private final JwtProperties jwtProperties;

    private SecretKey getSigningKey() {
        // The secret MUST be at least 256 bits for HS256, or JJWT rejects it
        return Keys.hmacShaKeyFor(jwtProperties.getSecret().getBytes(StandardCharsets.UTF_8));
    }

    public String generateAccessToken(UserDetails userDetails) {
        return Jwts.builder()
                .subject(userDetails.getUsername())
                .claim("roles", userDetails.getAuthorities().stream()
                        .map(GrantedAuthority::getAuthority).toList())
                .issuer(jwtProperties.getIssuer())
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis()
                        + jwtProperties.getAccessTokenExpirationMs()))
                .id(UUID.randomUUID().toString())          // jti, for blacklisting
                .signWith(getSigningKey())
                .compact();
    }

    public String extractUsername(String token) {
        return extractAllClaims(token).getSubject();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        try {
            Claims claims = extractAllClaims(token);       // throws if signature invalid
            return claims.getSubject().equals(userDetails.getUsername())
                    && claims.getExpiration().after(new Date());
        } catch (JwtException | IllegalArgumentException e) {
            log.warn("Invalid JWT: {}", e.getMessage());   // do NOT log the token itself
            return false;
        }
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parser()
                .verifyWith(getSigningKey())     // ★ signature verification happens HERE
                .requireIssuer(jwtProperties.getIssuer())
                .build()
                .parseSignedClaims(token)        // ★ parseSigned — NEVER parse unsigned
                .getPayload();
    }
}
```

**Two security-critical lines highlighted above:**
- `verifyWith(...)` + `parseSignedClaims(...)` — this is what actually validates the signature. Using an unsigned parse, or decoding the payload manually, means **an attacker can forge any claims they like**.
- Never log the token. It is a credential; a token in your log aggregator is a leaked credential.

### The authentication filter

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        final String authHeader = request.getHeader(HttpHeaders.AUTHORIZATION);

        // No token → continue unauthenticated. Let the authorization filter decide.
        // Do NOT reject here — public endpoints must still work.
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        try {
            final String jwt = authHeader.substring(7);
            final String username = jwtService.extractUsername(jwt);

            // Only authenticate if not already authenticated on this request
            if (username != null
                    && SecurityContextHolder.getContext().getAuthentication() == null) {

                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                if (jwtService.isTokenValid(jwt, userDetails)) {
                    UsernamePasswordAuthenticationToken authToken =
                            new UsernamePasswordAuthenticationToken(
                                    userDetails, null, userDetails.getAuthorities());
                    authToken.setDetails(
                            new WebAuthenticationDetailsSource().buildDetails(request));

                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (Exception e) {
            log.warn("JWT authentication failed: {}", e.getMessage());
            SecurityContextHolder.clearContext();
            // Fall through — ExceptionTranslationFilter will produce a proper 401
        }

        filterChain.doFilter(request, response);
    }
}
```

**Design note worth explaining:** the filter does **not** reject requests without a token. It simply doesn't authenticate them. Rejection is the `AuthorizationFilter`'s job, using the URL rules — which is what keeps `/api/v1/auth/login` and other public endpoints working.

### Login endpoint

```java
@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;
    private final RefreshTokenService refreshTokenService;

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@Valid @RequestBody LoginRequest request) {

        // Throws BadCredentialsException → 401 via the entry point
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                        request.getUsername(), request.getPassword()));

        UserDetails userDetails = (UserDetails) authentication.getPrincipal();

        return ResponseEntity.ok(AuthResponse.builder()
                .accessToken(jwtService.generateAccessToken(userDetails))
                .refreshToken(refreshTokenService.create(userDetails.getUsername()).getToken())
                .tokenType("Bearer")
                .expiresIn(900)
                .build());
    }
}
```

## 43.6 The revocation problem and refresh tokens

**The core weakness:** a JWT is valid until it expires. If a token is stolen — or a user is deactivated, or their role is revoked — **the server cannot invalidate it**. There's nothing to delete.

### Mitigations

| Strategy | How | Trade-off |
|---|---|---|
| **Short expiry** | 15-minute access tokens | Small exposure window. **The primary defence.** |
| **Refresh tokens** | Long-lived, stored in the DB, revocable | Reintroduces some state — deliberately |
| **Blacklist** | Store revoked `jti` values in Redis until expiry | A lookup per request — partly gives up statelessness |
| **Token version** | A counter on the user row; invalidate all older tokens | One lookup per request |

**The standard production pattern:**

```
ACCESS TOKEN                          REFRESH TOKEN
────────────                          ─────────────
Short: 15 minutes                     Long: 7 days
Stateless — signature only            STORED IN THE DATABASE
Carries roles                         Minimal claims
Sent with every request               Sent only to /auth/refresh
Cannot be revoked                     CAN be revoked (delete the row)
```

**Why this design is defensible:** the access token stays stateless and fast for the 99% of requests. The refresh token is deliberately stateful, so logout and revocation actually work — we accept one database lookup every 15 minutes rather than on every request.

**Refresh token rotation:** issue a new refresh token on each use and invalidate the old one. If an old refresh token is ever reused, that signals theft — revoke the entire family and force re-login. This is the current best practice.

## 43.7 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Sensitive data in the payload** | Anyone can decode it — it is not encrypted |
| **Weak or committed secret** | Attacker forges tokens = full compromise |
| **No expiry (`exp`)** | Token valid forever |
| Very long access-token expiry | Huge window for a stolen token |
| **Not verifying the signature** | Forged tokens accepted |
| Accepting `alg: none` | Classic JWT vulnerability — signature skipped entirely |
| Logging the token | Credential leaked into log aggregation |
| No refresh mechanism | Users forced to log in constantly, or expiry set dangerously long |
| Expecting instant revocation | Doesn't exist without extra state |
| Token in a URL query parameter | Recorded in access logs and browser history |

**The `alg: none` attack, explained:** the JWT spec permits an unsecured token with `"alg": "none"` and an empty signature. If your library is configured to accept the algorithm declared *in the token*, an attacker rewrites the header to `none`, edits the payload to `"roles": ["ADMIN"]`, and sends it with no signature. **Always pin the expected algorithm server-side** — which `verifyWith(key)` + `parseSignedClaims()` does.

## 43.8 TL Questions

**Q: What is a JWT and why do we use it?**
A: A signed, self-contained token carrying the user's identity and roles. Because the server verifies it by checking the signature rather than looking it up, any instance can authenticate any request — which is what lets us scale horizontally without sticky sessions or a shared session store.

**Q: Is the token encrypted?**
A: No. The payload is Base64-encoded and anyone can decode and read it. The signature guarantees it hasn't been tampered with, not that it's secret. So we never put anything sensitive in the claims.

**Q: What happens if someone steals a token?**
A: They can use it until it expires, and we can't invalidate it — that's the fundamental trade-off of statelessness. We mitigate it with a fifteen-minute access-token lifetime, and a longer refresh token that *is* stored in the database and can be revoked.

**Q: How does logout work?**
A: We delete the refresh token server-side, so no new access tokens can be issued. The current access token stays valid for up to fifteen more minutes. If we needed immediate revocation we'd add a Redis blacklist keyed on the token ID, accepting a lookup per request.

**Q: Where does the client store the token?**
A: There's no risk-free option. An `httpOnly`, `Secure`, `SameSite` cookie is the most defensible, because JavaScript can't read it — but then we must re-enable CSRF protection. `localStorage` is simpler but readable by any injected script, so an XSS becomes full account takeover.

**Q: What's the biggest implementation risk?**
A: Not verifying the signature properly, or accepting the algorithm declared inside the token. That allows the `alg: none` attack, where an attacker rewrites the header, edits the claims to give themselves admin, and sends it unsigned. We pin the algorithm and key server-side.

**Q: Why not just use sessions?**
A: Sessions need shared state — sticky sessions or a Redis session store — which adds infrastructure and a failure point. For a microservice architecture where several services must independently validate the same identity, a signed token is simpler. For a single server-rendered app, sessions would be a reasonable choice.

## 43.9 Interview Questions

**Beginner — What are the three parts of a JWT?**
Header, payload and signature, Base64URL-encoded and separated by dots.

**Beginner — Where is the token sent?**
In the `Authorization` header as `Bearer <token>`.

**Intermediate — Is a JWT encrypted?**
No — encoded only. Use JWE if confidentiality is required.

**Intermediate — Why use both an access and a refresh token?**
The access token is short-lived and stateless so most requests need no lookup; the refresh token is long-lived, stored server-side and revocable, so logout and revocation actually work. It balances performance against control.

**Advanced — How can a JWT be revoked?**
Not directly. Options are short expiry (the main defence), a Redis blacklist of `jti` values kept until natural expiry, or a token-version counter on the user record — both of which trade some statelessness for control.

**Advanced — What is the `alg: none` vulnerability?**
The spec allows an unsecured JWT with no signature. If the server trusts the algorithm declared in the token header, an attacker sets it to `none`, modifies the claims freely and omits the signature. Mitigation: always pin the expected algorithm and key server-side.

**Advanced — HS256 vs RS256?**
HS256 is symmetric — the same secret signs and verifies, so every verifying service needs the signing secret and could forge tokens. RS256 is asymmetric — a private key signs and a public key verifies, so downstream services can validate without any ability to issue tokens. **RS256 is the correct choice in a multi-service architecture.**

## 43.10 Quick Revision

1. JWT = header + payload + signature, Base64URL-encoded, dot-separated.
2. **Payload is encoded, NOT encrypted** — never put secrets in it.
3. Signature gives **integrity**, not confidentiality.
4. Stateless → any instance validates → horizontal scaling.
5. **Cannot be revoked** — mitigate with short expiry.
6. Access token ~15 min (stateless); refresh token ~7 days (stored, revocable).
7. Rotate refresh tokens; reuse of an old one signals theft.
8. Always `verifyWith(key)` + `parseSignedClaims` — never trust the token's `alg`.
9. Keep the secret in the environment, at least 256 bits for HS256.
10. **RS256 for multi-service** — verifiers can't forge.
11. Send in the `Authorization` header, never a query parameter.
12. Never log tokens.

## 43.11 TL Explanation (speak this)

> "We use JWT because it's stateless — the token carries the user's identity and roles and is signed, so any service instance can verify it by checking the signature without a session store. The critical thing everyone should know is that the payload is only Base64-encoded, not encrypted, so anyone can read the claims; the signature proves it wasn't tampered with, nothing more. The trade-off we accept is that a token can't be revoked before it expires, which is why access tokens live fifteen minutes and we pair them with a longer refresh token that *is* stored in the database and can be deleted on logout. And we pin the signing algorithm server-side rather than trusting the token header, which is what prevents the `alg: none` forgery attack."

---
---

# PART E — BUILD, CONFIGURATION AND DEPLOYMENT

---

[← 42. Spring Security](42-spring-security.md) | [Contents](../README.md) | [44. Maven →](../part-e-build-deployment/44-maven.md)
