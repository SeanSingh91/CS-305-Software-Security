# CS-305-Software-Security
**Develpor** Sean Singh 
**Date** June 2026 
|---------------------------------------
## Project Overview
This project is a refactored Spring Boot web application built for **Artemis Financial**, a financial 
consulting firm that develops individualized savings, retirement, investment, and insurance plans for its customers. 
Artemis Financial needed to modernize its public-facing web application by adding secure communication mechanisms and a 
data verification step to ensure that data transmitted between the server and clients has not been altered in transit.

The refactoring accomplishes two primary goals:

1. **Data integrity verification** — a SHA-256 cryptographic hash (checksum) endpoint that demonstrates how Artemis Financial
2. can verify that transmitted data has not been tampered with.
3. **Secure communications** — migration of the application from plain HTTP to HTTPS using a self-signed X.509 certificate
4. backed by a 2048-bit RSA key, served over TLS on port 8443.

|----------------------------------------
## Competencies Demonstrated
- Writing secure communications through the application of current encryption technologies
- Designing and implementing code that complies with software security testing protocols

|----------------------------------------

## Technologies Used


| Technology | Version | 
|---|---|
| Java | 1.8 (runs on JDK 11+) |
| Spring Boot | 2.2.4.RELEASE |
| Apache Maven | 3.9.x |
| OWASP Dependency-Check | 10.0.2 |
| Keystore format | PKCS12 |
| Hash algorithm | SHA-256 (FIPS 180-4) |
| Certificate algorithm | SHA384withRSA, 2048-bit RSA key | 

|----------------------------------------

## Project Structure

```
ssl-server/
|- src/
│   |- main/
│   │   |- java/com/snhu/sslserver/
│   │   │   |- SslServerApplication.java   <- Main app + /hash endpoint
│   │   |- resources/
│   │       |- application.properties       <- HTTPS/SSL configuration
│   │       |- keystore.p12                 <- PKCS12 self-signed certificate
│   |- test/
│       |- java/com/snhu/sslserver/
│           |- SslServerApplicationTests.java
|- pom.xml                                  <- Maven build + OWASP plugin
|- mvnw / mvnw.cmd                          <- Maven wrapper scripts
|- README.md
```

|---------------------------------------

## Key Changes from Original Scaffold

### 1. `SslServerApplication.java`
Added a `@RestController` annotation and a new `/hash` GET endpoint that:
- Takes a fixed data string (`"Sean Singh checksum verification"`)
- Computes its SHA-256 digest using `java.security.MessageDigest`
- Returns the original data and its 64-character hexadecimal hash to the browser

**Expected output at `https://localhost:8443/hash`:**
```
Data: Sean Singh checksum verification
Hash: 01b8f58cfa72b1345236a4e4de507d7704c511dd4fe006ce42054644c5b9f9e7
```

### 2. `application.properties`
Replaced all placeholder `????` values with working SSL configuration:
```properties
server.port=8443
server.ssl.key-alias=artemis
server.ssl.key-store-password=changeit
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-type=PKCS12
```

### 3. `pom.xml`
Updated the OWASP Dependency-Check Maven plugin from the original `5.3.0` to `10.0.2` (required for compatibility 
with NIST's current NVD API 2.0), with OSS Index analyzer disabled:
```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>10.0.2</version>
    <configuration>
        <ossindexAnalyzerEnabled>false</ossindexAnalyzerEnabled>
    </configuration>
    <executions>
        <execution>
            <goals><goal>check</goal></goals>
        </execution>
    </executions>
</plugin>
```

### 4. Certificate generation
A self-signed PKCS12 certificate was generated using Java Keytool and stored in `src/main/resources/keystore.p12`:
```bash
keytool -genkeypair -alias artemis -keyalg RSA -keysize 2048 \
  -validity 90 -storetype PKCS12 -keystore keystore.p12 \
  -storepass changeit \
  -dname "CN=localhost, OU=Development, O=Artemis Financial, L=Boston, ST=MA, C=US"
```

|---------------------------------------------

## How to Run

### Prerequisites
- JDK 8 or higher (tested on JDK 21)
- Apache Maven 3.6+ (or use the included `mvnw` wrapper)

### Steps

1. Clone or download this repository
2. Open a terminal in the `ssl-server` directory
3. Run the application:
   ```bash
   mvn spring-boot:run
   ```
   or using the Maven wrapper:
   ```bash
   ./mvnw spring-boot:run
   ```
4. Watch the console for:
   ```
   Tomcat started on port(s): 8443 (https)
   Started SslServerApplication in X.XXX seconds
   ```
5. Open a browser and navigate to:
   ```
   https://localhost:8443/hash
   ```
6. Accept the self-signed certificate warning (click **Advanced → Proceed to localhost**) — this is expected behavior for a
   development self-signed certificate
7. The page will display the data string and its SHA-256 checksum

### Running the dependency-check scan
```bash
mvn verify
```
The HTML report is generated at `target/dependency-check-report.html`.

|---------------------------------------------

## Security Notes

- The `keystore.p12` password (`changeit`) is the default keytool password and is intentionally weak for development purposes.
- In a production deployment, this should be replaced with a strong, randomly generated password stored outside of source
  control (e.g., in an environment variable or secrets manager).
- The self-signed certificate is suitable for development and testing only. A production deployment would use a certificate
  issued by a trusted Certificate Authority (CA).
- The dependency-check scan identified existing CVEs in the bundled Tomcat 9.0.30 server inherited from the Spring Boot 2.2.4 parent.
  These are pre-existing platform dependencies, not introduced by this refactoring. Remediation would require upgrading
  the Spring Boot parent version.

|---------------------------------------------

## Algorithm Summary

| Component | Algorithm | Purpose |
|---|---|---|
| Checksum endpoint | SHA-256 | Data integrity verification |
| Certificate key pair | RSA-2048 | TLS handshake / asymmetric encryption |
| Certificate signature | SHA384withRSA | Certificate authenticity |
| Transport encryption | TLS (via HTTPS) | Secure communication channel |

SHA-256 was selected because it is the current NIST standard (FIPS PUB 180-4), provides a 256-bit security margin well beyond 
current computational feasibility, and is the hash algorithm used internally by the TLS cipher suites negotiated over 
HTTPS — keeping the security model consistent end to end.

|--------------------------------------------

## References

- National Institute of Standards and Technology. (2015). *Secure hash standard (SHS)* (FIPS PUB 180-4).
  https://doi.org/10.6028/NIST.FIPS.180-4
- National Institute of Standards and Technology. (2020). *Recommendation for key management: Part 1 – General* (NIST SP 800-57 Part 1, Rev. 5).
- https://doi.org/10.6028/NIST.SP.800-57pt1r5
- OWASP Foundation. (n.d.). OWASP dependency-check.
- https://owasp.org/www-project-dependency-check/
- Oracle. (n.d.). Java security standard algorithm names. https://docs.oracle.com/en/java/javase/17/docs/specs/security/standard-names.html
- VMware. (n.d.). *Spring Boot reference documentation* (Version 2.2.4).
- https://docs.spring.io/spring-boot/docs/2.2.4.RELEASE/reference/htmlsingle/

|--------------------------------------------
