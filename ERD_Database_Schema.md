# Entity Relationship Diagram (ERD) - GW-ImpactAI Learning Platform

## Database Schema & Relationships

### ER Diagram (PlantUML)

```plantuml
@startuml ER_Diagram
!define TABLENAME(x) class x << (T,#FFAAAA) >>
!define PRIMARY_KEY(x) x
!define FOREIGN_KEY(x) x
!define FIELD(x) x

TABLENAME(Users) {
  PRIMARY_KEY(id) : ObjectId
  --
  name : String
  email : String (unique)
  password : String (hashed)
  role : String (student/teacher/admin)
  preferences : Object {theme, notifications}
  createdAt : DateTime
  updatedAt : DateTime
}

TABLENAME(LearningSession) {
  PRIMARY_KEY(id) : ObjectId
  --
  FOREIGN_KEY(userId) : ObjectId
  title : String
  description : String
  contentType : String (pdf/text)
  status : String (active/completed/archived)
  createdAt : DateTime
  updatedAt : DateTime
}

TABLENAME(ProcessedContent) {
  PRIMARY_KEY(id) : ObjectId
  --
  FOREIGN_KEY(sessionId) : ObjectId
  FOREIGN_KEY(userId) : ObjectId
  originalText : String
  summary : String
  contentType : String
  processingTime : Number
  createdAt : DateTime
}

TABLENAME(Flashcard) {
  PRIMARY_KEY(id) : ObjectId
  --
  FOREIGN_KEY(sessionId) : ObjectId
  FOREIGN_KEY(userId) : ObjectId
  question : String
  answer : String
  difficulty : String (easy/medium/hard)
  category : String
  tags : Array<String>
  reviewCount : Number
  lastReviewedAt : DateTime
  createdAt : DateTime
}

TABLENAME(QuizQuestion) {
  PRIMARY_KEY(id) : ObjectId
  --
  FOREIGN_KEY(sessionId) : ObjectId
  FOREIGN_KEY(userId) : ObjectId
  question : String
  options : Array<String>
  correctAnswer : String
  explanation : String
  difficulty : String
  category : String
  createdAt : DateTime
}

TABLENAME(QuizAttempt) {
  PRIMARY_KEY(id) : ObjectId
  --
  FOREIGN_KEY(userId) : ObjectId
  FOREIGN_KEY(quizId) : ObjectId
  answers : Array<Object>
  score : Number
  percentage : Number
  timeTaken : Number
  status : String (completed/in_progress)
  completedAt : DateTime
  createdAt : DateTime
}

TABLENAME(AudioFile) {
  PRIMARY_KEY(id) : ObjectId
  --
  FOREIGN_KEY(contentId) : ObjectId
  FOREIGN_KEY(userId) : ObjectId
  filename : String
  filePath : String
  duration : Number
  format : String (mp3)
  size : Number
  createdAt : DateTime
}

TABLENAME(UserProgress) {
  PRIMARY_KEY(id) : ObjectId
  --
  FOREIGN_KEY(userId) : ObjectId
  totalSessionsCompleted : Number
  totalFlashcardsCreated : Number
  totalQuizzesAttempted : Number
  averageScore : Number
  streakDays : Number
  lastActiveAt : DateTime
  updatedAt : DateTime
}

Users "1" -- "*" LearningSession
Users "1" -- "*" ProcessedContent
Users "1" -- "*" Flashcard
Users "1" -- "*" QuizQuestion
Users "1" -- "*" QuizAttempt
Users "1" -- "*" AudioFile
Users "1" -- "1" UserProgress

LearningSession "1" -- "*" ProcessedContent
LearningSession "1" -- "*" Flashcard
LearningSession "1" -- "*" QuizQuestion
LearningSession "1" -- "*" AudioFile

ProcessedContent "1" -- "*" AudioFile

@enduml
```

## Mermaid ER Diagram

```mermaid
erDiagram
    USERS ||--o{ LEARNING_SESSION : creates
    USERS ||--o{ PROCESSED_CONTENT : generates
    USERS ||--o{ FLASHCARD : creates
    USERS ||--o{ QUIZ_QUESTION : creates
    USERS ||--o{ QUIZ_ATTEMPT : takes
    USERS ||--o{ AUDIO_FILE : owns
    USERS ||--|| USER_PROGRESS : tracks
    
    LEARNING_SESSION ||--o{ PROCESSED_CONTENT : contains
    LEARNING_SESSION ||--o{ FLASHCARD : generates
    LEARNING_SESSION ||--o{ QUIZ_QUESTION : includes
    LEARNING_SESSION ||--o{ AUDIO_FILE : produces
    
    PROCESSED_CONTENT ||--o{ AUDIO_FILE : generates
    
    QUIZ_QUESTION ||--o{ QUIZ_ATTEMPT : includes

    USERS {
        ObjectId id PK
        string name
        string email UK
        string password
        string role
        object preferences
        timestamp createdAt
        timestamp updatedAt
    }

    LEARNING_SESSION {
        ObjectId id PK
        ObjectId userId FK
        string title
        string description
        string contentType
        string status
        timestamp createdAt
        timestamp updatedAt
    }

    PROCESSED_CONTENT {
        ObjectId id PK
        ObjectId sessionId FK
        ObjectId userId FK
        string originalText
        string summary
        string contentType
        number processingTime
        timestamp createdAt
    }

    FLASHCARD {
        ObjectId id PK
        ObjectId sessionId FK
        ObjectId userId FK
        string question
        string answer
        string difficulty
        string category
        string[] tags
        number reviewCount
        timestamp lastReviewedAt
        timestamp createdAt
    }

    QUIZ_QUESTION {
        ObjectId id PK
        ObjectId sessionId FK
        ObjectId userId FK
        string question
        string[] options
        string correctAnswer
        string explanation
        string difficulty
        string category
        timestamp createdAt
    }

    QUIZ_ATTEMPT {
        ObjectId id PK
        ObjectId userId FK
        ObjectId quizId FK
        object[] answers
        number score
        number percentage
        number timeTaken
        string status
        timestamp completedAt
        timestamp createdAt
    }

    AUDIO_FILE {
        ObjectId id PK
        ObjectId contentId FK
        ObjectId userId FK
        string filename
        string filePath
        number duration
        string format
        number size
        timestamp createdAt
    }

    USER_PROGRESS {
        ObjectId id PK
        ObjectId userId FK
        number totalSessionsCompleted
        number totalFlashcardsCreated
        number totalQuizzesAttempted
        number averageScore
        number streakDays
        timestamp lastActiveAt
        timestamp updatedAt
    }
```

## Database Schema Details

### Users Collection

```json
{
  "_id": ObjectId,
  "name": "John Doe",
  "email": "john@example.com",
  "password": "$2b$12$...", // bcrypt hash
  "role": "student", // student | teacher | admin
  "preferences": {
    "theme": "dark", // light | dark | system
    "notifications": true,
    "language": "en"
  },
  "createdAt": ISODate("2024-01-15"),
  "updatedAt": ISODate("2024-01-15")
}
```

### LearningSession Collection

```json
{
  "_id": ObjectId,
  "userId": ObjectId,
  "title": "Biology Chapter 5",
  "description": "Study material for biology",
  "contentType": "pdf", // pdf | text | video
  "status": "active", // active | completed | archived
  "fileMetadata": {
    "originalFilename": "biology.pdf",
    "uploadedSize": 2048576,
    "pages": 45
  },
  "createdAt": ISODate("2024-01-15"),
  "updatedAt": ISODate("2024-01-16")
}
```

### ProcessedContent Collection

```json
{
  "_id": ObjectId,
  "sessionId": ObjectId,
  "userId": ObjectId,
  "originalText": "Full extracted text...",
  "summary": "Condensed summary...",
  "contentType": "pdf",
  "processingTime": 4200, // milliseconds
  "metadata": {
    "wordCount": 5000,
    "summaryWordCount": 500,
    "extractionMethod": "PyMuPDF"
  },
  "createdAt": ISODate("2024-01-15")
}
```

### Flashcard Collection

```json
{
  "_id": ObjectId,
  "sessionId": ObjectId,
  "userId": ObjectId,
  "question": "What is photosynthesis?",
  "answer": "Photosynthesis is the process by which plants convert light energy...",
  "difficulty": "medium", // easy | medium | hard
  "category": "Biology",
  "tags": ["photosynthesis", "biology", "plants"],
  "reviewCount": 5,
  "lastReviewedAt": ISODate("2024-01-16"),
  "createdAt": ISODate("2024-01-15")
}
```

### QuizQuestion Collection

```json
{
  "_id": ObjectId,
  "sessionId": ObjectId,
  "userId": ObjectId,
  "question": "Which organelle is responsible for energy production?",
  "options": [
    "Nucleus",
    "Mitochondria",
    "Ribosome",
    "Golgi Apparatus"
  ],
  "correctAnswer": "Mitochondria",
  "explanation": "Mitochondria is the powerhouse of the cell...",
  "difficulty": "easy",
  "category": "Cell Biology",
  "createdAt": ISODate("2024-01-15")
}
```

### QuizAttempt Collection

```json
{
  "_id": ObjectId,
  "userId": ObjectId,
  "quizId": ObjectId,
  "answers": [
    {
      "questionId": ObjectId,
      "selectedAnswer": "Mitochondria",
      "isCorrect": true,
      "timeTaken": 15000 // milliseconds
    }
  ],
  "score": 8,
  "totalQuestions": 10,
  "percentage": 80,
  "timeTaken": 450000, // milliseconds
  "status": "completed",
  "completedAt": ISODate("2024-01-16"),
  "createdAt": ISODate("2024-01-16")
}
```

### AudioFile Collection

```json
{
  "_id": ObjectId,
  "contentId": ObjectId,
  "userId": ObjectId,
  "filename": "summary_2024_01_15.mp3",
  "filePath": "/tts_chunks/summary_2024_01_15.mp3",
  "duration": 180000, // milliseconds
  "format": "mp3",
  "size": 2883840, // bytes
  "metadata": {
    "generator": "ElevenLabs",
    "voice": "neutral",
    "speed": 1.0
  },
  "createdAt": ISODate("2024-01-15")
}
```

### UserProgress Collection

```json
{
  "_id": ObjectId,
  "userId": ObjectId,
  "stats": {
    "totalSessionsCompleted": 15,
    "totalFlashcardsCreated": 342,
    "totalQuizzesAttempted": 28,
    "averageScore": 78.5,
    "bestScore": 95,
    "worstScore": 62
  },
  "engagement": {
    "streakDays": 7,
    "longestStreak": 14,
    "totalHoursLearned": 42.5
  },
  "lastActiveAt": ISODate("2024-01-16"),
  "updatedAt": ISODate("2024-01-16")
}
```

## Data Relationships

### One-to-Many Relationships

1. **Users → LearningSession**
   - One user can create multiple learning sessions
   - Foreign Key: `userId` in LearningSession

2. **Users → Flashcard**
   - One user can create multiple flashcards
   - Foreign Key: `userId` in Flashcard

3. **Users → QuizQuestion**
   - One user (or teacher) can create multiple quiz questions
   - Foreign Key: `userId` in QuizQuestion

4. **Users → QuizAttempt**
   - One user can take multiple quiz attempts
   - Foreign Key: `userId` in QuizAttempt

5. **LearningSession → ProcessedContent**
   - One session generates one processed content (1:1 in practice)
   - Foreign Key: `sessionId` in ProcessedContent

6. **LearningSession → Flashcard**
   - One session can have multiple flashcards
   - Foreign Key: `sessionId` in Flashcard

7. **LearningSession → QuizQuestion**
   - One session can have multiple quiz questions
   - Foreign Key: `sessionId` in QuizQuestion

### One-to-One Relationships

1. **Users ↔ UserProgress**
   - Each user has exactly one progress record
   - Foreign Key: `userId` in UserProgress

## Indexing Strategy

```javascript
// Users Collection
db.users.createIndex({ "email": 1 }, { unique: true })
db.users.createIndex({ "role": 1 })

// LearningSession Collection
db.learning_session.createIndex({ "userId": 1 })
db.learning_session.createIndex({ "status": 1 })
db.learning_session.createIndex({ "createdAt": -1 })

// ProcessedContent Collection
db.processed_content.createIndex({ "userId": 1 })
db.processed_content.createIndex({ "sessionId": 1 })
db.processed_content.createIndex({ "createdAt": -1 })

// Flashcard Collection
db.flashcard.createIndex({ "userId": 1 })
db.flashcard.createIndex({ "sessionId": 1 })
db.flashcard.createIndex({ "difficulty": 1 })
db.flashcard.createIndex({ "tags": 1 })

// QuizAttempt Collection
db.quiz_attempt.createIndex({ "userId": 1 })
db.quiz_attempt.createIndex({ "completedAt": -1 })
```

## Data Aggregation Pipeline Examples

### User Learning Statistics
```javascript
db.users.aggregate([
  {
    $lookup: {
      from: "learning_session",
      localField: "_id",
      foreignField: "userId",
      as: "sessions"
    }
  },
  {
    $lookup: {
      from: "quiz_attempt",
      localField: "_id",
      foreignField: "userId",
      as: "attempts"
    }
  },
  {
    $project: {
      name: 1,
      email: 1,
      totalSessions: { $size: "$sessions" },
      totalQuizAttempts: { $size: "$attempts" },
      averageScore: { $avg: "$attempts.percentage" }
    }
  }
])
```

## Data Validation Rules

| Collection | Field | Validation |
|-----------|-------|-----------|
| Users | email | Required, Unique, Valid email format |
| Users | password | Required, Min 8 chars, hashed |
| Flashcard | question | Required, Max 500 chars |
| Flashcard | answer | Required, Max 2000 chars |
| QuizQuestion | options | Required, Exactly 4 options |
| QuizAttempt | percentage | 0-100 numeric range |
| AudioFile | duration | Positive integer, milliseconds |

## Backup & Recovery Strategy

1. **Backup Frequency**: Daily automated backups
2. **Retention**: Keep 30 days of daily backups
3. **Recovery Time Objective (RTO)**: 2 hours
4. **Recovery Point Objective (RPO)**: 1 hour
5. **Backup Location**: Cloud storage with geo-replication
