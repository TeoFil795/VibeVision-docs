# Session Log - iOS App Store Release
**Data**: 25 Gennaio 2026
**Versione**: 1.0.18 (documentazione) | 1.0.7 build 10 (App Store)
**Durata sessione**: ~2 ore

---

## Obiettivo
Configurare ambiente di sviluppo iOS su Mac e rilasciare aggiornamento app su App Store.

---

## Contesto Iniziale
- App WodVision già pubblicata su App Store (versione 1.0.1)
- Sviluppo precedente fatto su Windows (solo Android)
- Nuovo Mac configurato con Xcode e Flutter
- Necessità di testare e rilasciare aggiornamento iOS

---

## Step 1: Setup Ambiente iOS

### Verifica Configurazione
- Flutter SDK installato
- Xcode 26.2 installato
- CocoaPods installato via Homebrew

### Risoluzione Conflitto CocoaPods
**Problema**: Firebase/Crashlytics version mismatch
```
In Podfile.lock: Firebase/Crashlytics (= 11.8.0)
Required: Firebase/Crashlytics (= 11.15.0)
```

**Soluzione**:
```bash
cd ios
rm -rf Pods Podfile.lock
pod install --repo-update
```

---

## Step 2: Test su Device Fisico

### Configurazione iPhone 16
1. Collegato iPhone 16 via USB
2. Abilitata **Modalità Sviluppatore** su iPhone:
   - Impostazioni → Privacy e Sicurezza → Modalità Sviluppatore → ON
   - Riavvio richiesto
3. Autorizzato computer su iPhone

### Test App
- Build di debug eseguita con successo
- App funzionante su iOS
- RevenueCat paywall verificato funzionante
- Note: Performance "clunky" in debug mode (normale - JIT vs AOT)

---

## Step 3: Preparazione Release

### Aggiornamento Versione
```yaml
# pubspec.yaml
version: 1.0.7+10
```

### Primo Tentativo Upload - FALLITO
**Errore**:
```
Validation failed: Invalid Bundle. The bundle Runner.app/Frameworks/App.framework
does not support the minimum OS Version specified in the Info.plist.
```

**Causa**: IPHONEOS_DEPLOYMENT_TARGET misto nel progetto
```
# project.pbxproj conteneva:
IPHONEOS_DEPLOYMENT_TARGET = 12.0;  # ❌ Alcune configurazioni
IPHONEOS_DEPLOYMENT_TARGET = 15.6;  # ✅ Altre configurazioni
```

### Fix Deployment Target
```bash
# Allineato tutto a 15.6
sed -i '' 's/IPHONEOS_DEPLOYMENT_TARGET = 12.0/IPHONEOS_DEPLOYMENT_TARGET = 15.6/g' \
  ios/Runner.xcodeproj/project.pbxproj
```

### Rigenerazione File iOS
```bash
flutter clean
flutter pub get
flutter build ios --release --no-codesign
```

Questo ha aggiornato:
- `ios/Flutter/Generated.xcconfig` con FLUTTER_BUILD_NUMBER=10
- `ios/Flutter/AppFrameworkInfo.plist`
- `ios/Podfile.lock`

---

## Step 4: Archive e Upload

### Processo in Xcode
1. Aperto `Runner.xcworkspace`
2. Selezionato target "Any iOS Device (arm64)"
3. Product → Clean Build Folder (Shift+Cmd+K)
4. Product → Archive
5. Distribute App → App Store Connect → Upload

### Risultato Upload
```
Upload completed with warnings:

⚠️ Upload Symbols Failed
The archive did not include a dSYM for the objective_c.framework
```

**Nota**: Warning sui dSYM è ignorabile - riguarda solo Firebase Crashlytics per la decodifica dei crash report. Non blocca la pubblicazione.

---

## Step 5: Configurazione SSH per Push

### Problema
```
git@github.com: Permission denied (publickey).
```

### Causa
Chiave SSH esistente ma non caricata nell'agent.

### Soluzione
```bash
ssh-add ~/.ssh/id_ed25519
```

---

## File Modificati

### VibeVision-mobile
| File | Modifica |
|------|----------|
| `pubspec.yaml` | version 1.0.7+10 |
| `ios/Runner.xcodeproj/project.pbxproj` | IPHONEOS_DEPLOYMENT_TARGET = 15.6 (allineato) |
| `ios/Podfile.lock` | Firebase 11.15.0 |
| `ios/Flutter/AppFrameworkInfo.plist` | Rigenerato |
| `ios/Runner.xcodeproj/xcshareddata/xcschemes/Runner.xcscheme` | Aggiornato |
| `android/.gitignore` | Fix trailing newline |
| `CLAUDE.md` | Changelog v1.0.18 |

### VibeVision-docs
| File | Modifica |
|------|----------|
| `claude.md` | Changelog v1.0.18 |
| `SESSION_LOG_2026-01-25_IOS_RELEASE.md` | Questo file |

---

## Commits

### VibeVision-mobile
```
feat: iOS App Store release v1.0.7 (build 10)

- Fix IPHONEOS_DEPLOYMENT_TARGET alignment (12.0 → 15.6)
- Update CocoaPods dependencies (Firebase 11.15.0)
- Bump version to 1.0.7+10
- Update documentation with changelog v1.0.18
```

```
chore: fix trailing newline in android/.gitignore
```

### VibeVision-docs
```
docs: changelog v1.0.18 - iOS App Store release

- Build v1.0.7 (10) uploaded to App Store Connect
- Fix iOS deployment target alignment
- Fix CocoaPods Firebase conflict
- Tested on iPhone 16 (iOS 26.2)
```

---

## Troubleshooting Reference

### Errore: "minimum OS Version" validation failed
**Causa**: IPHONEOS_DEPLOYMENT_TARGET non allineato tra le configurazioni
**Fix**:
```bash
grep -r "IPHONEOS_DEPLOYMENT_TARGET" ios/Runner.xcodeproj/project.pbxproj
# Se ci sono versioni diverse, allineare tutte alla stessa
sed -i '' 's/IPHONEOS_DEPLOYMENT_TARGET = OLD/IPHONEOS_DEPLOYMENT_TARGET = NEW/g' \
  ios/Runner.xcodeproj/project.pbxproj
```

### Errore: Version non aggiornata in Archive
**Causa**: Xcode usa file cached
**Fix**:
```bash
flutter clean
flutter pub get
flutter build ios --release --no-codesign
# Poi in Xcode: Clean Build Folder + Archive
```

### Errore: CocoaPods version conflict
**Fix**:
```bash
cd ios
rm -rf Pods Podfile.lock
pod install --repo-update
```

### Warning: dSYM Upload Failed
**Impatto**: Nessuno sulla pubblicazione
**Causa**: Firebase Crashlytics non riesce a caricare i simboli di debug
**Note**: I crash report saranno meno leggibili ma l'app funziona normalmente

---

## Note Aggiuntive

### RevenueCat su iOS
- Paywall funziona correttamente
- Per dare accesso gratuito: Dashboard RevenueCat → Customers → Grant Promotional
- Utenti identificati tramite App User ID (non email)

### Performance Debug vs Release
- Debug: Lento/scattoso (JIT compilation)
- Release: Fluido (AOT compilation)
- Per testare performance reali: `flutter run --release`

---

## Prossimi Step
1. ⏳ Attendere review Apple (1-2 giorni lavorativi)
2. 📧 Monitorare email per feedback/approvazione
3. 🎉 Se approvata, app disponibile su App Store

---

## Stato Finale
- ✅ Ambiente iOS configurato su Mac
- ✅ Test su iPhone 16 completato
- ✅ Build 1.0.7 (10) caricata su App Store Connect
- ✅ Documentazione aggiornata
- ✅ Codice pushato su GitHub
- ⏳ In attesa review Apple
