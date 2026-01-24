# WodVision - Presentazione

**Data**: Gennaio 2026
**Versione App**: 1.0.8
**Status**: Beta Testing - Production Ready

---

## 🎯 Cos'è WodVision

**WodVision** è un'app mobile per smartphone (Android e iOS) che utilizza l'intelligenza artificiale per analizzare video di esercizi CrossFit e fornire feedback tecnico automatico sulla corretta esecuzione del movimento.

### Il Problema che Risolviamo

Gli atleti CrossFit spesso si allenano da soli, senza un coach presente, o comunque in classi molto numerose, rendendo difficile:
- Identificare errori tecnici in tempo reale
- Monitorare i progressi oggettivamente
- Evitare infortuni da esecuzione scorretta
- Ricevere feedback dettagliato su ogni movimento (in classi numerosi è diffiicle seguire tutti uno ad uno)

**WodVision porta un "coach virtuale AI" nella tasca di ogni atleta.**

---

## 📱 Come Funziona (per l'Utente)

1. **Registra un video**: L'atleta filma se stesso mentre esegue un esercizio (deadlift, squat, pull-up, ecc.)
2. **Carica il video**: Upload tramite app con scelta dell'esercizio
3. **Analisi AI**: L'intelligenza artificiale analizza il movimento in ~2-3 minuti
4. **Ricevi il Report**: Video annotato (con skeleton figure sopra l'atleta, molto instagrammabile) + score numerico (0-100) + feedback dettagliato su:
   - **Form** (postura e tecnica)
   - **Speed** (velocità di esecuzione)
   - **Stability** (equilibrio e controllo)

### Esempio Output
```
Deadlift Conventional - Score: 87/100
✅ Form: 92 - Ottima posizione schiena neutra
⚠️ Speed: 78 - Fase eccentrica troppo veloce
✅ Stability: 91 - Controllo del bilanciere eccellente

Suggerimenti:
- Rallenta la discesa del bilanciere (3-4 secondi)
- Mantieni tensione sui glutei durante tutta l'alzata
```
Il feedback reale è molto più dettagliato ovviamente, è circa una pagina, ed è scaricabile tramite pdf così da poterlo girare direttamente al coach o a chi si vuole.
---

## 💡 Tecnologia (Spiegazione Semplificata)

L'app combina 3 tecnologie AI:

1. **Rilevamento Scheletro** - Traccia 33 punti del corpo in movimento
2. **Riconoscimento Oggetti** - Identifica attrezzature (bilanciere, palla medica, box)
3. **Analisi Linguistica AI** - Genera feedback personalizzato in linguaggio naturale

Il tutto gira su server cloud Google, garantendo analisi veloci senza consumare batteria o memoria del telefono.

**Movimenti Supportati**: 35+ esercizi (tutti i principali movimenti CrossFit + varianti)

---

## 💰 Modello di Business

### Piani di Abbonamento (via Google Play / App Store)

| Piano | Prezzo | Caratteristiche |
|-------|--------|-----------------|
| **Base** | **9,49€/mese** | Tutti i movimenti + analisi illimitate |
| **Premium** | **79,99€/anno** | Base + risparmio 30% + early adopter lock |

**Nota**: I prezzi per i primi iscritti sono bloccati anche se aumenteremo in futuro (loyalty pricing).
In più, sono previsti AB test continui sul pricing e paywall (per questo è stato integrato RevenueCat)

### Strategia Revenue
- **Modello SaaS** (abbonamento ricorrente)
- **Free trial** disponibile per provare il servizio (7 giorni)
- **Gestione pagamenti**: RevenueCat (industry standard, usato da app come Duolingo)
- **Zero frizioni**: Abbonamenti gestiti direttamente da Google Play/App Store

---

## 💵 Costi Operativi

### Infrastruttura Cloud (Mensili)

| Servizio | Costo Mensile | Utilizzo |
|----------|---------------|----------|
| **DigitalOcean Server** | ~$16/mese | Backend API + Database |
| **Google Cloud Run** | ~$0-50/mese | AI Processing (scala con utilizzo) |
| **Firebase Storage** | ~$0-25/mese | Storage video |
| **Gemini AI API** | ~$0-100/mese | Analisi linguaggio naturale |
| **RevenueCat** | **Gratis** fino a $2,500/mese revenue | Gestione abbonamenti |

**Totale Costi Fissi**: ~$12-200/mese a seconda del volume

**Nota Importante**: I costi AI scalano con l'utilizzo (pay-per-use), quindi crescono proporzionalmente ai ricavi.

### Break-Even
- **2 abbonati mensili** coprono i costi minimi
- **Margine netto**: ~80% dopo break-even (modello SaaS altamente scalabile)

---

## 📊 Stato Attuale e Metriche

### Sviluppo
- ✅ **App completata**: Android + iOS pronte e già in produzione
- ✅ **35+ movimenti**: Database completo esercizi CrossFit
- ✅ **AI funzionante**: Testata e validata su video reali (ovviamente migliorabile)
- ✅ **Pagamenti integrati**: RevenueCat production-ready
- ✅ **Infrastruttura scalabile**: Supporta migliaia di utenti

### Testing
- 🧪 **Fase**: Beta testing interno
- 🧪 **Feedback**: Positivo su accuratezza AI e UX
- 🧪 **Prossimo step**: Beta pubblica su Google Play (della nuova versione)

### Utenti Attuali
- **Database**: Sistema di registrazione completo
- **Notifiche push**: Configurate per engagement
- **Analytics**: Pronte per tracciare conversioni

---

## 🚀 Opportunità e Potenziale

### Mercato Target

**CrossFit Globale**:
- 15,000+ box CrossFit nel mondo
- Milioni di praticanti
- Comunità altamente engaged e disposta a spendere (Gli atleti crossfit usano mediamente 2-3 app per allenarsi, è un segmento altospendente)

**Home Athletes**: Segmento in crescita post-pandemia, atleti che si allenano in garage o home gym. (potenzialità di crossover oltre il crossfit)

### Vantaggi Competitivi

1. **First Mover**: Non esistono competitor diretti con analisi AI per CrossFit
2. **Tecnologia Proprietaria**: Pipeline AI custom-built
3. **Prezzo Accessibile**: 9€/mese (testabile/modificabile) vs coach privato (50-100€/ora)
4. **Network Effect**: Community di atleti che condividono risultati
5. **Scalabilità**
### Canali di Crescita

- **Instagram/TikTok**: Demo video "prima/dopo" coaching AI
- **Box CrossFit**: Partnership B2B2C (box offrono ai propri membri)
- **Influencer**: Atleti CrossFit con seguito
- **App Store SEO**: Ottimizzazione per "crossfit training" keywords
- **Referral Program**: Già implementato (ogni utente ha link personale)

---

## 🎯 Prossimi 3-6 Mesi (Roadmap)

### Q1 2026 - Launch
- [ ] Beta pubblica su Google Play (100-1,000 utenti)
- [ ] Raccolta feedback e iterazione
- [ ] Campagne marketing organico (Instagram/TikTok)
- [ ] Obiettivo: 50 abbonati paganti

### Q2 2026 - Growth
- [ ] Campagne ads a pagamento (se budget disponibile)
- [ ] Obiettivo: 200-500 abbonati

### Q3 2026 - Scale
- [ ] Espansione mercati
- [ ] Feature avanzate (tracking progressi, community)
- [ ] Programma affiliazione per box
- [ ] Obiettivo: 1,000+ abbonati

**Revenue Target 6 Mesi**: $1000 MRR (Monthly Recurring Revenue, ottimistica ovviamente, ma non impossibile)

---

## 🤝 Perché

1. **Prodotto funzionante**: testato, ma sicuramente migliorabile
2. **Zero Debito Tecnico**: Architettura pulita, scalabile, sicura
3. **Costi Bassi**: Break-even a 3 abbonati, rischio minimo
4. **Mercato Caldo**: CrossFit in crescita, AI mainstream
5. **Timing COVID**: Normalizzazione home training
6. **Tech Stack Premium**: Google AI, RevenueCat

---

## 📋 In Sintesi

| Aspetto | Dettaglio |
|---------|-----------|
| **Cosa fa** | Coaching AI per esercizi CrossFit |
| **Stato** | Production |
| **Costi fissi** | $12-200/mese (bassi) |
| **Prezzo** | 9.49€/mese o 79.99€/anno |
| **Break-even** | 3 abbonati |
| **Margine** | 80 % |
| **Mercato** | Milioni di praticanti CrossFit globali |
| **Competitor** | Nessuno con AI per CrossFit |
| **Tech** | AI Google (Gemini), infrastruttura cloud scalabile |
| **Opportunità** | First mover in mercato non servito |

Stats attuali 
**Lancio 4 Aprile 2025**
Attività di marketing fatte pari a zero, solo un po' di Build In public con il mio profilo Linkedin (da riprendere, molto utile)

Download: 199 (169 IOS, 30 Android)
Revenue: 195$ (77 IOS, 118 Android)
