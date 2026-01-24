# DOCUMENTAZIONE TECNICA COMPLETA - WODVISION APP

**Versione App**: 1.0.6 (Build 7)
**Data Documentazione**: 11 Gennaio 2026
**Autore**: Documentazione Tecnica Completa

---

## INDICE

1. [Panoramica Generale](#1-panoramica-generale)
2. [Architettura Sistema](#2-architettura-sistema)
3. [Frontend - App Mobile Flutter](#3-frontend-app-mobile-flutter)
4. [Backend - Laravel API](#4-backend-laravel-api)
5. [Database MySQL](#5-database-mysql)
6. [Servizi Esterni](#6-servizi-esterni)
7. [Infrastruttura e Hosting](#7-infrastruttura-e-hosting)
8. [Flussi Funzionali Principali](#8-flussi-funzionali-principali)
9. [API Endpoints Reference](#9-api-endpoints-reference)
10. [Sicurezza e Privacy](#10-sicurezza-e-privacy)
11. [Manutenzione e Deploy](#11-manutenzione-e-deploy)
12. [Troubleshooting Comune](#12-troubleshooting-comune)

---

## 1. PANORAMICA GENERALE

### 1.1 Cos'è WodVision

**WodVision** è un'applicazione mobile iOS/Android per l'analisi video di esercizi CrossFit tramite intelligenza artificiale.

**Funzionalità Principali**:
- Upload video esercizi CrossFit da camera o galleria
- Analisi AI movimento e postura
- Report dettagliati con score e feedback
- Storico attività e progressi
- Sistema abbonamenti (Free, Premium, Pro)
- Notifiche push
- Sistema referral

**Target Utenti**: Atleti CrossFit, personal trainer, box CrossFit

---

## 2. ARCHITETTURA SISTEMA

### 2.1 Diagramma Architettura Completa

```
┌─────────────────────────────────────────────────────────────────┐
│                    UTENTI FINALI                                │
│              (iOS App + Android App)                            │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 │ HTTPS
                 │
    ┌────────────┴────────────────────────────────────┐
    │                                                  │
    ▼                                                  ▼
┌─────────────────────────┐              ┌──────────────────────────┐
│   FRONTEND (Flutter)    │              │   FIREBASE SERVICES      │
│                         │              │   Project: wodvision-52d46│
│ Repository:             │              ├──────────────────────────┤
│ github.com/TeoFil795/   │              │ - Cloud Messaging (FCM)  │
│ WodVision2              │              │ - Analytics              │
│                         │              │ - Crashlytics            │
│ Tech Stack:             │              │ - Storage (video/files)  │
│ - Dart 3.5.3+           │              └──────────────────────────┘
│ - Flutter               │
│ - Provider (state mgmt) │
└────────────┬────────────┘
             │
             │ REST API (HTTPS)
             │ Base URL: https://admin.wodvision.app/
             │
             ▼
┌──────────────────────────────────────────────────────────────────┐
│             BACKEND API (Laravel 11 PHP)                         │
│                                                                  │
│ Server: DigitalOcean Droplet                                    │
│ IP: 64.226.127.138                                              │
│ Domain: admin.wodvision.app                                     │
│                                                                  │
│ Repository: gitlab.anviam.com/php/crossfit                      │
│                                                                  │
│ Location: /var/www/html/crossfit/                              │
│                                                                  │
│ Tech Stack:                                                     │
│ ├─ Laravel 11 (PHP Framework)                                  │
│ ├─ Nginx (Web Server)                                          │
│ ├─ MySQL 8.x (Database)                                        │
│ ├─ Ubuntu 24.10 (OS)                                           │
│ └─ Composer (Dependency Manager)                               │
└────────────┬─────────────────────────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────────────────────────┐
│                    DATABASE MYSQL                                │
│                                                                  │
│ Host: 127.0.0.1 (locale sulla VM)                               │
│ Database: wodvision                                             │
│ User: app_user                                                  │
│                                                                  │
│ Tabelle: 30 tabelle                                             │
│ ├─ users (utenti registrati)                                   │
│ ├─ journeys (video analizzati)                                 │
│ ├─ subscriptions (piani abbonamento)                           │
│ ├─ transactions (pagamenti)                                    │
│ └─ ... (vedi sezione Database)                                 │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                    SERVIZI ESTERNI                               │
├──────────────────────────────────────────────────────────────────┤
│ - Stripe (Pagamenti) - Modalità TEST                            │
│ - Brevo SMTP (Email transazionali)                              │
│ - Firebase Services (notifiche, analytics, storage)             │
└──────────────────────────────────────────────────────────────────┘
```

### 2.2 Flusso Dati Principale

```
1. USER (Mobile App)
   ↓
2. UPLOAD VIDEO → Firebase Storage
   ↓
3. API CALL → Backend Laravel (admin.wodvision.app)
   ↓
4. ELABORAZIONE AI → Analisi movimento
   ↓
5. SALVATAGGIO → MySQL Database
   ↓
6. NOTIFICA PUSH → Firebase Cloud Messaging
   ↓
7. USER riceve notifica e visualizza report
```

---

## 3. FRONTEND - APP MOBILE FLUTTER

### 3.1 Informazioni Repository

**Repository GitHub**: `https://github.com/TeoFil795/WodVision2`
**Location Locale**: `C:\Users\kogy9\Desktop\WodVision2`
**Linguaggio**: Dart 3.5.3+
**Framework**: Flutter

### 3.2 Struttura Directory Frontend

```
WodVision2/
├── lib/
│   ├── main.dart                    # Entry point app
│   ├── firebase_options.dart        # Config Firebase
│   │
│   ├── core/                        # Servizi core
│   │   ├── api_constant.dart        # Endpoint API
│   │   └── api_service.dart         # HTTP client wrapper
│   │
│   ├── models/                      # Data models (23 file)
│   │   ├── user_model.dart
│   │   ├── journey_model.dart
│   │   ├── subscription_plans_model.dart
│   │   ├── notification_model.dart
│   │   └── ...
│   │
│   ├── providers/                   # State Management (13 provider)
│   │   ├── auth_provider.dart
│   │   ├── home_provider.dart
│   │   ├── user_details_provider.dart
│   │   ├── theme_provider.dart
│   │   └── ...
│   │
│   ├── screens/                     # UI Screens (24 screen)
│   │   ├── authorization_screens/
│   │   │   ├── login_screen.dart
│   │   │   ├── signup_screen.dart
│   │   │   └── verify_account_screen.dart
│   │   ├── home/
│   │   │   ├── home_screen.dart
│   │   │   ├── analyze_and_upload_screen.dart
│   │   │   └── analyzed_report_screen.dart
│   │   └── subscriptionPlan/
│   │
│   ├── widgets/                     # Componenti riutilizzabili
│   │   ├── custom_button.dart
│   │   ├── custom_text_field.dart
│   │   └── ...
│   │
│   ├── resources/                   # Risorse globali
│   │   ├── app_routes.dart          # Sistema routing
│   │   ├── app_strings.dart         # Stringhe
│   │   ├── app_themes.dart          # Temi light/dark
│   │   └── app_assets.dart
│   │
│   └── helpers/                     # Utility
│       ├── shared_preferences_helper.dart
│       ├── notification_controller.dart
│       └── utils.dart
│
├── assets/                          # Asset statici
│   ├── images/                      # Immagini e icone
│   ├── fonts/                       # Font custom
│   └── jsons/                       # Animazioni Lottie
│
├── android/                         # Config Android
│   └── app/
│       ├── build.gradle
│       └── google-services.json
│
├── ios/                             # Config iOS
│   └── Runner/
│
├── pubspec.yaml                     # Dipendenze Flutter
└── README.md
```

### 3.3 Dipendenze Principali Frontend

```yaml
# State Management
provider: ^6.1.2

# Networking
http: ^1.3.0
dio: ^5.8.0

# Firebase
firebase_core: ^3.12.1
firebase_messaging: ^15.2.4
firebase_analytics: ^11.4.4
firebase_crashlytics: ^4.3.4

# Video & Media
video_player: ^2.9.2
chewie: ^1.10.0
image_picker: ^1.1.2
video_compress: ^3.1.4

# Storage
shared_preferences: ^2.5.3
path_provider: ^2.1.5

# UI/UX
google_fonts: ^6.2.1
lottie: ^3.3.1
carousel_slider: ^5.0.0

# Notifiche
awesome_notifications: ^0.10.1

# Pagamenti
flutter_inapp_purchase: ^5.6.1
```

### 3.4 Pattern Architetturale Frontend

**Pattern**: Provider Pattern + MVC modificato

```
SCREEN (UI)
    ↓ Consumer<Provider>
PROVIDER (Business Logic)
    ↓ notifyListeners()
MODEL (Data)
    ↓ toJson() / fromJson()
API SERVICE (HTTP)
```

**Esempio Flusso Login**:
```dart
1. User tap "Login" button in LoginScreen
2. LoginScreen chiama AuthProvider.login(email, password)
3. AuthProvider fa HTTP POST a /api/login tramite ApiService
4. ApiService ritorna response
5. AuthProvider parsea response in UserModel
6. AuthProvider salva token in SharedPreferences
7. AuthProvider chiama notifyListeners()
8. LoginScreen (in ascolto) naviga a HomeScreen
```

### 3.5 Configurazione Firebase Frontend

**Project ID**: `wodvision-52d46`
**Project Number**: `262190625946`

**Android**:
- App ID: `1:262190625946:android:5dc730d24ca98698951b23`
- Package: `com.crossfit.movement`

**iOS**:
- App ID: `1:262190625946:ios:a6984f17675abb5f951b23`
- Bundle ID: `com.crossfit.movement`

**Storage Bucket**: `wodvision-52d46.firebasestorage.app`

### 3.6 Entry Point App

**File**: `lib/main.dart`

**Flusso Inizializzazione**:
```dart
main() async {
  1. WidgetsFlutterBinding.ensureInitialized()
  2. Firebase.initializeApp()
  3. Setup Crashlytics
  4. Get FCM Token
  5. Load token da SharedPreferences
  6. Set orientation (portrait only)
  7. runApp(MyApp())
}

MyApp:
  - MultiProvider setup (13 provider)
  - MaterialApp con routing
  - Tema light/dark
  - Initial route: "/" (GetStartedScreen)
```

**Logica Route Iniziale**:
```dart
if (token exists && isInfoUpdated) {
  → HomeScreen
} else if (token exists && !isInfoUpdated) {
  → UserCompleteDetails
} else {
  → GetStartedScreen
}
```

### 3.7 API Communication

**Base URL**: `https://admin.wodvision.app/`

**ApiService Methods**:
```dart
- get(endpoint, {useToken})
- post(endpoint, body, {useToken})
- put(endpoint, body)
- delete(endpoint)
- multipart(endpoint, file, fieldName, fields)
```

**Autenticazione**: Bearer Token da SharedPreferences

**Esempio chiamata**:
```dart
final response = await ApiService.post(
  'api/login',
  {'email': email, 'password': password},
  useToken: false
);
```

---

## 4. BACKEND - LARAVEL API

### 4.1 Informazioni Repository Backend

**Repository GitLab**: `https://gitlab.anviam.com/php/crossfit`
**Location Server**: `/var/www/html/crossfit/`
**Framework**: Laravel 11
**Linguaggio**: PHP 8.x

### 4.2 Struttura Directory Backend

```
/var/www/html/crossfit/
├── app/
│   ├── Http/
│   │   └── Controllers/
│   │       └── Api/
│   │           ├── UserController.php
│   │           ├── HomeController.php
│   │           ├── JourneyController.php
│   │           ├── SubscriptionController.php
│   │           └── NotificationController.php
│   │
│   ├── Models/
│   │   ├── User.php
│   │   ├── Journey.php
│   │   ├── Subscription.php
│   │   └── ...
│   │
│   └── ...
│
├── config/                          # Configurazioni Laravel
│   ├── database.php
│   ├── mail.php
│   └── services.php
│
├── database/
│   ├── migrations/                  # Schema database
│   └── seeders/
│
├── routes/
│   ├── api.php                      # Route API
│   └── web.php
│
├── public/                          # Document root Nginx
│   └── index.php                    # Entry point
│
├── storage/                         # File storage
│   ├── app/
│   ├── logs/                        # Log Laravel
│   └── framework/
│
├── .env                             # CONFIGURAZIONI SENSIBILI
├── composer.json                    # Dipendenze PHP
├── artisan                          # CLI Laravel
└── vendor/                          # Dipendenze installate
```

### 4.3 Configurazione Backend (.env)

**File**: `/var/www/html/crossfit/.env`

```env
APP_NAME=WODVision!
APP_ENV=local
APP_URL=http://64.226.127.138
APP_DEBUG=true

# Database MySQL
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=wodvision
DB_USERNAME=app_user
DB_PASSWORD=your_database_password_here

# Email (Brevo SMTP)
MAIL_MAILER=smtp
MAIL_HOST=smtp-relay.brevo.com
MAIL_PORT=2525
MAIL_USERNAME=your_smtp_username@smtp-brevo.com
MAIL_PASSWORD=your_smtp_password_here
MAIL_FROM_ADDRESS=admin@wodvision.app

# Stripe Payments (TEST MODE)
STRIPE_KEY=pk_test_YOUR_STRIPE_PUBLIC_KEY_HERE
STRIPE_SECRET=sk_test_YOUR_STRIPE_SECRET_KEY_HERE

# Session & Cache
SESSION_DRIVER=database
CACHE_STORE=database
QUEUE_CONNECTION=database
```

### 4.4 Controllers Backend

#### UserController.php
**Responsabilità**: Autenticazione, gestione profilo utente

**Metodi principali**:
```php
- register(Request)           # Registrazione nuovo utente
- login(Request)              # Login + JWT token
- verifyOtp(Request)          # Verifica OTP email
- resendOtp(Request)          # Rimanda OTP
- forgotPassword(Request)     # Reset password
- createNewPassword(Request)  # Crea nuova password
- submitUserInfo(Request)     # Completa profilo (età, peso, ecc.)
- profile()                   # Get profilo utente
- updateProfilePic(Request)   # Cambia foto profilo
- deleteAccount()             # Elimina account
- logout()                    # Logout
```

#### HomeController.php
**Responsabilità**: Dati homepage, journey, tipi movimento

**Metodi principali**:
```php
- homeData()                  # Dati dashboard home
- getAllJourneys()            # Lista tutti i journey utente
- getJourneyById($id)         # Dettaglio journey + report AI
- getMovementTypes()          # Lista tipi movimento (squat, deadlift, ecc.)
- getAllBodyParts()           # Parti del corpo per analisi
- handleReferral($code)       # Gestione codice referral
```

#### JourneyController.php
**Responsabilità**: Upload video, analisi AI, gestione journey

**Metodi principali**:
```php
- uploadChunk(Request)        # Upload chunk video (multipart)
- getJourneys()               # Lista journey con filtri
- clearHistory()              # Cancella storico
```

**Nota**: Upload video usa chunking per file grandi (progress bar)

#### SubscriptionController.php
**Responsabilità**: Piani abbonamento, pagamenti Stripe

**Metodi principali**:
```php
- index()                     # Lista piani disponibili
- show($id)                   # Dettaglio piano
- addCard(Request)            # Aggiungi carta credito
- getCards()                  # Lista carte salvate
- createSubscription(Request) # Crea abbonamento Stripe
```

#### NotificationController.php
**Responsabilità**: Gestione notifiche push

**Metodi principali**:
```php
- getNotifications()          # Lista notifiche utente
- markAsRead($id)             # Segna notifica come letta
- markAllAsRead()             # Segna tutte come lette
- deleteNotification($id)     # Elimina notifica
- notificationPreference()    # Preferenze notifiche
- checkAppVersion()           # Check update app disponibili
```

### 4.5 Autenticazione Backend

**Sistema**: Laravel Sanctum (Token-based)

**Flow**:
```
1. POST /api/login
   Body: {email, password}

2. Backend verifica credenziali

3. Response:
   {
     "token": "1|abc123...",
     "user": {...}
   }

4. App salva token in SharedPreferences

5. Ogni richiesta successiva:
   Headers: {
     "Authorization": "Bearer 1|abc123..."
   }
```

**Middleware**: `auth.sanctum.api` su route protette

### 4.6 Rate Limiting

**Configurazione**: 60 richieste/minuto per IP o User ID

```php
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)
        ->by(optional($request->user())->id ?: $request->ip());
});
```

**Eccezione**: `upload-chunk` NO rate limiting (file grandi)

---

## 5. DATABASE MYSQL

### 5.1 Informazioni Database

**Host**: `127.0.0.1` (locale sulla VM)
**Port**: `3306`
**Database**: `wodvision`
**User**: `app_user`
**Password**: `ADFRhjdk98hdj`

**Accesso**:
```bash
mysql -u app_user -pADFRhjdk98hdj wodvision
```

### 5.2 Schema Database Completo

**Totale Tabelle**: 30

#### Tabelle Principali

##### users
**Descrizione**: Utenti registrati

**Campi chiave**:
```sql
- id (PK)
- name
- email (unique)
- password (hashed)
- phone
- profile_image
- age
- weight
- height
- gender (male/female)
- goal (lose_weight, gain_muscle, improve_performance)
- activity_level
- referral_code (unique)
- is_verified (boolean)
- email_verified_at
- created_at
- updated_at
```

##### journeys
**Descrizione**: Video analizzati e risultati AI

**Campi chiave**:
```sql
- id (PK)
- user_id (FK → users)
- movement_type_id (FK → movement_types)
- video_url (Firebase Storage URL)
- thumbnail_url
- analyzed_video_url (video con overlay AI)
- status (pending, processing, completed, failed)
- overall_score (0-100)
- ai_feedback (JSON con analisi dettagliata)
- duration (secondi)
- file_size
- created_at
- updated_at
```

**Esempio ai_feedback JSON**:
```json
{
  "overall_score": 85,
  "body_parts": [
    {
      "name": "Knees",
      "score": 90,
      "feedback": "Good alignment throughout the movement"
    },
    {
      "name": "Back",
      "score": 80,
      "feedback": "Minor rounding at bottom position"
    }
  ],
  "suggestions": [
    "Focus on keeping chest up",
    "Improve ankle mobility"
  ]
}
```

##### journey_responses
**Descrizione**: Risposte dettagliate analisi AI per ogni journey

**Campi chiave**:
```sql
- id (PK)
- journey_id (FK → journeys)
- body_part_id (FK → body_parts)
- score (0-100)
- feedback (text)
- created_at
```

##### subscriptions
**Descrizione**: Piani abbonamento disponibili

**Campi chiave**:
```sql
- id (PK)
- name (Free, Premium, Pro)
- description
- price (decimal)
- billing_period (monthly, yearly)
- stripe_price_id
- max_videos_per_month
- features (JSON)
- is_active (boolean)
- created_at
- updated_at
```

**Esempio features JSON**:
```json
{
  "max_videos": -1,  // -1 = unlimited
  "advanced_analysis": true,
  "download_reports": true,
  "priority_support": true,
  "no_ads": true
}
```

##### user_subscriptions
**Descrizione**: Abbonamenti utenti attivi

**Campi chiave**:
```sql
- id (PK)
- user_id (FK → users)
- subscription_id (FK → subscriptions)
- stripe_subscription_id
- status (active, canceled, expired, past_due)
- starts_at
- ends_at
- canceled_at
- created_at
- updated_at
```

##### transactions
**Descrizione**: Storico pagamenti

**Campi chiave**:
```sql
- id (PK)
- user_id (FK → users)
- subscription_id (FK → subscriptions)
- stripe_payment_intent_id
- amount
- currency (USD)
- status (succeeded, failed, pending)
- payment_method
- created_at
```

##### notifications
**Descrizione**: Notifiche push inviate

**Campi chiave**:
```sql
- id (PK)
- user_id (FK → users)
- title
- body
- type (journey_completed, subscription_expiring, general)
- data (JSON payload)
- read_at (timestamp)
- created_at
```

##### movement_types
**Descrizione**: Tipi di esercizi CrossFit

**Campi chiave**:
```sql
- id (PK)
- name (Squat, Deadlift, Snatch, Clean, ecc.)
- description
- icon_url
- is_active
```

**Esempi**:
- Back Squat
- Front Squat
- Overhead Squat
- Deadlift
- Clean & Jerk
- Snatch
- Pull-up
- Handstand Push-up

##### body_parts
**Descrizione**: Parti del corpo analizzate

**Campi chiave**:
```sql
- id (PK)
- name (Knees, Back, Hips, Shoulders, ecc.)
- description
```

##### sub_body_parts
**Descrizione**: Sotto-parti corpo (dettaglio)

**Campi chiave**:
```sql
- id (PK)
- body_part_id (FK → body_parts)
- name
```

##### body_injuries
**Descrizione**: Infortuni corpo (per feedback personalizzato)

**Campi chiave**:
```sql
- id (PK)
- body_part_id (FK → body_parts)
- name
- description
```

##### referrals
**Descrizione**: Sistema referral utenti

**Campi chiave**:
```sql
- id (PK)
- referrer_id (FK → users, chi ha invitato)
- referred_id (FK → users, chi è stato invitato)
- reward_amount
- status (pending, completed)
- created_at
```

##### fcm_tokens
**Descrizione**: Token Firebase Cloud Messaging per notifiche push

**Campi chiave**:
```sql
- id (PK)
- user_id (FK → users)
- token (FCM token string)
- device_type (ios, android)
- is_active
- created_at
- updated_at
```

##### app_versions
**Descrizione**: Versioni app per force update

**Campi chiave**:
```sql
- id (PK)
- platform (ios, android)
- version (es. 1.0.6)
- build_number (es. 7)
- force_update (boolean)
- message
- created_at
```

##### chunk_uploads_logs
**Descrizione**: Log upload chunk video

**Campi chiave**:
```sql
- id (PK)
- user_id (FK → users)
- filename
- chunk_index
- total_chunks
- status (uploading, completed, failed)
- created_at
```

##### Altre Tabelle Laravel

- **cache**: Cache applicazione
- **cache_locks**: Lock per cache
- **sessions**: Sessioni utente
- **jobs**: Queue job asincroni
- **job_batches**: Batch di job
- **failed_jobs**: Job falliti
- **personal_access_tokens**: Token Sanctum
- **password_reset_tokens**: Reset password
- **migrations**: Storico migrazioni DB

### 5.3 Relazioni Database

```
users (1) ──< (N) journeys
users (1) ──< (N) user_subscriptions
users (1) ──< (N) notifications
users (1) ──< (N) transactions
users (1) ──< (N) fcm_tokens

journeys (N) ─> (1) movement_types
journeys (1) ──< (N) journey_responses

journey_responses (N) ─> (1) body_parts

subscriptions (1) ──< (N) user_subscriptions
subscriptions (1) ──< (N) transactions

body_parts (1) ──< (N) sub_body_parts
body_parts (1) ──< (N) body_injuries

referrals (N) ─> (1) users (referrer)
referrals (N) ─> (1) users (referred)
```

### 5.4 Backup Database

**Comando Backup**:
```bash
# Full backup
mysqldump -u app_user -pADFRhjdk98hdj wodvision > wodvision_backup_$(date +%Y%m%d).sql

# Solo struttura (no data)
mysqldump -u app_user -pADFRhjdk98hdj --no-data wodvision > wodvision_schema.sql

# Solo dati users e subscriptions
mysqldump -u app_user -pADFRhjdk98hdj wodvision users subscriptions > critical_data.sql
```

**Restore**:
```bash
mysql -u app_user -pADFRhjdk98hdj wodvision < wodvision_backup_20260111.sql
```

**Automazione Backup** (Cron Job):
```bash
# Aggiungere a crontab: crontab -e
0 2 * * * mysqldump -u app_user -pADFRhjdk98hdj wodvision > /backup/wodvision_$(date +\%Y\%m\%d).sql
```

---

## 6. SERVIZI ESTERNI

### 6.1 Firebase

**Project ID**: `wodvision-52d46`
**Console**: https://console.firebase.google.com/project/wodvision-52d46

**Servizi Attivi**:

#### Cloud Messaging (FCM)
**Uso**: Notifiche push iOS/Android

**Flow**:
```
1. App ottiene FCM token al primo avvio
2. Token salvato in tabella fcm_tokens
3. Backend invia notifica quando journey completato:
   POST https://fcm.googleapis.com/fcm/send
   Body: {
     "to": "<fcm_token>",
     "notification": {
       "title": "Video Analysis Complete",
       "body": "Your Squat analysis is ready!"
     }
   }
4. App riceve notifica e naviga a report
```

#### Firebase Storage
**Bucket**: `wodvision-52d46.firebasestorage.app`

**Uso**: Storage video caricati utenti

**Struttura**:
```
/videos/
  /user_123/
    /journey_456/
      original.mp4
      analyzed.mp4
      thumbnail.jpg
```

**Regole Sicurezza**:
```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /videos/{userId}/{journeyId}/{file} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

#### Analytics
**Uso**: Tracciamento comportamento utenti

**Eventi tracciati**:
- `screen_view` (visualizzazione screen)
- `video_upload` (upload video)
- `subscription_purchase` (acquisto abbonamento)
- `journey_completed` (analisi completata)

#### Crashlytics
**Uso**: Monitoring crash app

**Integrazione**: Automatica via FlutterFire

### 6.2 Stripe (Pagamenti)

**Modalità**: TEST (da migrare a LIVE)

**API Keys**:
- Publishable Key: `pk_test_51NFVXUSGCpj8igx3...`
- Secret Key: `sk_test_51NFVXUSGCpj8igx3...`

**Dashboard**: https://dashboard.stripe.com/

**Prodotti Configurati**:
1. **Premium Monthly** - $9.99/mese
2. **Pro Monthly** - $19.99/mese
3. **Premium Yearly** - $99.99/anno

**Flow Abbonamento**:
```
1. User seleziona piano in app
2. App chiama POST /api/payments/add-card
   → Backend crea Stripe Customer
   → Salva carta con Stripe.js

3. App chiama POST /api/payments/create-subscription
   Body: {subscription_id: 2}

4. Backend:
   - Crea Stripe Subscription
   - Salva in user_subscriptions
   - Salva transaction in transactions

5. Response: {
     "subscription": {...},
     "status": "active"
   }

6. App naviga a success screen
```

**Webhook Stripe** (da configurare):
```
Endpoint: https://admin.wodvision.app/api/stripe/webhook
Eventi:
- customer.subscription.deleted
- invoice.payment_succeeded
- invoice.payment_failed
```

### 6.3 Brevo (Email SMTP)

**Uso**: Email transazionali

**Configurazione**:
- Host: `smtp-relay.brevo.com`
- Port: `2525`
- Username: `887b8d003@smtp-brevo.com`
- Password: `2yOC5WFf7pJA4R93`

**Email Inviate**:
1. **Welcome Email** (post registrazione)
2. **OTP Verification** (codice verifica)
3. **Password Reset** (link reset)
4. **Subscription Confirmation** (conferma abbonamento)
5. **Journey Completed** (analisi pronta)

**Template Email** (in Laravel):
```php
Mail::to($user->email)->send(new JourneyCompletedMail($journey));
```

---

## 7. INFRASTRUTTURA E HOSTING

### 7.1 DigitalOcean Droplet

**Provider**: DigitalOcean
**Tipo**: Virtual Machine (Droplet)
**IP Pubblico**: `64.226.127.138`
**Dominio**: `admin.wodvision.app` → `64.226.127.138`

**Specifica VM**:
- CPU: 1 vCPU Intel
- RAM: 2 GB
- Storage: 70 GB SSD
- Bandwidth: Probabilmente 2 TB/mese
- Regione: Frankfurt (fra1)
- OS: Ubuntu 24.10 (GNU/Linux 6.11.0-29-generic x86_64)

**Hostname**: `ubuntu-s-1vcpu-2gb-70gb-intel-fra1-01`

**Costo Stimato**: ~$18-24/mese

### 7.2 Stack Software VM

```
┌─────────────────────────────────────┐
│         Ubuntu 24.10                │
├─────────────────────────────────────┤
│ Nginx (Web Server)                  │
│  ↓ proxy_pass                       │
│ PHP-FPM 8.x                         │
│  ↓ execute                          │
│ Laravel 11 Application              │
│  ↓ query                            │
│ MySQL 8.x Database                  │
└─────────────────────────────────────┘
```

**Software Installato**:
- Nginx (web server)
- PHP 8.x + PHP-FPM
- MySQL 8.x
- Composer (PHP dependency manager)
- Git (version control)

### 7.3 Configurazione Nginx

**File**: `/etc/nginx/sites-enabled/default`

**Configurazione Base** (stimata):
```nginx
server {
    listen 80;
    server_name admin.wodvision.app 64.226.127.138;

    root /var/www/html/crossfit/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

**Note**:
- NO HTTPS configurato (solo HTTP)
- **SECURITY RISK**: Dati in chiaro su rete

### 7.4 Accesso SSH

**Comando**:
```bash
ssh root@64.226.127.138
```

**Credenziali**: Password resettabile da DigitalOcean Console

**Alternative**:
- DigitalOcean Web Console (browser)
- SSH Key (da configurare)

### 7.5 DNS Configuration

**Domain**: `wodvision.app`
**Subdomain**: `admin.wodvision.app`

**Record DNS** (stimato):
```
Type: A
Name: admin
Value: 64.226.127.138
TTL: 3600
```

**Gestione DNS**: Da verificare (DigitalOcean, Cloudflare, Namecheap?)

### 7.6 Monitoring e Logs

**Laravel Logs**:
```bash
# Location
/var/www/html/crossfit/storage/logs/laravel.log

# Tail real-time
tail -f /var/www/html/crossfit/storage/logs/laravel.log

# Ultimi errori
grep ERROR /var/www/html/crossfit/storage/logs/laravel.log | tail -20
```

**Nginx Logs**:
```bash
# Access log
tail -f /var/log/nginx/access.log

# Error log
tail -f /var/log/nginx/error.log
```

**MySQL Logs**:
```bash
# Error log
tail -f /var/log/mysql/error.log
```

**System Resources**:
```bash
# CPU e RAM
htop

# Disk usage
df -h

# Traffico rete
iftop
```

---

## 8. FLUSSI FUNZIONALI PRINCIPALI

### 8.1 Registrazione Nuovo Utente

```
┌─────────────┐
│    USER     │
└──────┬──────┘
       │
       │ 1. Tap "Sign Up"
       ▼
┌─────────────────────────┐
│  SignupScreen (Flutter) │
│  Campi:                 │
│  - Name                 │
│  - Email                │
│  - Password             │
│  - Confirm Password     │
└──────┬──────────────────┘
       │
       │ 2. POST /api/register
       │    {name, email, password}
       ▼
┌─────────────────────────┐
│ Backend Laravel         │
│ UserController::register│
│                         │
│ 1. Validate input       │
│ 2. Hash password        │
│ 3. Generate OTP (6 digit)│
│ 4. Create user (unverified)│
│ 5. Send OTP email       │
│ 6. Return response      │
└──────┬──────────────────┘
       │
       │ 3. Response: {user, otp_sent: true}
       ▼
┌─────────────────────────┐
│ VerifyAccountScreen     │
│ Input: 6-digit OTP      │
└──────┬──────────────────┘
       │
       │ 4. POST /api/verify-otp
       │    {email, otp}
       ▼
┌─────────────────────────┐
│ Backend                 │
│ UserController::verifyOtp│
│                         │
│ 1. Check OTP correct    │
│ 2. Mark user verified   │
│ 3. Generate auth token  │
│ 4. Return token + user  │
└──────┬──────────────────┘
       │
       │ 5. Response: {token, user}
       ▼
┌─────────────────────────┐
│ App saves:              │
│ - Token (SharedPrefs)   │
│ - User (SharedPrefs)    │
│ - isInfoUpdated = false │
└──────┬──────────────────┘
       │
       │ 6. Navigate to
       ▼
┌─────────────────────────┐
│ UserCompleteDetailsScreen│
│ Campi:                  │
│ - Gender                │
│ - Age                   │
│ - Weight                │
│ - Height                │
│ - Goal                  │
│ - Activity Level        │
└──────┬──────────────────┘
       │
       │ 7. POST /api/user-info-submit
       ▼
┌─────────────────────────┐
│ Backend                 │
│ Update user record      │
└──────┬──────────────────┘
       │
       │ 8. Set isInfoUpdated = true
       ▼
┌─────────────────────────┐
│ SubscriptionPlansScreen │
│ Scelta piano:           │
│ - Free                  │
│ - Premium ($9.99/mo)    │
│ - Pro ($19.99/mo)       │
└──────┬──────────────────┘
       │
       │ 9. Se Free → Skip payment
       │    Se Premium/Pro → Payment
       ▼
┌─────────────────────────┐
│      HomeScreen         │
└─────────────────────────┘
```

### 8.2 Login Utente Esistente

```
┌─────────────┐
│  LoginScreen│
│  - Email    │
│  - Password │
└──────┬──────┘
       │
       │ POST /api/login
       ▼
┌─────────────────────────┐
│ Backend                 │
│ UserController::login   │
│                         │
│ 1. Find user by email   │
│ 2. Verify password hash │
│ 3. Check is_verified    │
│ 4. Generate token       │
│ 5. Get FCM token        │
│ 6. Update fcm_tokens    │
└──────┬──────────────────┘
       │
       │ Response: {token, user}
       ▼
┌─────────────────────────┐
│ App:                    │
│ - Save token            │
│ - Save user             │
│ - Navigate to HomeScreen│
└─────────────────────────┘
```

### 8.3 Upload e Analisi Video

```
┌─────────────┐
│  HomeScreen │
│             │
│ [+ Upload]  │
└──────┬──────┘
       │
       │ Tap Upload
       ▼
┌─────────────────────────┐
│ FileSelectionScreen     │
│ Opzioni:                │
│ - 📷 Camera             │
│ - 🖼️ Gallery            │
└──────┬──────────────────┘
       │
       │ User seleziona Camera
       ▼
┌─────────────────────────┐
│ ImagePicker.pickVideo() │
│ → Registra video        │
└──────┬──────────────────┘
       │
       │ Video file selected
       ▼
┌─────────────────────────┐
│ AnalyzeAndUploadScreen  │
│                         │
│ 1. Preview video        │
│ 2. Seleziona:           │
│    - Movement Type      │
│      (Squat, Deadlift)  │
│    - Body Part Focus    │
│      (Knees, Back, Hips)│
└──────┬──────────────────┘
       │
       │ 3. Tap "Analyze"
       ▼
┌─────────────────────────┐
│ Video Processing        │
│                         │
│ 1. Compress video       │
│    (VideoCompress pkg)  │
│ 2. Generate thumbnail   │
│ 3. Split in chunks      │
│    (es. 5MB per chunk)  │
└──────┬──────────────────┘
       │
       │ 4. Upload chunks sequentially
       ▼
┌─────────────────────────────────────┐
│ Loop: for each chunk                │
│                                     │
│  POST /api/upload-chunk             │
│  Headers: {Authorization: Bearer}   │
│  Body (multipart):                  │
│    - video_chunk (file)             │
│    - chunk_index (int)              │
│    - total_chunks (int)             │
│    - movement_type_id (int)         │
│    - body_part_id (int)             │
│    - filename (string)              │
│                                     │
│  Backend:                           │
│    1. Save chunk to temp storage    │
│    2. Log in chunk_uploads_logs     │
│    3. If last chunk:                │
│       - Merge all chunks            │
│       - Upload to Firebase Storage  │
│       - Create journey record       │
│       - Trigger AI analysis (queue) │
└──────┬──────────────────────────────┘
       │
       │ 5. All chunks uploaded
       ▼
┌─────────────────────────┐
│ UploadSuccessScreen     │
│ "Video uploaded!        │
│  Analysis in progress..." │
└──────┬──────────────────┘
       │
       │ Backend Queue Job
       ▼
┌─────────────────────────┐
│ AI Analysis Process     │
│ (Job asincrono)         │
│                         │
│ 1. Download video       │
│ 2. Process con AI model │
│ 3. Extract keypoints    │
│ 4. Calculate scores     │
│ 5. Generate feedback    │
│ 6. Create analyzed video│
│    (con overlay skeleton)│
│ 7. Upload analyzed video│
│ 8. Update journey:      │
│    - status = completed │
│    - overall_score      │
│    - ai_feedback (JSON) │
│ 9. Create journey_responses│
│ 10. Send push notification│
└──────┬──────────────────┘
       │
       │ FCM Push
       ▼
┌─────────────────────────┐
│ User riceve notifica:   │
│ "Your Squat analysis    │
│  is ready! Score: 85/100"│
└──────┬──────────────────┘
       │
       │ Tap notifica
       ▼
┌─────────────────────────┐
│ AnalyzedReportScreen    │
│                         │
│ 1. GET /api/home/       │
│    get-journeys-by-id/456│
│                         │
│ 2. Display:             │
│    - Video analyzed     │
│    - Overall Score: 85  │
│    - Body parts breakdown:│
│      • Knees: 90 ✓      │
│      • Back: 80 ⚠️      │
│      • Hips: 85 ✓       │
│    - AI Feedback text   │
│    - Suggestions        │
│                         │
│ 3. Actions:             │
│    - Download PDF       │
│    - Share              │
│    - Save to gallery    │
└─────────────────────────┘
```

**Note Tecniche Upload**:
- Chunk size: 5 MB (configurabile)
- Timeout per chunk: 60 secondi
- Retry automatico se fallisce
- Progress bar basato su chunk completati / totali

### 8.4 Acquisto Abbonamento Premium

```
┌─────────────┐
│ ProfileTab  │
│             │
│ [Upgrade]   │
└──────┬──────┘
       │
       │ Tap Upgrade
       ▼
┌─────────────────────────┐
│ SubscriptionPlansScreen │
│                         │
│ ┌─────────────────────┐ │
│ │ FREE                │ │
│ │ $0/month            │ │
│ │ - 5 videos/month    │ │
│ │ - Basic analysis    │ │
│ └─────────────────────┘ │
│                         │
│ ┌─────────────────────┐ │
│ │ PREMIUM ⭐          │ │
│ │ $9.99/month         │ │
│ │ - Unlimited videos  │ │
│ │ - Advanced analysis │ │
│ │ - Download reports  │ │
│ └─────────────────────┘ │
│                         │
│ ┌─────────────────────┐ │
│ │ PRO 🏆              │ │
│ │ $19.99/month        │ │
│ │ - Everything Premium│ │
│ │ - Coaching insights │ │
│ │ - Priority support  │ │
│ └─────────────────────┘ │
└──────┬──────────────────┘
       │
       │ User seleziona Premium
       ▼
┌─────────────────────────┐
│ AddPaymentMethodsScreen │
│                         │
│ Campi carta:            │
│ - Card Number           │
│ - Expiry (MM/YY)        │
│ - CVV                   │
│ - Cardholder Name       │
└──────┬──────────────────┘
       │
       │ POST /api/payments/add-card
       │ {card_number, exp_month, exp_year, cvc}
       ▼
┌─────────────────────────┐
│ Backend                 │
│ SubscriptionController  │
│ ::addCard()             │
│                         │
│ 1. Create Stripe Customer│
│    (se non esiste)      │
│ 2. Tokenize carta       │
│    (Stripe.js)          │
│ 3. Attach payment method│
│ 4. Save customer_id     │
│    in users table       │
└──────┬──────────────────┘
       │
       │ Response: {card_saved: true}
       ▼
┌─────────────────────────┐
│ App mostra:             │
│ "Payment method added"  │
│                         │
│ [Confirm Subscription]  │
└──────┬──────────────────┘
       │
       │ Tap Confirm
       │ POST /api/payments/create-subscription
       │ {subscription_id: 2}  // Premium
       ▼
┌─────────────────────────┐
│ Backend                 │
│ ::createSubscription()  │
│                         │
│ 1. Get Stripe customer  │
│ 2. Get subscription plan│
│    (Premium: $9.99/mo)  │
│ 3. Create Stripe        │
│    Subscription:        │
│    stripe.subscriptions.│
│    create({             │
│      customer: cus_xxx, │
│      items: [{          │
│        price: price_xxx │
│      }]                 │
│    })                   │
│ 4. First payment        │
│    (automatically charged)│
│ 5. Save user_subscription│
│    record               │
│ 6. Save transaction     │
│ 7. Send confirmation email│
└──────┬──────────────────┘
       │
       │ Response: {
       │   subscription: {...},
       │   status: "active"
       │ }
       ▼
┌─────────────────────────┐
│ Success Screen          │
│ "Welcome to Premium! 🎉"│
│                         │
│ Features unlocked:      │
│ ✓ Unlimited videos      │
│ ✓ Advanced AI analysis  │
│ ✓ Download PDF reports  │
└──────┬──────────────────┘
       │
       │ Navigate to HomeScreen
       ▼
┌─────────────────────────┐
│ HomeScreen              │
│ (now with Premium badge)│
└─────────────────────────┘
```

**Gestione Rinnovi Automatici**:
```
Stripe Webhook → https://admin.wodvision.app/api/stripe/webhook

Eventi:
1. invoice.payment_succeeded
   → Update user_subscription (extend ends_at)
   → Create transaction record
   → Send receipt email

2. invoice.payment_failed
   → Update status = "past_due"
   → Send payment failed email
   → Retry payment (Stripe automatic)

3. customer.subscription.deleted
   → Update status = "canceled"
   → Send cancellation email
```

### 8.5 Sistema Notifiche Push

```
┌─────────────────────────┐
│ Backend Event           │
│ (es. Journey completed) │
└──────┬──────────────────┘
       │
       │ 1. Trigger notification
       ▼
┌─────────────────────────┐
│ NotificationController  │
│ ::sendPushNotification()│
│                         │
│ 1. Get user FCM token   │
│    from fcm_tokens table│
│ 2. Create notification  │
│    record in DB         │
│ 3. Prepare FCM payload  │
└──────┬──────────────────┘
       │
       │ 2. POST to FCM API
       │    https://fcm.googleapis.com/fcm/send
       ▼
┌─────────────────────────┐
│ Firebase Cloud Messaging│
│                         │
│ Payload:                │
│ {                       │
│   "to": "<fcm_token>",  │
│   "notification": {     │
│     "title": "Analysis  │
│              Complete", │
│     "body": "Your Squat │
│             is ready!"  │
│   },                    │
│   "data": {             │
│     "journey_id": "456",│
│     "type": "journey_   │
│             completed"  │
│   }                     │
│ }                       │
└──────┬──────────────────┘
       │
       │ 3. FCM delivers to device
       ▼
┌─────────────────────────┐
│ Mobile App (Flutter)    │
│ FirebaseMessaging       │
│ .onMessage.listen()     │
│                         │
│ if (app in foreground): │
│   - Show local notif    │
│     (AwesomeNotifications)│
│                         │
│ if (app in background): │
│   - System notification │
│   - onMessageOpenedApp  │
│     when tapped         │
└──────┬──────────────────┘
       │
       │ 4. User taps notification
       ▼
┌─────────────────────────┐
│ App Navigation          │
│                         │
│ if (type == "journey_   │
│     completed"):        │
│   Navigate to           │
│   AnalyzedReportScreen  │
│   (journey_id: 456)     │
│                         │
│ if (type == "subscription│
│     _expiring"):        │
│   Navigate to           │
│   SubscriptionScreen    │
└─────────────────────────┘
```

**Tipi Notifiche**:
1. `journey_completed` - Analisi video completata
2. `subscription_expiring` - Abbonamento in scadenza (7 giorni prima)
3. `payment_failed` - Pagamento fallito
4. `referral_reward` - Ricompensa referral guadagnata
5. `app_update` - Nuova versione disponibile
6. `general` - Notifiche marketing/news

---

## 9. API ENDPOINTS REFERENCE

### 9.1 Base URL

```
https://admin.wodvision.app/api/
```

### 9.2 Autenticazione

**Tutte le route protette richiedono**:
```
Headers: {
  "Authorization": "Bearer <token>",
  "Accept": "application/json"
}
```

### 9.3 Endpoints Pubblici (no auth)

#### POST /register
**Descrizione**: Registrazione nuovo utente

**Body**:
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "SecurePass123",
  "password_confirmation": "SecurePass123"
}
```

**Response 200**:
```json
{
  "success": true,
  "message": "OTP sent to your email",
  "user": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "is_verified": false
  }
}
```

#### POST /login
**Descrizione**: Login utente

**Body**:
```json
{
  "email": "john@example.com",
  "password": "SecurePass123",
  "fcm_token": "dGhpc19pc19hX2ZjbV90b2tlbg=="
}
```

**Response 200**:
```json
{
  "success": true,
  "token": "1|abc123def456...",
  "user": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "profile_image": "https://...",
    "subscription_type": "premium"
  }
}
```

#### POST /verify-otp
**Descrizione**: Verifica codice OTP

**Body**:
```json
{
  "email": "john@example.com",
  "otp": "123456"
}
```

**Response 200**:
```json
{
  "success": true,
  "message": "Account verified",
  "token": "1|abc123...",
  "user": {...}
}
```

#### POST /resend-otp
**Descrizione**: Rimanda OTP

**Body**:
```json
{
  "email": "john@example.com"
}
```

#### POST /forgot-password
**Descrizione**: Richiesta reset password

**Body**:
```json
{
  "email": "john@example.com"
}
```

#### POST /create-new-password
**Descrizione**: Imposta nuova password dopo reset

**Body**:
```json
{
  "email": "john@example.com",
  "token": "reset_token_from_email",
  "password": "NewSecurePass123",
  "password_confirmation": "NewSecurePass123"
}
```

### 9.4 Endpoints Autenticati - User

#### GET /profile
**Descrizione**: Ottieni profilo utente corrente

**Response 200**:
```json
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "profile_image": "https://...",
  "age": 30,
  "weight": 80.5,
  "height": 180,
  "gender": "male",
  "goal": "improve_performance",
  "activity_level": "very_active",
  "referral_code": "JOHN123",
  "subscription": {
    "type": "premium",
    "status": "active",
    "ends_at": "2026-02-11"
  }
}
```

#### POST /user-info-submit
**Descrizione**: Completa/aggiorna profilo utente

**Body**:
```json
{
  "age": 30,
  "weight": 80.5,
  "height": 180,
  "gender": "male",
  "goal": "improve_performance",
  "activity_level": "very_active"
}
```

#### POST /profile/update-profile-image
**Descrizione**: Cambia foto profilo

**Body** (multipart/form-data):
```
profile_image: <file>
```

#### POST /logout
**Descrizione**: Logout (invalida token)

**Response 200**:
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

#### GET /delete-account
**Descrizione**: Elimina account (GDPR compliance)

**Response 200**:
```json
{
  "success": true,
  "message": "Account deleted successfully"
}
```

**Note**: Elimina user + tutti dati correlati (journeys, subscriptions, ecc.)

### 9.5 Endpoints Autenticati - Home & Journeys

#### GET /home
**Descrizione**: Dati dashboard home

**Response 200**:
```json
{
  "user": {...},
  "recent_activities": [
    {
      "id": 456,
      "movement_type": "Back Squat",
      "overall_score": 85,
      "created_at": "2026-01-10T15:30:00Z",
      "thumbnail_url": "https://..."
    }
  ],
  "stats": {
    "total_videos": 24,
    "avg_score": 82.5,
    "videos_this_month": 5
  }
}
```

#### GET /home/get-all-journeys
**Descrizione**: Lista tutti journey utente (con paginazione)

**Query Params**:
- `page` (default: 1)
- `per_page` (default: 20)
- `movement_type_id` (filtro opzionale)

**Response 200**:
```json
{
  "data": [
    {
      "id": 456,
      "movement_type": {
        "id": 1,
        "name": "Back Squat"
      },
      "overall_score": 85,
      "status": "completed",
      "created_at": "2026-01-10T15:30:00Z",
      "thumbnail_url": "https://...",
      "video_url": "https://..."
    }
  ],
  "pagination": {
    "current_page": 1,
    "total_pages": 3,
    "total": 48
  }
}
```

#### GET /home/get-journeys-by-id/{id}
**Descrizione**: Dettaglio journey con report completo

**Response 200**:
```json
{
  "id": 456,
  "movement_type": "Back Squat",
  "overall_score": 85,
  "video_url": "https://firebase.../original.mp4",
  "analyzed_video_url": "https://firebase.../analyzed.mp4",
  "thumbnail_url": "https://firebase.../thumb.jpg",
  "duration": 15,
  "created_at": "2026-01-10T15:30:00Z",
  "ai_feedback": {
    "overall": "Good form with minor improvements needed",
    "strengths": [
      "Consistent depth",
      "Good bar path"
    ],
    "weaknesses": [
      "Slight knee valgus at bottom",
      "Could improve hip drive"
    ]
  },
  "body_parts_analysis": [
    {
      "body_part": "Knees",
      "score": 90,
      "feedback": "Excellent tracking, minimal valgus"
    },
    {
      "body_part": "Back",
      "score": 80,
      "feedback": "Minor rounding at bottom position"
    },
    {
      "body_part": "Hips",
      "score": 85,
      "feedback": "Good depth, improve drive out of hole"
    }
  ]
}
```

#### GET /home/get-movement-types
**Descrizione**: Lista tipi movimento disponibili

**Response 200**:
```json
[
  {
    "id": 1,
    "name": "Back Squat",
    "description": "Barbell back squat",
    "icon_url": "https://.../squat.png"
  },
  {
    "id": 2,
    "name": "Deadlift",
    "description": "Conventional deadlift",
    "icon_url": "https://.../deadlift.png"
  }
]
```

#### GET /home/get-body-parts
**Descrizione**: Lista parti corpo per analisi

**Response 200**:
```json
[
  {
    "id": 1,
    "name": "Knees"
  },
  {
    "id": 2,
    "name": "Back"
  },
  {
    "id": 3,
    "name": "Hips"
  }
]
```

#### POST /upload-chunk
**Descrizione**: Upload chunk video (NO rate limiting)

**Body** (multipart/form-data):
```
video_chunk: <file>
chunk_index: 0
total_chunks: 10
movement_type_id: 1
body_part_id: 2
filename: "squat_video.mp4"
```

**Response 200** (chunk intermedio):
```json
{
  "success": true,
  "message": "Chunk 1/10 uploaded",
  "progress": 10
}
```

**Response 200** (ultimo chunk):
```json
{
  "success": true,
  "message": "Upload complete, analysis started",
  "journey_id": 789
}
```

#### GET /journeys/clear-history
**Descrizione**: Cancella storico journey utente

**Response 200**:
```json
{
  "success": true,
  "message": "History cleared"
}
```

### 9.6 Endpoints Autenticati - Subscriptions

#### GET /subscriptions
**Descrizione**: Lista piani abbonamento disponibili

**Response 200**:
```json
[
  {
    "id": 1,
    "name": "Free",
    "price": 0,
    "billing_period": "monthly",
    "features": {
      "max_videos": 5,
      "advanced_analysis": false,
      "download_reports": false
    }
  },
  {
    "id": 2,
    "name": "Premium",
    "price": 9.99,
    "billing_period": "monthly",
    "stripe_price_id": "price_xxx",
    "features": {
      "max_videos": -1,
      "advanced_analysis": true,
      "download_reports": true
    }
  }
]
```

#### GET /subscriptions/{id}
**Descrizione**: Dettaglio piano specifico

#### GET /profile/user-subscription
**Descrizione**: Abbonamento corrente utente

**Response 200**:
```json
{
  "subscription": {
    "id": 2,
    "name": "Premium",
    "price": 9.99
  },
  "status": "active",
  "starts_at": "2026-01-01",
  "ends_at": "2026-02-01",
  "auto_renew": true,
  "stripe_subscription_id": "sub_xxx"
}
```

#### POST /payments/add-card
**Descrizione**: Aggiungi carta credito

**Body**:
```json
{
  "card_number": "4242424242424242",
  "exp_month": 12,
  "exp_year": 2027,
  "cvc": "123",
  "cardholder_name": "John Doe"
}
```

**Response 200**:
```json
{
  "success": true,
  "message": "Card added successfully",
  "card": {
    "id": "card_xxx",
    "last4": "4242",
    "brand": "Visa"
  }
}
```

#### GET /payments/get-cards
**Descrizione**: Lista carte salvate

**Response 200**:
```json
[
  {
    "id": "card_xxx",
    "brand": "Visa",
    "last4": "4242",
    "exp_month": 12,
    "exp_year": 2027,
    "is_default": true
  }
]
```

#### POST /payments/create-subscription
**Descrizione**: Crea abbonamento

**Body**:
```json
{
  "subscription_id": 2,
  "payment_method_id": "card_xxx"
}
```

**Response 200**:
```json
{
  "success": true,
  "message": "Subscription created",
  "subscription": {
    "id": 456,
    "status": "active",
    "ends_at": "2026-02-11"
  }
}
```

### 9.7 Endpoints Autenticati - Notifications

#### GET /notifications
**Descrizione**: Lista notifiche utente

**Query Params**:
- `page` (default: 1)
- `unread_only` (boolean, default: false)

**Response 200**:
```json
{
  "data": [
    {
      "id": 789,
      "title": "Video Analysis Complete",
      "body": "Your Back Squat analysis is ready! Score: 85/100",
      "type": "journey_completed",
      "data": {
        "journey_id": 456
      },
      "read_at": null,
      "created_at": "2026-01-11T10:30:00Z"
    }
  ],
  "unread_count": 3
}
```

#### GET /notifications/mark-as-read/{id}
**Descrizione**: Segna notifica come letta

#### GET /notifications/mark-all-as-read
**Descrizione**: Segna tutte come lette

#### GET /notifications/delete-notification/{id}
**Descrizione**: Elimina notifica

#### GET /notifications/notification-preference
**Descrizione**: Get/Set preferenze notifiche

**Response 200**:
```json
{
  "email_notifications": true,
  "push_notifications": true,
  "marketing_emails": false
}
```

#### GET /notifications/check-version-update
**Descrizione**: Check se nuova versione app disponibile

**Query Params**:
- `platform` (ios/android)
- `current_version` (es. 1.0.6)
- `current_build` (es. 7)

**Response 200** (update disponibile):
```json
{
  "update_available": true,
  "latest_version": "1.0.7",
  "latest_build": 8,
  "force_update": false,
  "message": "New features and bug fixes available!"
}
```

**Response 200** (nessun update):
```json
{
  "update_available": false
}
```

### 9.8 Referral Endpoints

#### GET /referral/{code}
**Descrizione**: Gestisce link referral (public)

**Esempio**: `https://admin.wodvision.app/api/referral/JOHN123`

**Response**: Redirect a app store con tracking

---

## 10. SICUREZZA E PRIVACY

### 10.1 Vulnerabilità Identificate

#### CRITICHE (da fixare SUBITO)

1. **NO HTTPS sul backend**
   - **Rischio**: Dati utente (password, token, video) in chiaro su rete
   - **Fix**: Configurare SSL/TLS con Let's Encrypt
   ```bash
   # Su VM
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d admin.wodvision.app
   ```

2. **Stripe in modalità TEST**
   - **Rischio**: Pagamenti reali non funzionano
   - **Fix**: Migrare a Stripe Live Mode
   - Cambiare API keys in `.env`
   - Configurare webhook produzione

3. **API Keys pubbliche nel codice**
   - **Rischio**: Chiunque può decompilare APK e rubare keys
   - **File**: `firebase_options.dart`
   - **Fix**: Configurare Firebase App Check

4. **Token JWT in SharedPreferences (plain text)**
   - **Rischio**: Su Android rooted, token leggibile
   - **Fix**: Usare `flutter_secure_storage`

5. **Password database in .env leggibile**
   - **Fix**: Usare variabili ambiente o secrets manager

#### MEDIE

6. **Nessun rate limiting su alcuni endpoint**
   - `upload-chunk` esplicitamente escluso
   - **Rischio**: Possibile DoS attack
   - **Fix**: Implementare rate limiting intelligente

7. **Nessun SSL Certificate Pinning**
   - **Rischio**: Man-in-the-middle attacks
   - **Fix**: Implementare cert pinning in app

8. **Debug mode abilitato in produzione**
   - `APP_DEBUG=true` in `.env`
   - **Rischio**: Stack traces rivelano struttura backend
   - **Fix**: Set `APP_DEBUG=false`

9. **Nessuna validazione file upload lato backend**
   - **Rischio**: Upload file malevoli
   - **Fix**: Validare MIME type, dimensione, scansione virus

#### BASSE

10. **Logs verbosi accessibili**
    - `/var/www/html/crossfit/storage/logs/laravel.log`
    - **Fix**: Rotazione logs, permessi restrittivi

11. **MySQL accessible da localhost only**
    - **Good**: Non esposto a internet
    - **Miglioramento**: Firewall rules esplicite

### 10.2 Checklist Sicurezza Immediata

#### Backend (Laravel)

- [ ] **Abilita HTTPS**
  ```bash
  sudo certbot --nginx -d admin.wodvision.app
  ```

- [ ] **Disabilita Debug Mode**
  ```env
  # .env
  APP_DEBUG=false
  APP_ENV=production
  ```

- [ ] **Cambia Database Password**
  ```bash
  mysql -u root -p
  ALTER USER 'app_user'@'localhost' IDENTIFIED BY 'NEW_SECURE_PASSWORD_HERE';
  FLUSH PRIVILEGES;

  # Aggiorna .env
  DB_PASSWORD=NEW_SECURE_PASSWORD_HERE
  ```

- [ ] **Cambia APP_KEY**
  ```bash
  php artisan key:generate
  ```

- [ ] **Abilita CORS corretto**
  ```php
  // config/cors.php
  'allowed_origins' => ['https://admin.wodvision.app'],
  ```

- [ ] **Validazione Input Strict**
  ```php
  // In ogni Controller
  $request->validate([
    'email' => 'required|email|max:255',
    'password' => 'required|min:8|regex:/[A-Z]/|regex:/[0-9]/',
  ]);
  ```

- [ ] **Sanitize File Uploads**
  ```php
  $file = $request->file('video_chunk');

  // Validate
  $request->validate([
    'video_chunk' => 'required|file|mimetypes:video/mp4,video/quicktime|max:102400'
  ]);

  // Rename file (no user input in filename)
  $filename = uniqid() . '.mp4';
  ```

- [ ] **Configura Backup Automatici Database**
  ```bash
  # Crontab
  0 2 * * * mysqldump -u app_user -pYOUR_DB_PASSWORD wodvision > /backup/db_$(date +\%Y\%m\%d).sql
  ```

#### Frontend (Flutter)

- [ ] **Migrare token storage a flutter_secure_storage**
  ```dart
  // Invece di SharedPreferences
  final storage = FlutterSecureStorage();
  await storage.write(key: 'auth_token', value: token);
  ```

- [ ] **Implementare Firebase App Check**
  ```dart
  await FirebaseAppCheck.instance.activate(
    webRecaptchaSiteKey: 'recaptcha-v3-site-key',
  );
  ```

- [ ] **Aggiungere SSL Pinning**
  ```yaml
  # pubspec.yaml
  dependencies:
    http_certificate_pinning: ^2.0.0
  ```

- [ ] **Validare file prima upload**
  ```dart
  if (!file.path.endsWith('.mp4')) {
    throw Exception('Only MP4 videos allowed');
  }
  if (file.lengthSync() > 100 * 1024 * 1024) {
    throw Exception('Video too large (max 100MB)');
  }
  ```

- [ ] **Obfuscate codice su release**
  ```bash
  flutter build apk --obfuscate --split-debug-info=/symbols
  ```

#### Infrastruttura

- [ ] **Configurare Firewall**
  ```bash
  sudo ufw allow 22/tcp   # SSH
  sudo ufw allow 80/tcp   # HTTP
  sudo ufw allow 443/tcp  # HTTPS
  sudo ufw enable
  ```

- [ ] **Disabilita root login SSH**
  ```bash
  # /etc/ssh/sshd_config
  PermitRootLogin no
  PasswordAuthentication no  # Solo SSH keys

  sudo systemctl restart sshd
  ```

- [ ] **Setup fail2ban** (protezione brute force)
  ```bash
  sudo apt install fail2ban
  sudo systemctl enable fail2ban
  ```

- [ ] **Monitoring e Alerting**
  - DigitalOcean Monitoring (CPU, RAM, disk)
  - Uptime monitoring (UptimeRobot gratis)
  - Log aggregation (Papertrail)

### 10.3 GDPR Compliance

#### Dati Personali Raccolti

```
- Nome
- Email
- Telefono
- Età
- Peso, Altezza
- Genere
- Video esercizi (potenzialmente riconoscibili)
- Foto profilo
- Indirizzo IP (logs)
- Device info
- Location (se richiesto)
```

#### Implementazioni Necessarie

1. **Privacy Policy**
   - Creare pagina Privacy Policy
   - Link in signup e settings
   - Aggiornare con dettagli storage dati

2. **Terms of Service**
   - Creare ToS
   - Checkbox accettazione in signup

3. **Consent Management**
   ```dart
   // In signup
   Checkbox(
     value: acceptPrivacy,
     onChanged: (value) => setState(() => acceptPrivacy = value),
   );
   Text('I accept Privacy Policy and Terms of Service');
   ```

4. **Right to Access** (già implementato)
   - `GET /profile` ritorna tutti dati utente

5. **Right to Erasure** (già implementato)
   - `GET /delete-account` elimina tutto

6. **Data Export** (da implementare)
   ```php
   // New endpoint
   GET /api/export-my-data

   // Response: JSON con TUTTI i dati utente
   {
     "user": {...},
     "journeys": [...],
     "transactions": [...],
     "notifications": [...]
   }
   ```

7. **Cookie Banner** (se web version)
   - Notifica uso cookie
   - Opzione accept/reject

8. **Data Retention Policy**
   - Definire quanto tempo conservare dati
   - Auto-delete account inattivi > 2 anni
   - Delete video > 1 anno

### 10.4 Best Practices Coding

#### Laravel Backend

```php
// 1. SEMPRE validare input
public function store(Request $request) {
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users',
    ]);

    // Use $validated, NOT $request->all()
    User::create($validated);
}

// 2. Proteggere da SQL Injection (Eloquent già lo fa)
// GOOD
User::where('email', $email)->first();

// BAD - mai fare così
DB::raw("SELECT * FROM users WHERE email = '$email'");

// 3. Hash passwords
$user->password = Hash::make($request->password);

// 4. Sanitize output
echo e($user->name);  // Escapes HTML

// 5. Autorizzazione
$this->authorize('update', $journey);  // Policy check
```

#### Flutter Frontend

```dart
// 1. SEMPRE gestire errori HTTP
try {
  final response = await ApiService.get('endpoint');
  if (response.statusCode == 200) {
    // Success
  } else {
    throw Exception('API Error: ${response.statusCode}');
  }
} catch (e) {
  // Show user-friendly error
  showToast('Something went wrong. Please try again.');
}

// 2. Validare input prima di inviare
if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email)) {
  return 'Invalid email';
}

// 3. Non loggare dati sensibili
print('User logged in');  // GOOD
print('Token: $token');   // BAD - mai loggare token/password

// 4. Timeout su richieste HTTP
final response = await http.get(uri).timeout(
  Duration(seconds: 30),
  onTimeout: () => throw TimeoutException('Request timeout'),
);
```

---

## 11. MANUTENZIONE E DEPLOY

### 11.1 Deploy Backend (Laravel)

#### Via SSH Manuale

```bash
# 1. SSH nella VM
ssh root@64.226.127.138

# 2. Naviga nella directory
cd /var/www/html/crossfit

# 3. Pull ultimi cambiamenti da GitLab
git pull origin main

# 4. Installa/aggiorna dipendenze
composer install --optimize-autoloader --no-dev

# 5. Esegui migrazioni database (se ci sono)
php artisan migrate --force

# 6. Clear cache
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# 7. Ottimizza per produzione
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 8. Restart PHP-FPM
sudo systemctl restart php8.2-fpm

# 9. Restart Nginx (se necessario)
sudo systemctl restart nginx
```

#### Automazione con Git Hooks (Consigliato)

**Setup**:
```bash
# Sulla VM
cd /var/www/html/crossfit
nano .git/hooks/post-merge

# Contenuto:
#!/bin/bash
composer install --optimize-autoloader --no-dev
php artisan migrate --force
php artisan cache:clear
php artisan config:cache
php artisan route:cache
sudo systemctl restart php8.2-fpm

# Rendi eseguibile
chmod +x .git/hooks/post-merge
```

Ora ogni `git pull` eseguirà automaticamente questi comandi.

### 11.2 Deploy Frontend (Flutter)

#### Android (Google Play)

```bash
# 1. Update version in pubspec.yaml
version: 1.0.7+8  # 1.0.7 = version name, 8 = build number

# 2. Build App Bundle (formato richiesto da Play Store)
flutter build appbundle --release --obfuscate --split-debug-info=/symbols

# Output: build/app/outputs/bundle/release/app-release.aab

# 3. Upload su Google Play Console
# - Vai su https://play.google.com/console
# - Seleziona app WodVision
# - Release > Production > Create new release
# - Upload app-release.aab
# - Compila release notes
# - Submit for review
```

**Release Notes Template**:
```
What's New in v1.0.7:

✨ New Features:
- [Feature 1]
- [Feature 2]

🐛 Bug Fixes:
- Fixed crash on video upload
- Improved performance

🔒 Security:
- Enhanced data encryption
```

#### iOS (App Store)

```bash
# 1. Update version in pubspec.yaml
version: 1.0.7+8

# 2. Build iOS
flutter build ios --release

# 3. Apri Xcode
open ios/Runner.xcworkspace

# 4. In Xcode:
# - Product > Archive
# - Window > Organizer
# - Seleziona archive > Distribute App
# - App Store Connect > Upload
# - Aspetta processing (può richiedere 30-60 min)

# 5. Su App Store Connect
# - https://appstoreconnect.apple.com/
# - My Apps > WodVision
# - TestFlight (per beta) o App Store (per release)
# - Submit for Review
```

### 11.3 Migrazioni Database

**Creare nuova migrazione**:
```bash
# Sulla VM
cd /var/www/html/crossfit
php artisan make:migration add_premium_features_to_subscriptions

# Edit: database/migrations/2026_01_11_xxx_add_premium_features_to_subscriptions.php
```

**Esempio migrazione**:
```php
public function up() {
    Schema::table('subscriptions', function (Blueprint $table) {
        $table->json('premium_features')->nullable();
        $table->integer('max_ai_analysis_per_day')->default(5);
    });
}

public function down() {
    Schema::table('subscriptions', function (Blueprint $table) {
        $table->dropColumn(['premium_features', 'max_ai_analysis_per_day']);
    });
}
```

**Eseguire migrazione**:
```bash
# Test locale prima
php artisan migrate

# Rollback se problemi
php artisan migrate:rollback

# In produzione
php artisan migrate --force
```

### 11.4 Backup Strategy

#### Backup Database

**Script Backup** (`/root/backup_db.sh`):
```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/mysql"
DB_NAME="wodvision"
DB_USER="app_user"
DB_PASS="ADFRhjdk98hdj"

mkdir -p $BACKUP_DIR

# Full backup
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME | gzip > $BACKUP_DIR/wodvision_$DATE.sql.gz

# Delete backups older than 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: wodvision_$DATE.sql.gz"
```

**Cron Job**:
```bash
# crontab -e
0 2 * * * /root/backup_db.sh >> /var/log/backup.log 2>&1
```

#### Backup Codice

**Script Backup Code** (`/root/backup_code.sh`):
```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/code"

mkdir -p $BACKUP_DIR

# Backup Laravel code
tar -czf $BACKUP_DIR/laravel_$DATE.tar.gz \
  --exclude='vendor' \
  --exclude='node_modules' \
  --exclude='storage/logs/*' \
  /var/www/html/crossfit/

# Delete old backups (>14 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +14 -delete
```

#### Backup Firebase Storage (Video)

**Usa gsutil** (Google Cloud SDK):
```bash
# Install gsutil
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud init

# Backup storage bucket
gsutil -m cp -r gs://wodvision-52d46.firebasestorage.app /backup/firebase/
```

#### Disaster Recovery Plan

**Scenario: VM corrotta/persa**

1. **Crea nuova VM DigitalOcean**
2. **Restore Database**:
   ```bash
   gunzip < wodvision_20260111.sql.gz | mysql -u app_user -p wodvision
   ```
3. **Restore Codice**:
   ```bash
   tar -xzf laravel_20260111.tar.gz -C /var/www/html/
   ```
4. **Reinstalla Stack**:
   ```bash
   sudo apt update
   sudo apt install nginx mysql-server php8.2-fpm php8.2-mysql composer
   ```
5. **Configura Nginx, PHP, MySQL**
6. **Update DNS** (punta a nuovo IP)
7. **Test completo**

**Recovery Time Objective (RTO)**: 4-6 ore
**Recovery Point Objective (RPO)**: 24 ore (backup giornaliero)

### 11.5 Monitoring e Alerting

#### Uptime Monitoring

**UptimeRobot** (gratuito per 50 monitor):
1. Vai su https://uptimerobot.com/
2. Aggiungi monitor:
   - URL: `https://admin.wodvision.app/api/test`
   - Intervallo: 5 minuti
   - Alert via email se down

#### Performance Monitoring

**Laravel Telescope** (per debug):
```bash
composer require laravel/telescope
php artisan telescope:install
php artisan migrate
```

Accesso: `https://admin.wodvision.app/telescope`

**IMPORTANTE**: Disabilitare in produzione o proteggere con auth

#### Log Aggregation

**Papertrail** (gratuito 50MB/mese):
```bash
# Install rsyslog remote logging
echo "*.*          @@logs.papertrailapp.com:12345" | sudo tee -a /etc/rsyslog.conf
sudo systemctl restart rsyslog
```

#### Server Monitoring

**DigitalOcean Monitoring** (incluso):
- Droplet → Graphs
- Metriche: CPU, RAM, Disk I/O, Bandwidth
- Alert se CPU > 90% per 5 min

---

## 12. TROUBLESHOOTING COMUNE

### 12.1 App Mobile

#### Problema: App crash al lancio

**Sintomi**: White screen, crash immediato

**Cause possibili**:
1. Firebase non inizializzato
2. API base URL errato
3. Token corrotto in SharedPreferences

**Fix**:
```dart
// 1. Verifica firebase_options.dart corretto
// 2. Clear app data (Settings > Apps > WodVision > Clear Data)
// 3. Reinstalla app

// Debug:
flutter run --release  // Test in release mode
flutter logs           // Vedi crash logs
```

#### Problema: Login fallisce sempre

**Sintomi**: "Invalid credentials" anche con password corretta

**Cause**:
1. Backend down
2. Endpoint errato
3. Email non verificata

**Fix**:
```dart
// 1. Test API manualmente
curl -X POST https://admin.wodvision.app/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"password"}'

// 2. Verifica user in database
mysql> SELECT email, is_verified FROM users WHERE email='test@test.com';

// 3. Se is_verified=0, marca come verificato
mysql> UPDATE users SET is_verified=1 WHERE email='test@test.com';
```

#### Problema: Video upload fallisce

**Sintomi**: Progress bar ferma, timeout

**Cause**:
1. File troppo grande
2. Timeout HTTP
3. Backend disco pieno

**Fix**:
```dart
// 1. Aumenta timeout
final response = await http.post(uri).timeout(Duration(minutes: 5));

// 2. Riduci chunk size
final chunkSize = 2 * 1024 * 1024;  // 2MB invece di 5MB

// 3. Verifica compressione attiva
await VideoCompress.compressVideo(file.path, quality: VideoQuality.MediumQuality);
```

#### Problema: Notifiche push non arrivano

**Sintomi**: Nessuna notifica ricevuta

**Fix**:
```dart
// 1. Verifica FCM token salvato
final token = await FirebaseMessaging.instance.getToken();
print('FCM Token: $token');

// 2. Test invio notifica da Firebase Console
// Firebase Console > Cloud Messaging > Send test message

// 3. Verifica permessi notifiche (iOS)
// Settings > WodVision > Notifications > Allow

// 4. Android: verifica Google Play Services aggiornato
```

### 12.2 Backend Laravel

#### Problema: 500 Internal Server Error

**Sintomi**: API risponde con 500

**Debug**:
```bash
# 1. Vedi logs Laravel
tail -f /var/www/html/crossfit/storage/logs/laravel.log

# 2. Vedi logs Nginx
tail -f /var/log/nginx/error.log

# 3. Abilita debug temporaneamente
# .env: APP_DEBUG=true (poi disabilita!)

# 4. Controlla permessi
sudo chown -R www-data:www-data /var/www/html/crossfit/storage
sudo chmod -R 775 /var/www/html/crossfit/storage
```

#### Problema: Database connection refused

**Sintomi**: "SQLSTATE[HY000] [2002] Connection refused"

**Fix**:
```bash
# 1. Verifica MySQL running
sudo systemctl status mysql

# 2. Se down, avvia
sudo systemctl start mysql

# 3. Verifica credenziali .env
mysql -u app_user -pADFRhjdk98hdj wodvision

# 4. Se password errata, reset
mysql -u root -p
ALTER USER 'app_user'@'localhost' IDENTIFIED BY 'NEW_PASSWORD';
```

#### Problema: Disco pieno (upload video falliscono)

**Sintomi**: Upload error, logs "No space left on device"

**Fix**:
```bash
# 1. Verifica spazio disco
df -h

# 2. Trova file grandi
du -h /var/www/html/crossfit/storage | sort -h | tail -20

# 3. Delete old logs
rm /var/www/html/crossfit/storage/logs/laravel-*.log

# 4. Delete temp upload chunks non completati
find /var/www/html/crossfit/storage/app/chunks -mtime +7 -delete

# 5. Rotate logs
php artisan log:clear
```

#### Problema: Stripe webhook non funziona

**Sintomi**: Pagamenti riusciti ma subscription non attiva in app

**Debug**:
```bash
# 1. Verifica webhook endpoint configurato su Stripe
# Dashboard Stripe > Developers > Webhooks
# Endpoint: https://admin.wodvision.app/api/stripe/webhook

# 2. Vedi webhook logs su Stripe (mostra errori)

# 3. Test locale webhook
stripe listen --forward-to localhost:8000/api/stripe/webhook

# 4. Verifica signature verification nel controller
```

### 12.3 Infrastruttura

#### Problema: Sito down (502 Bad Gateway)

**Sintomi**: `https://admin.wodvision.app` non risponde

**Fix**:
```bash
# 1. SSH nella VM
ssh root@64.226.127.138

# 2. Verifica servizi
sudo systemctl status nginx
sudo systemctl status php8.2-fpm
sudo systemctl status mysql

# 3. Restart servizi
sudo systemctl restart nginx
sudo systemctl restart php8.2-fpm

# 4. Verifica logs
tail -f /var/log/nginx/error.log
```

#### Problema: SSH connection refused

**Sintomi**: `ssh: connect to host 64.226.127.138 port 22: Connection refused`

**Fix**:
```bash
# 1. Usa DigitalOcean Web Console
# DigitalOcean > Droplet > Access > Launch Console

# 2. Verifica SSH service
sudo systemctl status sshd
sudo systemctl restart sshd

# 3. Verifica firewall
sudo ufw status
sudo ufw allow 22/tcp
```

#### Problema: VM lenta / CPU 100%

**Cause possibili**:
1. Processo runaway (loop infinito)
2. MySQL query pesanti
3. Attacco DDoS
4. Backup in corso

**Fix**:
```bash
# 1. Verifica processi CPU-intensive
top
# Premi 'P' per sort by CPU

# 2. Se processo PHP runaway
sudo killall php-fpm
sudo systemctl restart php8.2-fpm

# 3. Verifica MySQL query lente
mysql> SHOW PROCESSLIST;

# 4. Kill query lenta
mysql> KILL <process_id>;

# 5. Se DDoS, abilita fail2ban
sudo apt install fail2ban
```

### 12.4 Common Errors e Soluzioni

| Errore | Causa | Soluzione |
|--------|-------|-----------|
| `TokenMismatchException` | CSRF token expired | Aumenta `SESSION_LIFETIME` in .env |
| `Class not found` | Composer autoload out of date | `composer dump-autoload` |
| `Too many connections` (MySQL) | Connection pool esaurito | Aumenta `max_connections` MySQL config |
| `Memory limit exceeded` | PHP memory low per upload | Aumenta `memory_limit` in php.ini |
| `Maximum execution time` | Script timeout | Aumenta `max_execution_time` php.ini |
| Firebase Storage 403 | Regole sicurezza troppo restrittive | Verifica Firebase Storage Rules |
| Stripe webhook 401 | Signature verification failed | Verifica webhook secret corretto |

---

## APPENDICE A: COMANDI UTILI

### A.1 Backend (Laravel)

```bash
# Artisan commands
php artisan list                    # Lista tutti comandi
php artisan migrate                 # Esegui migrazioni
php artisan migrate:rollback        # Rollback ultima migrazione
php artisan migrate:fresh           # Drop all tables + migrate
php artisan db:seed                 # Popola database con dati test
php artisan make:controller ApiController
php artisan make:model Journey -m   # Model + migration
php artisan route:list              # Lista tutte route
php artisan tinker                  # REPL PHP interattivo
php artisan queue:work              # Esegui queue worker
php artisan schedule:run            # Esegui scheduled tasks

# Cache
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan optimize:clear          # Clear all caches

# Logs
tail -f storage/logs/laravel.log
php artisan log:clear

# Testing
php artisan test
php artisan test --filter=UserTest
```

### A.2 Database (MySQL)

```bash
# Accesso
mysql -u app_user -pADFRhjdk98hdj wodvision

# Query utili
mysql> SELECT COUNT(*) FROM users;
mysql> SELECT COUNT(*) FROM journeys WHERE status='completed';
mysql> SELECT * FROM users ORDER BY created_at DESC LIMIT 10;
mysql> SHOW TABLES;
mysql> DESCRIBE users;
mysql> SHOW PROCESSLIST;

# Backup/Restore
mysqldump -u app_user -pPASS wodvision > backup.sql
mysql -u app_user -pPASS wodvision < backup.sql

# Performance
mysql> SHOW STATUS LIKE 'Threads_connected';
mysql> SHOW VARIABLES LIKE 'max_connections';
```

### A.3 Server (Ubuntu)

```bash
# System info
uname -a                            # Kernel version
lsb_release -a                      # Ubuntu version
df -h                               # Disk usage
free -h                             # Memory usage
htop                                # Process monitor

# Services
sudo systemctl status nginx
sudo systemctl restart nginx
sudo systemctl enable nginx         # Auto-start on boot
journalctl -u nginx -f              # Nginx logs real-time

# Firewall
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# Users
whoami                              # Current user
who                                 # Logged in users
last                                # Login history

# Files
find /var/www -name "*.log"
grep -r "ERROR" /var/log/
du -sh /var/www/html/crossfit/      # Directory size
```

### A.4 Git

```bash
# Status e info
git status
git log --oneline -10
git branch -a
git remote -v

# Pull changes
git pull origin main
git fetch origin

# Stash (save uncommitted changes)
git stash
git stash pop

# Reset (careful!)
git reset --hard HEAD               # Discard all local changes
git clean -fd                       # Remove untracked files

# History
git log --author="John"
git log --since="2 weeks ago"
git show <commit_hash>
```

### A.5 Flutter

```bash
# Run app
flutter run                         # Debug mode
flutter run --release               # Release mode
flutter run -d chrome               # Web version

# Build
flutter build apk --release
flutter build ios --release
flutter build web

# Clean
flutter clean
flutter pub get

# Analyze
flutter analyze
flutter doctor                      # Check setup

# Test
flutter test
flutter test test/widget_test.dart

# Devices
flutter devices
flutter emulators
flutter emulators --launch <id>
```

---

## APPENDICE B: RIFERIMENTI ESTERNI

### B.1 Documentazione Ufficiale

- **Laravel**: https://laravel.com/docs/11.x
- **Flutter**: https://docs.flutter.dev/
- **Dart**: https://dart.dev/guides
- **Firebase**: https://firebase.google.com/docs
- **MySQL**: https://dev.mysql.com/doc/
- **Nginx**: https://nginx.org/en/docs/
- **Stripe**: https://stripe.com/docs/api
- **DigitalOcean**: https://docs.digitalocean.com/

### B.2 Tools e Servizi

**Development**:
- VS Code: https://code.visualstudio.com/
- Android Studio: https://developer.android.com/studio
- Postman: https://www.postman.com/
- TablePlus (MySQL GUI): https://tableplus.com/

**Monitoring**:
- UptimeRobot: https://uptimerobot.com/
- Papertrail: https://www.papertrail.com/
- Sentry (error tracking): https://sentry.io/

**Testing**:
- Stripe Test Cards: https://stripe.com/docs/testing
- Firebase Test Lab: https://firebase.google.com/docs/test-lab

### B.3 Community e Supporto

- Stack Overflow Flutter: https://stackoverflow.com/questions/tagged/flutter
- Stack Overflow Laravel: https://stackoverflow.com/questions/tagged/laravel
- r/FlutterDev: https://reddit.com/r/FlutterDev
- Laravel Discord: https://discord.gg/laravel
- Flutter Discord: https://discord.gg/flutter

---

## APPENDICE C: GLOSSARIO TECNICO

**API (Application Programming Interface)**: Interfaccia che permette comunicazione tra app e server

**Backend**: Server-side dell'applicazione (Laravel PHP)

**CORS (Cross-Origin Resource Sharing)**: Meccanismo sicurezza browser per richieste cross-domain

**CRUD (Create Read Update Delete)**: Operazioni base database

**DTO (Data Transfer Object)**: Oggetto per trasferire dati tra layers

**Eloquent**: ORM (Object-Relational Mapping) di Laravel

**FCM (Firebase Cloud Messaging)**: Servizio notifiche push Google

**Frontend**: Client-side dell'applicazione (Flutter app mobile)

**JWT (JSON Web Token)**: Standard token autenticazione

**Migration**: File che definisce cambiamenti schema database

**Middleware**: Software che intercetta richieste HTTP (es. auth check)

**Model**: Classe che rappresenta tabella database (Laravel Eloquent)

**ORM (Object-Relational Mapping)**: Mappa oggetti a tabelle database

**Provider (Flutter)**: Package state management

**Rate Limiting**: Limite numero richieste per prevenire abuse

**REST (Representational State Transfer)**: Architettura API HTTP

**Sanctum**: Sistema autenticazione token Laravel

**Seeder**: File per popolare database con dati test

**SSH (Secure Shell)**: Protocollo accesso remoto server

**SSL/TLS**: Protocollo crittografia HTTPS

**Widget (Flutter)**: Elemento UI riutilizzabile

---

## CONCLUSIONE

Questa documentazione copre l'intera architettura dell'app **WodVision**, dalla app mobile Flutter al backend Laravel, database MySQL, e servizi esterni (Firebase, Stripe, Brevo).

**Prossimi Step Consigliati**:

1. **Immediati** (1-2 giorni):
   - ✅ Abilita HTTPS su backend
   - ✅ Cambia password database
   - ✅ Disabilita APP_DEBUG

2. **Breve Termine** (1 settimana):
   - Migra Stripe a Live Mode
   - Implementa backup automatici
   - Setup monitoring (UptimeRobot)

3. **Medio Termine** (1 mese):
   - Implementa Firebase App Check
   - Migra token storage a flutter_secure_storage
   - Crea Privacy Policy e ToS

4. **Lungo Termine** (3 mesi):
   - Ottimizzazioni performance
   - Testing automatizzato
   - CI/CD pipeline

**Contatti Developer Originali**:
- Repository Backend: `gitlab.anviam.com/php/crossfit`
- Chiedi accesso GitLab per future modifiche

**Supporto**:
Se hai domande su questa documentazione o hai bisogno di aiuto, contattami.

---

**FINE DOCUMENTAZIONE**

*Versione 1.0 - 11 Gennaio 2026*
