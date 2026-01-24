# 📚 DOCUMENTAZIONE TECNICA - BACKEND PYTHON WODVISION

**Versione**: 1.0
**Data**: 11 Gennaio 2026
**Autore**: Analisi completa del backend Python per analisi video CrossFit
---

## 📋 INDICE

1. [Panoramica Generale](#1-panoramica-generale)
2. [Architettura del Sistema](#2-architettura-del-sistema)
3. [Struttura dei File](#3-struttura-dei-file)
4. [Tecnologie Utilizzate](#4-tecnologie-utilizzate)
5. [API Endpoints](#5-api-endpoints)
6. [Componenti Principali](#6-componenti-principali)
7. [Processo di Analisi Video](#7-processo-di-analisi-video)
8. [Skeleton Detection (MediaPipe)](#8-skeleton-detection-mediapipe)
9. [Analisi con Gemini AI](#9-analisi-con-gemini-ai)
10. [Criteri di Movimento](#10-criteri-di-movimento)
11. [Configurazioni e Parametri](#11-configurazioni-e-parametri)
12. [Deployment su Google Cloud Run](#12-deployment-su-google-cloud-run)
13. [Modelli YOLO Custom](#13-modelli-yolo-custom)
14. [Storage e File Management](#14-storage-e-file-management)
15. [Logging e Monitoring](#15-logging-e-monitoring)
16. [Come Modificare](#16-come-modificare)
17. [Troubleshooting](#17-troubleshooting)
18. [Costi e Performance](#18-costi-e-performance)

---

## 1. PANORAMICA GENERALE

### Scopo del Backend Python

Questo backend è un **microservizio Python** deployato su **Google Cloud Run** che:

1. ✅ **Riceve video** dall'app Flutter tramite Laravel
2. ✅ **Applica skeleton detection** usando MediaPipe
3. ✅ **Analizza il movimento** con Gemini AI 2.0 Flash
4. ✅ **Rileva oggetti** (barbell, box, ball) con YOLO custom
5. ✅ **Restituisce video processato** + analisi JSON strutturata

### Posizionamento nell'Architettura

```
┌─────────────────┐
│   App Flutter   │
│   (Frontend)    │
└────────┬────────┘
         │
         ↓ HTTP Request
┌─────────────────┐
│ Laravel Backend │  (DigitalOcean)
│ (API Gateway)   │  - Gestione utenti
└────────┬────────┘  - Database MySQL
         │           - Upload chunks
         ↓ HTTP POST
┌─────────────────────────────┐
│ PYTHON BACKEND (Cloud Run)  │  ← QUESTO DOCUMENTO
│ - MediaPipe Pose Detection  │
│ - Gemini AI Analysis        │
│ - YOLO Object Detection     │
└────────┬────────────────────┘
         │
         ↓ Return JSON + Video URL
┌─────────────────┐
│ Google Cloud    │
│ Storage Bucket  │
│ (Video Output)  │
└─────────────────┘
```

### URL del Servizio

**Production URL**: `https://movement-analysis-250284968641.us-central1.run.app`

---

## 2. ARCHITETTURA DEL SISTEMA

### Stack Tecnologico

| Componente | Tecnologia | Versione |
|------------|------------|----------|
| **Framework Web** | FastAPI | 0.115.7 |
| **Server ASGI** | Uvicorn | 0.34.0 |
| **Pose Detection** | MediaPipe | 0.10.21 |
| **Object Detection** | YOLO (Ultralytics) | 8.3.51 |
| **AI Analysis** | Gemini 2.0 Flash | google-generativeai 0.8.4 |
| **Computer Vision** | OpenCV | 4.11.0.86 |
| **Cloud Storage** | Google Cloud Storage | 2.19.0 |
| **Logging** | Loguru | 0.7.3 |
| **Container** | Docker | Python 3.10-slim |

### Pattern Architetturale

**Microservizi + Event-Driven Processing**

```
Request → Middleware → Async Processing → Parallel Tasks → Response
    ↓         ↓              ↓                   ↓            ↓
Request ID  Logging    Video Processing    Gemini AI    JSON + URL
                       MediaPipe Skeleton   Analysis
                       YOLO Objects
```

### Caratteristiche Chiave

- ⚡ **Async/Await**: Processing parallelo di video e AI
- 🔄 **Stateless**: Ogni request è indipendente
- 📦 **Containerizzato**: Docker image su GCR
- 🚀 **Auto-scaling**: Cloud Run gestisce il traffico
- 🔒 **Autenticazione**: Bearer token Google Cloud

---

## 3. STRUTTURA DEI FILE

### File Tree

```
movement-analysis-code/
│
├── server.py                  # FastAPI server principale
├── main.py                    # Pipeline video processing
├── config.py                  # Configurazioni (colori, style, movimenti)
├── requirements.txt           # Dipendenze Python
├── Dockerfile                 # Container configuration
├── cloudbuild.yaml            # Google Cloud Build config
│
├── pose_detector.py           # MediaPipe pose detection
├── visualizer.py              # Disegno skeleton e overlay
├── angle_calculator.py        # Calcolo angoli articolazioni
├── object_detector.py         # YOLO object detection
├── llm_analyzer.py            # Gemini AI integration
├── movement_criteria.py       # Knowledge base esercizi (127KB!)
│
├── video_loader.py            # Caricamento/salvataggio video
├── storage_utils.py           # Google Cloud Storage manager
├── logger_config.py           # Logging configuration
├── service-account.json       # GCP credentials
│
├── models/                    # YOLO trained models
│   ├── barbell.pt            # Modello bilanciere
│   ├── box.pt                # Modello box jump
│   └── ball.pt               # Modello wall ball
│
├── uploads/                   # Temp input videos
├── output/                    # Temp processed videos
└── logs/                      # Log files
```

### Dimensioni File Chiave

| File | Dimensione | Descrizione |
|------|-----------|-------------|
| `movement_criteria.py` | 127 KB | Knowledge base 35+ esercizi |
| `visualizer.py` | 12 KB | Rendering skeleton |
| `llm_analyzer.py` | 8 KB | Gemini integration |
| `server.py` | 10 KB | FastAPI endpoints |
| `models/barbell.pt` | ~6 MB | YOLO weights |

---

## 4. TECNOLOGIE UTILIZZATE

### FastAPI Framework

**File**: `server.py`

```python
app = FastAPI()

# Features utilizzate:
- Async endpoints
- File upload multipart/form-data
- Dependency injection (Request ID)
- Middleware custom
- Static file serving
- Streaming response
```

**Vantaggi**:
- ⚡ Performance elevate (async nativo)
- 📝 Auto-documentazione API (Swagger UI)
- ✅ Validazione automatica input
- 🔧 Dependency injection nativo

### MediaPipe Pose

**File**: `pose_detector.py`

```python
mp.solutions.pose.Pose(
    min_detection_confidence=0.8,
    min_tracking_confidence=0.5,
    model_complexity=1,
    static_image_mode=False,
    enable_segmentation=False
)
```

**Capabilities**:
- 33 landmark points sul corpo umano
- Tracking real-time frame-by-frame
- Confidenza detection: 80%
- Confidenza tracking: 50%

### YOLO (Ultralytics)

**File**: `object_detector.py`

```python
YOLO("models/barbell.pt")
```

**Modelli Custom Trainati**:
- `barbell.pt`: Rileva barbell, weights, empty bar
- `box.pt`: Rileva box per box jumps
- `ball.pt`: Rileva medicine ball

**Classes**:
- Class 0: weights (dischi bilanciere)
- Class 1: barbell (barra)
- Class 2: empty (barra vuota)

### Gemini 2.0 Flash

**File**: `llm_analyzer.py`

```python
genai.GenerativeModel(
    model_name="gemini-2.0-flash",
    generation_config={
        "temperature": 1,
        "max_output_tokens": 8192,
        "response_mime_type": "application/json"
    }
)
```

**Caratteristiche**:
- Analisi video nativa
- Output JSON strutturato
- Context window: 1M token
- Multimodal (video + text)

### Google Cloud Storage

**File**: `storage_utils.py`

```python
CloudStorageManager()
- Bucket: movement-analysis-videos
- Upload video processati
- URL pubblici
```

---

## 5. API ENDPOINTS

### POST /analyze

**Endpoint principale** per l'analisi movimento.

#### Request

```http
POST /analyze
Content-Type: multipart/form-data

Fields:
- movement: string (nome esercizio)
- video: file (MP4)
- userInfo: string (info utente, es: "age:35")
```

#### Response Success (200)

```json
{
  "status": "success",
  "request_id": "uuid-v4",
  "video_url": "https://storage.googleapis.com/.../processed_videos/video.mp4",
  "llm_analysis": "{\"result\": \"...\", \"form\": 85, \"speed\": 78, \"stability\": 92}"
}
```

#### Response Error (500)

```json
{
  "message": "Analysis failed",
  "error_type": "ValueError",
  "error_details": "Invalid movement name"
}
```

#### Headers

```
Authorization: Bearer <google-cloud-token>
X-Request-ID: uuid-v4 (response)
```

#### Timeout

- Request timeout: **3600 secondi** (1 ora)
- Container timeout: **3600 secondi**

---

### GET /video/{video_name}

**Download video processato** (streaming).

#### Request

```http
GET /video/20250111_123456_processed.mp4
```

#### Response

```http
HTTP/1.1 200 OK
Content-Type: video/mp4
Accept-Ranges: bytes
Content-Disposition: attachment; filename="20250111_123456_processed.mp4"

[Binary video data streamed in 8KB chunks]
```

---

## 6. COMPONENTI PRINCIPALI

### 6.1 Server (server.py)

**Responsabilità**:
- Gestione richieste HTTP
- Orchestrazione processing parallelo
- Cleanup file temporanei
- Error handling centralizzato

**Key Functions**:

```python
@app.post("/analyze")
async def analyze_movement(...)
    # 1. Salva video temporaneo
    # 2. Avvia task paralleli:
    #    - process_video_async (skeleton)
    #    - analyze_video_with_llm (Gemini)
    # 3. Upload a Cloud Storage
    # 4. Ritorna JSON response
```

**Middleware**:

```python
class RequestIDMiddleware(BaseHTTPMiddleware)
    # Genera UUID per ogni request
    # Logging contestuale
    # Headers X-Request-ID
```

---

### 6.2 Main Pipeline (main.py)

**Responsabilità**:
- Orchestrazione processing video
- Coordinate detection (pose + objects)
- Rendering finale

**Flusso**:

```python
def main(video_path, output_path, movement_name):
    1. Carica configurazione movimento
    2. Inizializza detector (Pose, Object)
    3. Per ogni frame:
       a. Detect landmarks (MediaPipe)
       b. Detect objects (YOLO se richiesto)
       c. Calcola angoli articolazioni
       d. Disegna skeleton
       e. Disegna oggetti
       f. Scrivi frame processato
    4. Salva video output
```

---

### 6.3 Pose Detector (pose_detector.py)

**MediaPipe Pose Wrapper**

```python
class PoseDetector:
    def detect_landmarks(frame) -> landmarks
        # Ritorna 33 landmark points
        # Formato: [x, y, z, visibility]

    def get_landmark_coordinates(landmarks, point, shape)
        # Converte landmark normalizzato a pixel

    def get_landmark_point(landmarks, point)
        # Ritorna coordinate normalizzate [0-1]
```

**Landmark Points** (33 totali):
- 0-10: Faccia (esclusi dal rendering)
- 11-12: Spalle
- 13-16: Braccia e gomiti
- 17-22: Mani (escluse dal rendering)
- 23-24: Anche
- 25-28: Gambe e ginocchia
- 29-32: Piedi (esclusi dal rendering)

---

### 6.4 Visualizer (visualizer.py)

**Rendering Skeleton e Overlay**

#### Funzioni Chiave:

```python
def draw_body_landmarks(image, landmarks)
    # Disegna:
    # - Linee connessione landmark (bianco + grigio shadow)
    # - Punti articolazioni colorati
    # - Lato sinistro: ARANCIONE
    # - Lato destro: AZZURRO
```

```python
def draw_angle(image, points, angle, color)
    # Disegna:
    # - Arco semicircolare all'articolazione
    # - Valore angolo in gradi
    # - Overlay semi-trasparente
```

```python
def draw_detections(frame, detections)
    # Disegna oggetti rilevati:
    # - Bounding box (verde)
    # - Linea tra 2 weights (barbell center line)
    # - Cerchi ai punti di contatto
```

#### Styling:

**Colori** (RGB format):
```python
WHITE = (255, 255, 255)       # Linee skeleton
GRAY = (128, 128, 128)        # Ombre
ORANGE = (0, 165, 255)        # Punti lato sinistro
LIGHT_BLUE = (255, 191, 0)    # Punti lato destro
GREEN = (0, 255, 0)           # Oggetti (barbell/box)
```

**Dimensioni**:
```python
LINE_THICKNESS = 2            # Spessore linee
POINT_SIZE = 5                # Raggio cerchi
FONT_SCALE = 0.7              # Dimensione testo
```

---

### 6.5 Angle Calculator (angle_calculator.py)

**Calcolo Angoli Articolazioni**

```python
def calculate_angle(a, b, c) -> float
    # Calcola angolo tra 3 punti
    # Usa arctan2 per angolo in radianti
    # Converte a gradi
    # Range: 0-180°
```

**Articolazioni Tracciate**:

| Articolazione | Punti (Left) | Punti (Right) |
|---------------|--------------|---------------|
| **Knee** | Hip → Knee → Ankle | Hip → Knee → Ankle |
| **Hip** | Shoulder → Hip → Knee | Shoulder → Hip → Knee |
| **Elbow** | Shoulder → Elbow → Wrist | Shoulder → Elbow → Wrist |

**Output**:
```python
{
    'left_knee': {'angle': 145.2, 'points': (p1, p2, p3)},
    'right_knee': {'angle': 143.8, 'points': (p1, p2, p3)},
    'left_hip': {'angle': 92.5, 'points': (p1, p2, p3)},
    ...
}
```

---

### 6.6 Object Detector (object_detector.py)

**YOLO Custom Object Detection**

#### Initialization:

```python
ObjectDetector(object_type="barbell")
    # Carica modello: models/barbell.pt
    # Confidence threshold: 0.5 (50%)
```

#### Detection Logic:

```python
def detect(frame) -> List[dict]
    # Ritorna: [{
    #   'bbox': (x1, y1, x2, y2),
    #   'confidence': 0.87,
    #   'class': 1,  # barbell
    #   'object_type': 'barbell'
    # }]
```

#### Smart Barbell Detection:

```python
def detect_with_confidence(frame)
    # Logica avanzata:
    # 1. Rileva weights, barbell, empty bar
    # 2. Valida: barbell deve contenere esattamente 2 weights
    # 3. Calcola center line tra i 2 weights
    # 4. Ritorna solo detection valide
```

**Validazione Barbell**:
- ✅ Barbell deve avere 2 weights dentro bbox
- ✅ Weights devono essere centrati nel barbell
- ✅ Se solo 2 empty bars → disegna linea verde

---

### 6.7 LLM Analyzer (llm_analyzer.py)

**Gemini AI Integration**

#### Workflow:

```python
async def analyze_video_with_llm(
    video_path,
    request_id,
    movement_name,
    userInfo
)
    1. Upload video a Gemini
    2. Attendi processing (ACTIVE state)
    3. Recupera criteri movimento da MOVEMENT_CRITERIA
    4. Costruisci prompt strutturato
    5. Invia a Gemini 2.0 Flash
    6. Ritorna JSON analysis
```

#### Prompt Structure:

```python
prompt = f"""
YOU ARE A CROSSFIT (L3) AND BIOMECHANICS EXPERT

INPUTS:
- Knowledge Base: {criteria_text}
- User Info: {userInfo}

OUTPUT STRUCTURE:
1. Overview of Key Errors
2. Actionable Corrections
3. Regressions/Progressions (based on score)
4. Mobility Advice
5. Conclusion

FORMAT: Markdown with **bold**, bullet points

JSON SCHEMA:
{{
  "result": "string",    // Detailed feedback
  "form": number,        // 70-100
  "speed": number,       // 70-100
  "stability": number    // 70-100
}}
"""
```

#### Model Configuration:

```python
generation_config = {
    "temperature": 1,           # Creatività
    "top_p": 0.95,             # Nucleus sampling
    "top_k": 40,               # Top-K sampling
    "max_output_tokens": 8192,  # Max output
    "response_schema": {...},   # JSON schema enforcement
    "response_mime_type": "application/json"
}
```

#### Error Handling:

- ❌ Video processing timeout → Retry automatico
- ❌ Gemini API error → Log dettagliato + HTTPException
- ❌ Invalid response format → Schema validation fail

---

## 7. PROCESSO DI ANALISI VIDEO

### Flow Diagram Completo

```
┌─────────────────────────────────────────────────────────────┐
│                    REQUEST RICEVUTA                          │
│  POST /analyze (movement, video, userInfo)                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│                MIDDLEWARE REQUEST ID                         │
│  UUID generato: 550e8400-e29b-41d4-a716-446655440000       │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│              SALVATAGGIO VIDEO TEMPORANEO                    │
│  /tmp/uploads/20250111_123456_input.mp4                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
         ┌───────────┴───────────┐
         │                       │
         ↓                       ↓
┌──────────────────┐    ┌──────────────────┐
│ TASK 1: VIDEO    │    │ TASK 2: GEMINI   │
│ PROCESSING       │    │ AI ANALYSIS      │
│                  │    │                  │
│ - MediaPipe Pose │    │ - Upload video   │
│ - YOLO Objects   │    │ - Get criteria   │
│ - Draw Skeleton  │    │ - Run analysis   │
│ - Save Output    │    │ - Parse JSON     │
└──────────┬───────┘    └───────┬──────────┘
           │                    │
           │ asyncio.gather()   │
           └──────────┬─────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────────────────┐
│            UPLOAD A CLOUD STORAGE                            │
│  gs://movement-analysis-videos/processed_videos/...mp4      │
│  Public URL: https://storage.googleapis.com/...             │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│                 CLEANUP FILE TEMPORANEI                      │
│  - Rimuovi /tmp/uploads/input.mp4                          │
│  - Rimuovi /tmp/uploads/output.mp4                         │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│                    RESPONSE JSON                             │
│  {                                                           │
│    "status": "success",                                      │
│    "request_id": "550e8400-...",                            │
│    "video_url": "https://storage...",                       │
│    "llm_analysis": "{...}"                                  │
│  }                                                           │
└─────────────────────────────────────────────────────────────┘
```

### Timing Stimato

| Fase | Tempo Medio | Note |
|------|-------------|------|
| Upload video | 2-5 sec | Dipende da dimensione |
| MediaPipe processing | 10-30 sec | ~30 FPS processing |
| Gemini upload | 5-10 sec | Video a Gemini API |
| Gemini analysis | 20-60 sec | Dipende da lunghezza video |
| Cloud Storage upload | 3-8 sec | Video processato |
| **TOTALE** | **40-113 sec** | 0.7-2 minuti |

---

## 8. SKELETON DETECTION (MEDIAPIPE)

### Configurazione Pose Detection

```python
# pose_detector.py
mp.solutions.pose.Pose(
    min_detection_confidence=0.8,   # 80% confidenza minima
    min_tracking_confidence=0.5,    # 50% tracking frame-to-frame
    model_complexity=1,             # Modello medio (0=lite, 2=full)
    static_image_mode=False,        # Video mode (tracking enabled)
    enable_segmentation=False       # No background segmentation
)
```

### 33 Landmark Points

MediaPipe rileva **33 punti** sul corpo:

#### Punti Visualizzati (20):
```
11-12: LEFT_SHOULDER, RIGHT_SHOULDER
13-14: LEFT_ELBOW, RIGHT_ELBOW
15-16: LEFT_WRIST, RIGHT_WRIST
23-24: LEFT_HIP, RIGHT_HIP
25-26: LEFT_KNEE, RIGHT_KNEE
27-28: LEFT_ANKLE, RIGHT_ANKLE
```

#### Punti Esclusi (13):
```python
# visualizer.py - excluded_landmarks
NOSE, LEFT_EYE, RIGHT_EYE, MOUTH (faccia)
LEFT_PINKY, LEFT_INDEX, LEFT_THUMB (mano sx)
RIGHT_PINKY, RIGHT_INDEX, RIGHT_THUMB (mano dx)
LEFT_HEEL, LEFT_FOOT_INDEX (piede sx)
RIGHT_HEEL, RIGHT_FOOT_INDEX (piede dx)
```

### Rendering Skeleton

**Linee di Connessione**:
```python
# visualizer.py:55-56
cv2.line(overlay, start, end, Colors.GRAY, THICKNESS+2)  # Ombra
cv2.line(image, start, end, Colors.WHITE, THICKNESS)      # Linea
```

**Punti Articolazioni**:
```python
# Lato SINISTRO (Arancione)
cv2.circle(image, (x, y), POINT_SIZE, Colors.ORANGE, -1)

# Lato DESTRO (Azzurro)
cv2.circle(image, (x, y), POINT_SIZE, Colors.LIGHT_BLUE, -1)

# Centro corpo (Bianco)
cv2.circle(image, (x, y), POINT_SIZE, Colors.WHITE, -1)
```

**Effetti Visivi**:
```python
# Highlight interno ai punti
cv2.circle(image, (x, y), POINT_SIZE-3, (255, 200, 200), -1)

# Blend overlay per trasparenza
cv2.addWeighted(overlay, 0.2, image, 0.8, 0, image)
```

### Visualizzazione Angoli

```python
# visualizer.py:82-153
def draw_angle(image, points, angle, color):
    # 1. Calcola vettori tra i 3 punti
    # 2. Disegna arco ellittico all'articolazione
    # 3. Overlay semi-trasparente (40%)
    # 4. Testo con valore angolo (int degrees)

    cv2.ellipse(overlay, center, radius, angle, ...)
    cv2.putText(image, f'{int(angle)}°', position, ...)
```

**Colori Angoli**:
- Arancione: Articolazioni lato sinistro
- Azzurro: Articolazioni lato destro

---

## 9. ANALISI CON GEMINI AI

### API Key e Autenticazione

**API Key**: `AIzaSyC7QxjlBPGTec_wpRmCxGwFj9QV7P-j6dQ`

```python
# llm_analyzer.py:15
genai.configure(api_key=os.getenv("GEMINI_API_KEY"))
```

**Nota**: La chiave è anche hardcoded nel Cloud Run YAML.

### Modello Utilizzato

**Gemini 2.0 Flash Experimental**

```python
# llm_analyzer.py:143-146
model = genai.GenerativeModel(
    model_name="gemini-2.0-flash",
    generation_config=generation_config
)
```

**Caratteristiche**:
- 🎥 Input video nativo (fino a 1 ora)
- 📝 Context window: 1M token
- ⚡ Velocità: ~2-3 sec per 1K token output
- 💰 Costo: $0.075/1M token input, $0.30/1M output

### Upload Video a Gemini

```python
# llm_analyzer.py:25-31
def upload_to_gemini(path, mime_type=None):
    file = genai.upload_file(path, mime_type="video/mp4")
    return file  # URI: gs://generativeai-uploads/...
```

**Processo**:
1. Video caricato su Google AI storage
2. Processing asincrono (state: PROCESSING)
3. Attesa fino a state: ACTIVE
4. URI disponibile per inference

### Wait for Processing

```python
# llm_analyzer.py:33-41
async def wait_for_files_active(files):
    while file.state.name == "PROCESSING":
        await asyncio.sleep(2)  # Check ogni 2 secondi

    if file.state.name != "ACTIVE":
        raise Exception("Processing failed")
```

### Prompt Engineering

**Struttura Prompt** (llm_analyzer.py:48-113):

```
┌────────────────────────────────────────┐
│ ROLE: CrossFit L3 Expert               │
├────────────────────────────────────────┤
│ INPUTS:                                │
│ - Knowledge Base (movement criteria)   │
│ - User Info (age, weight, etc.)       │
│ - Video (analyzed by Gemini)          │
├────────────────────────────────────────┤
│ REQUIRED OUTPUT:                       │
│ 1. Overview of Key Errors             │
│ 2. Biomechanical Rationale            │
│ 3. Actionable Corrections             │
│ 4. Regressions (if score < 80)        │
│ 5. Progressions (if score >= 80)      │
│ 6. Mobility Exercises (1-2)           │
│ 7. Conclusion                          │
├────────────────────────────────────────┤
│ FORMAT: Markdown                       │
│ SCHEMA: JSON enforced                  │
└────────────────────────────────────────┘
```

### JSON Schema Enforcement

```python
# llm_analyzer.py:121-140
response_schema = glm.Schema(
    type=glm.Type.OBJECT,
    required=["result", "form", "speed", "stability"],
    properties={
        "result": glm.Schema(type=glm.Type.STRING),
        "form": glm.Schema(type=glm.Type.NUMBER),
        "speed": glm.Schema(type=glm.Type.NUMBER),
        "stability": glm.Schema(type=glm.Type.NUMBER)
    }
)
```

**Esempio Output**:
```json
{
  "result": "**Overview**\n- Hip shoots up too early\n- Bar path deviates...",
  "form": 75,
  "speed": 82,
  "stability": 88
}
```

### Chat Session

```python
# llm_analyzer.py:155-165
chat_session = model.start_chat(
    history=[{
        "role": "user",
        "parts": [uploaded_video_file]
    }]
)

response = chat_session.send_message(prompt)
return response.text  # JSON string
```

### Error Handling

```python
try:
    response = analyze_video_with_llm(...)
except Exception as e:
    logger.error(f"LLM analysis failed: {e}")
    raise HTTPException(500, detail={
        "error_type": type(e).__name__,
        "error_message": str(e)
    })
```

---

## 10. CRITERI DI MOVIMENTO

### File: movement_criteria.py

**Dimensione**: 127 KB (3.476 righe)

**Struttura**:
```python
MOVEMENT_CRITERIA = {
    "deadliftConventional": [...],
    "sumoDeadlift": [...],
    "overheadLunge": [...],
    # ... 35+ movimenti
}
```

### Contenuto per Movimento

Ogni movimento include:

1. ✅ **Perfect Form**: Setup, esecuzione, lockout
2. ✅ **Important Cues**: Verbal cues per atleta
3. ✅ **Regressions**: Versioni semplificate
4. ✅ **Progressions**: Versioni avanzate
5. ✅ **Mobility Exercises**: Stretching specifici

### Esempio: Deadlift Conventional

```python
"deadliftConventional": [
    # PERFECT FORM
    "Initial Setup:",
    "Feet hip-width apart, barbell over mid-foot.",
    "Grip just outside the knees, arms extended straight.",

    "Lift-Off (First Pull):",
    "Drive through heels, maintaining neutral spine.",

    "Lockout (Final Position):",
    "Stand fully erect, shoulders pulled back.",

    # CUES
    "Important cues:",
    "'Chest up, hips back.'",
    "'Push the ground away.'",

    # REGRESSIONS
    "Regression: Elevated Deadlift (Blocks or Plates):",
    "Beneficial for beginners to reduce range of motion.",

    # PROGRESSIONS
    "Progression: Deficit Deadlift:",
    "Performing while standing on elevated surface.",

    # MOBILITY
    "Hamstring Stretch with Band (PNF Stretching):",
    "Essential for hamstring flexibility."
]
```

### Movimenti Disponibili (35+)

| Categoria | Movimenti |
|-----------|-----------|
| **Deadlifts** | conventional, sumo |
| **Squats** | back, front, overhead, air |
| **Olympic Lifts** | clean, snatch, jerk (+ varianti) |
| **Presses** | shoulder, push press, push jerk |
| **Gymnastics** | pull-up, muscle-up, HSPU, handstand walk |
| **Metabolic** | burpee, box jump, wall ball, thruster |
| **Skills** | rope climb, toes-to-bar |

### Come Aggiungere Nuovo Movimento

```python
# movement_criteria.py
MOVEMENT_CRITERIA["myNewMovement"] = [
    "Perfect Form",
    "Step 1: ...",
    "Step 2: ...",

    "Important cues:",
    "'Cue 1'",

    "Regressions and Progressions",
    "Regression: Easier version",
    "Progression: Harder version",

    "Mobility Exercises",
    "Exercise 1: Description"
]
```

```python
# config.py - Movements.MOVEMENTS
"myNewMovement": {
    'angles': ['knee', 'hip', 'elbow'],
    'object': 'barbell'  # or None, 'box', 'ball'
}
```

---

## 11. CONFIGURAZIONI E PARAMETRI

### File: config.py

#### Colors Class

```python
class Colors:
    WHITE = (255, 255, 255)       # Skeleton lines
    GRAY = (128, 128, 128)        # Shadows
    BLACK = (0, 0, 0)
    ORANGE = (0, 165, 255)        # Left side points (BGR!)
    LIGHT_BLUE = (255, 191, 0)    # Right side points (BGR!)
    GREEN = (0, 255, 0)           # Objects (barbell/box)
    RED = (0, 0, 255)
```

**⚠️ IMPORTANTE**: OpenCV usa formato **BGR**, non RGB!

#### Style Class

```python
class Style:
    LINE_THICKNESS = 2     # Skeleton lines
    POINT_SIZE = 5         # Joint circles
    FONT_SCALE = 0.7       # Text size
```

#### PoseConfig Class

```python
class PoseConfig:
    MIN_DETECTION_CONFIDENCE = 0.8   # 80%
    MIN_TRACKING_CONFIDENCE = 0.5    # 50%
    BARBELL_DISTANCE_THRESHOLD = 40  # pixels
```

#### VideoConfig Class

```python
class VideoConfig:
    OUTPUT_PATH = 'output/output_video.mp4'
    FPS = 30.0
```

#### Movements Class

```python
class Movements:
    MOVEMENTS = {
        'deadliftConventional': {
            'angles': ['knee', 'hip'],
            'object': 'barbell'
        },
        'boxJump': {
            'angles': ['knee', 'hip'],
            'object': 'box'
        },
        'burpee': {
            'angles': ['elbow', 'knee', 'hip'],
            'object': None
        }
    }
```

**Oggetti Supportati**:
- `'barbell'`: YOLO barbell detection
- `'box'`: YOLO box detection
- `'ball'`: YOLO ball detection
- `None`: Solo pose detection

---

## 12. DEPLOYMENT SU GOOGLE CLOUD RUN

### Docker Configuration

**File**: `Dockerfile`

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libgl1-mesa-glx \
    libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy YOLO models
COPY models/barbell.pt /app/models/
COPY models/box.pt /app/models/
COPY models/ball.pt /app/models/

# Copy application code
COPY . .

# Create temp directories
RUN mkdir -p /tmp/uploads /tmp/output

ENV PORT 8080

CMD exec uvicorn server:app --host 0.0.0.0 --port ${PORT}
```

### Cloud Build Configuration

**File**: `cloudbuild.yaml`

```yaml
steps:
  # Build Docker image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/movement-analysis', '.']

  # Push to Container Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/movement-analysis']

  # Deploy to Cloud Run
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - 'run'
      - 'deploy'
      - 'movement-analysis'
      - '--image=gcr.io/$PROJECT_ID/movement-analysis'
      - '--region=us-central1'
      - '--platform=managed'
```

### Cloud Run Configuration

**Project**: `peak-ascent-452414-k2`
**Service Name**: `movement-analysis`
**Region**: `us-central1`

**Container Settings**:
```yaml
spec:
  containerConcurrency: 80        # Max concurrent requests
  timeoutSeconds: 3600            # 1 hour timeout
  serviceAccountName: 250284968641-compute@developer.gserviceaccount.com

  containers:
  - name: movement-analysis-1
    image: gcr.io/peak-ascent-452414-k2/movement-analysis

    ports:
    - containerPort: 8080

    env:
    - name: CLOUD_STORAGE_BUCKET
      value: movement-analysis-videos
    - name: GEMINI_API_KEY
      value: AIzaSyC7QxjlBPGTec_wpRmCxGwFj9QV7P-j6dQ

    resources:
      limits:
        cpu: '8'                  # 8 vCPU
        memory: 16Gi              # 16 GB RAM

    startupProbe:
      timeoutSeconds: 240
      periodSeconds: 240
      failureThreshold: 1
      tcpSocket:
        port: 8080
```

### Build e Deploy Manuale

**1. Build Docker Image**:
```bash
cd /path/to/movement-analysis-code

docker build -t gcr.io/peak-ascent-452414-k2/movement-analysis:latest .
```

**2. Push to Google Container Registry**:
```bash
docker push gcr.io/peak-ascent-452414-k2/movement-analysis:latest
```

**3. Deploy to Cloud Run**:
```bash
gcloud run deploy movement-analysis \
  --image gcr.io/peak-ascent-452414-k2/movement-analysis:latest \
  --region us-central1 \
  --platform managed \
  --memory 16Gi \
  --cpu 8 \
  --timeout 3600 \
  --concurrency 80 \
  --set-env-vars CLOUD_STORAGE_BUCKET=movement-analysis-videos,GEMINI_API_KEY=your_gemini_api_key_here
```

**4. Verifica Deploy**:
```bash
gcloud run services describe movement-analysis \
  --region us-central1 \
  --format yaml
```

### Auto-Deploy con Cloud Build

**Trigger Setup**:
1. Vai a Cloud Build → Triggers
2. Crea trigger da repository (se disponibile)
3. Ogni push a `main` → build automatico

**Build Trigger YAML**:
```yaml
name: movement-analysis-deploy
filename: cloudbuild.yaml
includedFiles:
  - "**/*.py"
  - "Dockerfile"
  - "requirements.txt"
substitutions:
  _REGION: us-central1
  _SERVICE_NAME: movement-analysis
```

---

## 13. MODELLI YOLO CUSTOM

### File dei Modelli

**Directory**: `models/`

```
models/
├── barbell.pt  (~6 MB)  # Barbell detection
├── box.pt      (~6 MB)  # Box jump detection
└── ball.pt     (~6 MB)  # Medicine ball detection
```

### Barbell Model

**File**: `models/barbell.pt`

**Classes** (3):
```python
0: weights      # Dischi bilanciere
1: barbell      # Barra caricata
2: empty        # Barra vuota
```

**Training Data**: Custom dataset CrossFit
**Architecture**: YOLOv8 (Ultralytics)
**Input Size**: 640x640
**Confidence Threshold**: 0.5 (50%)

**Logica di Validazione**:
```python
# object_detector.py:86-109
if weights_inside == 2:
    # Barbell valido solo se contiene esattamente 2 weights
    valid_detections.append(barbell)
```

### Box Model

**File**: `models/box.pt`

**Classes** (1):
```python
0: box  # Box per box jumps
```

**Uso**: Esercizi come boxJump

### Ball Model

**File**: `models/ball.pt`

**Classes** (1):
```python
0: ball  # Medicine ball
```

**Uso**: Esercizi come wallBall

### Come Usare i Modelli

```python
# Automatico in base al movimento
movement_config = Movements.get_movement_config('deadliftConventional')
object_type = movement_config['object']  # 'barbell'

if object_type in ['barbell', 'ball', 'box']:
    detector = ObjectDetector(object_type)
    detections = detector.detect(frame)
```

### Re-training Modelli

**Requisiti**:
- Dataset annotato (Roboflow, LabelImg)
- GPU per training (Google Colab, local GPU)
- Ultralytics YOLO framework

**Training Script**:
```python
from ultralytics import YOLO

model = YOLO('yolov8n.pt')  # Pretrained base

results = model.train(
    data='dataset.yaml',     # Dataset config
    epochs=100,
    imgsz=640,
    batch=16,
    name='barbell_custom'
)

model.export(format='pt')    # Export to .pt
```

**Dataset YAML**:
```yaml
# dataset.yaml
path: /path/to/dataset
train: images/train
val: images/val

names:
  0: weights
  1: barbell
  2: empty
```

---

## 14. STORAGE E FILE MANAGEMENT

### Google Cloud Storage

**Bucket Name**: `movement-analysis-videos`

**Struttura**:
```
movement-analysis-videos/
└── processed_videos/
    ├── 20250111_123456_processed.mp4
    ├── 20250111_134521_processed.mp4
    └── ...
```

**Classe Manager**: `storage_utils.py`

```python
class CloudStorageManager:
    def __init__(self):
        self.client = storage.Client()
        self.bucket_name = 'movement-analysis-videos'
        self.bucket = self.client.bucket(self.bucket_name)

    def upload_video(self, source_path, dest_name):
        blob = self.bucket.blob(dest_name)
        blob.upload_from_filename(source_path)
        blob.make_public()
        return blob.public_url
```

**URL Pubblici**:
```
https://storage.googleapis.com/movement-analysis-videos/processed_videos/video.mp4
```

### File Temporanei

**Directory**:
```
/tmp/uploads/    # Input videos
/tmp/output/     # Processed videos (unused, ora su GCS)
```

**Lifecycle**:
1. Upload request → Save to `/tmp/uploads/`
2. Processing → Read from `/tmp/uploads/`
3. Output → Upload to GCS
4. Cleanup → Delete from `/tmp/uploads/`

**Cleanup Logic** (server.py:213-228):
```python
finally:
    for path in [temp_input_path, temp_output_path]:
        if os.path.exists(path):
            try:
                os.remove(path)
                logger.info(f"Removed: {path}")
            except Exception as e:
                logger.error(f"Cleanup error: {e}")
```

### Video Loader

**Classe**: `video_loader.py`

```python
class VideoLoader:
    def __init__(self, video_path, output_path):
        self.cap = cv2.VideoCapture(video_path)
        self.fps = self.cap.get(cv2.CAP_PROP_FPS)
        self.width = int(self.cap.get(cv2.CAP_PROP_FRAME_WIDTH))
        self.height = int(self.cap.get(cv2.CAP_PROP_FRAME_HEIGHT))

        fourcc = cv2.VideoWriter_fourcc(*'mp4v')
        self.out = cv2.VideoWriter(
            output_path, fourcc, self.fps, (self.width, self.height)
        )

    def read_frame(self):
        return self.cap.read()

    def write_frame(self, frame):
        self.out.write(frame)

    def release(self):
        self.cap.release()
        self.out.release()
```

---

## 15. LOGGING E MONITORING

### Logger Configuration

**File**: `logger_config.py`

```python
from loguru import logger

logger.add(
    "logs/app_{time}.log",
    rotation="500 MB",
    retention="10 days",
    level="INFO"
)
```

**Features**:
- Rotation automatica ogni 500 MB
- Retention: 10 giorni
- Livelli: DEBUG, INFO, WARNING, ERROR
- Formato: `{time} | {level} | {message}`

### Request ID Tracking

**Middleware** (server.py:40-72):
```python
class RequestIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        request_id = str(uuid.uuid4())
        request.state.request_id = request_id

        logger_with_context = logging.LoggerAdapter(
            logger, {"request_id": request_id}
        )

        logger_with_context.info(f"Request started: {request.url}")
        response = await call_next(request)
        logger_with_context.info(f"Request completed: {response.status_code}")

        response.headers["X-Request-ID"] = request_id
        return response
```

**Output Example**:
```
2026-01-11 12:34:56 | INFO | [request_id: 550e8400...] Request started: POST /analyze
2026-01-11 12:35:42 | INFO | [request_id: 550e8400...] Video processing completed
2026-01-11 12:36:15 | INFO | [request_id: 550e8400...] LLM analysis completed
2026-01-11 12:36:20 | INFO | [request_id: 550e8400...] Request completed: 200
```

### Structured Logging

**Example** (llm_analyzer.py):
```python
logger_with_context = logging.LoggerAdapter(
    logger, {
        "request_id": request_id,
        "movement": movement_name,
        "video_size": file_size_mb
    }
)

logger_with_context.info(f"Starting LLM analysis")
logger_with_context.error(
    f"Analysis failed: {error}",
    extra={"error_context": {...}}
)
```

### Cloud Logging

**Cloud Run → Logs Explorer**

Query esempio:
```
resource.type="cloud_run_revision"
resource.labels.service_name="movement-analysis"
severity>=ERROR
```

**Metriche Disponibili**:
- Request count
- Request latency (P50, P95, P99)
- Error rate
- CPU utilization
- Memory usage
- Container instance count

---

## 16. COME MODIFICARE

### 16.1 Modificare Colori Skeleton

**File**: `config.py`

```python
# PRIMA (Originale)
class Colors:
    ORANGE = (0, 165, 255)      # Left side
    LIGHT_BLUE = (255, 191, 0)  # Right side

# DOPO (Esempio: Tutto Rosso)
class Colors:
    ORANGE = (0, 0, 255)        # BGR Red
    LIGHT_BLUE = (0, 0, 255)    # BGR Red
```

**Deploy**:
1. Modifica `config.py`
2. Build: `docker build -t gcr.io/.../movement-analysis:v2 .`
3. Push: `docker push gcr.io/.../movement-analysis:v2`
4. Deploy: `gcloud run deploy movement-analysis --image ...v2`

---

### 16.2 Modificare Spessore Skeleton

**File**: `config.py`

```python
# PRIMA
class Style:
    LINE_THICKNESS = 2
    POINT_SIZE = 5

# DOPO (Più spesso e visibile)
class Style:
    LINE_THICKNESS = 4
    POINT_SIZE = 8
```

---

### 16.3 Mostrare Punti Nascosti (es. Dita)

**File**: `visualizer.py`

```python
# PRIMA (Esclusi)
self.excluded_landmarks = {
    self.mp_pose.PoseLandmark.LEFT_PINKY,
    self.mp_pose.PoseLandmark.LEFT_INDEX,
    # ...
}

# DOPO (Inclusi - commenta o rimuovi)
self.excluded_landmarks = {
    # self.mp_pose.PoseLandmark.LEFT_PINKY,  # Ora visibile!
    # self.mp_pose.PoseLandmark.LEFT_INDEX,
}
```

---

### 16.4 Cambiare Modello Gemini

**File**: `llm_analyzer.py`

```python
# PRIMA (Gemini 2.0 Flash)
model = genai.GenerativeModel(
    model_name="gemini-2.0-flash"
)

# OPZIONI:
# gemini-1.5-pro         # Più accurato, più costoso
# gemini-1.5-flash       # Bilanciato
# gemini-1.5-flash-8b    # Più economico
```

---

### 16.5 Modificare Temperature Gemini

**File**: `llm_analyzer.py`

```python
# PRIMA (Creativo)
generation_config = {
    "temperature": 1,  # Range: 0-2
}

# DOPO (Più deterministico)
generation_config = {
    "temperature": 0.3,  # Meno variazione
}
```

**Effetti**:
- `0.0-0.3`: Output deterministico, ripetibile
- `0.4-0.7`: Bilanciato
- `0.8-1.2`: Creativo, vario
- `1.3-2.0`: Molto creativo (rischio incoerenza)

---

### 16.6 Aggiungere Nuovo Esercizio

**Step 1**: Aggiungi criteri (movement_criteria.py)
```python
MOVEMENT_CRITERIA["pistolSquat"] = [
    "Perfect Form",
    "Setup: Single leg, other leg extended forward",
    "Execution: Descend controlled, knee tracking toes",
    "Lockout: Full hip and knee extension",

    "Important cues:",
    "'Keep chest up'",
    "'Drive through heel'",

    "Regressions and Progressions",
    "Regression: Box-assisted pistol squat",
    "Progression: Weighted pistol squat",

    "Mobility Exercises",
    "Hip flexor stretch: Essential for extended leg",
    "Ankle mobility: Critical for depth"
]
```

**Step 2**: Aggiungi configurazione (config.py)
```python
class Movements:
    MOVEMENTS = {
        # ... existing movements
        'pistolSquat': {
            'angles': ['knee', 'hip'],
            'object': None
        }
    }
```

**Step 3**: Deploy e testa
```bash
# Test locale
python main.py input.mp4 output.mp4 pistolSquat

# Deploy
docker build -t ... && docker push ... && gcloud run deploy ...
```

---

### 16.7 Modificare Confidence Threshold YOLO

**File**: `object_detector.py`

```python
# PRIMA (50% confidence)
self.conf_threshold = 0.5

# DOPO (Più conservativo)
self.conf_threshold = 0.7  # Solo detection molto sicure

# DOPO (Più permissivo)
self.conf_threshold = 0.3  # Più detection, possibili falsi positivi
```

---

### 16.8 Aumentare Resources Cloud Run

**Console Cloud Run** → Service → Edit & Deploy New Revision

```yaml
resources:
  limits:
    cpu: '16'      # Da 8 a 16 vCPU (max)
    memory: 32Gi   # Da 16GB a 32GB (max)
```

**O via gcloud**:
```bash
gcloud run services update movement-analysis \
  --region us-central1 \
  --cpu 16 \
  --memory 32Gi
```

**Effetti**:
- ✅ Processing più veloce
- ✅ Più richieste concorrenti
- ❌ Costi maggiori

---

## 17. TROUBLESHOOTING

### Problema: Video Processing Timeout

**Sintomo**: Request timeout dopo 3600 secondi

**Cause**:
- Video troppo lungo (>10 minuti)
- FPS troppo alto (>60 FPS)
- Risoluzione troppo alta (>1080p)

**Soluzioni**:

1. **Aumentare timeout** (max Cloud Run):
```bash
gcloud run services update movement-analysis \
  --timeout 3600  # Already at max
```

2. **Pre-processare video lato Laravel**:
```php
// MergeChunksJob.php - convertVideo()
// Già implementato: riduce a 480p
$scaleFilter = "scale=854:-2"
```

3. **Ridurre FPS**:
```python
# video_loader.py
self.fps = min(self.cap.get(cv2.CAP_PROP_FPS), 30)  # Max 30 FPS
```

---

### Problema: Gemini API Quota Exceeded

**Sintomo**:
```
Error 429: Resource has been exhausted (e.g. check quota)
```

**Cause**:
- Troppi video processati in poco tempo
- Video troppo lunghi (>1 ora)
- Quota giornaliera superata

**Soluzioni**:

1. **Controlla quota** (Google AI Studio):
   - https://aistudio.google.com/app/apikey
   - Quota attuale visibile per API key

2. **Implementa rate limiting**:
```python
# server.py
from slowapi import Limiter

limiter = Limiter(key_func=get_remote_address)

@app.post("/analyze")
@limiter.limit("10/minute")  # Max 10 requests/min
async def analyze_movement(...):
```

3. **Switch to paid tier** (se in free tier):
   - Google Cloud Console → Billing
   - Enable billing su progetto

---

### Problema: MediaPipe Non Rileva Persona

**Sintomo**: Skeleton non appare sul video

**Cause**:
- Persona troppo piccola nel frame
- Illuminazione scarsa
- Persona parzialmente nascosta
- Confidence threshold troppo alta

**Soluzioni**:

1. **Abbassa confidence**:
```python
# pose_detector.py
min_detection_confidence=0.5,  # Da 0.8 a 0.5
min_tracking_confidence=0.3,   # Da 0.5 a 0.3
```

2. **Preprocessing video**:
```python
# main.py - dopo read_frame()
frame = cv2.convertScaleAbs(frame, alpha=1.2, beta=20)  # Aumenta contrasto
```

3. **Usa model_complexity=2**:
```python
# pose_detector.py
model_complexity=2,  # Full model (più lento ma accurato)
```

---

### Problema: YOLO Non Rileva Oggetti

**Sintomo**: Barbell/box non evidenziati

**Cause**:
- Oggetto fuori frame
- Angolazione inusuale
- Confidence threshold troppo alta
- Modello non trainato per quel tipo

**Soluzioni**:

1. **Abbassa threshold**:
```python
# object_detector.py
self.conf_threshold = 0.3  # Da 0.5 a 0.3
```

2. **Re-train modello** con più varianti angolazioni

3. **Disabilita validazione 2-weights**:
```python
# object_detector.py:106
# Commenta questa condizione per accettare qualsiasi detection
# if weights_inside == 2:
```

---

### Problema: Container Out of Memory

**Sintomo**:
```
Error: Container instance exceeded memory limit
```

**Cause**:
- Video troppo grande in memoria
- Memory leak
- Troppi frame processati contemporaneamente

**Soluzioni**:

1. **Aumenta memory limit**:
```bash
gcloud run services update movement-analysis \
  --memory 32Gi  # Da 16Gi a 32Gi
```

2. **Processa batch di frame**:
```python
# main.py
BATCH_SIZE = 10
frames_batch = []

while True:
    success, frame = video_loader.read_frame()
    frames_batch.append(frame)

    if len(frames_batch) >= BATCH_SIZE:
        # Process batch
        for f in frames_batch:
            # ... process
        frames_batch.clear()  # Free memory
```

3. **Esplicita garbage collection**:
```python
import gc

# Dopo processing video
gc.collect()
```

---

### Problema: Gemini Ritorna Errore Schema

**Sintomo**:
```
Error: Response does not match expected schema
```

**Cause**:
- Gemini non segue JSON schema
- Field mancanti (form, speed, stability)
- Tipo errato (string invece di number)

**Soluzioni**:

1. **Fallback parsing**:
```python
# llm_analyzer.py
import json

try:
    response_json = json.loads(response.text)

    # Validate e fornisci default
    if 'form' not in response_json:
        response_json['form'] = 70
    if 'speed' not in response_json:
        response_json['speed'] = 70
    # ...
except json.JSONDecodeError:
    logger.error("Invalid JSON from Gemini")
    raise
```

2. **Prompt più esplicito**:
```python
prompt += """
CRITICAL: Your response MUST be valid JSON with ALL these fields:
- result (string)
- form (number 70-100)
- speed (number 70-100)
- stability (number 70-100)
"""
```

---

### Problema: Cloud Storage Upload Fails

**Sintomo**:
```
Error: 403 Forbidden - Insufficient permission
```

**Cause**:
- Service account senza permessi
- Bucket non esiste
- Bucket in progetto diverso

**Soluzioni**:

1. **Verifica service account**:
```bash
gcloud iam service-accounts describe \
  250284968641-compute@developer.gserviceaccount.com
```

2. **Aggiungi permessi**:
```bash
gsutil iam ch \
  serviceAccount:250284968641-compute@developer.gserviceaccount.com:roles/storage.objectAdmin \
  gs://movement-analysis-videos
```

3. **Verifica bucket**:
```bash
gsutil ls gs://movement-analysis-videos
```

---

## 18. COSTI E PERFORMANCE

### Costi Google Cloud Run

**Pricing** (us-central1):
- **CPU**: $0.00002400 per vCPU-second
- **Memory**: $0.00000250 per GB-second
- **Requests**: $0.40 per million

**Configurazione Attuale**:
- 8 vCPU
- 16 GB RAM
- Timeout: 3600s (1 ora max)

**Costo per Request** (esempio: 60 sec processing):
```
CPU: 8 vCPU × 60 sec × $0.000024 = $0.01152
Memory: 16 GB × 60 sec × $0.0000025 = $0.0024
Request: $0.0000004
-------------------------------------------
TOTALE: ~$0.014 per request (60 sec)
```

**Stima Mensile** (1000 video/mese, 60 sec avg):
```
1000 × $0.014 = $14/mese
```

### Costi Gemini AI

**Gemini 2.0 Flash Pricing**:
- **Input**: $0.075 per 1M token
- **Output**: $0.30 per 1M token

**Token per Video** (stima):
- Input: ~10-20K token (video 30-60 sec)
- Output: ~2-3K token (analisi dettagliata)

**Costo per Video**:
```
Input: 15K token × $0.075 / 1M = $0.001125
Output: 2.5K token × $0.30 / 1M = $0.00075
-------------------------------------------
TOTALE: ~$0.0019 per video
```

**Stima Mensile** (1000 video/mese):
```
1000 × $0.0019 = $1.90/mese
```

### Costi Cloud Storage

**Pricing**:
- **Storage**: $0.020 per GB/mese (Standard)
- **Network egress**: $0.12 per GB (worldwide)

**Video Output Size**: ~5-10 MB

**Stima Mensile** (1000 video, 7.5 MB avg):
```
Storage: 1000 × 7.5MB = 7.5GB × $0.020 = $0.15/mese
Egress: 1000 download × 7.5MB = 7.5GB × $0.12 = $0.90/mese
-------------------------------------------
TOTALE: ~$1.05/mese
```

### Costo Totale Stimato

**1000 video al mese**:
```
Cloud Run:     $14.00
Gemini AI:     $ 1.90
Cloud Storage: $ 1.05
-------------------
TOTALE:        $16.95/mese
```

**Per singolo video**: ~$0.017 (1.7 centesimi)

### Performance Metrics

**Latency Breakdown** (video 30 sec, 1080p):

| Fase | Tempo | % Totale |
|------|-------|----------|
| Video upload | 2-3 sec | 4% |
| MediaPipe processing | 15-20 sec | 30% |
| YOLO detection | 5-8 sec | 12% |
| Gemini upload | 3-5 sec | 8% |
| Gemini analysis | 20-30 sec | 42% |
| Cloud Storage upload | 2-3 sec | 4% |
| **TOTALE** | **47-69 sec** | **100%** |

**Throughput**:
- **Serial**: ~1 video ogni 60 secondi
- **Parallel (80 concurrency)**: ~80 video/minuto teorici
- **Realistic**: ~20-30 video/minuto con auto-scaling

**Optimizations**:

1. **Ridurre risoluzione input**:
   - 1080p → 720p: -30% tempo processing
   - 720p → 480p: -50% tempo processing

2. **Parallelizzare meglio**:
   ```python
   # Già implementato in server.py:137-143
   tasks = [
       process_video_async(...),      # MediaPipe + YOLO
       analyze_video_with_llm(...)    # Gemini
   ]
   await asyncio.gather(*tasks)
   ```

3. **Cache Gemini results** (se stessi video):
   - Redis cache con request hash
   - TTL: 1 ora

4. **Batch processing** YOLO:
   - Process 10 frame insieme invece che 1 alla volta
   - Trade-off: più memory, meno tempo

---

## 📝 APPENDICI

### A. Environment Variables

**Cloud Run Container**:
```bash
CLOUD_STORAGE_BUCKET=movement-analysis-videos
GEMINI_API_KEY=your_gemini_api_key_here
PORT=8080
```

**Locale (.env)**:
```bash
CLOUD_STORAGE_BUCKET=movement-analysis-videos
GEMINI_API_KEY=your-api-key-here
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

---

### B. Dipendenze Complete

**requirements.txt**:
```
fastapi==0.115.7
uvicorn==0.34.0
python-multipart==0.0.20
aiofiles==24.1.0
python-dotenv==1.0.1
opencv-python==4.11.0.86
mediapipe==0.10.21
ultralytics==8.3.51
google-generativeai==0.8.4
numpy==1.26.4
pillow==10.4.0
python-socketio==5.11.4
requests==2.32.3
tqdm==4.67.1
loguru==0.7.3
google-cloud-storage==2.19.0
```

---

### C. API Response Examples

**Success Response**:
```json
{
  "status": "success",
  "request_id": "550e8400-e29b-41d4-a716-446655440000",
  "video_url": "https://storage.googleapis.com/movement-analysis-videos/processed_videos/20250111_123456_processed.mp4",
  "llm_analysis": "{\"result\":\"**Overview of Key Errors**\\n- Hip shoots up before shoulders\\n- Bar drifts forward\\n- Insufficient lat engagement\\n\\n**Biomechanical Rationale**\\nWhen hips rise faster than shoulders, the back angle becomes more horizontal, forcing the lower back to compensate...\\n\\n**Actionable Corrections**\\n1. Focus on pushing the floor away while maintaining chest up\\n2. Engage lats by 'bending the bar' cue\\n3. Practice Romanian deadlifts to feel proper hip hinge\\n\\n**Regressions** (Score: 75)\\n- Elevated deadlift from blocks (2-4 inches)\\n- Reduces range of motion, helps establish proper positions\\n\\n**Mobility Exercises**\\n1. Hamstring PNF stretching: 3×60sec per leg\\n2. Thoracic spine foam rolling: 3×90sec\\n\\n**Conclusion**\\nMain focus: Keep bar closer to body throughout lift. Film next attempt from side angle to verify bar path improvement.\",\"form\":75,\"speed\":82,\"stability\":88}"
}
```

**Error Response**:
```json
{
  "message": "Task execution failed",
  "error_type": "ValueError",
  "error_details": "Invalid movement name: squatConventional"
}
```

---

### D. Useful Commands

**Local Development**:
```bash
# Install dependencies
pip install -r requirements.txt

# Run locally
uvicorn server:app --reload --port 8080

# Test endpoint
curl -X POST http://localhost:8080/analyze \
  -F "movement=deadliftConventional" \
  -F "video=@test_video.mp4" \
  -F "userInfo=age:35"
```

**Docker**:
```bash
# Build
docker build -t movement-analysis .

# Run locally
docker run -p 8080:8080 \
  -e GEMINI_API_KEY=your-key \
  movement-analysis

# Shell into container
docker run -it movement-analysis /bin/bash
```

**Cloud Run**:
```bash
# View logs
gcloud run services logs read movement-analysis \
  --region us-central1 \
  --limit 100

# Describe service
gcloud run services describe movement-analysis \
  --region us-central1

# List revisions
gcloud run revisions list \
  --service movement-analysis \
  --region us-central1
```

**Cloud Storage**:
```bash
# List videos
gsutil ls gs://movement-analysis-videos/processed_videos/

# Download video
gsutil cp gs://movement-analysis-videos/processed_videos/video.mp4 .

# Delete old videos (>30 days)
gsutil -m rm gs://movement-analysis-videos/processed_videos/**
```

---

### E. Link Utili

**Documentazione**:
- [FastAPI](https://fastapi.tiangolo.com/)
- [MediaPipe Pose](https://google.github.io/mediapipe/solutions/pose.html)
- [Ultralytics YOLO](https://docs.ultralytics.com/)
- [Google Gemini API](https://ai.google.dev/docs)
- [Cloud Run](https://cloud.google.com/run/docs)

**Console**:
- [Google Cloud Console](https://console.cloud.google.com/)
- [Cloud Run Service](https://console.cloud.google.com/run?project=peak-ascent-452414-k2)
- [Container Registry](https://console.cloud.google.com/gcr/images/peak-ascent-452414-k2)
- [Cloud Storage](https://console.cloud.google.com/storage/browser/movement-analysis-videos)
- [Google AI Studio](https://aistudio.google.com/)

---

## 🎯 CONCLUSIONI

Questo backend Python è un microservizio complesso che combina:

✅ **Computer Vision** (MediaPipe, YOLO)
✅ **Artificial Intelligence** (Gemini 2.0 Flash)
✅ **Cloud Infrastructure** (Cloud Run, Cloud Storage)
✅ **Web Framework** (FastAPI)

**Punti di Forza**:
- 🚀 Scalabilità automatica
- 🎯 Analisi accurata movimenti
- 💰 Costi contenuti (~$0.017/video)
- 🔧 Facilmente modificabile
- 📊 Logging dettagliato

**Aree di Miglioramento**:
- ⚡ Ottimizzazione performance (batch processing)
- 💾 Caching Gemini results
- 🔒 Rate limiting più robusto
- 📈 Metriche avanzate (Prometheus)
- 🧪 Test suite automatizzati

**Next Steps**:
1. Implementare test automatici (pytest)
2. Aggiungere health check endpoint
3. Ottimizzare processing video (GPU?)
4. Migliorare error handling Gemini
5. Dashboard monitoring (Grafana)

---

**Documentazione creata il**: 11 Gennaio 2026
**Versione Backend**: Latest (Gemini 2.0 Flash)
**Autore Documentazione**: Claude Sonnet 4.5

---

*Per domande o supporto, contattare il team di sviluppo WodVision.*
