The fundamental difference between your past flow and the current flow is the transition from Data Authentication (JWS) to Data Confidentiality (JWE).
Here is how the lifecycle of your token has changed across the three files:
1. Token Issuance (The Login Flow)
 * Past Flow: When a user logged in, HmacJwtUtil.java took the user's details (email, phone, branch, etc.), converted them into plain Base64 text, and attached an HMAC signature using signWith(). If an attacker intercepted the token and pasted it into a tool like jwt.io, they could easily read all the personal data.
 * Current Flow: HmacJwtUtil.java now uses an AES symmetric key to actively encrypt the payload using encryptWith(key, Jwts.KEY.DIR, Jwts.ENC.A256GCM). The user details are transformed into unreadable ciphertext. If someone inspects the token now, the payload is completely hidden.
2. Token Validation (The API Request Flow)
 * Past Flow: When a user made an API request (like fetching a profile), Spring Security passed the token to the NimbusReactiveJwtDecoder inside JwtDecoderConfig.java. Nimbus simply looked at the signature and verified it using the HMAC secret.
 * Current Flow: Nimbus has been completely removed. Now, when a request arrives, Spring Security passes the token to the custom JJWT jwtDecoder() bean. This decoder uses the AES key to decrypt the ciphertext (decryptWith(key)), parses the hidden claims (parseEncryptedClaims(token)), and maps them into a Spring Security context so your application can securely authorize the user.
3. Internal Token Extraction (The Logout & Refresh Flow)
 * Past Flow: During logout, AuthController.java needed to read the token to find the jti (JWT ID) so it could blacklist the session in Redis. The extractClaims method used Jwts.parser().verifyWith() to read the plain Base64 claims.
 * Current Flow: Because the token is now fully encrypted, reading it requires a decryption step. The extractClaims method in AuthController.java was updated to decryptWith() and parseEncryptedClaims(). It now unlocks the token, extracts the jti, and successfully blacklists it in Redis.
