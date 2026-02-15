# AI Saathi - Design Document

## 1. System Overview

### How AI Saathi Works
AI Saathi is built as a conversational AI assistant that acts as a bridge between citizens and essential services. When a user asks a question (via voice or text), the system:
1. Understands the question in the user's local language
2. Identifies what the user needs (scheme information, job search, healthcare guidance, etc.)
3. Searches through its knowledge base of government schemes, opportunities, and resources
4. Checks if the user is eligible based on their profile
5. Provides a personalized, easy-to-understand response
6. Delivers the answer back in the user's preferred language (voice or text)

### Purpose of the Architecture
The architecture is designed to be:
- **Simple**: Easy to build and demonstrate during the hackathon
- **Modular**: Each component can be developed and tested independently
- **Scalable**: Can grow from prototype to production
- **Accessible**: Works on low-end devices with poor connectivity
- **Multilingual**: Supports multiple Indian languages from day one

## 2. Architecture Design

The system follows a layered architecture where each layer has a specific responsibility:

### Layer 1: User Interface Layer
**Purpose**: Provide multiple ways for users to interact with AI Saathi

**Components**:
- **Mobile App** (Primary Interface)
  - Simple, voice-first design
  - Large buttons and clear text
  - Works on Android (minimum version 6.0)
  - Offline capability for basic features
  
- **Web App** (Secondary Interface)
  - Browser-based access for desktop users
  - Responsive design for mobile browsers
  - No installation required
  
- **WhatsApp Bot** (Future Enhancement)
  - Reach users on familiar platform
  - No app installation needed
  - Works on feature phones

**Key Features**:
- Voice input button (tap to speak)
- Text input option
- Language selector
- Simple chat interface
- Profile setup (one-time)

### Layer 2: Voice & Language Processing Layer
**Purpose**: Convert speech to text and detect the user's language

**Components**:
- **Speech-to-Text (STT) Service**
  - Converts voice input to text
  - Handles multiple Indian languages
  - Filters background noise
  - Returns transcribed text
  
- **Language Detection**
  - Automatically identifies the language being used
  - Handles code-mixing (e.g., Hindi + English)
  - Sets language context for downstream processing
  
- **Text-to-Speech (TTS) Service**
  - Converts AI responses back to voice
  - Natural-sounding voices in local languages
  - Adjustable speech rate

**Technology Choice**:
- Google Cloud Speech-to-Text API (supports 10+ Indian languages)
- Language detection using built-in API capabilities
- Google Cloud Text-to-Speech for voice output

### Layer 3: AI Intelligence Layer
**Purpose**: Understand user intent and generate helpful responses

**Components**:
- **Natural Language Processing (NLP)**
  - Intent classification (what does the user want?)
  - Entity extraction (key information from the query)
  - Context management (remember conversation history)
  
- **Large Language Model (LLM)**
  - Core AI brain of the system
  - Understands questions in natural language
  - Generates human-like responses
  - Explains complex information simply
  - Adapts tone for different user groups

**How It Works**:
1. User query arrives as text (from STT or direct text input)
2. NLP identifies intent: "scheme_search", "eligibility_check", "job_search", etc.
3. NLP extracts entities: location, age, occupation, scheme name, etc.
4. LLM receives structured query + user profile + relevant knowledge
5. LLM generates personalized, contextual response
6. Response is formatted for output

**Technology Choice**:
- OpenAI GPT-4 or GPT-3.5-turbo for LLM
- Custom prompt engineering for Indian context
- Langchain for orchestration and memory management

### Layer 4: Knowledge Base Layer
**Purpose**: Store and retrieve information about schemes, jobs, education, and healthcare

**Data Categories**:

**Government Schemes Database**
- Scheme name and description
- Eligibility criteria (age, income, location, category)
- Required documents
- Application process and deadlines
- Official website links
- Contact information

**Jobs Database**
- Job title and description
- Employer information
- Location and salary range
- Required qualifications
- Application links
- Posting date

**Education Database**
- Scholarships and financial aid
- Educational institutions
- Courses and programs
- Admission requirements
- Fees and duration

**Healthcare Database**
- Nearby hospitals and clinics
- Health insurance schemes
- Vaccination programs
- Emergency contacts
- Health helplines

**Data Structure**:
```json
{
  "scheme_id": "PM_KISAN_2024",
  "name": "PM-KISAN",
  "category": "agriculture",
  "description": "Income support for farmers",
  "eligibility": {
    "occupation": ["farmer"],
    "land_holding": "up to 2 hectares",
    "income": "any"
  },
  "benefits": "₹6000 per year in 3 installments",
  "documents": ["Aadhaar", "Land records", "Bank account"],
  "application_process": "Online through PM-KISAN portal",
  "languages": ["hi", "en", "ta", "te"]
}
```

**Technology Choice**:
- PostgreSQL for structured data (schemes, jobs)
- Vector database (Pinecone or Weaviate) for semantic search
- Regular data updates from public sources

### Layer 5: Recommendation Engine
**Purpose**: Match users with relevant schemes and opportunities

**Components**:

**User Profile Manager**
- Stores user information (age, location, occupation, income level)
- Updates profile based on conversations
- Maintains privacy (minimal data collection)

**Eligibility Checker**
- Compares user profile against scheme criteria
- Returns eligible/not eligible status
- Explains why user is or isn't eligible
- Suggests alternative schemes if not eligible

**Personalization Engine**
- Ranks recommendations based on relevance
- Considers user's past queries and interests
- Prioritizes high-impact schemes
- Filters by location and language

**Matching Algorithm**:
1. Extract user profile attributes
2. Query knowledge base for relevant items
3. Apply eligibility filters
4. Score and rank results
5. Return top 5-10 recommendations

**Technology Choice**:
- Python-based recommendation logic
- Redis for caching user profiles
- Rule-based + ML hybrid approach

### Layer 6: Output Layer
**Purpose**: Deliver responses to users in their preferred format

**Components**:

**Response Formatter**
- Structures AI output for display
- Adds formatting (bullet points, steps)
- Includes action buttons (Apply Now, Learn More)
- Handles error messages gracefully

**Text-to-Speech Converter**
- Converts text responses to voice
- Matches user's input language
- Natural pronunciation of Indian names and terms

**Multi-Channel Delivery**
- Sends response to appropriate interface (app, web, WhatsApp)
- Handles retries if delivery fails
- Logs interactions for improvement

## 3. Data Flow

### Complete User Journey (Voice Input Example)

**Step 1: User Input**
- User opens mobile app
- Taps "Speak" button
- Says: "Mujhe kisan yojana ke baare mein batao" (Tell me about farmer schemes)

**Step 2: Voice Processing**
- Audio captured by mobile app
- Sent to Speech-to-Text API
- Returns: "mujhe kisan yojana ke baare mein batao"
- Language detected: Hindi (hi)

**Step 3: Intent Understanding**
- Text sent to AI Intelligence Layer
- NLP identifies intent: "scheme_search"
- Entities extracted: category="agriculture", language="hi"
- User profile retrieved: occupation="farmer", location="Maharashtra"

**Step 4: Knowledge Retrieval**
- Query sent to Knowledge Base Layer
- Searches for agriculture schemes
- Filters by location (Maharashtra + Central schemes)
- Returns: PM-KISAN, Maharashtra Farmer Scheme, Crop Insurance

**Step 5: Eligibility Check**
- Recommendation Engine checks each scheme
- Compares user profile with eligibility criteria
- PM-KISAN: Eligible ✓
- Maharashtra Farmer Scheme: Eligible ✓
- Crop Insurance: Eligible ✓

**Step 6: Response Generation**
- LLM receives: user query + eligible schemes + user profile
- Generates response in Hindi:
  "आपके लिए 3 योजनाएं उपलब्ध हैं:
  1. PM-KISAN - ₹6000 प्रति वर्ष
  2. Maharashtra Farmer Scheme - ₹10,000 प्रति वर्ष
  3. Crop Insurance - फसल सुरक्षा
  क्या आप किसी योजना के बारे में विस्तार से जानना चाहेंगे?"

**Step 7: Voice Output**
- Text response sent to Text-to-Speech API
- Converts to Hindi audio
- Audio file returned to mobile app

**Step 8: User Receives Response**
- App plays audio response
- Also displays text on screen
- Shows action buttons: "Apply for PM-KISAN", "Learn More"

**Step 9: Follow-up (Optional)**
- User can ask follow-up questions
- System maintains conversation context
- Can drill down into specific schemes

### Data Flow Diagram (Simplified)

```
User → Mobile App → API Gateway → Voice Processing → AI Intelligence
                                                            ↓
User ← Mobile App ← Output Layer ← Recommendation ← Knowledge Base
```

## 4. Technology Stack

### Frontend Technologies

**Mobile App**
- **Framework**: React Native
- **Why**: Single codebase for Android and iOS, large community support
- **UI Library**: React Native Paper (Material Design)
- **State Management**: Redux Toolkit
- **Voice Recording**: react-native-audio-recorder-player

**Web App**
- **Framework**: React.js
- **Why**: Fast development, reusable components
- **UI Library**: Material-UI (MUI)
- **State Management**: Redux Toolkit
- **Styling**: Tailwind CSS

### Backend Technologies

**API Server**
- **Framework**: Node.js + Express.js
- **Why**: Fast, scalable, JavaScript ecosystem
- **Alternative**: Python + FastAPI (if team prefers Python)

**API Gateway**
- **Service**: AWS API Gateway
- **Why**: Handles routing, rate limiting, authentication

**Serverless Functions**
- **Service**: AWS Lambda
- **Why**: Cost-effective for hackathon, auto-scaling
- **Use Cases**: Voice processing, eligibility checks, data formatting

### AI Services

**Large Language Model**
- **Primary**: OpenAI GPT-4 or GPT-3.5-turbo
- **API**: OpenAI API
- **Orchestration**: Langchain (Python)

**Speech-to-Text**
- **Service**: Google Cloud Speech-to-Text API
- **Languages**: Hindi, Tamil, Telugu, Bengali, Marathi, Gujarati, Kannada, Malayalam

**Text-to-Speech**
- **Service**: Google Cloud Text-to-Speech API
- **Voice Quality**: WaveNet voices for natural sound

**Language Detection**
- **Service**: Google Cloud Translation API (language detection)

### Databases

**Primary Database**
- **Type**: PostgreSQL (AWS RDS)
- **Why**: Reliable, supports complex queries, good for structured data
- **Stores**: User profiles, schemes, jobs, education data

**Vector Database**
- **Service**: Pinecone or Weaviate
- **Why**: Fast semantic search for finding relevant schemes
- **Stores**: Embeddings of scheme descriptions

**Cache Layer**
- **Service**: Redis (AWS ElastiCache)
- **Why**: Fast access to frequently requested data
- **Stores**: User sessions, popular queries, scheme summaries

### Cloud Infrastructure (AWS)

**Compute**
- AWS Lambda (serverless functions)
- AWS EC2 (if needed for backend server)

**Storage**
- AWS S3 (audio files, static assets)
- AWS RDS (PostgreSQL database)

**Networking**
- AWS API Gateway (API management)
- AWS CloudFront (CDN for fast content delivery)

**Monitoring**
- AWS CloudWatch (logs and metrics)

**Why AWS?**
- Free tier available for hackathon
- Comprehensive services
- Easy to scale
- Good documentation

### Development Tools

**Version Control**: Git + GitHub
**API Testing**: Postman
**Code Editor**: VS Code
**Project Management**: Trello or Notion
**Communication**: Slack or Discord

## 5. Scalability Considerations

### Multi-Language Expansion
**Current Approach**:
- Start with 3-4 major languages (Hindi, English, Tamil, Telugu)
- Use language codes (ISO 639-1): hi, en, ta, te

**Scaling Strategy**:
- Add new languages by updating STT/TTS configurations
- Translate scheme data using Google Translate API
- Store translations in database with language code
- No code changes needed for new languages

**Implementation**:
```javascript
const supportedLanguages = {
  'hi': 'Hindi',
  'en': 'English',
  'ta': 'Tamil',
  'te': 'Telugu',
  'bn': 'Bengali',
  'mr': 'Marathi'
};
```

### Cloud-Based Scaling
**Horizontal Scaling**:
- AWS Lambda auto-scales based on request volume
- Add more Lambda instances during high traffic
- No manual intervention needed

**Database Scaling**:
- PostgreSQL read replicas for heavy read operations
- Redis cache reduces database load
- Vector database (Pinecone) scales automatically

**Cost Optimization**:
- Use AWS Lambda (pay per request)
- Cache frequently accessed data
- Compress audio files
- Optimize API calls to external services

### Modular System Design
**Benefits**:
- Each layer can be updated independently
- Easy to swap technologies (e.g., change LLM provider)
- Team members can work on different modules simultaneously
- Testing is easier with isolated components

**Module Independence**:
- Voice Processing: Can switch from Google to AWS Transcribe
- AI Layer: Can switch from OpenAI to local LLM
- Database: Can migrate from PostgreSQL to MongoDB
- Frontend: Can add new interfaces (WhatsApp, Telegram) without backend changes

## 6. Security and Privacy

### User Data Protection

**Data Encryption**
- All data in transit encrypted (HTTPS/TLS)
- Database encryption at rest (AWS RDS encryption)
- Audio files encrypted in S3

**Authentication**
- Phone number-based authentication (OTP)
- JWT tokens for session management
- Token expiry after 7 days

**Authorization**
- Users can only access their own data
- Role-based access for admin panel
- API rate limiting to prevent abuse

### Minimal Sensitive Data Storage

**What We Store**:
- Phone number (for login)
- Basic profile: age range, location (district level), occupation
- Conversation history (for context)

**What We DON'T Store**:
- Full name (optional, user choice)
- Aadhaar number or government IDs
- Bank account details
- Exact address
- Income details (only range: <2L, 2-5L, >5L)

**Data Retention**:
- Conversation history: 30 days
- User profile: Until user deletes account
- Audio files: Deleted after processing (not stored)

### Privacy Policy

**Transparency**:
- Clear privacy policy shown during signup
- Explain what data is collected and why
- Option to delete account and all data
- No sharing of data with third parties

**User Control**:
- Users can view their stored data
- Users can edit their profile
- Users can delete conversation history
- Users can opt out of personalization

### Compliance

**For Hackathon**:
- Basic privacy policy
- Secure data handling practices
- No real user data (use test data)

**For Production** (Future):
- GDPR compliance (if expanding internationally)
- Indian data protection laws
- Regular security audits
- Penetration testing

## 7. Future Improvements

### Offline Mode
**Challenge**: Users in rural areas have intermittent connectivity

**Solution**:
- Download frequently asked schemes to mobile app
- Basic chatbot works offline with cached data
- Sync when internet is available
- Progressive Web App (PWA) for web version

**Implementation**:
- Use React Native AsyncStorage
- Store top 100 schemes locally
- Update cache weekly
- Show "Offline Mode" indicator

### Community Feedback Learning
**Challenge**: AI responses may not always be accurate or helpful

**Solution**:
- Add thumbs up/down buttons after each response
- Collect user feedback on answer quality
- Use feedback to improve prompts and knowledge base
- Identify common questions that need better answers

**Implementation**:
- Feedback stored in database
- Weekly review of low-rated responses
- Update LLM prompts based on patterns
- Add missing information to knowledge base

**Advanced** (Future):
- Fine-tune LLM on successful conversations
- Reinforcement learning from human feedback (RLHF)
- A/B testing different response styles

### Integration with Government APIs
**Challenge**: Data in knowledge base may become outdated

**Solution**:
- Integrate with official government APIs (when available)
- Real-time scheme information
- Direct application submission
- Status tracking for applications

**Potential Integrations**:
- DigiLocker API (document verification)
- UMANG API (government services)
- National Career Service API (job listings)
- State government portals

**Benefits**:
- Always up-to-date information
- Reduced manual data entry
- End-to-end service (discovery to application)
- Official verification of eligibility

### Additional Future Features

**Multi-Modal Input**
- Upload documents for eligibility verification
- Image recognition for form filling
- QR code scanning for scheme details

**Community Features**
- User forums for peer support
- Success stories from beneficiaries
- Local language content creation

**Advanced Personalization**
- Proactive notifications about new schemes
- Reminders for application deadlines
- Life event-based recommendations (marriage, childbirth, etc.)

**Analytics Dashboard**
- For government officials to see adoption
- Identify gaps in scheme awareness
- Regional insights and trends

## 8. Implementation Roadmap (Hackathon Timeline)

### Day 1: Foundation
- Set up project structure
- Initialize frontend (React Native + React.js)
- Set up backend (Node.js + Express)
- Configure AWS services
- Create database schema

### Day 2: Core Features
- Implement voice input/output
- Integrate OpenAI API
- Build basic chat interface
- Create sample knowledge base (20-30 schemes)
- Implement language detection

### Day 3: Intelligence
- Build recommendation engine
- Implement eligibility checker
- Add user profile management
- Test with sample queries
- Refine AI prompts

### Day 4: Polish & Demo
- UI/UX improvements
- Add error handling
- Create demo video
- Prepare presentation
- Test with team members
- Deploy to cloud

## 9. Success Criteria

### Technical Success
- System responds within 5 seconds
- Voice recognition accuracy >80%
- Supports 3+ languages
- Handles 50+ concurrent users
- Zero critical bugs during demo

### User Experience Success
- First-time users can get an answer within 2 minutes
- Clear and understandable responses
- Smooth voice interaction
- Works on low-end Android devices

### Impact Success
- Demonstrates real-world value
- Solves a genuine problem
- Scalable architecture
- Impresses hackathon judges
- Potential for real deployment

---

**Document Version**: 1.0  
**Last Updated**: Hackathon Kickoff  
**Team**: AI Saathi Development Team
