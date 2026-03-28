# 💡 LearnAI - Comprehensive Project Documentation

This document provides an exhaustive, in-depth overview of the **LearnAI** project. It details the architecture, all technologies, libraries, machine learning models, external APIs used, and exactly how they interact to provide an accessible learning platform for visually, hearing, and cognitively impaired students.

---

## 1. Project Architecture Reference
The LearnAI project uses a decoupled **Client-Server Architecture**. 
* **Frontend (Client)**: A modern Single Page Application (SPA) built with React and Vite. It handles the user interface, routing, animations, file uploads via drag-and-drop, and the Web Speech API for voice navigation and local text-to-speech.
* **Backend (Server)**: A Python-based RESTful API powered by Flask. It handles intensive processing including PDF text extraction, AI summarization via HuggingFace models, AI question/answer generation, and connecting to the ElevenLabs API for realistic audio.

Both the client and server run on localhost simultaneously (Ports `5173` and `5000` respectively). They communicate via JSON payloads over HTTP POST/GET requests.

---

## 2. Frontend Technologies & Tooling
The frontend is built for speed, reactivity, and accessibility. Located in the `/client` directory.

### Core Frameworks
* **React 18** (`react`, `react-dom`): The core library used to build the component-based UI. Utilizes React Hooks (`useState`, `useEffect`, `useCallback`, `useRef`) for robust state management without class components.
* **Vite** (`vite`): The build tool and development server. It replaces Webpack/CRA, offering near-instant Hot Module Replacement (HMR) and ultra-fast builds.

### UI & Styling
* **Tailwind CSS** (`tailwindcss`, `postcss`, `autoprefixer`): The primary utility-first CSS framework. It allows for rapid prototyping and styling directly in the JSX classes without writing custom CSS files. 
* **Framer Motion** (`framer-motion`): Used for fluid, production-ready animations. It handles page entrance animations, hover states, and smooth card flips (used in the Flashcards UI).
* **Material UI / MUI** (`@mui/material`, `@emotion/react`): Used primarily for creating the structured Light/Dark theme provider out of the box (`createTheme`, `ThemeProvider`).
* **Lucide React** (`lucide-react`): Library providing the sleek, modern SVG icons used throughout the project (e.g., `UploadIcon`, `Book`, `Brain`, `Music`, `FileText`).

### Routing & Functionality
* **React Router DOM v6** (`react-router-dom`): Handles client-side routing. Allows users to switch between `/home`, `/dashboard`, `/upload`, and `/summary` seamlessly without reloading the webpage.
* **React Dropzone** (`react-dropzone`): Handles the drag-and-drop file upload zone on the Upload page. It restricts uploads to PDFs.

### Accessibility & Voice Tools
* **React Speech Recognition** (`react-speech-recognition`): Wraps the browser's native Web Speech API to provide the "Voice Navigation" feature. It continuously listens for commands (like "Go to Upload") and fires routing callbacks automatically.
* **React Speech Kit** (`react-speech-kit`): Wraps the browser's native `speechSynthesis` API. Used predominantly within the Voice Assistant to announce actions locally (e.g., "You are on the upload page").

---

## 3. Backend Technologies & Tooling
The backend is dedicated to running heavy Machine Learning tasks natively. Located in the `/server` directory.

### Core Frameworks
* **Python 3**: The core backend language chosen for its dominant Machine Learning ecosystem.
* **Flask** (`Flask`): A lightweight WSGI web application framework. It serves the REST endpoints (like `POST /summary` and `GET /audio/<filename>`).
* **Flask-CORS** (`flask-cors`): Cross-Origin Resource Sharing. Essential because the React app runs on port `5173` and the Flask app on port `5000`. This allows the browser to securely accept HTTP requests between the two.
* **Dotenv** (`python-dotenv`): Securely loads environment variables (API keys) from a local `.env` file instead of hardcoding them into the Python source code.

### File Processing
* **PyMuPDF** (`fitz`, `PyMuPDF`): A high-performance PDF rendering and extraction library. Given an uploaded `.pdf` file by the user, this script loops through all the pages and strips the raw plaintext for the AI to process.

---

## 4. Machine Learning & AI Models (HuggingFace)
LearnAI heavily relies on the `transformers` library to run free, open-source Machine Learning models dynamically on your local system without relying on expensive paid APIs (like OpenAI GPT-4).

### The Pipeline Architecture
The backend uses **Pipeline** objects from the `transformers` library, which automatically abstract away complex tokenization processes, model loading, and tensor mathematics into simple function calls.

**1. Summarization Model**: `facebook/bart-large-cnn`
* **Size**: Large (~1.6GB)
* **Purpose**: This model was trained by Facebook specifically to summarize news articles (CNN/DailyMail dataset).
* **Usage**: In `summarizer.py` and `app.py`, the massive extracted PDF text is broken up into smaller "chunks" (max 500-700 words) because ML models have a strict input size limit (context window). BART processes each chunk individually and compresses the document's meaning while retaining critical facts.

**2. Question Generation Model**: `valhalla/t5-base-qg-hl`
* **Size**: Medium (~890MB)
* **Purpose**: A modified Google T5 model specifically fine-tuned for generating quiz questions from statements.
* **Usage**: Used in `flashcard_generator.py`. It reads the summarized text generated by BART and asks specific questions about the data. Generating 10 to 15 diverse questions automatically to form the "Front" of the Flashcards.

**3. Question Answering Model**: `distilbert-base-cased-distilled-squad`
* **Size**: Small (~260MB)
* **Purpose**: A fast, compressed model trained on SQuAD (Stanford Question Answering Dataset). Given a Question (from T5) and a Context (from the summary), it highlights the exact phrase that answers the question.
* **Usage**: Used in `flashcard_generator.py` to auto-generate the "Back" of the Flashcards or the "Correct Answer" in the Quiz.

---

## 5. External APIs
Instead of running heavy, expensive models natively for audio and web search, the project connects to third-party endpoints.

### ElevenLabs API (Text-to-Speech)
* **Purpose**: Generates hyper-realistic, human-like voice dictation for visually impaired students. Web Speech API is robotic; ElevenLabs is natural.
* **Usage in `tts_model.py`**: Given the final summarized text, the script authenticates with `ELEVENLABS_API_KEY`, sends the text via an HTTP `POST` request, and immediately streams back an `.mp3` encoded audio file. It saves these into a `tts_chunks` folder. If multiple chunks are generated, a simple binary concatenation or `pydub` joins them.

### Tavily API (Intelligent Web Search)
* **Purpose**: Experimental/Fallback search functionality. 
* **Usage in `flashcard_generator.py`**: If the internal AI (`distilbert`) fails to confidently extract a good flashcard answer from the summary, the backend uses `TAVILY_API_KEY` to search the live internet for a factual answer to the generated question.

---

## 6. End-to-End Data Flow Execution

This represents exactly what happens when you click "Generate" in the app:

1. **Frontend Request**: The user selects checkboxes (Summary, Flashcards, Audio) and drops a PDF. `Upload.jsx` bundles the binary file and the checklist into `FormData` and `fetch()` POSTs it to the Flask server.
2. **PDF Parsing (app.py)**: Flask saves the file to a temp directory. PyMuPDF (`fitz`) rips the document apart and stitches all the raw text into a single large string.
3. **Summarizer Pipeline (BART)**: The plain text is chopped into blocks of 700 words. Each block is sent to the `BART` brain to summarize. All summarized blocks are stitched back together into a final summary.
4. **Audio Engine (tts_model.py)**: If checking "Audio", the backend takes the final summary and chunks it by sentences. These are pushed to the ElevenLabs API, returning a `.mp3` file URL to serve to the client.
5. **Flashcard/Quiz Engine (flashcard_generator.py)**: If checked, the summary goes to the `T5` brain which spits out 15 raw questions. These are deduplicated. The unique questions are then piped into the `DistilBERT` brain alongside the summary to fish out the correct answers. These form JSON Arrays.
6. **Response Processing**: Flask packages the `summary string`, `audio_url string`, and the `flashcard`/`quiz` arrays into a single JSON object and HTTP 200 responses back to Vite.
7. **Frontend Rendering**: React transitions to the `Summary.jsx` page using `useNavigate` state context. The Framer Motion cards mount, the HTML5 Audio tag mounts the new `[audio_url]`, and the user begins studying.

---

## 7. Known Edge Cases and Resilience Strategies
* **First Run Model Freeze**: If the HuggingFace models are not cached locally in your `~/.cache/huggingface/hub` folder, the entire request pauses while Python downloads Gigabytes of model `.bin` or `.safetensors` files. To mitigate crashes, `do_sample=False` and strict tensor lengths are defined.
* **ElevenLabs Quota Failures**: ElevenLabs free tier is strict. If the API returns 401 Unauthenticated or 429 Too Many Requests, `server/app.py` catches the `ValueError` and purposefully nullifies the audio response instead of breaking the entire app. This is why you will see the frontend gracefully fall back to a Yellow Warning Box instead of crashing the site.
* **Browser Speech Blocks**: Modern browsers heavily restrict Speech Synthesis. The `VoiceAssistant.jsx` prevents breaking out by ensuring interaction happens before continuous listening.
