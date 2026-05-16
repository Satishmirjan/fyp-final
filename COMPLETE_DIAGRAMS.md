# Complete Diagram Repository - All PlantUML & Mermaid Diagrams

## 1. HIGH-LEVEL ARCHITECTURE - PlantUML

```plantuml
@startuml "HLD_Complete_Architecture"
!theme plain
skinparam linetype ortho

rectangle "Client Layer\n(React Browser)" as CLIENT {
  component "UI Components\n(Pages & Widgets)" as UI
  component "Voice Assistant\n(Speech Recognition)" as VOICE
  component "Auth Context\n(State Management)" as AUTH
}

rectangle "API Gateway\n(Flask Routing)" as GATEWAY {
  component "HTTP/REST Router" as ROUTER
}

rectangle "Backend Services\n(Flask Applications)" as SERVICES {
  component "Authentication\nService" as AUTH_SVC
  component "PDF Processing\nService" as PDF_SVC
  component "Summarization\nService" as SUMM_SVC
  component "Flashcard\nGenerator" as FLASH_SVC
  component "Quiz\nGenerator" as QUIZ_SVC
  component "TTS\nService" as TTS_SVC
}

rectangle "External AI/ML\nServices" as EXTERNAL {
  component "Hugging Face\nTransformers\n(BART-Large-CNN)" as HF
  component "Google Gemini\nAPI" as GEMINI
  component "ElevenLabs\nTTS API" as ELEVEN
}

rectangle "Database\nLayer" as DATA {
  database "MongoDB\nCluster" as MONGO
}

rectangle "File\nStorage" as STORAGE {
  component "Audio Files\n(MP3 Chunks)" as AUDIO
  component "Temp Files\n(PDF Upload)" as TEMP
}

CLIENT --> ROUTER
ROUTER --> AUTH_SVC
ROUTER --> PDF_SVC
ROUTER --> SUMM_SVC
ROUTER --> FLASH_SVC
ROUTER --> QUIZ_SVC
ROUTER --> TTS_SVC

AUTH_SVC --> MONGO
PDF_SVC --> SUMM_SVC
SUMM_SVC --> HF
FLASH_SVC --> GEMINI
QUIZ_SVC --> GEMINI
TTS_SVC --> ELEVEN
TTS_SVC --> AUDIO
PDF_SVC --> TEMP

@enduml
```

## 2. FRONTEND COMPONENT HIERARCHY - PlantUML

```plantuml
@startuml "Frontend_Component_Tree"
!theme plain

package "App Structure" {
  component "AppWrapper" as ROOT
  
  package "Theme & Context" {
    component "ThemeProvider" as THEME
    component "AuthProvider" as AUTH_PROV
  }
  
  package "Layout Components" {
    component "Router" as ROUTER
    component "Navbar" as NAV
    component "VoiceAssistant" as VOICE
    component "AnimatedBackground" as BG
  }
  
  package "Page Components" {
    component "Home" as HOME
    component "Login/Signup" as AUTH_PAGES
    component "Upload" as UPLOAD
    component "Summary" as SUMMARY
    component "StudyRoom" as STUDY
    component "QuizSession" as QUIZ
    component "Analytics" as ANALYTICS
    component "TeacherTools" as TEACHER
    component "Settings" as SETTINGS
  }
  
  package "Utility Components" {
    component "ProtectedRoute" as PROTECTED
  }
  
  package "Services" {
    component "geminiService" as GEMINI_SVC
  }
}

ROOT --> THEME
ROOT --> AUTH_PROV
THEME --> ROUTER
AUTH_PROV --> NAV
ROUTER --> HOME
ROUTER --> AUTH_PAGES
ROUTER --> PROTECTED
PROTECTED --> UPLOAD
PROTECTED --> SUMMARY
PROTECTED --> STUDY
PROTECTED --> QUIZ
PROTECTED --> ANALYTICS
PROTECTED --> TEACHER
PROTECTED --> SETTINGS
VOICE --> SPEECH_KIT
UPLOAD --> GEMINI_SVC

@enduml
```

## 3. BACKEND SERVICE ARCHITECTURE - PlantUML

```plantuml
@startuml "Backend_Services_Architecture"
!theme plain

rectangle "Flask Application" as APP {
  package "Entry Point" {
    component "app.py" as MAIN
  }
  
  package "Middleware Layer" {
    component "CORS Middleware" as CORS
    component "Error Handler" as ERROR
    component "JWT Validator" as JWT_VAL
  }
  
  package "Route Handlers" {
    component "POST /api/signup" as ROUTE_SIGNUP
    component "POST /api/login" as ROUTE_LOGIN
    component "POST /summary" as ROUTE_SUMMARY
    component "GET /audio/<file>" as ROUTE_AUDIO
  }
  
  package "Business Logic" {
    component "Auth Service" as AUTH
    component "PDF Processor" as PDF
    component "Summarizer" as SUMM
    component "Flashcard Gen" as FLASH
    component "Quiz Gen" as QUIZ
    component "TTS Handler" as TTS
  }
  
  package "Data Access" {
    component "MongoDB Client" as MONGO_CLIENT
    component "Users Collection" as USERS
  }
  
  package "External Libraries" {
    component "PyMuPDF (fitz)" as PDFLIB
    component "HF Transformers" as HFTRANS
    component "PyJWT" as JWT_LIB
    component "Werkzeug" as WERKZEUG
    component "Requests" as REQUESTS
  }
}

MAIN --> CORS
MAIN --> ERROR
MAIN --> ROUTE_SIGNUP
MAIN --> ROUTE_LOGIN
MAIN --> ROUTE_SUMMARY
MAIN --> ROUTE_AUDIO

ROUTE_SIGNUP --> AUTH
ROUTE_LOGIN --> AUTH
ROUTE_SUMMARY --> PDF
ROUTE_SUMMARY --> SUMM
ROUTE_SUMMARY --> FLASH
ROUTE_SUMMARY --> QUIZ
ROUTE_SUMMARY --> TTS
ROUTE_AUDIO --> TTS

AUTH --> JWT_LIB
AUTH --> WERKZEUG
AUTH --> MONGO_CLIENT
PDF --> PDFLIB
SUMM --> HFTRANS
FLASH --> HFTRANS
QUIZ --> HFTRANS
TTS --> REQUESTS

MONGO_CLIENT --> USERS

@enduml
```

## 4. DATABASE ENTITY RELATIONSHIP - PlantUML

```plantuml
@startuml "Database_ER_Diagram"
!theme plain

entity "Users" as USERS {
  * _id : ObjectId
  --
  name : String
  email : String (unique)
  password : String (hashed)
  role : String
  preferences : Object
  createdAt : DateTime
  updatedAt : DateTime
}

entity "LearningSession" as SESSION {
  * _id : ObjectId
  --
  userId : ObjectId (FK)
  title : String
  description : String
  contentType : String
  status : String
  createdAt : DateTime
  updatedAt : DateTime
}

entity "ProcessedContent" as CONTENT {
  * _id : ObjectId
  --
  sessionId : ObjectId (FK)
  userId : ObjectId (FK)
  originalText : String
  summary : String
  processingTime : Number
  createdAt : DateTime
}

entity "Flashcard" as FLASHCARD {
  * _id : ObjectId
  --
  sessionId : ObjectId (FK)
  userId : ObjectId (FK)
  question : String
  answer : String
  difficulty : String
  category : String
  reviewCount : Number
  createdAt : DateTime
}

entity "QuizQuestion" as QUIZ_Q {
  * _id : ObjectId
  --
  sessionId : ObjectId (FK)
  userId : ObjectId (FK)
  question : String
  options : Array
  correctAnswer : String
  difficulty : String
  createdAt : DateTime
}

entity "QuizAttempt" as QUIZ_A {
  * _id : ObjectId
  --
  userId : ObjectId (FK)
  quizId : ObjectId (FK)
  score : Number
  percentage : Number
  status : String
  createdAt : DateTime
}

entity "AudioFile" as AUDIO {
  * _id : ObjectId
  --
  contentId : ObjectId (FK)
  userId : ObjectId (FK)
  filename : String
  duration : Number
  createdAt : DateTime
}

entity "UserProgress" as PROGRESS {
  * _id : ObjectId
  --
  userId : ObjectId (FK - unique)
  totalSessions : Number
  averageScore : Number
  streakDays : Number
  updatedAt : DateTime
}

USERS ||--o{ SESSION : creates
USERS ||--o{ CONTENT : generates
USERS ||--o{ FLASHCARD : creates
USERS ||--o{ QUIZ_Q : creates
USERS ||--o{ QUIZ_A : takes
USERS ||--o{ AUDIO : owns
USERS ||--|| PROGRESS : tracks

SESSION ||--o{ CONTENT : contains
SESSION ||--o{ FLASHCARD : generates
SESSION ||--o{ QUIZ_Q : includes
SESSION ||--o{ AUDIO : produces

CONTENT ||--o{ AUDIO : generates
QUIZ_Q ||--o{ QUIZ_A : includes

@enduml
```

## 5. USER JOURNEY FLOW - Mermaid

```mermaid
flowchart TD
    Start([User Visits App]) --> CheckAuth{Authenticated?}
    CheckAuth -->|No| AuthPage[Login/Signup Page]
    AuthPage --> SignupFlow{New User?}
    SignupFlow -->|Yes| Signup["POST /api/signup<br/>Create Account"]
    SignupFlow -->|No| Login["POST /api/login<br/>Get JWT Token"]
    Signup --> AuthSuccess["Token Stored<br/>localStorage"]
    Login --> AuthSuccess
    AuthSuccess --> Home["Home Page<br/>Features Overview"]
    CheckAuth -->|Yes| Home
    
    Home --> ChooseAction{User Action}
    
    ChooseAction -->|Upload PDF| UploadFlow["Upload Page"]
    UploadFlow --> SelectOptions["Select Processing<br/>Options"]
    SelectOptions --> ProcessPDF["POST /summary<br/>Send File"]
    ProcessPDF --> ExtractText["Backend:<br/>Extract Text"]
    ExtractText --> ProcessOptions{Options<br/>Selected?}
    
    ProcessOptions -->|Summary| Summarize["BART Model<br/>Summarization"]
    ProcessOptions -->|Flashcards| GenFlash["Gemini API<br/>Generate Cards"]
    ProcessOptions -->|Quiz| GenQuiz["Gemini API<br/>Generate Quiz"]
    ProcessOptions -->|Audio| GenAudio["ElevenLabs<br/>TTS Generation"]
    
    Summarize --> StoreDB["Store Results<br/>MongoDB"]
    GenFlash --> StoreDB
    GenQuiz --> StoreDB
    GenAudio --> StoreDB
    
    StoreDB --> DisplayResults["Summary Page<br/>Show Results"]
    DisplayResults --> StudyChoice{Study Mode?}
    
    StudyChoice -->|Review Cards| StudyRoom["StudyRoom<br/>Flashcard Review"]
    StudyChoice -->|Take Quiz| QuizSession["QuizSession<br/>Answer Questions"]
    StudyChoice -->|View Analytics| Analytics["Analytics Page<br/>Progress Stats"]
    
    StudyRoom --> SaveProgress["Update Progress<br/>MongoDB"]
    QuizSession --> SubmitQuiz["Store Attempt<br/>Calculate Score"]
    SubmitQuiz --> SaveProgress
    SaveProgress --> ViewStats["View Updated<br/>Statistics"]
    ViewStats --> EndFlow([End Session])
    
    Analytics --> EndFlow
    Home --> ChooseAction
```

## 6. DATA PROCESSING PIPELINE - Mermaid

```mermaid
flowchart LR
    A["📄 PDF Upload"] --> B["File Received<br/>Flask Backend"]
    B --> C["Temp Storage<br/>System Temp"]
    C --> D["Extract Text<br/>PyMuPDF fitz"]
    D --> E["Validate Content<br/>Check Not Empty"]
    E --> F{Processing<br/>Options}
    
    F -->|Summary ON| G["Split into Chunks<br/>Max 700 words"]
    G --> H["BART Summarization<br/>Hugging Face"]
    H --> I["Store Summary<br/>MongoDB"]
    
    F -->|Flashcards ON| J["Flashcard Generation<br/>Google Gemini"]
    J --> K["Store Flashcards<br/>MongoDB"]
    
    F -->|Quiz ON| L["Quiz Generation<br/>Google Gemini"]
    L --> M["Store Questions<br/>MongoDB"]
    
    F -->|Audio ON| N["TTS Processing<br/>ElevenLabs API"]
    N --> O["Save MP3 Chunks<br/>File System"]
    
    I --> P["Response to Frontend"]
    K --> P
    M --> P
    O --> P
    
    P --> Q["Display Results<br/>React Components"]
    Q --> R["User Studies<br/>Material"]
```

## 7. AUTHENTICATION & SECURITY FLOW - Mermaid

```mermaid
flowchart TD
    Start([User Starts]) --> Cred["Enter Credentials<br/>Email + Password"]
    Cred --> Check{Existing<br/>User?}
    
    Check -->|No| Signup["POST /api/signup"]
    Check -->|Yes| Login["POST /api/login"]
    
    Signup --> ValidEmail["Validate Email<br/>Format & Uniqueness"]
    ValidEmail --> HashPass["Bcrypt Hash<br/>Password"]
    HashPass --> CreateUser["Insert User<br/>MongoDB"]
    CreateUser --> SignupOK["Return 201<br/>User Created"]
    
    Login --> FindUser["Find User<br/>By Email"]
    FindUser --> UserExists{User<br/>Found?}
    UserExists -->|No| LoginFail1["Return 401<br/>Invalid Credentials"]
    UserExists -->|Yes| VerifyPass["Bcrypt Verify<br/>Password"]
    
    VerifyPass --> MatchOK{Password<br/>Match?}
    MatchOK -->|No| LoginFail2["Return 401<br/>Invalid Credentials"]
    MatchOK -->|Yes| GenJWT["Generate JWT<br/>Payload: email, exp"]
    
    GenJWT --> SignJWT["Sign with<br/>SECRET_KEY"]
    SignJWT --> Return["Return 200<br/>Token + User Info"]
    
    SignupOK --> Client["Frontend:<br/>Store Token<br/>localStorage"]
    Return --> Client
    
    Client --> Usage["Include in Headers<br/>Authorization: Bearer {token}"]
    Usage --> Protected["Access Protected<br/>Routes"]
    
    Protected --> Validate["Backend:<br/>Validate Token"]
    Validate --> ValidJWT{Token<br/>Valid?}
    ValidJWT -->|No| Reject["Return 401<br/>Token Invalid"]
    ValidJWT -->|Yes| Allowed["Access Granted<br/>Route Handler"]
    
    Reject --> Refresh["Frontend:<br/>Clear Token<br/>Redirect Login"]
    Allowed --> Success["Process Request<br/>Return Data"]
```

## 8. SYSTEM SEQUENCE DIAGRAM - Mermaid

```mermaid
sequenceDiagram
    actor User
    participant Browser as React Browser
    participant API as Flask API
    participant DB as MongoDB
    participant External as External APIs
    participant Storage as File Storage

    User->>Browser: Upload PDF + Select Options
    Browser->>Browser: FormData Creation
    Browser->>API: POST /summary (File + Options)
    
    API->>Storage: Save Temp File
    Storage-->>API: File Path
    
    API->>API: Extract Text (PyMuPDF)
    
    rect rgb(200, 220, 255)
    Note over API: Parallel Processing
    
    par Summary Generation
        API->>External: BART Summarization
        External-->>API: Summary Text
    and Flashcard Generation
        API->>External: Gemini: Generate Flashcards
        External-->>API: Flashcard Array
    and Quiz Generation
        API->>External: Gemini: Generate Quiz
        External-->>API: Quiz Array
    and Audio Generation
        API->>External: ElevenLabs: TTS
        External-->>API: Audio File
    end
    end
    
    API->>Storage: Save Audio File
    Storage-->>API: Audio Path
    
    API->>DB: Store All Results
    DB-->>API: Success
    
    API->>Storage: Delete Temp File
    
    API-->>Browser: JSON Response (Summary, Audio, Cards, Quiz)
    Browser->>Browser: Parse Results
    Browser->>User: Display Summary Page
```

## 9. DEPLOYMENT ARCHITECTURE - Mermaid

```mermaid
graph TB
    subgraph "Development Environment"
        DevBrowser["Developer Browser<br/>localhost:5173"]
        DevServer["Vite Dev Server<br/>localhost:5173"]
        DevAPI["Flask Dev Server<br/>localhost:5000"]
        DevDB["Local MongoDB<br/>mongodb://localhost"]
    end
    
    subgraph "Production Environment"
        CDN["CDN<br/>CloudFront/Vercel"]
        Frontend["React Build<br/>Static Assets"]
        Load["Load Balancer"]
        API1["Flask API #1<br/>Gunicorn"]
        API2["Flask API #2<br/>Gunicorn"]
        Cache["Redis Cache"]
        MongoDB["MongoDB Atlas<br/>Production"]
        Storage["Cloud Storage<br/>Audio Files"]
    end
    
    subgraph "External Services"
        HF["Hugging Face<br/>API"]
        Gemini["Google Gemini<br/>API"]
        Eleven["ElevenLabs<br/>API"]
    end
    
    DevBrowser --> DevServer
    DevServer --> DevAPI
    DevAPI --> DevDB
    
    Frontend --> CDN
    CDN --> Load
    Load --> API1
    Load --> API2
    
    API1 --> Cache
    API2 --> Cache
    Cache --> MongoDB
    
    API1 --> Storage
    API2 --> Storage
    
    API1 --> HF
    API1 --> Gemini
    API1 --> Eleven
    
    API2 --> HF
    API2 --> Gemini
    API2 --> Eleven
```

## 10. ERROR HANDLING FLOW - Mermaid

```mermaid
flowchart TD
    A["API Request"] --> B{"Request<br/>Valid?"}
    B -->|No| C["400 Bad Request"]
    B -->|Yes| D{"User<br/>Authenticated?"}
    D -->|No Token| E["401 Unauthorized"]
    D -->|Invalid Token| F["401 Invalid Token"]
    D -->|Expired Token| G["401 Token Expired"]
    D -->|Valid| H{"Process<br/>Request"}
    
    H -->|File Error| I["400 File Upload Error"]
    H -->|Processing Error| J["500 Processing Failed"]
    H -->|Database Error| K["500 Database Error"]
    H -->|External API Error| L["503 Service Unavailable"]
    H -->|Success| M["200 Success<br/>Return Data"]
    
    C --> LogError["Log Error<br/>Traceback"]
    E --> LogError
    F --> LogError
    G --> LogError
    I --> LogError
    J --> LogError
    K --> LogError
    L --> LogError
    
    LogError --> ResponseError["Generic Error<br/>Message to Client"]
    M --> ResponseSuccess["Return<br/>Data + 200"]
    
    ResponseError --> Browser["Browser:<br/>Handle Error"]
    ResponseSuccess --> Browser
    
    Browser --> Frontend{Frontend<br/>Logic}
    Frontend -->|401| Redirect["Redirect to Login"]
    Frontend -->|400| ShowForm["Show Form Error"]
    Frontend -->|500| ShowGeneric["Show 'Try Again'"]
```

## 11. MERMAID - Complete Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        React["⚛️ React Application"]
        Pages["📄 Pages<br/>Home, Login, Upload,<br/>Summary, Quiz, Analytics"]
        Components["🔧 Components<br/>Navbar, Voice Assistant,<br/>Protected Routes"]
        Context["🔐 Auth Context<br/>State Management"]
    end
    
    subgraph "API Gateway"
        Flask["🔌 Flask Router<br/>HTTP/REST Endpoints"]
    end
    
    subgraph "Business Logic"
        Auth["🔑 Auth Service"]
        PDF["📄 PDF Processor"]
        Summ["📝 Summarization"]
        Flash["💳 Flashcard Gen"]
        Quiz["❓ Quiz Gen"]
        TTS["🔊 TTS Service"]
    end
    
    subgraph "External Services"
        HF["🤖 Hugging Face<br/>BART Summarization"]
        Gemini["✨ Google Gemini<br/>Content Generation"]
        Eleven["🎙️ ElevenLabs<br/>Text-to-Speech"]
    end
    
    subgraph "Storage & DB"
        MongoDB["🗄️ MongoDB<br/>User Data"]
        FileSystem["💾 File System<br/>Audio/Temp Files"]
    end
    
    React --> Pages
    React --> Components
    React --> Context
    
    Pages --> Flask
    Components --> Flask
    Context --> Auth
    
    Flask --> Auth
    Flask --> PDF
    Flask --> Summ
    Flask --> Flash
    Flask --> Quiz
    Flask --> TTS
    
    Auth --> MongoDB
    PDF --> Summ
    Summ --> HF
    Flash --> Gemini
    Quiz --> Gemini
    TTS --> Eleven
    TTS --> FileSystem
    
    style React fill:#61dafb,stroke:#333,color:#000
    style HF fill:#ffd700,stroke:#333,color:#000
    style Gemini fill:#4285f4,stroke:#333,color:#fff
    style Eleven fill:#1f1f1f,stroke:#fff,color:#fff
    style MongoDB fill:#13aa52,stroke:#333,color:#fff
```

---

## Usage Instructions

### Viewing PlantUML Diagrams
1. Copy the PlantUML code into http://www.plantuml.com/plantuml/uml/
2. Or use VS Code extension: "PlantUML"
3. Or integrate with Draw.io or Miro for team collaboration

### Viewing Mermaid Diagrams
1. Copy code into https://mermaid.live/
2. Or use GitHub markdown (auto-renders)
3. Or use Mermaid VS Code extension

### Best Practices for Using These Diagrams
- Use HLD for high-level discussions with stakeholders
- Reference LLD for development and implementation
- Use ERD for database design and optimization discussions
- Share sequence diagrams during API integration meetings
- Use deployment diagrams for DevOps and infrastructure planning

---

## Diagram Legend

```
Colors:
- Green: Storage/Database
- Blue: Services/Processing
- Yellow: External Services
- Purple: User/Client
- Red: Errors/Warnings
- Orange: Files/Media

Symbols:
→  : Data Flow
←  : Response
↔  : Bidirectional
📄 : Document/File
🔐 : Security/Auth
🔧 : Tools/Components
⚙️  : Configuration
```

---

