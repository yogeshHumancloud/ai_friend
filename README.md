# Caring AI Mobile App - System Architecture

## Overall Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  React Native   │    │   FastAPI        │    │  LLM Service    │
│  Mobile App     │◄──►│   Backend        │◄──►│  (3B Model)     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │   Message Queue  │
                       │   (RabbitMQ)     │
                       └──────────────────┘
                                │
                ┌───────────────┼───────────────┐
                ▼               ▼               ▼
      ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
      │  ChromaDB   │  │  Supabase   │  │  Redis      │
      │  (RAG)      │  │  (Auth/DB)  │  │  (Cache)    │
      └─────────────┘  └─────────────┘  └─────────────┘
```

## Component Details

### 1. React Native Mobile App

**Core Features:**
- Chat interface with conversation bubbles
- Voice input/output capabilities
- Real-time messaging
- User authentication flow
- Offline message queue

**Key Libraries:**
```json
{
  "dependencies": {
    "@react-native-async-storage/async-storage": "^1.19.0",
    "@react-native-voice/voice": "^3.2.4",
    "react-native-tts": "^4.1.0",
    "@supabase/supabase-js": "^2.38.0",
    "react-native-websocket": "^1.0.2",
    "react-native-sound": "^0.11.2"
  }
}
```

**App Structure:**
```
src/
├── components/
│   ├── ChatBubble.js
│   ├── VoiceRecorder.js
│   └── TypingIndicator.js
├── screens/
│   ├── LoginScreen.js
│   ├── ChatScreen.js
│   └── ProfileScreen.js
├── services/
│   ├── ApiService.js
│   ├── AuthService.js
│   └── VoiceService.js
└── utils/
    ├── storage.js
    └── constants.js
```

### 2. FastAPI Backend Service

**API Endpoints:**
```python
# Authentication & User Management
POST   /api/auth/login
POST   /api/auth/register
GET    /api/auth/profile
POST   /api/auth/refresh

# Chat & Conversation
POST   /api/chat/message
GET    /api/chat/conversations/{user_id}
GET    /api/chat/history/{conversation_id}
DELETE /api/chat/conversation/{conversation_id}

# Voice Processing
POST   /api/voice/transcribe
POST   /api/voice/synthesize

# Health & Status
GET    /api/health
GET    /api/status
```

**Service Structure:**
```
backend/
├── app/
│   ├── api/
│   │   ├── auth.py
│   │   ├── chat.py
│   │   └── voice.py
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── database.py
│   ├── services/
│   │   ├── llm_service.py
│   │   ├── rag_service.py
│   │   ├── queue_service.py
│   │   └── vector_service.py
│   └── models/
│       ├── user.py
│       ├── conversation.py
│       └── message.py
├── requirements.txt
└── main.py
```

### 3. Message Queue System (RabbitMQ)

**Queue Structure:**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   chat.input    │    │  llm.processing │    │  chat.response  │
│   (User msgs)   │───►│  (LLM tasks)    │───►│  (AI responses) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │  rag.indexing   │
                       │  (Memory store) │
                       └─────────────────┘
```

**Queue Configuration:**
```python
# Queue definitions
QUEUES = {
    'chat_input': {
        'durable': True,
        'priority': 10
    },
    'llm_processing': {
        'durable': True,
        'priority': 8
    },
    'rag_indexing': {
        'durable': True,
        'priority': 5
    },
    'chat_response': {
        'durable': True,
        'priority': 10
    }
}
```

### 4. Authentication System (Supabase)

**Database Schema:**
```sql
-- Users table (managed by Supabase Auth)
users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
)

-- User profiles
user_profiles (
  id UUID REFERENCES users(id),
  display_name VARCHAR(100),
  avatar_url TEXT,
  preferences JSONB,
  created_at TIMESTAMP DEFAULT NOW()
)

-- Conversations
conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  title VARCHAR(200),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
)

-- Messages
messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID REFERENCES conversations(id),
  user_id UUID REFERENCES users(id),
  content TEXT NOT NULL,
  message_type VARCHAR(20) DEFAULT 'text', -- 'text', 'voice', 'system'
  is_from_ai BOOLEAN DEFAULT FALSE,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
)
```

### 5. RAG System (ChromaDB)

**Vector Storage Structure:**
```python
# Collection schema
collections = {
    "user_conversations": {
        "embeddings": "sentence-transformers/all-MiniLM-L6-v2",
        "metadata": {
            "user_id": str,
            "conversation_id": str,
            "timestamp": datetime,
            "message_type": str,
            "emotional_context": str
        }
    },
    "user_preferences": {
        "embeddings": "sentence-transformers/all-MiniLM-L6-v2",
        "metadata": {
            "user_id": str,
            "preference_type": str,
            "importance": float
        }
    }
}
```

**RAG Service Functions:**
```python
class RAGService:
    def store_conversation(self, user_id, message, response)
    def retrieve_relevant_context(self, user_id, current_message, k=5)
    def update_user_preferences(self, user_id, preferences)
    def get_conversation_summary(self, conversation_id)
```

## Data Flow

### 1. Message Processing Flow
```
User Message → Mobile App → FastAPI → RabbitMQ (chat.input)
                                   ↓
Worker picks up → RAG retrieval → LLM processing → RabbitMQ (chat.response)
                                   ↓
FastAPI receives → WebSocket/Push → Mobile App → User
                                   ↓
Parallel: RabbitMQ (rag.indexing) → ChromaDB storage
```

### 2. Authentication Flow
```
User Login → React Native → Supabase Auth → JWT Token
                          ↓
Store token → AsyncStorage → API requests (Authorization header)
```

### 3. Voice Processing Flow
```
Voice Input → Speech-to-Text → Text Message Flow
                            ↓
AI Response → Text-to-Speech → Audio Output
```

## Deployment Architecture

### Development Environment
```yaml
# docker-compose.yml
version: '3.8'
services:
  fastapi:
    build: ./backend
    ports: ["8000:8000"]
    depends_on: [rabbitmq, chromadb, redis]
    
  rabbitmq:
    image: rabbitmq:3-management
    ports: ["5672:5672", "15672:15672"]
    
  chromadb:
    image: chromadb/chroma
    ports: ["8001:8000"]
    volumes: ["./chroma_data:/chroma/chroma"]
    
  redis:
    image: redis:alpine
    ports: ["6379:6379"]
    
  llm-service:
    build: ./llm-service
    ports: ["8002:8000"]
    runtime: nvidia  # For GPU support
```

### Production Environment
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Load      │    │   FastAPI   │    │   Worker    │
│ Balancer    │───►│   Cluster   │───►│   Nodes     │
│ (Nginx)     │    │ (Gunicorn)  │    │ (Celery)    │
└─────────────┘    └─────────────┘    └─────────────┘
                            │                │
                    ┌───────┼────────────────┼───────┐
                    ▼       ▼                ▼       ▼
              ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
              │RabbitMQ │ │ChromaDB │ │Supabase │ │  Redis  │
              │Cluster  │ │Cluster  │ │  Cloud  │ │Cluster  │
              └─────────┘ └─────────┘ └─────────┘ └─────────┘
```

## Scalability Considerations

### 1. Horizontal Scaling
- **FastAPI**: Multiple instances behind load balancer
- **Worker Nodes**: Scale based on queue length
- **ChromaDB**: Distributed deployment for large datasets
- **RabbitMQ**: Cluster setup for high availability

### 2. Performance Optimization
- **Redis Caching**: Cache frequent RAG queries
- **Connection Pooling**: Database and queue connections
- **Async Processing**: Non-blocking I/O operations
- **Message Batching**: Batch similar operations

### 3. Monitoring & Observability
- **API Metrics**: Request latency, error rates
- **Queue Monitoring**: Message throughput, queue depth
- **Resource Usage**: CPU, memory, GPU utilization
- **User Analytics**: Conversation quality, engagement

## Security Implementation

### 1. Authentication & Authorization
```python
# JWT token validation
@jwt_required
async def protected_route(current_user: User = Depends(get_current_user)):
    return {"user": current_user}

# Rate limiting
@limiter.limit("10/minute")
async def chat_endpoint():
    pass
```

### 2. Data Protection
- **Encryption**: At rest and in transit (TLS 1.3)
- **Input Validation**: Sanitize all user inputs
- **CORS Configuration**: Restrict origins
- **API Key Management**: Secure LLM service access

This architecture provides a robust, scalable foundation for your caring AI mobile app with proper separation of concerns and modern best practices.
