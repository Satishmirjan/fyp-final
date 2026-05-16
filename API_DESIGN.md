# API Design Document
## GW ImpactAI Learning Platform & LearnAI — Complete API Reference

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture Summary](#architecture-summary)
3. [Flask REST API — GW ImpactAI Backend](#flask-rest-api--gw-impactai-backend)
   - [Base URL & CORS](#base-url--cors)
   - [Authentication](#authentication)
   - [Endpoints](#endpoints)
4. [Client-Side AI API — Gemini Service](#client-side-ai-api--gemini-service)
   - [processContent()](#processcontent)
   - [generateSchedule()](#generateschedule)
   - [speak()](#speak)
5. [Data Models & Schemas](#data-models--schemas)
6. [Error Handling](#error-handling)
7. [Environment Variables](#environment-variables)
8. [Internal Service Interfaces](#internal-service-interfaces)
9. [API Flow Diagrams](#api-flow-diagrams)

---

## Overview

The platform exposes **two distinct API surfaces**:

| API Surface | Type | Used By | Auth |
|---|---|---|---|
| **Flask REST API** | HTTP/JSON over REST | GW ImpactAI React frontend | JWT Bearer token |
| **Gemini Client API** | JavaScript service wrapper | Both React frontends | Gemini API key (client-side) |

```
User Browser
  ├─ GW ImpactAI React App
  │    ├─ → Flask Backend  (POST /summary · POST /api/login · GET /audio/:file)
  │    └─ → Google Gemini  (geminiService.js — processContent · speak · generateSchedule)
  │
  └─ LearnAI TypeScript App
       └─ → Google Gemini  (geminiService.ts — processContent · speak · generateSchedule)
```

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    GW ImpactAI Platform                         │
│                                                                 │
│  React Frontend (Port 5173)                                     │
│  ┌──────────────────┐    ┌─────────────────────────────────┐   │
│  │  React Pages     │    │  geminiService.js               │   │
│  │  (Upload, Quiz,  │    │  ┌───────────────────────────┐  │   │
│  │   Summary, etc.) │    │  │ processContent(text)       │  │   │
│  └────────┬─────────┘    │  │ generateSchedule(topic,wk) │  │   │
│           │              │  │ speak(text)                │  │   │
│           │ REST         │  └─────────────┬─────────────┘  │   │
│           │              └────────────────│─────────────────┘   │
│           ▼                               │ Gemini API          │
│  Flask Backend (Port 5000)                ▼                     │
│  ┌──────────────────────────────┐   Google Gemini API           │
│  │ POST  /summary               │   gemini-3-flash-preview      │
│  │ POST  /api/signup            │                               │
│  │ POST  /api/login             │                               │
│  │ GET   /audio/:filename       │                               │
│  └──────────────────────────────┘                               │
│           │                                                     │
│    ┌──────┴──────┐                                              │
│    ▼             ▼                                              │
│  MongoDB    HuggingFace Models                                  │
│  (users)    (BART · T5 · DistilBERT)                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Flask REST API — GW ImpactAI Backend

### Base URL & CORS

```
Base URL (development):  http://localhost:5000
Base URL (production):   https://<your-domain>/api

CORS Policy:
  origins: *  (all origins)
  allow_headers: *
  supports_credentials: true
```

### Authentication

The API uses **JWT (JSON Web Token)** Bearer authentication.

#### Token Format

```
Authorization: Bearer <jwt_token>
```

#### Token Properties

| Property | Value |
|---|---|
| Algorithm | HS256 |
| Expiry | 24 hours from login |
| Payload | `{ email: string, exp: timestamp }` |
| Secret | `SECRET_KEY` environment variable |

#### Protected Routes

Routes decorated with `@token_required` require a valid JWT. If the token is missing or invalid, the server returns `401 Unauthorized`.

```python
# Decorator behavior
def token_required(f):
    # Reads: Authorization: Bearer <token>
    # Decodes JWT → fetches user from MongoDB
    # Passes current_user to route handler
    # Returns 401 if token missing or invalid
```

---

## Endpoints

---

### POST /api/signup

Register a new user account.

**Auth required:** No

**Request**

```
Content-Type: application/json
```

```json
{
  "name":     "Jane Smith",
  "email":    "jane@example.com",
  "password": "securepassword123"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | No | Display name |
| `email` | string | Yes | Unique user email |
| `password` | string | Yes | Plain-text password (hashed server-side with pbkdf2:sha256) |

**Response — 201 Created**

```json
{
  "message": "User created successfully"
}
```

**Response — 400 Bad Request**

```json
{ "message": "Email and password required" }
```

```json
{ "message": "User already exists" }
```

```json
{ "message": "No data provided" }
```

**Response — 500 Internal Server Error**

```json
{ "message": "Database not configured" }
```

**Notes**
- Password is hashed using `werkzeug.security.generate_password_hash` (pbkdf2:sha256)
- Email uniqueness is enforced at the MongoDB document level
- Returns `201` on success regardless of whether MongoDB is connected (graceful degradation)

---

### POST /api/login

Authenticate an existing user and receive a JWT token.

**Auth required:** No

**Request**

```
Content-Type: application/json
```

```json
{
  "email":    "jane@example.com",
  "password": "securepassword123"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | string | Yes | Registered email address |
| `password` | string | Yes | User's plain-text password |

**Response — 200 OK**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "name":  "Jane Smith",
    "email": "jane@example.com"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `token` | string | JWT token — valid for 24 hours |
| `user.name` | string | User display name |
| `user.email` | string | User email address |

**Response — 400 Bad Request**

```json
{ "message": "No data provided" }
```

**Response — 401 Unauthorized**

```json
{ "message": "Invalid credentials" }
```

**Response — 500 Internal Server Error**

```json
{ "message": "Database not configured" }
```

**Frontend Usage**

```javascript
// AuthContext.jsx
const response = await fetch('/api/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password })
});
const data = await response.json();
localStorage.setItem('token', data.token);
```

---

### POST /summary

Process a PDF file and return AI-generated educational content.

**Auth required:** No  
**Content-Type:** `multipart/form-data`

**Request Fields**

| Field | Type | Required | Description |
|---|---|---|---|
| `file` | File (PDF) | Yes | The PDF document to process |
| `options` | JSON string | No | Feature toggles (see below) |

**Options JSON Schema**

```json
{
  "summary":    true,
  "flashcards": true,
  "quiz":       true,
  "audio":      false
}
```

| Option | Type | Default | Description |
|---|---|---|---|
| `summary` | boolean | false | Generate a text summary via BART model |
| `flashcards` | boolean | false | Generate Q&A flashcards via T5 + DistilBERT |
| `quiz` | boolean | false | Generate multiple-choice quiz from flashcards |
| `audio` | boolean | false | Convert summary to MP3 via gTTS |

**Processing Pipeline**

```
PDF File
  ↓  extract_text_from_pdf()       [PyMuPDF / fitz]
Raw Text
  ↓  split_into_chunks(max_words=700)
Text Chunks [chunk₁, chunk₂, ..., chunkₙ]
  ↓  summarize_large_text()        [facebook/bart-large-cnn]
Summary Text
  ├─ [if flashcards] generate_flashcards(max=10) [T5 + DistilBERT]
  ├─ [if quiz]       generate_quiz(num=5)         [wraps flashcards]
  └─ [if audio]      generate_speech()            [gTTS → MP3]
```

**Response — 200 OK**

```json
{
  "summary":   "The document covers...",
  "flashcards": [
    {
      "question": "What is machine learning?",
      "answer":   "A subset of AI that enables systems to learn from data."
    }
  ],
  "quiz": [
    {
      "id":             1,
      "question":       "What is machine learning?",
      "correct_answer": "A subset of AI that enables systems to learn from data.",
      "options": [
        "A subset of AI that enables systems to learn from data.",
        "A programming language for data analysis.",
        "None of the above (1)",
        "A database management system."
      ],
      "correctIndex": 0,
      "explanation":  "This specific detail was extracted directly from the uploaded material."
    }
  ],
  "audio_url": "/audio/output.mp3"
}
```

**Response Fields**

| Field | Type | Present when | Description |
|---|---|---|---|
| `summary` | string \| null | `options.summary = true` | Multi-paragraph BART summary |
| `flashcards` | array \| null | `options.flashcards = true` | Array of `{question, answer}` objects |
| `quiz` | array \| null | `options.quiz = true` | Array of multiple-choice question objects |
| `audio_url` | string \| null | `options.audio = true` + TTS success | Relative path to MP3 file |

**Flashcard Object Schema**

```json
{
  "question": "string — interrogative sentence ending with ?",
  "answer":   "string — extracted answer, max 25 words"
}
```

**Quiz Question Object Schema**

```json
{
  "id":             "number — 1-indexed",
  "question":       "string — the question text",
  "correct_answer": "string — correct answer text",
  "options":        ["string", "string", "string", "string"],
  "correctIndex":   "number — 0-3, index of correct answer in options[]",
  "explanation":    "string — explanation text"
}
```

**Response — 400 Bad Request**

```json
{ "error": "No file uploaded" }
```

```json
{ "error": "Empty or unreadable PDF" }
```

```json
{ "error": "Invalid options format: <detail>" }
```

**Response — 500 Internal Server Error**

```json
{
  "error":   "<exception message>",
  "message": "Internal Server Error"
}
```

**Frontend Usage (Upload.jsx)**

```javascript
const formData = new FormData();
formData.append('file', selectedFile);
formData.append('options', JSON.stringify({
  summary: true,
  flashcards: true,
  quiz: true,
  audio: false
}));

const response = await fetch('http://localhost:5000/summary', {
  method: 'POST',
  body: formData
});
const data = await response.json();
navigate('/summary', { state: data });
```

**Limits & Behavior**

| Constraint | Value |
|---|---|
| PDF chunk size | 700 words per chunk |
| BART max output length | 200 tokens |
| BART min output length | 60 tokens |
| Flashcard input limit | First 2000 chars of summary |
| QA context limit | First 3000 chars of summary |
| Max flashcards | 10 |
| Quiz questions | 5 (derived from flashcards) |
| Similarity dedup threshold | 85% (SequenceMatcher ratio) |
| Quiz distractor options | 3 distractors + 1 correct = 4 total |

---

### GET /audio/:filename

Serve a generated MP3 audio file.

**Auth required:** No

**URL Parameters**

| Parameter | Type | Description |
|---|---|---|
| `filename` | string | MP3 filename returned in `audio_url` from `/summary` |

**Example**

```
GET /audio/output.mp3
```

**Response — 200 OK**

```
Content-Type: audio/mpeg
Body: <binary MP3 data>
```

Served from the `tts_chunks/` directory on the server using `send_from_directory`.

**Response — 404 Not Found**

File not found in `tts_chunks/` directory.

**Frontend Usage**

```javascript
// Summary.jsx
const audioElement = new Audio(`http://localhost:5000${data.audio_url}`);
audioElement.play();
```

---

## Client-Side AI API — Gemini Service

Both platforms use a `geminiService` module that wraps the Google Gemini API. The GW ImpactAI version is JavaScript (`geminiService.js`); the LearnAI version is TypeScript (`geminiService.ts`). They share identical method signatures and response schemas.

**Configuration**

```javascript
// GW ImpactAI (geminiService.js)
import { GoogleGenAI, Type, Modality } from "@google/genai";

const apiKey = import.meta.env.VITE_GEMINI_API_KEY;
const ai = new GoogleGenAI({ apiKey });
```

```typescript
// LearnAI (geminiService.ts)
import { GoogleGenAI, Type, Modality } from "@google/genai";

const apiKey = process.env.API_KEY;
const ai = new GoogleGenAI({ apiKey });
```

**Models Used**

| Method | Model ID | Purpose |
|---|---|---|
| `processContent()` | `gemini-3-flash-preview` | Structured content generation |
| `generateSchedule()` | `gemini-3-flash-preview` | Teacher schedule generation |
| `speak()` | `gemini-3-flash-preview` | Text-to-speech audio output |

---

### processContent()

Generate a summary, flashcards, and quiz from educational text in a single Gemini API call.

**Signature**

```typescript
async processContent(text: string): Promise<LearningContent>
```

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `text` | string | Educational content to process (PDF extracted text or user-pasted text) |

**Gemini Request**

```javascript
ai.models.generateContent({
  model: 'gemini-3-flash-preview',
  contents: `Process this educational content. Provide a concise summary, 
             5 key flashcards (question/answer), and a 5-question multiple 
             choice quiz. Content: ${text}`,
  config: {
    responseMimeType: "application/json",
    responseSchema: { /* see schema below */ }
  }
})
```

**Response Schema (enforced by Gemini)**

```json
{
  "summary": "string",
  "flashcards": [
    {
      "question": "string",
      "answer":   "string"
    }
  ],
  "quiz": [
    {
      "question":     "string",
      "options":      ["string", "string", "string", "string"],
      "correctIndex": 0,
      "explanation":  "string"
    }
  ]
}
```

**Return Type**

```typescript
interface LearningContent {
  id:         string;    // generated client-side (Date.now())
  title:      string;    // first 50 chars of text
  summary:    string;
  flashcards: Flashcard[];
  quiz:       QuizQuestion[];
  timestamp:  number;
}
```

**Usage**

```javascript
// StudyRoom.jsx / StudyRoom.tsx
const content = await geminiService.processContent(pastedText);
setSavedContent(prev => [...prev, { ...content, id: Date.now() }]);
```

**Error Handling**

Throws on Gemini API error. Caller should wrap in `try/catch`.

---

### generateSchedule()

Generate a weekly teaching schedule for a given topic.

**Signature**

```typescript
async generateSchedule(topic: string, weeks: number): Promise<TeacherSchedule[]>
```

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `topic` | string | Subject/topic for the schedule (e.g., "Python Programming") |
| `weeks` | number | Number of weeks to plan |

**Gemini Request**

```javascript
ai.models.generateContent({
  model: 'gemini-3-flash-preview',
  contents: `Create a teaching schedule for "${topic}" spread over ${weeks} weeks.`,
  config: {
    responseMimeType: "application/json",
    responseSchema: { /* array of TeacherSchedule */ }
  }
})
```

**Response Schema**

```json
[
  {
    "week":        "Week 1",
    "topic":       "Introduction to Python",
    "objectives":  ["Understand variables", "Learn data types"],
    "activities":  ["Lecture: syntax basics", "Lab: Hello World exercise"]
  }
]
```

**Return Type**

```typescript
interface TeacherSchedule {
  week:       string;
  topic:      string;
  objectives: string[];
  activities: string[];
}
```

**Usage**

```javascript
// TeacherTools.jsx
const schedule = await geminiService.generateSchedule("Machine Learning", 8);
setSchedule(schedule);
```

---

### speak()

Convert text to speech using Gemini's audio modality and play it through the browser's Web Audio API.

**Signature**

```typescript
async speak(text: string): Promise<number | undefined>
```

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `text` | string | Text to convert to speech |

**Gemini Request**

```javascript
ai.models.generateContent({
  model: "gemini-3-flash-preview",
  contents: [{ parts: [{ text }] }],
  config: {
    responseModalities: [Modality.AUDIO],
    speechConfig: {
      voiceConfig: {
        prebuiltVoiceConfig: { voiceName: 'Kore' }
      }
    }
  }
})
```

**Audio Decoding Pipeline**

```
Gemini Response
  ↓  candidates[0].content.parts[0].inlineData.data
base64 string
  ↓  atob() → Uint8Array
Raw bytes (PCM Int16)
  ↓  Int16Array → normalize to [-1.0, 1.0]
Float32Array per channel
  ↓  AudioContext.createBuffer(channels=1, sampleRate=24000)
AudioBuffer
  ↓  createBufferSource().start()
Browser Audio Playback
```

**Return Value**

Returns the audio duration in milliseconds (`audioBuffer.duration * 1000`), or `undefined` if no audio data was returned.

**Voice Configuration**

| Property | Value |
|---|---|
| Voice name | `Kore` |
| Sample rate | 24000 Hz |
| Channels | 1 (mono) |
| Encoding | PCM Int16 (raw) |

**Usage**

```javascript
// Summary.jsx — "Listen" button
await geminiService.speak(summary);

// QuizSession.jsx — accessibility mode
await geminiService.speak(currentQuestion.question);

// VoiceAssistant.jsx — feedback
await geminiService.speak("Navigating to upload page");
```

**Error Handling**

Logs errors to console and returns `undefined`. Does not throw.

---

## Data Models & Schemas

### MongoDB — Users Collection

```
Database:   ai_learning
Collection: users
```

```json
{
  "_id":      "ObjectId",
  "name":     "string — user display name",
  "email":    "string — unique, indexed",
  "password": "string — pbkdf2:sha256 hash (Werkzeug)"
}
```

**Indexes**

| Field | Type | Constraint |
|---|---|---|
| `email` | string | Unique (enforced by application) |

---

### JWT Payload

```json
{
  "email": "jane@example.com",
  "exp":   1700000000
}
```

| Field | Type | Description |
|---|---|---|
| `email` | string | Used to look up the user in MongoDB on each authenticated request |
| `exp` | Unix timestamp | Token expiry — 24 hours from issue time |

---

### Flashcard

```typescript
interface Flashcard {
  question: string;  // ends with "?"
  answer:   string;  // max 25 words
}
```

---

### QuizQuestion (Flask version)

```typescript
interface QuizQuestion {
  id:             number;   // 1-indexed
  question:       string;
  correct_answer: string;
  options:        string[]; // 4 items, shuffled
  correctIndex:   number;   // 0–3
  explanation:    string;
}
```

### QuizQuestion (Gemini version)

```typescript
interface QuizQuestion {
  question:     string;
  options:      string[]; // 4 items
  correctIndex: number;   // 0–3
  explanation:  string;
}
```

---

### LearningContent (LearnAI / Client-side)

```typescript
interface LearningContent {
  id:         string;
  title:      string;
  summary:    string;
  flashcards: Flashcard[];
  quiz:       QuizQuestion[];
  timestamp:  number;  // Date.now()
}
```

---

### TeacherSchedule

```typescript
interface TeacherSchedule {
  week:       string;    // e.g. "Week 1"
  topic:      string;    // e.g. "Introduction to Variables"
  objectives: string[];  // learning outcomes
  activities: string[];  // classroom activities
}
```

---

### UserProgress (LearnAI)

```typescript
interface UserProgress {
  topic:          string;
  score:          number;
  totalQuestions: number;
  date:           string;  // ISO date string
}
```

---

## Error Handling

### Flask Global Error Handler

All unhandled exceptions are caught by:

```python
@app.errorhandler(Exception)
def handle_exception(e):
    traceback.print_exc()
    return jsonify({
        "error":   str(e),
        "message": "Internal Server Error"
    }), 500
```

CORS headers are set on error responses:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: *
```

### HTTP Status Code Reference

| Code | Meaning | Trigger |
|---|---|---|
| `200` | OK | Successful GET or POST |
| `201` | Created | Successful signup |
| `400` | Bad Request | Missing fields, invalid JSON, empty PDF |
| `401` | Unauthorized | Invalid credentials, missing/invalid JWT |
| `500` | Internal Server Error | Unhandled exception, DB not configured |

### JWT Error Response

```json
{ "message": "Token is missing!" }
{ "message": "Token is invalid!" }
```

### Gemini API Errors

Errors from `geminiService.processContent()` and `generateSchedule()` are re-thrown to the caller. `speak()` catches internally and returns `undefined`.

```javascript
// Recommended caller pattern
try {
  const content = await geminiService.processContent(text);
} catch (err) {
  console.error('Gemini API error:', err);
  // Show user-friendly error message
}
```

---

## Environment Variables

### Flask Backend (`server/.env`)

| Variable | Required | Description | Example |
|---|---|---|---|
| `SECRET_KEY` | Yes | JWT signing secret | `super-secret-jwt-key-change-in-prod` |
| `MONGODB_URI` | Yes | MongoDB connection string | `mongodb+srv://user:pass@cluster.mongodb.net/` |
| `ELEVENLABS_API_KEY` | No | ElevenLabs TTS key (unused, gTTS is active) | `el_xxxxxxxx` |

**Fallback behavior:**
- `SECRET_KEY` defaults to `"super-secret-jwt-key"` if not set (insecure — always set in production)
- `MONGODB_URI` missing → auth routes return 500; `/summary` still works

### React Frontend (`client/.env`)

| Variable | Required | Description |
|---|---|---|
| `VITE_GEMINI_API_KEY` | Yes | Google Gemini API key for client-side AI calls |

### LearnAI (`learnai/.env`)

| Variable | Required | Description |
|---|---|---|
| `API_KEY` | Yes | Google Gemini API key |

---

## Internal Service Interfaces

### generate_speech() — tts_model.py

```python
def generate_speech(
    text:        str,
    output_dir:  str = "tts_chunks",
    output_file: str = "output.mp3"
) -> str:
    """
    Convert text to speech using gTTS and save as MP3.

    Returns:
        str: Absolute path to the saved MP3 file.

    Raises:
        ValueError: If text is empty or blank.
        Exception:  If gTTS conversion fails.
    """
```

**gTTS configuration**

| Property | Value |
|---|---|
| Language | `en` (English) |
| Speed | Normal (`slow=False`) |
| Output | `tts_chunks/output.mp3` |

---

### generate_flashcards() — flashcard_generator.py

```python
def generate_flashcards(
    summary_text:   str,
    max_flashcards: int = 10
) -> list[dict]:
    """
    Generate Q&A flashcard pairs from summarized text.

    Returns:
        List of {"question": str, "answer": str} dicts.
        Empty list on failure or insufficient input.

    Requirements:
        summary_text must be >= 100 characters.
    """
```

**Model pipeline**

| Step | Model | Config |
|---|---|---|
| Question generation | `valhalla/t5-base-qg-hl` | `max_length=256, do_sample=True, top_k=50, top_p=0.95, num_return_sequences=15` |
| Question answering | `distilbert-base-cased-distilled-squad` | `pipeline("question-answering")` |
| Deduplication | `difflib.SequenceMatcher` | `threshold=0.85` |
| Input truncation | — | First 2000 chars for QG, first 3000 chars for QA |
| Answer truncation | — | Max 25 words |

---

### generate_quiz() — flashcard_generator.py

```python
def generate_quiz(
    summary_text:  str,
    num_questions: int = 5
) -> list[dict]:
    """
    Generate multiple-choice quiz from summary text.
    Internally calls generate_flashcards() and wraps results.

    Returns:
        List of quiz question dicts with id, question, correct_answer,
        options (4 items, shuffled), correctIndex, explanation.
    """
```

**Distractor selection**

```
All flashcard answers → distractor pool
For each question:
  Remove own answer from pool → candidate distractors
  random.sample(candidates, min(3, len))
  Pad with "None of the above (N)" if < 3 distractors
  options = [correct] + [3 distractors]
  random.shuffle(options)
  correctIndex = options.index(correct_answer)
```

---

## API Flow Diagrams

### Authentication Flow

```
Client                          Flask                        MongoDB
  │                               │                              │
  │  POST /api/signup             │                              │
  │  {name, email, password}──────►                              │
  │                               │  hash_password(pw)           │
  │                               │  find_one({email})──────────►│
  │                               │◄──────────────── existing?   │
  │                               │  insert_one({...hashed})────►│
  │◄──────────── 201 Created      │                              │
  │                               │                              │
  │  POST /api/login              │                              │
  │  {email, password}────────────►                              │
  │                               │  find_one({email})──────────►│
  │                               │◄──────────────── user doc    │
  │                               │  check_password_hash()       │
  │                               │  jwt.encode({email, exp})    │
  │◄──── 200 {token, user} ───────│                              │
  │                               │                              │
  │  POST /summary                │                              │
  │  Authorization: Bearer <jwt>  │                              │
  │  (if @token_required applied) │                              │
  │                               │  jwt.decode(token)           │
  │                               │  find_one({email})──────────►│
  │                               │◄──────────────── current_user│
```

### PDF Processing Flow

```
Client                   Flask                  ML Services
  │                        │                        │
  │  POST /summary          │                        │
  │  file=<pdf>             │                        │
  │  options={...}──────────►                        │
  │                         │  extract_text(pdf)     │
  │                         │  split_chunks(700w)    │
  │                         │                        │
  │                         │──summarize()──────────►│ BART
  │                         │◄──────────── summary   │
  │                         │                        │
  │                         │──generate_flashcards()─►│ T5 + DistilBERT
  │                         │◄──────────── Q&A pairs  │
  │                         │                        │
  │                         │──generate_quiz()───────►│ wraps flashcards
  │                         │◄──────────── MC quiz    │
  │                         │                        │
  │                         │──generate_speech()─────►│ gTTS
  │                         │◄──────────── output.mp3 │
  │                         │                        │
  │◄── 200 {summary,         │                        │
  │    flashcards, quiz,     │                        │
  │    audio_url} ───────────│                        │
  │                          │                        │
  │  GET /audio/output.mp3   │                        │
  │──────────────────────────►                        │
  │◄───── MP3 binary ─────── │  send_from_directory() │
```

### Gemini Client API Flow (processContent)

```
React Component        geminiService.js        Google Gemini API
      │                      │                        │
      │  processContent(text) │                        │
      │──────────────────────►                        │
      │                      │  generateContent({     │
      │                      │    model,              │
      │                      │    contents: prompt,   │
      │                      │    config: {schema}    │
      │                      │  })────────────────────►
      │                      │                        │ Generate JSON
      │                      │◄──────── response.text │ matching schema
      │                      │  JSON.parse()          │
      │◄── LearningContent ──│                        │
```

---

## Quick Reference Card

### Flask REST Endpoints Summary

```
Method  Path                  Auth   Content-Type            Purpose
──────  ────                  ────   ────────────            ───────
POST    /api/signup            —     application/json        Register user
POST    /api/login             —     application/json        Login → JWT
POST    /summary               —     multipart/form-data     Process PDF → AI content
GET     /audio/:filename       —     —                       Stream MP3 audio
```

### Client Gemini Service Summary

```
Method              Auth              Returns             Purpose
──────              ────              ───────             ───────
processContent(txt) Gemini API Key    LearningContent     Summary + flashcards + quiz
generateSchedule()  Gemini API Key    TeacherSchedule[]   Weekly teaching plan
speak(txt)          Gemini API Key    number (ms) | void  TTS audio playback
```

### Status Code Quick Reference

```
200 — Success
201 — Resource created (signup)
400 — Bad input (missing file, invalid JSON, empty PDF)
401 — Unauthorized (wrong credentials, invalid/missing JWT)
500 — Server error (DB not configured, unhandled exception)
```

---

*Document covers: GW ImpactAI Hackathon Platform + LearnAI Adaptive Education Platform*  
*Backend: Flask 3.1.0 · Python 3.x · MongoDB · HuggingFace Transformers*  
*Frontend: React 18/19 · Vite 6 · Google Gemini API (@google/genai)*
