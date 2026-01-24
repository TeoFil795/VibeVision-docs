# WodVision - Session Log: Security & Stability Audit

**Data**: 15 Gennaio 2026
**Versione**: 1.0.6 → 1.0.7
**Obiettivo**: Code review completo + fix critici pre-lancio

---

## Executive Summary

Oggi ho fatto un audit completo del codice di WodVision con l'aiuto di Claude (AI). Come founder non-tecnico, volevo assicurarmi che l'app fosse sicura e stabile prima di acquisire più utenti.

**Risultato**: 4 vulnerabilità critiche risolte, UX migliorata significativamente.

---

## Problemi Identificati e Risolti

### PROBLEMA 1: Loading Infinito sugli Abbonamenti

**Sintomo**: Quando un utente selezionava un piano di abbonamento, la schermata rimaneva in loading per sempre.

**Causa Root**: Le chiamate HTTP non avevano timeout. Se il server non rispondeva (o rispondeva lentamente), l'app aspettava all'infinito.

**Soluzione**:
```dart
// PRIMA (problematico)
final response = await http.post(uri, headers: headers, body: jsonEncode(body));

// DOPO (con timeout)
final response = await http.post(uri, headers: headers, body: jsonEncode(body))
    .timeout(ApiTimeout.defaultTimeout);  // 30 secondi
```

**File modificati**:
- `lib/core/api_service.dart` - Aggiunto sistema di timeout configurabile
- `lib/providers/subscription_plans_provider.dart` - Gestione errori timeout

**Impatto utente**: L'app ora mostra un messaggio chiaro dopo 30 secondi invece di bloccarsi.

---

### PROBLEMA 2: Errori Silenziosi

**Sintomo**: Quando qualcosa andava storto, l'utente non vedeva nessun messaggio. L'app sembrava semplicemente non funzionare.

**Causa Root**: I blocchi `catch` facevano solo `debugPrint()` invece di mostrare feedback all'utente.

**Soluzione**:
```dart
// PRIMA (errore silenzioso)
catch (e) {
  debugPrint('Error: $e');  // Solo per sviluppatori!
}

// DOPO (feedback utente)
catch (e) {
  Utils.showToastMessage(
    context, false, 'Error',
    'Something went wrong. Please try again.',
  );
  debugPrint('Error: $e');
}
```

**Impatto utente**: Ora riceve sempre feedback quando qualcosa va storto.

---

### PROBLEMA 3: Token di Autenticazione Non Sicuro

**Sintomo**: Il token di login era salvato in SharedPreferences (non criptato) e stampato nei log.

**Rischio**: Chiunque con accesso al dispositivo o ai log poteva rubare la sessione dell'utente.

**Soluzione**:
```dart
// PRIMA (insicuro)
SharedPreferences prefs = await SharedPreferences.getInstance();
await prefs.setString('token', value);  // Non criptato!
print('Bearer $token');  // Esposto nei log!

// DOPO (sicuro)
static const FlutterSecureStorage _secureStorage = FlutterSecureStorage(
  aOptions: AndroidOptions(encryptedSharedPreferences: true),
);
await _secureStorage.write(key: key, value: value);  // Criptato!
debugPrint('API Call: POST $endpoint');  // Token non esposto
```

**File modificati**:
- `lib/helpers/shared_preferences_helper.dart` - Storage sicuro per token
- `lib/core/api_service.dart` - Rimosso print del token
- `pubspec.yaml` - Aggiunto `flutter_secure_storage: ^9.2.2`

**Impatto sicurezza**: Token ora criptato con Android KeyStore / iOS Keychain.

---

### PROBLEMA 4: Traffico HTTP Non Sicuro Permesso

**Sintomo**: L'app permetteva connessioni HTTP (non criptate) invece di forzare HTTPS.

**Rischio**: Attacchi man-in-the-middle potevano intercettare dati sensibili.

**Soluzione**:
```xml
<!-- PRIMA (insicuro) -->
<application android:usesCleartextTraffic="true">

<!-- DOPO (sicuro) -->
<application android:usesCleartextTraffic="false">
```

**File modificati**:
- `android/app/src/main/AndroidManifest.xml`
- `android/app/src/main/res/xml/network_security_config.xml`

**Impatto sicurezza**: Solo connessioni HTTPS permesse.

---

### PROBLEMA 5: API Key Gemini Esposta

**Sintomo**: L'API key per Gemini AI era hardcoded nel file `cloudbuild.yaml` (visibile su GitHub).

**Rischio**: Chiunque poteva usare la nostra quota API o accumulare costi sul nostro account.

**Soluzione**:
```yaml
# PRIMA (esposta)
- '--set-env-vars'
- 'GEMINI_API_KEY=AIzaSy...'  # Visibile a tutti!

# DOPO (sicura)
- '--set-secrets'
- 'GEMINI_API_KEY=GEMINI_API_KEY:latest'  # Da Secret Manager
```

**Configurazione aggiuntiva**:
```bash
# Cloud Run ora usa Secret Manager
gcloud run services update movement-analysis \
  --remove-env-vars=GEMINI_API_KEY \
  --set-secrets=GEMINI_API_KEY=GEMINI_API_KEY:latest
```

**File modificati**:
- `cloudbuild.yaml` (Python backend)
- `.gitignore` creato per proteggere secrets futuri

**Impatto sicurezza**: API key non più esposta nel codice sorgente.

---

## Riepilogo Modifiche

### Flutter App (7 file)
| File | Tipo Modifica |
|------|---------------|
| `lib/core/api_service.dart` | Timeout + rimozione log token |
| `lib/providers/subscription_plans_provider.dart` | Error handling |
| `lib/helpers/shared_preferences_helper.dart` | Secure storage |
| `pubspec.yaml` | Nuova dipendenza |
| `android/app/src/main/AndroidManifest.xml` | HTTPS only |
| `android/app/src/main/res/xml/network_security_config.xml` | Security config |

### Python Backend (4 file)
| File | Tipo Modifica |
|------|---------------|
| `Dockerfile` | Fix pacchetto deprecato |
| `cloudbuild.yaml` | Secret Manager |
| `server.py` | Validazione input |
| `.gitignore` | Protezione secrets |

---

## Commit Effettuati

```
656edaa - fix: add timeout to all API calls (30s default, 5min for AI operations)
bd04219 - fix: add proper error handling and user feedback for subscription flow
4b035e9 - security: implement secure token storage and enforce HTTPS
[deploy] - Python backend deployed with Secret Manager integration
```

---

## Prossimi Passi (FASE 2-3)

1. **RevenueCat Integration** - Semplificare gestione abbonamenti iOS/Android
2. **Mixpanel Integration** - Analytics per capire comportamento utenti

---

## Lezioni Apprese

1. **Timeout sono essenziali** - Mai fare chiamate HTTP senza timeout
2. **Feedback utente sempre** - Mai catch silenzioso in produzione
3. **Secrets mai nel codice** - Usare sempre Secret Manager o .env
4. **HTTPS obbligatorio** - Mai permettere traffico non criptato
5. **Secure storage per token** - SharedPreferences normale non basta

---

## Note per Altri Founder Non-Tecnici

Se stai costruendo un'app e non sei uno sviluppatore:
- Fai fare un security audit PRIMA di lanciare
- I problemi di sicurezza possono distruggere la fiducia degli utenti
- Investire in sicurezza costa meno che gestire una breach

---

*Questo log fa parte della serie "Build in Public" di WodVision.*
*Seguimi per aggiornamenti sul journey da founder non-tecnico.*
