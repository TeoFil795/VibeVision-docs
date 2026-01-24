# PIANO MIGRAZIONE REVENUECAT - 242 UTENTI LEGACY

**Data**: 23 Gennaio 2026
**Owner**: WodVision Backend Team
**Priorità**: 🔴 CRITICA
**Rischio**: ALTO (possibile perdita accesso premium se non gestito correttamente)

---

## 📊 STATO ATTUALE

### Problema
- **242 utenti registrati** nel DB Laravel con abbonamenti legacy (Stripe custom)
- RevenueCat integrato nell'app MA solo per **nuovi utenti** (da v1.0.8)
- Utenti esistenti con abbonamenti attivi **NON riconosciuti** da RevenueCat
- Dashboard legacy probabilmente **non aggiornata** con subscription status reale
- Quando aprono l'app: `isPro = false` anche se hanno subscription attiva

### Architettura Attuale

```
┌─────────────────────────────────────────┐
│  APP FLUTTER (RevenueCat SDK)           │
│                                         │
│  initRevenueCat(user_id)                │
│  ↓                                      │
│  isPro = false ❌ (non conosce legacy)  │
└─────────────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│  REVENUECAT CLOUD                       │
│                                         │
│  User 123: NO entitlements              │
│  User 456: NO entitlements              │
└─────────────────────────────────────────┘

         vs

┌─────────────────────────────────────────┐
│  DATABASE MYSQL (Laravel)               │
│                                         │
│  user_subscriptions:                    │
│  - user_id: 123                         │
│  - subscription_id: 2 (Premium)         │
│  - stripe_subscription_id: sub_xxx      │
│  - status: active ✅                    │
│  - ends_at: 2026-02-15                  │
└─────────────────────────────────────────┘
```

**Gap**: RevenueCat e DB Laravel non sono sincronizzati!

---

## 🎯 OBIETTIVI MIGRAZIONE

1. ✅ **Zero Downtime**: Nessun utente perde accesso premium durante migrazione
2. ✅ **100% Data Integrity**: Tutti gli abbonamenti attivi preservati
3. ✅ **RevenueCat as Source of Truth**: Dashboard unica per revenue/churn/MRR
4. ✅ **Webhook Sync**: Sincronizzazione automatica futura RevenueCat ↔ Laravel
5. ✅ **Decommission Legacy**: Dashboard vecchia disabilitata post-migrazione

---

## 📋 PIANO STEP-BY-STEP

### FASE 1: AUDIT COMPLETO (2-3 ore)

**Obiettivo**: Capire esattamente quanti utenti hanno subscription attiva e quale tipo.

#### Task 1.1: Query Database
```sql
-- Count totale utenti
SELECT COUNT(*) FROM users;
-- Output: 242

-- Count utenti con subscription attiva
SELECT COUNT(*)
FROM user_subscriptions
WHERE status = 'active'
AND ends_at > NOW();

-- Breakdown per subscription type
SELECT s.name, COUNT(*) as count
FROM user_subscriptions us
JOIN subscriptions s ON us.subscription_id = s.id
WHERE us.status = 'active' AND us.ends_at > NOW()
GROUP BY s.name;

-- Export completo per migrazione
SELECT
    u.id as user_id,
    u.email,
    u.name,
    us.subscription_id,
    s.name as subscription_name,
    us.stripe_subscription_id,
    us.status,
    us.starts_at,
    us.ends_at,
    us.canceled_at
FROM users u
LEFT JOIN user_subscriptions us ON u.id = us.user_id
LEFT JOIN subscriptions s ON us.subscription_id = s.id
WHERE us.status = 'active' OR us.status IS NULL
ORDER BY u.id;
```

**Output atteso**: CSV con tutti i 242 utenti + subscription status

#### Task 1.2: Analisi Dati
- [ ] Quanti utenti hanno `status='active'`?
- [ ] Quanti hanno `stripe_subscription_id NOT NULL`? (abbonamenti reali Stripe)
- [ ] Quanti hanno `ends_at` già scaduta ma `status='active'`? (dati inconsistenti!)
- [ ] Quanti utenti free (nessuna subscription)?

#### Task 1.3: Identificare Edge Cases
- Utenti con multiple subscriptions attive (bug DB?)
- Utenti con `status='canceled'` ma `ends_at` futura (grace period)
- Utenti con Stripe subscription ID mancante (test account?)

---

### FASE 2: SCRIPT IMPORTAZIONE REVENUECAT (4-6 ore)

**Obiettivo**: Creare script Laravel per importare utenti in RevenueCat via REST API.

#### RevenueCat REST API Reference

**Endpoint**: `POST https://api.revenuecat.com/v1/subscribers/{app_user_id}`

**Headers**:
```
Authorization: Bearer YOUR_REVENUECAT_API_KEY
X-Platform: android (or ios)
Content-Type: application/json
```

**Payload**:
```json
{
  "app_user_id": "123",
  "attributes": {
    "$email": "user@example.com",
    "$displayName": "John Doe"
  },
  "entitlements": {
    "pro": {
      "expires_date": "2026-02-15T23:59:59Z",
      "product_identifier": "monthly_premium"
    }
  }
}
```

#### Script Laravel: `ImportUsersToRevenueCat.php`

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\User;
use App\Models\UserSubscription;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class ImportUsersToRevenueCat extends Command
{
    protected $signature = 'revenuecat:import {--dry-run} {--user=}';
    protected $description = 'Import existing subscriptions to RevenueCat';

    private $apiKey;
    private $baseUrl = 'https://api.revenuecat.com/v1';

    public function __construct()
    {
        parent::__construct();
        $this->apiKey = env('REVENUECAT_API_KEY'); // Settare in .env
    }

    public function handle()
    {
        $dryRun = $this->option('dry-run');
        $userId = $this->option('user');

        $this->info('Starting RevenueCat import...');
        $this->info($dryRun ? 'DRY RUN MODE - No API calls' : 'LIVE MODE');

        // Query utenti da migrare
        $query = User::with('activeSubscription.subscription');

        if ($userId) {
            $query->where('id', $userId);
        }

        $users = $query->get();
        $this->info("Found {$users->count()} users to migrate");

        $bar = $this->output->createProgressBar($users->count());
        $stats = ['success' => 0, 'failed' => 0, 'skipped' => 0];

        foreach ($users as $user) {
            try {
                $result = $this->importUser($user, $dryRun);
                $stats[$result]++;
            } catch (\Exception $e) {
                $this->error("Error for user {$user->id}: {$e->getMessage()}");
                $stats['failed']++;
                Log::error('RevenueCat import failed', [
                    'user_id' => $user->id,
                    'error' => $e->getMessage()
                ]);
            }
            $bar->advance();
        }

        $bar->finish();
        $this->newLine(2);
        $this->info("Migration complete!");
        $this->table(['Status', 'Count'], [
            ['Success', $stats['success']],
            ['Failed', $stats['failed']],
            ['Skipped', $stats['skipped']]
        ]);
    }

    private function importUser(User $user, bool $dryRun): string
    {
        $subscription = $user->activeSubscription;

        // User senza subscription attiva
        if (!$subscription || $subscription->status !== 'active') {
            $this->line(" User {$user->id}: No active subscription - SKIP");
            return 'skipped';
        }

        // Check scadenza
        if ($subscription->ends_at < now()) {
            $this->line(" User {$user->id}: Subscription expired - SKIP");
            return 'skipped';
        }

        // Prepara payload
        $payload = [
            'app_user_id' => (string)$user->id,
            'attributes' => [
                '$email' => $user->email,
                '$displayName' => $user->name ?? 'User',
            ]
        ];

        // Aggiungi entitlement solo se subscription attiva
        $payload['entitlements'] = [
            'pro' => [
                'expires_date' => $subscription->ends_at->toIso8601String(),
                'product_identifier' => $this->mapProductId($subscription->subscription_id),
                'store' => 'app_store', // O 'play_store'
            ]
        ];

        if ($dryRun) {
            $this->line(" User {$user->id}: Would import - {$subscription->subscription->name} until {$subscription->ends_at}");
            return 'success';
        }

        // API Call
        $response = Http::withHeaders([
            'Authorization' => "Bearer {$this->apiKey}",
            'X-Platform' => 'android',
            'Content-Type' => 'application/json',
        ])->post("{$this->baseUrl}/subscribers/{$user->id}", $payload);

        if ($response->successful()) {
            $this->line(" User {$user->id}: SUCCESS");
            return 'success';
        } else {
            $this->error(" User {$user->id}: FAILED - {$response->body()}");
            return 'failed';
        }
    }

    private function mapProductId(int $subscriptionId): string
    {
        // Mappa subscription_id DB → product_id RevenueCat
        return match($subscriptionId) {
            1 => 'free', // Free
            2 => 'monthly_premium', // Premium Monthly
            3 => 'monthly_pro', // Pro Monthly
            4 => 'yearly_premium', // Premium Yearly
            default => 'free'
        };
    }
}
```

#### Task 2.1: Setup API Key
```bash
# .env Laravel
REVENUECAT_API_KEY=sk_xxxxxxxxxxxxxxxxxxxxx  # Secret key da RevenueCat dashboard
```

#### Task 2.2: Testing su Utenti Dummy
```bash
# Dry run (no API calls)
php artisan revenuecat:import --dry-run

# Test su singolo utente
php artisan revenuecat:import --user=123

# Full import (DRY RUN first!)
php artisan revenuecat:import --dry-run
php artisan revenuecat:import  # LIVE
```

---

### FASE 3: WEBHOOK SYNC REVENUECAT → LARAVEL (3-4 ore)

**Obiettivo**: Mantenere DB Laravel sincronizzato con RevenueCat per futuri eventi.

#### Eventi RevenueCat da Gestire

| Evento | Azione Laravel |
|--------|----------------|
| `INITIAL_PURCHASE` | Crea record in `user_subscriptions` |
| `RENEWAL` | Aggiorna `ends_at` |
| `CANCELLATION` | Set `status='canceled'`, `canceled_at=now()` |
| `EXPIRATION` | Set `status='expired'` |
| `UNCANCELLATION` | Set `status='active'`, `canceled_at=null` |

#### Controller: `RevenueCatWebhookController.php`

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use App\Models\User;
use App\Models\UserSubscription;
use App\Models\Subscription;
use Illuminate\Support\Facades\Log;

class RevenueCatWebhookController extends Controller
{
    public function handle(Request $request)
    {
        // 1. Verify webhook signature
        if (!$this->verifySignature($request)) {
            Log::warning('RevenueCat webhook: Invalid signature');
            return response()->json(['error' => 'Invalid signature'], 401);
        }

        $payload = $request->all();
        $event = $payload['event'];
        $appUserId = $event['app_user_id'];
        $entitlements = $event['entitlements'] ?? [];

        Log::info('RevenueCat webhook received', ['event_type' => $event['type'], 'user' => $appUserId]);

        // 2. Find user
        $user = User::find($appUserId);
        if (!$user) {
            Log::error('RevenueCat webhook: User not found', ['app_user_id' => $appUserId]);
            return response()->json(['error' => 'User not found'], 404);
        }

        // 3. Handle event
        try {
            match($event['type']) {
                'INITIAL_PURCHASE' => $this->handlePurchase($user, $entitlements),
                'RENEWAL' => $this->handleRenewal($user, $entitlements),
                'CANCELLATION' => $this->handleCancellation($user),
                'EXPIRATION' => $this->handleExpiration($user),
                'UNCANCELLATION' => $this->handleUncancellation($user),
                default => Log::info('RevenueCat webhook: Unhandled event type', ['type' => $event['type']])
            };
        } catch (\Exception $e) {
            Log::error('RevenueCat webhook processing failed', [
                'error' => $e->getMessage(),
                'user_id' => $user->id
            ]);
            return response()->json(['error' => 'Processing failed'], 500);
        }

        return response()->json(['status' => 'success']);
    }

    private function handlePurchase(User $user, array $entitlements)
    {
        if (!isset($entitlements['pro'])) return;

        $proEntitlement = $entitlements['pro'];
        $expiresAt = $proEntitlement['expires_date'];
        $productId = $proEntitlement['product_identifier'];

        // Mappa product_id → subscription_id
        $subscriptionId = $this->mapSubscriptionId($productId);

        UserSubscription::updateOrCreate(
            ['user_id' => $user->id],
            [
                'subscription_id' => $subscriptionId,
                'status' => 'active',
                'starts_at' => now(),
                'ends_at' => $expiresAt,
                'canceled_at' => null,
            ]
        );

        Log::info('RevenueCat: Purchase processed', ['user_id' => $user->id]);
    }

    private function handleRenewal(User $user, array $entitlements)
    {
        if (!isset($entitlements['pro'])) return;

        $expiresAt = $entitlements['pro']['expires_date'];

        UserSubscription::where('user_id', $user->id)
            ->update(['ends_at' => $expiresAt]);

        Log::info('RevenueCat: Renewal processed', ['user_id' => $user->id]);
    }

    private function handleCancellation(User $user)
    {
        UserSubscription::where('user_id', $user->id)
            ->update([
                'status' => 'canceled',
                'canceled_at' => now()
            ]);

        Log::info('RevenueCat: Cancellation processed', ['user_id' => $user->id]);
    }

    private function handleExpiration(User $user)
    {
        UserSubscription::where('user_id', $user->id)
            ->update(['status' => 'expired']);

        Log::info('RevenueCat: Expiration processed', ['user_id' => $user->id]);
    }

    private function handleUncancellation(User $user)
    {
        UserSubscription::where('user_id', $user->id)
            ->update([
                'status' => 'active',
                'canceled_at' => null
            ]);

        Log::info('RevenueCat: Uncancellation processed', ['user_id' => $user->id]);
    }

    private function verifySignature(Request $request): bool
    {
        // RevenueCat invia header X-RevenueCat-Signature
        $signature = $request->header('X-RevenueCat-Signature');
        $secret = env('REVENUECAT_WEBHOOK_SECRET');

        if (!$signature || !$secret) return false;

        $payload = $request->getContent();
        $expectedSignature = hash_hmac('sha256', $payload, $secret);

        return hash_equals($expectedSignature, $signature);
    }

    private function mapSubscriptionId(string $productId): int
    {
        return match($productId) {
            'monthly_premium' => 2,
            'monthly_pro' => 3,
            'yearly_premium' => 4,
            default => 1 // Free
        };
    }
}
```

#### Task 3.1: Route
```php
// routes/api.php
Route::post('/webhooks/revenuecat', [RevenueCatWebhookController::class, 'handle']);
```

#### Task 3.2: Configure Webhook in RevenueCat Dashboard
```
URL: https://admin.wodvision.app/api/webhooks/revenuecat
Events: All (INITIAL_PURCHASE, RENEWAL, CANCELLATION, EXPIRATION, UNCANCELLATION)
Secret: (generato da RevenueCat) → salvare in .env come REVENUECAT_WEBHOOK_SECRET
```

---

### FASE 4: TESTING (4-6 ore)

#### Test Plan

**4.1 Test su Staging (5 utenti dummy)**
- [ ] Creare 5 utenti test con diversi status:
  1. User con subscription active (ends_at futura)
  2. User con subscription canceled (ends_at futura - grace period)
  3. User con subscription expired (ends_at passata)
  4. User free (nessuna subscription)
  5. User con multiple subscriptions (edge case)

- [ ] Eseguire import su staging:
  ```bash
  php artisan revenuecat:import --dry-run
  php artisan revenuecat:import
  ```

- [ ] Verificare in RevenueCat Dashboard:
  - User 1: entitlement "pro" attivo ✅
  - User 2: entitlement "pro" attivo (fino a ends_at) ✅
  - User 3: nessun entitlement ✅
  - User 4: nessun entitlement ✅
  - User 5: solo 1 entitlement (ultimo attivo) ✅

- [ ] Testare app Flutter:
  - Login come User 1 → `isPro = true` ✅
  - Login come User 4 → `isPro = false` ✅

**4.2 Test Webhook**
- [ ] Simulare eventi RevenueCat (RevenueCat dashboard → Test Webhook)
- [ ] Verificare log Laravel: `storage/logs/laravel.log`
- [ ] Verificare DB: `user_subscriptions` aggiornata correttamente

**4.3 Test Flusso Completo**
- [ ] Nuovo utente fa purchase in-app
- [ ] Webhook ricevuto da Laravel
- [ ] DB aggiornato
- [ ] RevenueCat Dashboard aggiornato

---

### FASE 5: DEPLOY PRODUZIONE (2-3 ore)

#### Pre-Deploy Checklist
- [ ] Backup completo DB MySQL
- [ ] Script import testato su staging
- [ ] Webhook configurato e testato
- [ ] .env produzione aggiornato con API keys RevenueCat
- [ ] Rollback plan documentato

#### Deploy Steps

**5.1 Backup**
```bash
# SSH su server DigitalOcean
ssh root@64.226.127.138

# Backup DB
mysqldump -u app_user -p wodvision > /backup/wodvision_pre_revenuecat_migration_$(date +%Y%m%d).sql

# Backup .env
cp /var/www/html/crossfit/.env /backup/.env_backup
```

**5.2 Deploy Codice**
```bash
# Pull latest code
cd /var/www/html/crossfit
git pull origin main

# Update dependencies
composer install --no-dev --optimize-autoloader

# Clear cache
php artisan config:clear
php artisan cache:clear
php artisan route:clear
```

**5.3 Eseguire Import (Batch Processing)**
```bash
# Dry run FIRST!
php artisan revenuecat:import --dry-run > /tmp/revenuecat_import_dryrun.log

# Review log
less /tmp/revenuecat_import_dryrun.log

# LIVE import
php artisan revenuecat:import > /tmp/revenuecat_import_live.log 2>&1

# Monitor progress
tail -f /tmp/revenuecat_import_live.log
```

**5.4 Verifiche Post-Import**
```bash
# Check log
grep "SUCCESS\|FAILED" /tmp/revenuecat_import_live.log | wc -l

# Query DB per verificare integrità
mysql -u app_user -p wodvision <<EOF
SELECT
    COUNT(*) as total_users,
    SUM(CASE WHEN status='active' THEN 1 ELSE 0 END) as active_subs
FROM user_subscriptions;
EOF
```

**5.5 Verifiche RevenueCat Dashboard**
- Login: https://app.revenuecat.com
- Controllare: Active Subscribers count = numero utenti active nel DB
- Controllare: MRR (Monthly Recurring Revenue) corretto

---

### FASE 6: POST-MIGRATION (1-2 ore)

#### Task 6.1: Comunicazione Utenti
**SE necessario reset app**:
- Email/push notification: "Abbiamo migliorato il sistema abbonamenti. Per favore riavvia l'app."
- In-app banner: "Aggiornamento completato, riavvia l'app"

#### Task 6.2: Monitoring (primo mese)
- [ ] Monitorare webhook RevenueCat: errori in `storage/logs/laravel.log`
- [ ] Monitorare support tickets: utenti che segnalano "ho perso accesso premium"
- [ ] Confrontare MRR RevenueCat vs revenue storico
- [ ] Verificare churn rate pre/post migrazione

#### Task 6.3: Decommission Dashboard Legacy
- [ ] Disabilitare route dashboard vecchia (se esiste)
- [ ] Documentare che RevenueCat è source of truth
- [ ] Formare team su RevenueCat dashboard

---

## 🚨 RISCHI E MITIGAZIONI

| Rischio | Probabilità | Impact | Mitigazione |
|---------|-------------|---------|-------------|
| Utenti perdono accesso premium | Media | ALTO | Dry-run + staging test + batch processing + rollback plan |
| API RevenueCat down durante import | Bassa | MEDIO | Retry logic + batch processing (10 user per batch) |
| Webhook signature mismatch | Media | BASSO | Test webhook su staging first |
| DB inconsistenti (ends_at passata ma status=active) | Alta | MEDIO | Audit query pre-import per pulire dati |
| Multiple subscriptions attive per stesso user | Bassa | BASSO | Script prende solo ultima attiva |

---

## 📞 ROLLBACK PLAN

**Se qualcosa va storto durante import**:

```bash
# 1. Stop import
Ctrl+C

# 2. Restore DB backup
mysql -u app_user -p wodvision < /backup/wodvision_pre_revenuecat_migration_YYYYMMDD.sql

# 3. Rollback code
cd /var/www/html/crossfit
git reset --hard <previous-commit>
composer install

# 4. Clear cache
php artisan config:clear
php artisan cache:clear

# 5. Verify app still works with legacy system
curl https://admin.wodvision.app/api/subscriptions
```

**Se utenti segnalano perdita accesso post-import**:

```bash
# Re-import singolo utente
php artisan revenuecat:import --user=<user_id>

# Oppure grant manuale entitlement via RevenueCat dashboard:
# Dashboard → Customers → <user_id> → Grant Entitlement → "pro"
```

---

## 📚 DOCUMENTAZIONE POST-MIGRAZIONE

Da aggiornare:
- [ ] `DOCUMENTAZIONE_TECNICA_WODVISION.md`: sezione Subscriptions
- [ ] `claude.md`: aggiornare "RevenueCat è source of truth"
- [ ] README team: link dashboard RevenueCat + API keys location
- [ ] Onboarding nuovi dev: RevenueCat webhook flow

---

## ✅ SUCCESS CRITERIA

**La migrazione è considerata SUCCESS se**:

1. ✅ 100% utenti con `status='active'` nel DB hanno `isPro=true` nell'app
2. ✅ Zero utenti segnalano perdita accesso premium (0 support tickets)
3. ✅ Webhook RevenueCat → Laravel funziona al 100%
4. ✅ MRR RevenueCat dashboard = revenue storico (±5% tolleranza)
5. ✅ Nessun downtime durante migrazione
6. ✅ DB Laravel sincronizzato con RevenueCat (verifiche spot check)

---

## 📊 TIMELINE STIMATA

| Fase | Tempo | Owner |
|------|-------|-------|
| 1. Audit DB | 2-3 ore | Backend Dev |
| 2. Script Import | 4-6 ore | Backend Dev |
| 3. Webhook Setup | 3-4 ore | Backend Dev |
| 4. Testing Staging | 4-6 ore | QA + Backend Dev |
| 5. Deploy Produzione | 2-3 ore | DevOps + Backend Dev |
| 6. Post-Migration | 1-2 ore | Product Manager |

**TOTALE**: 16-24 ore lavorative (~3-5 giorni calendario)

---

## 🎯 NEXT STEPS IMMEDIATE

1. **Week 1**: Audit DB + Script import
2. **Week 2**: Testing staging + Webhook setup
3. **Week 3**: Deploy produzione (preferibilmente Lunedì mattina per monitoring)
4. **Week 4**: Post-migration monitoring + decommission legacy

**START DATE CONSIGLIATA**: Inizio settimana prossima (Lunedì)
**TARGET COMPLETION**: Fine mese (entro 31 Gennaio 2026)

---

*Documento creato: 23 Gennaio 2026*
*Owner: WodVision Backend Team*
*Status: DRAFT - In Review*
