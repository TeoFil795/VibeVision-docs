# Security Principles - WodVision

**Source**: Basato su [claude-code-workflows](https://github.com/OneRedOak/claude-code-workflows) di Anthropic Security Team

---

## 🛡️ Filosofia: Security by Design

**"Security is built into the development process from the start, not added as an afterthought"**

La sicurezza non è un checkbox, è una mentalità che permea ogni decisione di sviluppo.

---

## 🎯 Core Security Principles

### 1. Proactive Vulnerability Detection
**Cattura problemi PRIMA che vadano in produzione**

- Security review automatici su ogni PR
- Scan secrets prima di ogni commit
- Dependency security checks continui

### 2. Industry-Standard Frameworks
**Non inventare la ruota**

- **OWASP Top 10** come baseline
- Best practices consolidate dall'industria
- Standards verificati da security community

### 3. Defense in Depth
**Mai fidarsi di un singolo layer di protezione**

```
User Input → Validation → Sanitization → Parameterized Query → Encrypted Storage
           ↓            ↓               ↓                   ↓
        Layer 1      Layer 2         Layer 3           Layer 4
```

### 4. Least Privilege Principle
**Dai accesso SOLO a quello che serve**

- Database user con permessi minimi
- API keys con scope limitato
- Token con expiration

---

## 🚨 OWASP Top 10 - Checklist WodVision

### A01:2021 – Broken Access Control
- [ ] Ogni endpoint API verifica autenticazione
- [ ] User può accedere solo ai SUOI dati
- [ ] Admin routes protetti con role check

```dart
// ✅ GOOD
Future<Journey> getJourneyById(int id, String userId) async {
  final journey = await db.journeys.findById(id);
  if (journey.userId != userId) {
    throw UnauthorizedException('Cannot access other user\'s journey');
  }
  return journey;
}
```

### A02:2021 – Cryptographic Failures
- [x] Token salvati con flutter_secure_storage (v1.0.7)
- [x] HTTPS enforced, no cleartext traffic (v1.0.7)
- [x] Passwords hashate con bcrypt (Laravel)
- [ ] Sensitive data mai loggati

```dart
// ❌ BAD
debugPrint('User password: $password');

// ✅ GOOD
debugPrint('Login attempt for user: $email');
```

### A03:2021 – Injection
- [x] Query SQL sempre parametrizzate (Eloquent ORM)
- [ ] Input validation su TUTTI gli endpoint
- [ ] Sanitizzazione output HTML

```php
// ❌ BAD - SQL Injection vulnerability
$user = DB::select("SELECT * FROM users WHERE email = '$email'");

// ✅ GOOD - Parameterized query
$user = DB::table('users')->where('email', $email)->first();
```

### A04:2021 – Insecure Design
- [ ] Security requirements definiti PRIMA dello sviluppo
- [ ] Threat modeling per feature critiche
- [ ] Security review prima del merge

### A05:2021 – Security Misconfiguration
- [x] `.env` mai committato (.gitignore)
- [x] API keys in Secret Manager (v1.0.7)
- [ ] APP_DEBUG=false in produzione (Laravel)
- [ ] CORS configurato correttamente
- [x] Security headers configurati (AndroidManifest)

### A06:2021 – Vulnerable and Outdated Components
- [ ] Dipendenze aggiornate regolarmente
- [ ] Security patches applicate tempestivamente
- [ ] Monitor advisories per librerie critiche

```bash
# Check vulnerabilities regolarmente
flutter pub outdated
npm audit
composer audit
```

### A07:2021 – Identification and Authentication Failures
- [x] Token Sanctum con expiration
- [ ] Rate limiting su login (60 req/min)
- [ ] OTP per verifica email
- [ ] No password in logs

### A08:2021 – Software and Data Integrity Failures
- [ ] Code signing per release
- [ ] Webhook signature verification (Stripe)
- [ ] File upload validation (MIME type, size)

```dart
// ✅ GOOD - File validation
if (!allowedMimeTypes.contains(file.mimeType)) {
  throw SecurityException('Invalid file type');
}
if (file.lengthSync() > maxSize) {
  throw SecurityException('File too large');
}
```

### A09:2021 – Security Logging and Monitoring Failures
- [ ] Log failed login attempts
- [ ] Alert su anomalie (multiple failed payments)
- [ ] Log SENZA dati sensibili

```dart
// ❌ BAD
logger.error('Payment failed for card ${cardNumber}');

// ✅ GOOD
logger.error('Payment failed for user ${userId}', {
  'last4': cardNumber.substring(cardNumber.length - 4)
});
```

### A10:2021 – Server-Side Request Forgery (SSRF)
- [ ] Validate URL input
- [ ] Whitelist domains per webhook
- [ ] No user-controlled redirects

---

## 🔐 Security Review Criteria

### Severity Levels

**🔴 CRITICAL** - Fix IMMEDIATELY
- RCE (Remote Code Execution)
- Authentication bypass
- Direct data breach
- Hardcoded production secrets

**🟠 HIGH** - Fix before next release
- SQL injection
- XSS vulnerabilities
- Insecure deserialization
- Weak cryptography

**🟡 MEDIUM** - Fix soon
- Missing rate limiting
- Weak session management
- Information disclosure
- CSRF vulnerabilities

**🟢 LOW** - Defense in depth
- Missing security headers
- Verbose error messages
- Insecure cookies flags

### Confidence Threshold

Flag SOLO vulnerabilità con confidence > 80%
- 🎯 **High confidence (0.8-1.0)**: Clear vulnerability pattern
- ⚠️ **Medium confidence (0.5-0.8)**: Investigate further
- ❓ **Low confidence (<0.5)**: Likely false positive

---

## 🚫 Common Vulnerability Patterns

### 1. Input Validation Vulnerabilities

```dart
// ❌ BAD - No validation
await apiService.post('/register', {'email': userInput});

// ✅ GOOD - Validation
if (!EmailValidator.validate(email)) {
  throw ValidationException('Invalid email format');
}
await apiService.post('/register', {'email': email});
```

### 2. Authentication Bypass

```php
// ❌ BAD - Trusting client-side check
if (isset($_POST['isAdmin'])) {
    // Grant admin access
}

// ✅ GOOD - Server-side verification
if (Auth::user()->role === 'admin') {
    // Grant admin access
}
```

### 3. Hardcoded Secrets

```dart
// ❌ BAD - Secret in code
const String apiKey = 'AIzaSyC7Qxjl...';

// ✅ GOOD - From environment/secure storage
final apiKey = await secureStorage.read(key: 'api_key');
```

### 4. Improper Error Handling

```php
// ❌ BAD - Exposes stack trace
catch (Exception $e) {
    return response()->json(['error' => $e->getMessage()], 500);
}

// ✅ GOOD - Generic message
catch (Exception $e) {
    Log::error('Error in payment: ' . $e->getMessage());
    return response()->json(['error' => 'Payment failed'], 500);
}
```

---

## 🛠️ Security Tools & Practices

### Pre-Commit Checks
```bash
# Git hooks per security
- Check for hardcoded secrets (gitleaks, trufflehog)
- Scan for common vulnerabilities
- Validate .gitignore
```

### CI/CD Security
```yaml
# GitHub Actions security checks
- Dependency vulnerability scan
- SAST (Static Application Security Testing)
- Secret scanning
- License compliance
```

### Monitoring & Alerting
```
- Failed authentication attempts
- Unusual API usage patterns
- Large data exports
- Admin action logs
```

---

## 📋 Security Checklist per Feature

Quando implementi una nuova feature, verifica:

### Input Handling
- [ ] Input validation (tipo, lunghezza, formato)
- [ ] Sanitizzazione per output
- [ ] Rate limiting se esposta pubblicamente

### Authentication & Authorization
- [ ] Endpoint protetto con auth middleware
- [ ] User può accedere solo ai suoi dati
- [ ] Roles/permissions verificati

### Data Storage
- [ ] Dati sensibili mai in plain text
- [ ] Encryption at rest per PII
- [ ] Backup criptati

### Communication
- [ ] HTTPS only
- [ ] Secure headers (HSTS, CSP)
- [ ] Certificate pinning (produzione)

### Error Handling
- [ ] Nessun stack trace esposto
- [ ] Log senza dati sensibili
- [ ] Generic error messages al client

### Dependencies
- [ ] Librerie da fonti affidabili
- [ ] Vulnerabilità note verificate
- [ ] Versioni aggiornate

---

## 🎯 WodVision Specific Security

### Critical Assets
1. **User authentication tokens** - Compromissione = account takeover
2. **Video content** - Privacy utente
3. **Payment information** - PCI compliance
4. **Gemini API key** - Costi non autorizzati

### Attack Vectors da Monitorare
- **API abuse** - Rate limiting essenziale
- **Video upload abuse** - File size/type validation
- **Payment fraud** - Stripe Radar + custom logic
- **Account takeover** - MFA consigliato per Pro users

### Security Priorities
1. ✅ Token storage (FIXED v1.0.7)
2. ✅ API timeout (FIXED v1.0.7)
3. ✅ Secret management (FIXED v1.0.7)
4. 🔄 Rate limiting (IN PROGRESS)
5. ⏳ MFA (FUTURE)

---

## 🚨 Incident Response Plan

### Se trovi una vulnerabilità:

1. **Assess Severity**
   - Critical? Fix IMMEDIATELY, deploy hotfix
   - High? Fix in <24h
   - Medium/Low? Schedule for next sprint

2. **Document**
   - Come è stata trovata
   - Impatto potenziale
   - Fix applicato
   - Lesson learned

3. **Communicate**
   - Team notificato
   - Users notificati se dati compromessi (GDPR)
   - Post-mortem documentato

---

## 📚 Security Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Mobile Security](https://owasp.org/www-project-mobile-top-10/)
- [Flutter Security Best Practices](https://docs.flutter.dev/security)
- [Laravel Security Best Practices](https://laravel.com/docs/security)
- [Google Cloud Security](https://cloud.google.com/security/best-practices)

---

**Remember**: "Security is not a product, but a process." - Bruce Schneier

*Ultimo aggiornamento: 15 Gennaio 2026 (v1.0.7)*
