# Session Log - 25 Gennaio 2026 (Sessione 2)
## Upgrade Gemini 3 Flash + AI Disclaimer

**Durata sessione**: ~2 ore
**Obiettivo**: Upgrade modello Gemini e preparazione per produzione
**Risultato**: ✅ Gemini 3 Flash operativo + Disclaimer AI aggiunto

---

## Problema Iniziale

### Gemini 2.0 Flash Deprecato
- **Deadline**: 31 Marzo 2026 (shutdown definitivo)
- **Rischio**: App non funzionante se non aggiornata prima della deadline
- **Soluzione necessaria**: Migrare a modello più recente

---

## Analisi Costi-Benefici

### Modelli Valutati

| Modello | Input ($/1M) | Output ($/1M) | Status |
|---------|--------------|---------------|--------|
| Gemini 2.0 Flash | $0.10 | $0.40 | ⚠️ Deprecato |
| Gemini 2.5 Flash-Lite | $0.10 | $0.40 | ✅ Stabile |
| Gemini 2.5 Flash | $0.30 | $2.50 | ✅ Stabile |
| **Gemini 3 Flash** | $0.50 | $3.00 | ✅ Preview |
| Gemini 3 Pro | $2.00-4.00 | $12.00-18.00 | ✅ Preview |

### Stima Costi WodVision

Per 1000 analisi/mese:
- Gemini 2.0 Flash: ~$1.70/mese
- **Gemini 3 Flash**: ~$11.50/mese (+$10)
- Gemini 3 Pro: ~$46/mese (overkill)

### Decisione
**Gemini 3 Flash** con fallback automatico a **Gemini 2.5 Flash**

**Motivazione**:
- Costo incrementale minimo (+$10/mese ≈ 2 abbonamenti)
- Migliore qualità video understanding
- "Frontier intelligence" per multimodal
- Differenziazione competitiva

---

## Implementazione

### 1. Modifica `llm_analyzer.py`

**Aggiunta configurazione modelli** (riga 17-23):
```python
# Model configuration with fallback support
# Primary: Gemini 3 Flash (best multimodal understanding)
# Fallback: Gemini 2.5 Flash (stable, good performance)
GEMINI_MODELS = [
    "gemini-3-flash-preview",  # Primary - best for video analysis
    "gemini-2.5-flash",        # Fallback - stable and reliable
]
```

**Logica fallback** (righe 159-192):
```python
# Try models with fallback support
last_error = None
for model_name in GEMINI_MODELS:
    try:
        logger_with_context.info(f"Attempting analysis with model: {model_name}")
        model = genai.GenerativeModel(
            model_name=model_name,
            generation_config=generation_config
        )
        # ... chat session e analisi
        return response.text
    except Exception as model_error:
        last_error = model_error
        logger_with_context.warning(
            f"Model {model_name} failed: {str(model_error)}. Trying fallback..."
        )
        continue

# All models failed
raise Exception(f"All Gemini models failed. Last error: {str(last_error)}")
```

### 2. Fix Formattazione Markdown

Gemini 3 formattava diversamente il markdown. Aggiornate istruzioni:

```python
CRITICAL FORMATTING INSTRUCTIONS:
In the "result" field, format the content using **Markdown** with STRICT line break rules:
- ALWAYS add TWO newlines (blank line) BEFORE each **bold** section header
- Each section header must be on its OWN line
- NEVER put a section header and content on the same line

Example of CORRECT formatting:
"**Overview of Key Errors**\n\nYour main issue is...\n\n**Actionable Corrections**\n\n- Keep the bar closer"
```

### 3. Deploy Cloud Run

```bash
# Build
gcloud builds submit --tag gcr.io/peak-ascent-452414-k2/movement-analysis

# Deploy
gcloud run deploy movement-analysis \
  --image gcr.io/peak-ascent-452414-k2/movement-analysis \
  --region us-central1

# Risultato: Revision 00038-mdp (100% traffic)
```

---

## AI Disclaimer

### Requisito
Aggiungere disclaimer simile a Gemini per protezione legale:
> "Our AI is good, but can always make mistakes, we invite you to review this with your coach."

### Implementazione Flutter

**File**: `lib/screens/home/analyzed_report_screen.dart`

```dart
// AI Disclaimer (dopo il report container)
Padding(
  padding: const EdgeInsets.symmetric(horizontal: 4.0),
  child: Row(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Padding(
        padding: const EdgeInsets.only(top: 2.0),
        child: Icon(
          Icons.info_outline,
          size: 12,
          color: Colors.grey.shade500,
        ),
      ),
      const SizedBox(width: 4),
      Expanded(
        child: Text(
          'Our AI is good, but can always make mistakes, we invite you to review this with your coach.',
          style: GoogleFonts.poppins(
            textStyle: TextStyle(
              fontSize: 10,
              fontWeight: FontWeight.w400,
              color: Colors.grey.shade500,
            ),
          ),
        ),
      ),
    ],
  ),
),
```

**Caratteristiche**:
- Responsive: testo va a capo su schermi piccoli
- Discreto: font piccolo grigio con icona info
- Posizione: sotto il report AI, prima dei bottoni

---

## Test & Verifica

### Test Upload Video
1. Upload video dall'app ✅
2. Analisi completata (200 OK) ✅
3. Formattazione corretta (headers su righe separate) ✅
4. Disclaimer visibile e responsive ✅

### Cloud Run Logs
```
POST /analyze HTTP/1.1" 200 OK
Revision: movement-analysis-00038-mdp
```

---

## File Modificati

| File | Repo | Modifica |
|------|------|----------|
| `llm_analyzer.py` | wodvision-ai | Upgrade Gemini 3 + fallback + formattazione |
| `analyzed_report_screen.dart` | wodvision-mobile | AI Disclaimer responsive |
| `CLAUDE.md` | tutti | Aggiornato v1.0.17 |

---

## Commits

### wodvision-ai
1. `2480941` - feat: upgrade to Gemini 3 Flash with 2.5 Flash fallback
2. `f4d69c9` - docs: update CLAUDE.md v1.0.17 - Gemini 3 Flash upgrade
3. `601afd3` - fix: improve markdown formatting instructions for Gemini 3

### wodvision-mobile
1. `51d680f` - feat: add AI disclaimer to analyzed report screen
2. `16be364` - fix: make AI disclaimer responsive with text wrapping
3. `4b36fae` - chore: sync documentation and minor updates

---

## Prossimi Step

### Pre-Produzione
1. [ ] **Copy paywall RevenueCat** - Scrivere testi persuasivi
2. [ ] **Test su iOS** - Verificare funzionamento su Mac/Xcode
3. [ ] **Push in produzione** - Dopo test iOS

### Setup Mac per iOS
```bash
git clone https://github.com/TeoFil795/VibeVision-mobile.git wodvision-mobile
cd wodvision-mobile
flutter pub get
cd ios && pod install && cd ..
flutter run
```

---

## Lezioni Apprese

### 1. Gemini 3 Formatta Diversamente
- Le istruzioni di formattazione devono essere più esplicite
- Specificare "TWO newlines" e "OWN line" per headers
- Fornire esempio concreto nel prompt

### 2. Fallback è Essenziale
- Modelli preview possono avere downtime
- Sempre avere un piano B (modello stabile)
- Video uploadato una volta, riutilizzato per retry

### 3. Disclaimer Legale Importante
- Protegge da responsabilità su consigli errati
- Deve essere sempre visibile ma non invasivo
- Responsive per tutti i device

---

## Costi Aggiornati

### Stima Mensile (post-upgrade)
| Voce | Costo |
|------|-------|
| Cloud Run | ~$5-10 |
| Gemini 3 Flash (1000 analisi) | ~$11.50 |
| GCS Storage | ~$2-5 |
| **Totale** | ~$20-25/mese |

Coperto da ~3 abbonamenti mensili (9.49€ × 3 = 28.47€)

---

*Fine sessione: 25 Gennaio 2026, ~18:00*
*Sistema pronto per test iOS e produzione*
