# Coding Principles - WodVision

**Source**: Basato su [claude-code-workflows](https://github.com/OneRedOak/claude-code-workflows) di Anthropic Engineering Team

---

## 🎯 Filosofia: Pragmatic Quality

**"Battle-tested by Anthropic's engineering team building Claude Code with Claude Code"**

Qualità pragmatica significa codice che:
- **Funziona** - Risolve il problema effettivo
- **È manutenibile** - Altri (o tu fra 6 mesi) possono capirlo
- **È testabile** - Può essere verificato facilmente
- **È sicuro** - Non introduce vulnerabilità

---

## 📋 Code Review Standards

### Criteri di Valutazione

1. **Syntax & Completeness**
   - Nessun errore sintattico
   - Nessun import mancante
   - Nessun TODO critico non risolto

2. **Style Guide Adherence**
   - Naming conventions consistenti
   - Formattazione standard del linguaggio
   - Commenti dove necessario (non ovunque)

3. **Bug Detection**
   - Null checks dove necessario
   - Error handling appropriato
   - Edge cases considerati

4. **Business Logic Alignment**
   - Il codice fa quello che dovrebbe
   - Non introduce regressioni
   - Performance accettabile

---

## 🏗️ Architectural Standards

### Keep It Simple

```dart
// ❌ BAD - Over-engineering
abstract class ISubscriptionServiceFactory {
  ISubscriptionService createService();
}

// ✅ GOOD - Pragmatic
class SubscriptionService {
  Future<void> purchase(String planId) async { ... }
}
```

### Don't Repeat Yourself (DRY)

```dart
// ❌ BAD - Duplicazione
void showSuccessToast(BuildContext context, String message) {
  Utils.showToastMessage(context, true, 'Success', message);
}
void showErrorToast(BuildContext context, String message) {
  Utils.showToastMessage(context, false, 'Error', message);
}

// ✅ GOOD - Riutilizzo intelligente
void showToast(BuildContext context, bool isSuccess, String message) {
  Utils.showToastMessage(
    context,
    isSuccess,
    isSuccess ? 'Success' : 'Error',
    message
  );
}
```

### Single Responsibility Principle

Ogni classe/funzione fa **una cosa sola** e la fa bene.

```dart
// ❌ BAD
class VideoAnalyzer {
  void uploadVideo() { }
  void analyzeVideo() { }
  void saveToDatabase() { }
  void sendNotification() { }
}

// ✅ GOOD
class VideoUploader { }
class VideoAnalyzer { }
class VideoRepository { }
class NotificationService { }
```

---

## 🚫 Anti-Patterns da Evitare

### 1. God Objects
Classi che fanno troppo → Split in componenti più piccoli

### 2. Magic Numbers
```dart
// ❌ BAD
if (user.age > 18) { ... }

// ✅ GOOD
const int MINIMUM_AGE = 18;
if (user.age > MINIMUM_AGE) { ... }
```

### 3. Silent Failures
```dart
// ❌ BAD
catch (e) {
  debugPrint('Error: $e');
}

// ✅ GOOD
catch (e) {
  debugPrint('Error: $e');
  showToast(context, false, 'Operation failed. Please try again.');
}
```

### 4. Premature Optimization
"Premature optimization is the root of all evil" - Donald Knuth

Scrivi codice chiaro PRIMA, ottimizza DOPO se necessario.

---

## ✅ Best Practices

### Error Handling

```dart
// Sempre gestire gli errori in modo utile per l'utente
try {
  await apiService.post('/endpoint', data);
} on TimeoutException {
  showToast(context, false, 'Server took too long to respond');
} on SocketException {
  showToast(context, false, 'No internet connection');
} catch (e) {
  showToast(context, false, 'Something went wrong');
  debugPrint('Error: $e');
}
```

### Null Safety

```dart
// Usa null-aware operators
String? userName = user?.name ?? 'Guest';

// Controlla prima di usare
if (video != null && video.url.isNotEmpty) {
  playVideo(video.url);
}
```

### Async/Await

```dart
// ❌ BAD - Nested callbacks
getData().then((data) {
  processData(data).then((result) {
    saveResult(result).then((saved) {
      // callback hell
    });
  });
});

// ✅ GOOD - Linear flow
final data = await getData();
final result = await processData(data);
await saveResult(result);
```

---

## 🧪 Testing Standards

### Test Coverage Minima
- **Critical paths**: 100% (auth, payments, data loss prevention)
- **Business logic**: 80%+
- **UI**: Test manuali o E2E per flussi principali

### Test Pragmatici
Non serve testare tutto, ma testa quello che può rompersi costosamente.

```dart
// Test che vale la pena scrivere
test('User cannot purchase subscription without login', () async {
  final service = SubscriptionService(isLoggedIn: false);
  expect(() => service.purchase('premium'), throwsException);
});

// Test che non vale la pena
test('Button has correct color', () {
  // Il design può cambiare, questo test diventa tech debt
});
```

---

## 📚 Code Documentation

### Quando Commentare

```dart
// ✅ GOOD - Spiega il "perché"
// We use a 5-minute timeout for AI analysis because
// MediaPipe + YOLO + Gemini can take 2-3 minutes
const Duration aiTimeout = Duration(minutes: 5);

// ❌ BAD - Spiega il "cosa" (già ovvio dal codice)
// Increment counter by 1
counter++;
```

### Documenta API Pubbliche

```dart
/// Uploads video in chunks to avoid memory issues.
///
/// Throws [TimeoutException] if upload takes > 60 seconds.
/// Throws [NetworkException] if connection fails.
Future<void> uploadVideo(File video) async { ... }
```

---

## 🔄 Code Review Checklist

Prima di committare, verifica:

- [ ] Il codice compila senza warning
- [ ] I test passano (o test manuali fatti)
- [ ] Nessun `TODO` critico lasciato
- [ ] Nessun `print()` o `debugPrint()` dimenticato
- [ ] Nessun secret hardcoded
- [ ] Error handling presente dove serve
- [ ] Codice auto-esplicativo (o commentato dove necessario)
- [ ] Performance accettabili (nessun loop O(n²) evitabile)

---

## 🎓 Principi Generali

### KISS (Keep It Simple, Stupid)
Soluzione più semplice > Soluzione più elegante

### YAGNI (You Aren't Gonna Need It)
Non implementare feature "per il futuro" - implementale quando servono

### Boy Scout Rule
"Lascia il codice meglio di come l'hai trovato"

---

**Ricorda**: Il codice è scritto una volta, letto mille volte. Ottimizza per leggibilità.
