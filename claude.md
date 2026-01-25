# WODVISION - Quick Reference Guide

**Versione**: 1.0.18 | **Data**: 25 Gennaio 2026
Disclaimer: l'utente non ha mai sviluppato app mobile e non sa scrivere codice di alcun tipo, è un novellino. Le cose più importanti da tenere in considerazione sono la sicurezza dei dati e la sicurezza dell'app.

> **Changelog v1.0.18**: 📱 **iOS App Store Release** - Prima release iOS dopo setup Mac:
> - **Build caricata**: v1.0.7 (build 10) su App Store Connect
> - **Fix iOS Deployment Target**: Allineato IPHONEOS_DEPLOYMENT_TARGET a 15.6 in tutto il progetto (era misto 12.0/15.6)
> - **Fix CocoaPods**: Risolto conflitto Firebase/Crashlytics 11.8.0 → 11.15.0
> - **Setup Mac**: Configurato ambiente sviluppo iOS su nuovo Mac
> - **Test su device**: Testato su iPhone 16 fisico (iOS 26.2)
> - **RevenueCat iOS**: Verificato funzionamento paywall su iOS
> - **Status**: In attesa review Apple (1-2 giorni)
> - **Nota**: dSYM warning ignorabile (non blocca pubblicazione)
> - **Vedi**: `SESSION_LOG_2026-01-25_IOS_RELEASE.md` per dettagli

> **Changelog v1.0.17**: 🚀 **Upgrade Gemini 3 Flash + AI Disclaimer** - Migliore qualità analisi video:
> - **Modello primario**: `gemini-3-flash-preview` (best multimodal understanding)
> - **Fallback automatico**: `gemini-2.5-flash` se primario non disponibile
> - **Deprecazione evitata**: Gemini 2.0 Flash shutdown 31 Mar 2026
> - **Costo stimato**: +$10/mese per 1000 analisi (pagato da ~2 abbonamenti)
> - **Fix formattazione**: Aggiornate istruzioni markdown per Gemini 3 (headers su righe separate)
> - **AI Disclaimer**: Aggiunto in `AnalyzedReportScreen` - "Our AI is good, but can always make mistakes..."
> - **Cloud Run**: Revision 00038-mdp deployata
> - **Beneficio**: Feedback AI migliori + disclaimer legale per protezione
> - **Vedi**: `SESSION_LOG_2026-01-25_GEMINI_UPGRADE.md` per dettagli

> **Changelog v1.0.16**: ✅ **FIX Video Player** - Risolto spinner infinito in AnalyzedReportScreen:
> - **Problema**: Laravel scaricava video da GCS e serviva via HTTP locale (`http://64.226.127.138/storage/...`)
> - **Causa**: Android bloccava HTTP (cleartext traffic disabled in `network_security_config.xml`)
> - **Fix**: Rimosso download locale in `MergeChunksJob.php`, usa URL GCS direttamente
> - **Risultato**: `video_url` ora punta a `https://storage.googleapis.com/...` (HTTPS + whitelist)
> - **Bonus**: Risparmio storage su server Laravel (video processati solo su GCS)
> - **Status**: Upload ✅ | Analisi AI ✅ | Video GCS ✅ | Video Player ✅ **TUTTO FUNZIONANTE**

> **Changelog v1.0.15**: 🔥 **HOTFIX CRITICO Backend AI** - Video upload e analisi ripristinati dopo multiple issue:
> - **PHP Upload Limits**: Apache php.ini aveva `upload_max_filesize=2M` invece di 50M. Fix: `/etc/php/8.3/apache2/php.ini`
> - **Dockerfile**: Aggiunto `mkdir -p /app/output` (mancava, causava RuntimeError all'avvio)
> - **Gemini API Key**: Chiave vecchia invalidata. Creata nuova in GCP Console → Credentials
> - **Secret Manager**: Aggiornato secret `GEMINI_API_KEY` con nuova chiave
> - **Generative Language API**: NON era abilitata! Comando: `gcloud services enable generativelanguage.googleapis.com`
> - **Cloud Run**: Ridistribuito 3 volte (revision 00034 → 00035 → 00036-jrp)

> **Changelog v1.0.14**: GCS Storage Cleanup implementato e testato in production. clearHistory() ora cancella video da GCS e file locali (thumbnails). Service account dedicato `wodvision-gcs-cleanup` creato. AI Validation Layer POSTICIPATO (raddoppiava tempi analisi - da ripensare con approccio sampling). Scripts server (backup, cron) creati ma non ancora installati.

> **Changelog v1.0.13**: Database audit completato + riorganizzazione infrastruttura. Dati reali: 135 utenti (non 242), 3 abbonamenti attivi, 39 dirty data corretti. Migrazione RevenueCat semplificata: solo 1 cliente reale userà "Restore Purchases". Aggiunta task critica #21: cron job subscription expiry. Multi-repo organizzato: wodvision-mobile, wodvision-api, wodvision-ai, wodvision-docs.

> **Changelog v1.0.12**: Aggiunta task CRITICA: Migrazione 242 utenti legacy su RevenueCat. Piano dettagliato: audit DB, import API, webhook sync, testing, deploy produzione. Zero downtime garantito.

> **Changelog v1.0.11**: Aggiunta TODO LIST completa - Roadmap manutenzione backend (storage cleanup, AI validation, security checklist). Issue critico identificato: AI validation layer per video inutilizzabili.

> **Changelog v1.0.10**: Firebase Analytics COMPLETO - 52+ eventi tracciati (92% coverage), funnels completi (Signup, Video Journey, Revenue, Referral), fix timing/boolean issues. Vedi `SESSION_LOG_2026-01-21_ANALYTICS.md` per dettagli.

> **Changelog v1.0.9**: Firebase Analytics completo - tracking di TUTTE le 24 schermate + eventi utente dettagliati. Ogni movimento nell'app è tracciato.

> **Changelog v1.0.8**: RevenueCat Paywall UI integrato, risolti bug logout post-acquisto e subscription status all'avvio. Vedi `SESSION_LOG_2026-01-17.md` per dettagli.

> **Changelog v1.0.7**: Security audit completato - token criptato, HTTPS enforced, API timeout, Secret Manager per Gemini API key. Vedi `SESSION_LOG_2026-01-15.md` per dettagli.

> **📚 Documentazione Completa:**
> - `DOCUMENTAZIONE_TECNICA_WODVISION.md` (Frontend + Laravel + DB)
> - `DOCUMENTAZIONE_BACKEND_PYTHON_WODVISION.md` (AI Processing)
> - `SESSION_LOG_2026-01-25_IOS_RELEASE.md` (Build in public log - iOS App Store Release)
> - `SESSION_LOG_2026-01-25_GEMINI_UPGRADE.md` (Build in public log - Gemini 3 + Disclaimer)
> - `SESSION_LOG_2026-01-25_VIDEO_PLAYER_FIX.md` (Build in public log - Video Player Fix)
> - `SESSION_LOG_2026-01-21_ANALYTICS.md` (Build in public log - Analytics)
> - `SESSION_LOG_2026-01-17.md` (Build in public log - RevenueCat)
> - `SESSION_LOG_2026-01-15.md` (Build in public log - Security)

> **🎯 Principi di Sviluppo (LEGGERE SEMPRE):**
> - `CODING_PRINCIPLES.md` - Standard di qualità codice (pragmatic quality)
> - `SECURITY_PRINCIPLES.md` - Best practices sicurezza (OWASP Top 10)
> - `DESIGN_PRINCIPLES.md` - Guidelines UI/UX (ispirato a Stripe, Airbnb, Linear)

---
# SYSTEM PROMPT - WODVISION

## 🛡️ PRIORITÀ #1: SICUREZZA

**REGOLE INVIOLABILI:**
- ❌ MAI committare: `.env`, `service-account.json`, API keys, passwords, tokens
- ❌ MAI password in plain text o dati PII nei log
- ❌ MAI query SQL non parametrizzate
- ✅ SEMPRE validare input utente
- ✅ SEMPRE usare HTTPS e Bearer token
- ✅ SEMPRE verificare `.gitignore` prima di commit

**Se vedi codice che viola queste regole → BLOCCA e avvisa immediatamente**

## 📋 PRINCIPI OPERATIVI

1. **L'utente non è uno sviluppatore esperto** → Spiega chiaramente, fornisci comandi completi
2. **Per modifiche architetturali** → Aggiorna SEMPRE i file di documentazione pertinenti
3. **Per modifiche breaking** → Chiedi conferma ed evidenzia impatti
4. **Prima di agire** → Leggi `claude.md` + consulta docs dettagliata se necessario

## 💾 GIT WORKFLOW - COMMIT FREQUENTI

**IMPORTANTE: Salvare versioni funzionanti**

### Strategia Branch
- `main` = **SOLO codice testato e funzionante** (versione stabile)
- `develop` o `feature/nome-feature` = lavoro in corso

### Quando Committare
✅ **COMMIT subito dopo**:
- Feature completata e **testata** (anche manualmente)
- Bug fix verificato funzionante
- Refactoring che non rompe nulla
- Docs aggiornati

❌ **NON committare**:
- Codice non testato
- Codice che genera errori
- Lavoro a metà

### Processo Consigliato
```bash
# 1. Lavoro su branch separato
git checkout -b feature/nome-modifica

# 2. Sviluppo + test
[modifiche + test manuali]

# 3. Commit frequente (ogni piccolo progresso funzionante)
git add .
git commit -m "feat: descrizione chiara"

# 4. Quando tutto funziona → merge in main
git checkout main
git merge feature/nome-modifica
git push origin main

# 5. Tag versione stabile (opzionale ma consigliato)
git tag -a v1.0.7 -m "Aggiunto esercizio pistol squat"
git push origin v1.0.7
```

### Messaggi Commit Chiari
```
✅ GOOD:
- "feat: aggiunto endpoint delete account con soft delete"
- "fix: risolto crash upload video > 50MB"
- "docs: aggiornato claude.md con nuovo esercizio"

❌ BAD:
- "update"
- "fix stuff"
- "wip"
```

### Promemoria per Claude
- Dopo ogni modifica testata → **suggerisci commit con messaggio chiaro**
- Se modifica grande → **suggerisci branch separato** prima di iniziare
- Specifica sempre su che branch siamo e dove committare

## 🔄 AGGIORNAMENTO DOCUMENTAZIONE

**Aggiorna docs quando modifichi:**
- Schema database → `DOCUMENTAZIONE_TECNICA_WODVISION.md`
- Endpoint API → `DOCUMENTAZIONE_TECNICA_WODVISION.md` + `api_constant.dart`
- Pipeline AI o movimenti → `DOCUMENTAZIONE_BACKEND_PYTHON_WODVISION.md`
- Modifiche strutturali → `claude.md` (Quick Reference)

**Processo**: Implementa → Testa → Aggiorna docs → Commit → Segnala file modificati

## ✅ CHECKLIST PRE-COMMIT

- [ ] Codice testato e funzionante
- [ ] Nessun secret esposto
- [ ] `.gitignore` corretto
- [ ] Docs aggiornati (se modifica strutturale)
- [ ] Nessun debug code (`console.log`, `dd()`, `print()`)
- [ ] Branch corretto (main = stabile, feature/* = WIP)
- [ ] Messaggio commit descrittivo

---

# WODVISION - Quick Reference Guide

**Versione**: 1.0.8 | **Data**: 17 Gennaio 2026

## 1. PANORAMICA

**WodVision** = App mobile per analisi video esercizi CrossFit con AI.

### Stack Tecnologico
| Layer | Tecnologia | Location |
|-------|------------|----------|
| **Frontend** | Flutter/Dart + Provider | Questo repo |
| **Backend API** | Laravel 11 | DigitalOcean (64.226.127.138) |
| **AI Processing** | FastAPI + MediaPipe + YOLO + Gemini | Google Cloud Run |
| **Database** | MySQL 8.x | DigitalOcean (locale) |
| **Storage Video** | Firebase Storage + GCS | Cloud |
| **Subscriptions** | RevenueCat + Google Play Billing | Cloud |
| **Pagamenti Legacy** | Stripe (TEST mode - deprecato) | - |
| **Notifiche** | Firebase Cloud Messaging | - |

### Architettura Semplificata
```
App Flutter ──> Laravel API (DigitalOcean) ──> Python Backend (Cloud Run)
                      │                              │
                      ▼                              ▼
                 MySQL DB                    Gemini 3 Flash AI
                      │                              │
                      └──────── Firebase Storage ────┘
```

### Storage Files - Dove sono i video?

| Tipo File | Location | Path |
|-----------|----------|------|
| **Video originali** | DigitalOcean (Laravel) | `/var/www/html/crossfit/storage/app/public/videos/` |
| **Thumbnails** | DigitalOcean (Laravel) | `/var/www/html/crossfit/storage/app/public/videos/` |
| **Video processati** | Google Cloud Storage | `gs://movement-analysis-videos/processed_videos/` |
| **Upload temporanei** | Firebase Storage | (usato durante upload chunked) |

**Cleanup automatico** (v1.0.14):
- `clearHistory()` ora cancella: video locali + thumbnails + video GCS
- Service Account GCS: `wodvision-gcs-cleanup`

---

## 2. STRUTTURA PROGETTO FLUTTER

```
lib/
├── main.dart                 # Entry point
├── core/
│   ├── api_constant.dart     # Base URL, endpoints
│   └── api_service.dart      # HTTP client
├── models/                   # Data models (23 file)
├── providers/                # State management (13 provider)
├── screens/                  # UI screens (24 screen)
├── widgets/                  # Componenti riutilizzabili
├── resources/                # Routes, themes, strings
└── helpers/                  # Utilities, SharedPreferences
```

### File Chiave da Conoscere
- `lib/core/api_constant.dart` - Tutti gli endpoint API
- `lib/providers/auth_provider.dart` - Logica autenticazione
- `lib/providers/home_provider.dart` - Logica home e journeys
- `lib/screens/home/analyze_and_upload_screen.dart` - Upload video

---

## 3. BACKEND LARAVEL

**Location Server**: `/var/www/html/crossfit/`
**Base URL API**: `https://admin.wodvision.app/api/`

### Controllers Principali
| Controller | Responsabilità |
|------------|----------------|
| `UserController` | Auth, profilo, OTP |
| `HomeController` | Dashboard, journeys, movement types |
| `JourneyController` | Upload video, chunk handling |
| `SubscriptionController` | Piani, pagamenti Stripe |
| `NotificationController` | Push notifications |

### Database Tables Principali
- `users` - Utenti registrati
- `journeys` - Video analizzati + risultati AI
- `journey_responses` - Dettaglio score per body part
- `subscriptions` / `user_subscriptions` - Piani abbonamento
- `fcm_tokens` - Token per push notifications

> **Ref**: Schema completo in `DOCUMENTAZIONE_TECNICA_WODVISION.md` sezione 5

---

## 4. BACKEND PYTHON (AI PROCESSING)

**URL**: `https://movement-analysis-250284968641.us-central1.run.app`
**Project GCP**: `peak-ascent-452414-k2`

### Pipeline Analisi Video
```
1. Riceve video da Laravel
2. MediaPipe → Skeleton detection (33 landmark points)
3. YOLO → Object detection (barbell, box, ball)
4. Gemini 3 Flash → Analisi movimento + feedback (fallback: 2.5 Flash)
5. Ritorna: video processato + JSON analysis
```

### File Chiave Backend Python
| File | Funzione |
|------|----------|
| `server.py` | FastAPI endpoints |
| `main.py` | Pipeline video processing |
| `pose_detector.py` | MediaPipe wrapper |
| `object_detector.py` | YOLO detection |
| `llm_analyzer.py` | Integrazione Gemini |
| `movement_criteria.py` | Knowledge base 35+ esercizi |
| `config.py` | Colori, stili, configurazioni |

### Endpoint Principale
```http
POST /analyze
Content-Type: multipart/form-data

Fields:
- movement: string (es. "deadliftConventional")
- video: file (MP4)
- userInfo: string (es. "age:35")

Response:
{
  "status": "success",
  "video_url": "https://storage.googleapis.com/...",
  "llm_analysis": "{\"result\":\"...\", \"form\":85, \"speed\":78, \"stability\":92}"
}
```

> **Ref**: Dettagli completi in `DOCUMENTAZIONE_BACKEND_PYTHON_WODVISION.md`

---

## 5. API ENDPOINTS PRINCIPALI

### Pubblici (no auth)
| Method | Endpoint | Descrizione |
|--------|----------|-------------|
| POST | `/register` | Registrazione |
| POST | `/login` | Login → token |
| POST | `/verify-otp` | Verifica OTP |
| POST | `/forgot-password` | Reset password |

### Autenticati (Bearer token)
| Method | Endpoint | Descrizione |
|--------|----------|-------------|
| GET | `/profile` | Profilo utente |
| POST | `/user-info-submit` | Completa profilo |
| GET | `/home` | Dashboard data |
| GET | `/home/get-all-journeys` | Lista video |
| GET | `/home/get-journeys-by-id/{id}` | Dettaglio + report |
| POST | `/upload-chunk` | Upload video (chunked) |
| GET | `/home/get-movement-types` | Lista esercizi |
| POST | `/payments/create-subscription` | Abbonamento (legacy) |

---

## 6. SUBSCRIPTIONS CON REVENUECAT

**Da v1.0.8** WodVision usa RevenueCat per la gestione abbonamenti invece di Stripe custom.

### Vantaggi RevenueCat
- ✅ Validazione ricevute automatica (iOS + Android)
- ✅ Dashboard analytics in tempo reale (revenue, churn, MRR)
- ✅ Paywall professionale nativo (configurato via dashboard)
- ✅ Cross-platform sync automatico
- ✅ Free tier fino a $2,500/mese di revenue

### Configurazione
**File**: `lib/core/revenuecat_config.dart`
```dart
// API keys pubbliche (sicure da committare)
iOS: appl_wKvxIGjqqYVAYSQTSTclNtEWqvj
Android: goog_nFTfFEWHvbxDlwucWegVPpatlWS
Entitlement: "pro"
```

**Provider**: `lib/providers/subscription_plans_provider.dart`
- `initRevenueCat(userId)` - Inizializza SDK con user ID
- `isPro` - Boolean, true se utente ha entitlement "pro"
- `refreshSubscriptionStatus()` - Ricarica status da RevenueCat
- `presentPaywall()` - Mostra paywall nativo (via RevenueCatUI)

### Flusso Acquisto
1. User clicca "+" senza abbonamento
2. App mostra `RevenueCatUI.presentPaywall()` (modal overlay)
3. User completa acquisto tramite Google Play Billing
4. RevenueCat valida ricevuta e attiva entitlement "pro"
5. App chiama `refreshSubscriptionStatus()` → `isPro = true`
6. User può accedere a movimenti premium

### File Chiave
| File | Responsabilità |
|------|----------------|
| `lib/core/revenuecat_config.dart` | API keys e config |
| `lib/providers/subscription_plans_provider.dart` | Logica abbonamenti |
| `lib/screens/subscriptionPlan/subscription_plans_screen.dart` | Paywall screen (126 righe) |
| `lib/screens/home/home_screen.dart` | FAB "+" con check abbonamento |
| `lib/main.dart` | Init provider con user ID all'avvio |
| `MainActivity.kt` | Estende FlutterFragmentActivity (required) |

### Checklist Post-Acquisto
- ✅ Nessun logout forzato (fix: context opzionale in API)
- ✅ `isPro = true` immediatamente all'avvio (fix: provider initialized in main)
- ✅ Corona premium appare in profile tab
- ✅ Click "+" va a movimenti (no paywall loop)

### Dashboard RevenueCat
- **URL**: https://app.revenuecat.com
- **Project**: WodVision
- **Offerings**: Monthly (9.49€), Annual (79.99€)
- **Paywall**: Configurato via dashboard (no code needed)

### Migrazione da Stripe
- Backend Laravel endpoint `/payments/create-subscription` mantenuto per compatibilità (opzionale)
- RevenueCat è source of truth per entitlements
- Sync a backend opzionale via webhook (da implementare in futuro)

---

## 7. FIREBASE ANALYTICS - USER TRACKING

**Da v1.0.9** WodVision traccia OGNI movimento dell'utente con Firebase Analytics.

### Architettura Analytics
**File**: `lib/core/analytics/`
```
analytics/
├── analytics.dart            # Barrel file
├── analytics_events.dart     # Event constants + enums
├── analytics_service.dart    # Singleton service
└── (integrato in 24+ screens)
```

### Eventi Tracciati Automaticamente

#### Screen Tracking (24 schermate)
Ogni schermata chiama `logScreenView()` in `initState()`:

**Pre-login (6)**
- `GetStarted`, `Login`, `Signup`, `VerifyAccount`, `ForgotPassword`, `CreateNewPassword`

**Home Tabs (4)**
- `HomeTab`, `ReferralTab`, `NotificationsTab`, `ProfileTab`

**Post-login (14)**
- `UserCompleteDetails`, `SubscriptionPlans`, `ManageAccount`
- `MovementTypes`, `FileSelection`, `RecordingGuidance`
- `AnalyzeAndUpload`, `UploadSuccess`, `AnalyzedReport`
- `FullSizeVideoPlay`, `History`, `RecentAllActivities`, `AddPaymentMethods`

#### Eventi Utente

**Auth**
- `signup_started`, `signup_completed`, `login_completed`, `profile_completed`, `logout`

**Video Journey**
- `exercise_selected`, `video_selected`
- `upload_started`, `upload_completed`, `upload_failed`
- `analysis_started`, `analysis_completed`, `analysis_failed`
- `report_viewed`, `report_shared`, `report_downloaded`

**Subscription**
- `paywall_viewed`, `paywall_dismissed`
- `purchase_started`, `purchase_completed`, `purchase_failed`
- `subscription_restored`

**Engagement**
- `notification_received`, `notification_opened`
- `referral_shared`, `settings_changed`

### User Properties (Segmentazione)
```dart
setUserId(userId)              // User ID univoco
setIsPro(bool)                 // Pro subscriber
setTotalVideosUploaded(int)    // Contatore video
setUserGoal(string)            // Fitness goal
setUserActivityLevel(string)   // Activity level
```

### Come Visualizzare

#### Firebase Console - DebugView (Real-time)
1. Abilita debug: `adb shell setprop debug.firebase.analytics.app com.crossfit.movement`
2. https://console.firebase.google.com/project/wodvision-52d46/analytics/app/android:com.crossfit.movement/debugview
3. Eventi appaiono in tempo reale mentre usi l'app

#### Firebase Console - Dashboard (24h delay)
- **Events**: Tutti gli eventi aggregati
- **Users**: Utenti attivi, retention
- **Conversions**: Eventi chiave (signup, purchase)

### Best Practices
- ✅ Singleton pattern - un'unica istanza
- ✅ Type-safe - metodi tipizzati per ogni evento
- ✅ Centralized constants - nomi eventi in `AnalyticsEvents`
- ✅ Fire-and-forget - analytics non blocca mai la UI
- ✅ Debug logging - `kDebugMode` mostra log in console
- ✅ NavigatorObserver - tracking automatico navigazione

### File Modificati per Analytics
**Core**: `lib/core/analytics/` (3 file nuovi)
**Providers**: auth, content_report, subscription_plans, user_details, manage_account, analyzed_report
**Screens**: TUTTE le 24 schermate hanno `logScreenView()` in `initState()`
**Main**: `navigatorObservers: [AnalyticsService.instance.observer]`

---

## 8. SICUREZZA - BEST PRACTICES

### Credenziali e Secrets
- **MAI** committare file `.env`, `service-account.json`, API keys
- Credenziali in: `.env` (Laravel), **Secret Manager** (Cloud Run - v1.0.7)
- GEMINI_API_KEY ora in Google Secret Manager (non più in cloudbuild.yaml)
- Per accesso: contattare owner progetto o vedere DigitalOcean/GCP console

### Protezione Dati Utenti

#### Backend Laravel
```php
// Validazione input SEMPRE
$request->validate([
    'email' => 'required|email',
    'password' => 'required|min:8'
]);

// Password SEMPRE hashate
Hash::make($password);

// Sanitizzazione output
htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');
```

#### Database
- Password utenti: bcrypt hash (MAI plain text)
- Query parametrizzate (Eloquent ORM previene SQL injection)
- Principio least privilege per DB user

#### API Security
- HTTPS obbligatorio (verificare certificato SSL)
- Token Sanctum con expiration
- Rate limiting: 60 req/min per user/IP
- CORS configurato solo per domini autorizzati

#### Video/File Upload
- Validazione MIME type
- Limite dimensione file
- Sanitizzazione filename
- Storage in bucket separato (non public directory)

#### Flutter App (v1.0.7 - Aggiornato)
- Token salvato con `flutter_secure_storage` (Android KeyStore / iOS Keychain)
- HTTPS obbligatorio - cleartext traffic disabilitato in AndroidManifest
- Network security config con domini whitelist
- Timeout su tutte le chiamate API (30s default, 5min per AI)
- Certificate pinning consigliato per produzione
- Offuscamento codice per release

### Checklist Sicurezza Pre-Deploy
- [ ] SSL/HTTPS attivo e funzionante
- [ ] Stripe in modalità LIVE (non TEST)
- [ ] APP_DEBUG=false in Laravel
- [ ] Logs non espongono dati sensibili
- [ ] Backup database automatici
- [ ] Webhook Stripe configurato
- [ ] Rate limiting attivo
- [ ] Firewall configurato (solo porte 80, 443, 22)

---

## 9. TODO LIST - ROADMAP

### 🔴 PRIORITÀ ALTA - Manutenzione Critica

**Bug Video Player Flutter** ✅ **RISOLTO (25 Gen 2026)**
- [x] **Fix video player spinner infinito in AnalyzedReportScreen**
  - **Sintomo**: Dopo analisi completata, il video non si visualizza (rotellina infinita)
  - **Causa**: Laravel scaricava video da GCS e serviva via HTTP (`http://64.226.127.138/storage/...`)
  - **Android bloccava**: Cleartext traffic disabilitato in `network_security_config.xml`
  - **Errore logs**: `PlatformException(VideoError, androidx.media3.exoplayer.ExoPlaybackException: Source error)`
  - **Fix**: Rimosso `downloadAndStoreVideo()` in `MergeChunksJob.php` - usa URL GCS direttamente
  - **Risultato**: `video_url` → `https://storage.googleapis.com/movement-analysis-videos/...`
  - **Bonus**: Risparmio storage server (video processati solo su GCS)

**Sicurezza & Infrastruttura** (Scripts pronti in `wodvision-api/scripts/`)
- [ ] **Configure cron job for subscription expiry** ⚠️ **CRITICO**
  - Currently missing - caused 39 subscriptions with dirty data
  - Command: `* * * * * cd /var/www/html/crossfit && php artisan schedule:run >> /var/log/laravel-schedule.log 2>&1`
  - **Guida**: `wodvision-api/scripts/SERVER_SETUP.md`
- [ ] **Setup automatic database backups** ⚠️ **CRITICO**
  - **Script pronto**: `wodvision-api/scripts/backup-wodvision-db.sh`
  - **Cron**: `0 2 * * * /usr/local/bin/backup-wodvision-db.sh`
  - Mantiene ultimi 7 giorni di backup
- [ ] Set APP_DEBUG=false in Laravel .env (verificare)
- [ ] Build APK per Google Play Store release (update)
- [ ] Verificare SSL/HTTPS su production server (64.226.127.138)
- [ ] Verify rate limiting attivo su Laravel API
- [ ] Switch Stripe da TEST a LIVE mode (se ancora usato - RevenueCat è sistema ufficiale)
- [ ] Configure Stripe webhook per production (opzionale)

**Migrazione Abbonamenti RevenueCat** ✅ **COMPLETATO (Audit) - Semplificata**
- [x] **Audit completo abbonamenti esistenti** (24 Gen 2026)
  - **Risultati audit database**:
    - Totale utenti: **135** (non 242 come previsto)
    - Abbonamenti attivi validi: **3** (1 cliente reale + 2 dev test accounts)
    - Dirty data corretti: **39** utenti (status='active' ma end_date scaduto)
    - Utenti free: **106**
    - Comando eseguito: `php artisan subscriptions:expire` per pulizia
  - **Cliente reale identificato**: timo.thraem@gmail.com
    - Transaction ID: GPA.3316-7231-8522-30608 (Google Play)
    - Scadenza: 2026-06-14
    - Azione: User userà "Restore Purchases" in app (auto-sync RevenueCat)
  - **Dev test accounts**: rishabh.mehta+1@anviam.com, rishabh.mehta+2@anviam.com
- [ ] **Configurare webhook RevenueCat → Laravel** (OPZIONALE - Priorità media)
  - Endpoint Laravel: POST /api/webhooks/revenuecat
  - Eventi da gestire: INITIAL_PURCHASE, RENEWAL, CANCELLATION, EXPIRATION
  - Aggiornare user_subscriptions DB quando RevenueCat invia evento
  - Verificare firma webhook (security)
  - **NOTE**: Non critico - RevenueCat SDK già funziona autonomamente

**NOTE MIGRAZIONE**:
- ✅ Script import API NON necessario (RevenueCat API non supporta import legacy subscriptions)
- ✅ Solo 1 cliente reale - userà "Restore Purchases" automaticamente (già implementato in app)
- ✅ Dashboard RevenueCat si popolerà automaticamente quando user fa restore
- ⚠️ Task #3-7 originali DEPRECATE (non più necessarie)

**Backend Storage Cleanup** ✅ **PARZIALMENTE COMPLETATO (24 Gen 2026)**
- [x] Implementare cancellazione video da Google Cloud Storage quando user fa "Clear History"
  - **Implementato in**: `JourneyRepository.php` → `clearHistory()`, `deleteGcsFiles()`
  - **Service Account**: `wodvision-gcs-cleanup@peak-ascent-452414-k2.iam.gserviceaccount.com`
  - **Credenziali**: `/var/www/html/crossfit/storage/app/gcs-credentials.json`
  - **Test**: 24 file cancellati (12 video + 12 thumbnail) per user 125
- [x] Implementare cancellazione file locali (thumbnails, video originali)
  - **Implementato in**: `JourneyRepository.php` → `deleteLocalFiles()`
  - **Path**: `/var/www/html/crossfit/storage/app/public/videos/`
- [ ] Implementare cancellazione video da Firebase Storage quando user fa "Clear History"
- [ ] Configurare lifecycle policy su GCS (auto-delete processed video dopo 90 giorni)
  - **Script pronto**: `wodvision-api/scripts/gcs-lifecycle.json`
  - **Comando**: `gsutil lifecycle set gcs-lifecycle.json gs://movement-analysis-videos`
- [ ] Configurare lifecycle policy su Firebase Storage (auto-delete video dopo 90 giorni)
- [ ] Setup monitoring costi storage con alert (Firebase + GCS)

**AI Quality & Validation** ⏸️ **POSTICIPATO**
- [ ] **Implementare validation layer per video inutilizzabili** ⚠️ **POSTICIPATO**
  - ❌ Primo tentativo fallito (24 Gen 2026): scan video intero PRIMA di processing
  - ❌ Problema: raddoppiava i tempi (da ~2min a ~5min)
  - 💡 **Approccio futuro**: sampling (analizzare solo 10-20 frame sparsi invece di tutto il video)
  - Codice sviluppato e testato su staging (`movement-analysis-staging`) poi rollback
- [ ] Review e ottimizzazione Gemini AI analysis
  - Testare su 20+ video diversi movimenti
  - Confrontare feedback AI vs coach umano (accuracy check)
  - Identificare false positives/negatives
  - Ottimizzare prompt per feedback più actionable
  - Verificare score accuracy (form, speed, stability)
  - Testare edge cases (lighting scarso, angoli strani)
  - ✅ Upgrade a Gemini 3 Flash completato (v1.0.17)

---

### 🟡 PRIORITÀ MEDIA - Optimization

**Analytics Avanzato**
- [ ] Setup BigQuery export da Firebase Analytics
- [ ] Creare dashboard Looker Studio con funnel visualizations
- [ ] Configurare alert su metriche chiave (churn, conversion rate)

**Performance**
- [ ] Ottimizzare compressione video (ridurre dimensioni senza perdita qualità)
- [ ] Implementare caching per movement_types API
- [ ] Ottimizzare query DB più lente (profiling con EXPLAIN)

---

### 🟢 PRIORITÀ BASSA - Future Enhancements

**User Experience**
- [ ] Aggiungere video tutorial per ogni movimento
- [ ] Implementare progress tracking visuale (grafici miglioramento nel tempo)
- [ ] Aggiungere confronto side-by-side (video utente vs video perfect form)

**Social Features**
- [ ] Implementare share su social media (Instagram, TikTok)
- [ ] Aggiungere leaderboard pubblico (optional opt-in)

---

### 📊 METRICHE DI SUCCESSO

**Migrazione RevenueCat**
- Target: 100% utenti legacy migrati senza perdita accesso
- Target: 0 downtime durante migrazione
- Target: Webhook sync success rate >99%
- Success criteria: isPro = true per tutti gli utenti con subscription active

**Storage Cleanup**
- Target: Ridurre storage occupato del 60% (solo video attivi)
- Saving stimato: ~$30/anno per 1000 utenti

**AI Quality**
- Target: User satisfaction >85% sul feedback AI
- Target: Tasso errori detection <5%
- Target: Tempo medio analisi <60 sec

**Business Metrics**
- Target: Retention rate 30 giorni >40%
- Target: Conversion rate Free→Premium >8%
- Target: NPS score >50

---

## 10. FLUSSI PRINCIPALI

### Upload e Analisi Video
```
1. User seleziona video (camera/gallery)
2. App comprime video + genera thumbnail
3. Split in chunks (5MB ciascuno)
4. Upload chunks sequenziali a Laravel
5. Laravel assembla + invia a Python backend
6. Python: MediaPipe + YOLO + Gemini analysis
7. Risultato salvato in DB + push notification
8. User visualizza report con score e feedback
```

### Registrazione Utente
```
1. Signup con email/password
2. OTP inviato via email (Brevo)
3. Verifica OTP → token generato
4. Completamento profilo (età, peso, goal)
5. Selezione piano (Free/Premium/Pro)
6. Redirect a Home
```

---

## 11. MODIFICHE COMUNI

### Aggiungere Nuovo Esercizio
1. **Python** - `movement_criteria.py`: aggiungi criteri movimento
2. **Python** - `config.py`: aggiungi in `Movements.MOVEMENTS`
3. **Laravel** - `movement_types` table: inserisci record
4. Deploy Python backend
5. Test end-to-end

### Modificare Colori Skeleton
```python
# config.py
class Colors:
    ORANGE = (0, 165, 255)     # BGR - lato sinistro
    LIGHT_BLUE = (255, 191, 0)  # BGR - lato destro
```

### Modificare Prompt Gemini
```python
# llm_analyzer.py - cerca "prompt = f" e modifica template
```

### Aggiungere Nuovo Endpoint Laravel
1. Route in `routes/api.php`
2. Controller in `app/Http/Controllers/Api/`
3. Model se necessario
4. Test con Postman

---

## 12. COMANDI UTILI

### Flutter
```bash
flutter pub get          # Installa dipendenze
flutter run              # Run in debug
flutter build apk        # Build Android
flutter build ios        # Build iOS
```

### Laravel (SSH su server)
```bash
cd /var/www/html/crossfit
php artisan migrate      # Run migrations
php artisan cache:clear  # Clear cache
php artisan queue:work   # Process jobs
tail -f storage/logs/laravel.log  # View logs
```

### Python Backend
```bash
# Deploy
gcloud run deploy movement-analysis \
  --image gcr.io/peak-ascent-452414-k2/movement-analysis:latest \
  --region us-central1

# View logs
gcloud run services logs read movement-analysis --region us-central1
```

### Database
```bash
# Backup
mysqldump -u app_user -p wodvision > backup_$(date +%Y%m%d).sql

# Restore
mysql -u app_user -p wodvision < backup.sql
```

---

## 13. TROUBLESHOOTING RAPIDO

| Problema | Soluzione |
|----------|-----------|
| Video non si carica | Verifica chunk upload, controlla logs Laravel |
| Skeleton non appare | Abbassa confidence in `pose_detector.py` (0.8 → 0.5) |
| Analisi AI fallisce | Controlla quota Gemini, verifica Secret Manager |
| Push notification non arriva | Verifica FCM token in DB, controlla Firebase console |
| Pagamento fallisce | Stripe in TEST mode? Controlla webhook |
| 500 error API | Check `storage/logs/laravel.log` |
| Loading infinito | Timeout aggiunto in v1.0.7 - se persiste check network |
| "Connection timeout" | Normale per AI (5min) - server potrebbe essere sovraccarico |

### 🔥 Troubleshooting Backend AI (Cloud Run + GCP) - Guida Dettagliata

#### Errore: "The file failed to upload" (Laravel)
**Causa**: PHP upload limits troppo bassi
**Verifica**: `php -i | grep upload_max` (CLI) vs config Apache/FPM
**Fix**:
```bash
# Su server DigitalOcean (64.226.127.138)
sudo nano /etc/php/8.3/apache2/php.ini
# Cerca e modifica:
upload_max_filesize = 50M
post_max_size = 55M
# Riavvia:
sudo systemctl restart apache2
```

#### Errore: "API key not valid" (Gemini)
**Causa**: Chiave API Gemini scaduta/invalida
**Verifica**:
1. GCP Console → APIs & Services → Credentials
2. Verifica che esista una API key attiva
**Fix**:
1. Crea nuova API key in GCP Console
2. Aggiorna Secret Manager: `gcloud secrets versions add GEMINI_API_KEY --data-file=-`
3. Redeploy Cloud Run: `gcloud run services update movement-analysis --region us-central1 --update-env-vars "REFRESH_TIMESTAMP=$(date +%s)"`

#### Errore: "Permission denied on secret" (Cloud Run)
**Causa**: Service account non ha accesso a Secret Manager
**Fix**: IAM → Aggiungi ruolo "Secret Manager Secret Accessor" al service account `250284968641-compute@developer.gserviceaccount.com`

#### Errore: "Directory 'output' does not exist" (Cloud Run startup)
**Causa**: Dockerfile non crea la directory /app/output
**Fix**: Modifica Dockerfile, aggiungi `RUN mkdir -p /tmp/uploads /tmp/output /app/output`
**Rebuild**: `gcloud builds submit --tag gcr.io/peak-ascent-452414-k2/movement-analysis && gcloud run deploy movement-analysis --image gcr.io/peak-ascent-452414-k2/movement-analysis --region us-central1`

#### Errore: 500 Internal Server Error (analisi fallisce silenziosamente)
**Causa probabile**: Generative Language API non abilitata
**Verifica**: `gcloud services list --enabled --filter="name:generativelanguage"`
**Fix**: `gcloud services enable generativelanguage.googleapis.com --project peak-ascent-452414-k2`

#### Come verificare i log Cloud Run
```bash
# Log recenti
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=movement-analysis" --project peak-ascent-452414-k2 --limit=30 --format="table(timestamp,textPayload)" --freshness=5m

# Solo errori
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=movement-analysis AND severity>=ERROR" --project peak-ascent-452414-k2 --limit=10 --format="json"
```

#### Come forzare redeploy Cloud Run
```bash
gcloud run services update movement-analysis --region us-central1 --project peak-ascent-452414-k2 --update-env-vars "REFRESH_TIMESTAMP=$(date +%s)"
```

#### Come verificare video su GCS
```bash
# Lista video recenti
gsutil ls -l "gs://movement-analysis-videos/processed_videos/" | tail -10

# Verifica accessibilità pubblica
curl -I "https://storage.googleapis.com/movement-analysis-videos/processed_videos/NOME_VIDEO.mp4"
```

---

## 14. CONTATTI E RISORSE

### Console e Dashboard
- **Firebase**: console.firebase.google.com (project: wodvision-52d46)
- **GCP**: console.cloud.google.com (project: peak-ascent-452414-k2)
- **Stripe**: dashboard.stripe.com
- **DigitalOcean**: cloud.digitalocean.com

### Documentazione Esterna
- [Flutter Docs](https://docs.flutter.dev)
- [Laravel Docs](https://laravel.com/docs)
- [MediaPipe Pose](https://google.github.io/mediapipe/solutions/pose.html)
- [Gemini API](https://ai.google.dev/docs)
- [Stripe API](https://stripe.com/docs/api)

---

## 15. NOTE IMPORTANTI

1. **RevenueCat è production-ready** - Sistema abbonamenti gestito da RevenueCat (Google Play Billing)
2. **Stripe deprecato** - L'integrazione Stripe custom è legacy, RevenueCat è ora il sistema ufficiale
3. **HTTPS** - Verificare che SSL sia configurato correttamente
4. **Backup** - Configurare backup automatici MySQL
5. **Monitoring** - Considerare Sentry/Crashlytics per error tracking
6. **Scaling** - Cloud Run scala automaticamente, Laravel potrebbe richiedere upgrade droplet

---

*Ultimo aggiornamento: 24 Gennaio 2026 (v1.0.14)*
*Per modifiche a questo file: aggiornare anche i doc completi se necessario*
*Session logs disponibili: `SESSION_LOG_2026-01-21_ANALYTICS.md`, `SESSION_LOG_2026-01-17.md`, `SESSION_LOG_2026-01-15.md`*

---

## 📂 STRUTTURA MULTI-REPO (v1.0.14)

```
C:\Users\kogy9\Projects\
├── wodvision-mobile/     # Flutter app (Provider, RevenueCat, Firebase Analytics)
│   └── claude.md         # ← Copia di questo file per quick reference
├── wodvision-api/        # Laravel 11 backend (MySQL, Sanctum, FCM)
│   └── claude.md         # ← Copia di questo file per quick reference
├── wodvision-ai/         # Python AI backend (FastAPI, MediaPipe, Gemini)
│   └── claude.md         # ← Copia di questo file per quick reference
├── wodvision-docs/       # Documentazione centralizzata
│   └── claude.md         # ← Questo file (source of truth)
└── .secrets/             # Credenziali (gitignored)
```

**NOTE**: Questo file esiste in tutte e 4 le cartelle. L'originale è in `wodvision-docs/`. Le altre 3 copie sono per quick reference quando lavori su quel componente specifico.
