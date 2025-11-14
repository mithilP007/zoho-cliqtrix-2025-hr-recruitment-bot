# System Architecture

## Overview

This document outlines the technical architecture of the HR Recruitment Bot for Zoho Cliqtrix 2025.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        VISITOR INTERFACE                         │
│                      (Website/Mobile App)                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ZOHO SALESIQ BOT LAYER                       │
│              (Codeless Bot Builder + Deluge)                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  - Conversation Flow Management                          │   │
│  │  - Input Validation & User Data Collection              │   │
│  │  - Card/Carousel UI Components                          │   │
│  │  - Intent Detection & Response Generation               │   │
│  └─────────────────────────────────────────────────────────┘   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
            ▼              ▼              ▼
  ┌─────────────┐  ┌──────────────┐  ┌──────────┐
  │  ZIA AI     │  │ ZOHO FLOW    │  │  ZOHO    │
  │  PLATFORM   │  │  (Webhooks)  │  │  VAULT   │
  └──────┬──────┘  └──────┬───────┘  └────┬─────┘
         │                │                │
         │                │                │
         ▼                ▼                ▼
  ┌─────────────────────────────────────────────────────────┐
  │              INTEGRATION & BUSINESS LOGIC                │
  │  ┌─────────────────────────────────────────────────┐    │
  │  │  OAuth 2.0 Authentication Manager               │    │
  │  │  (Tokens managed via Zoho Vault)                │    │
  │  └─────────────────────────────────────────────────┘    │
  └──────────────────────┬──────────────────────────────────┘
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
       ▼                 ▼                 ▼
  ┌──────────┐    ┌──────────┐     ┌──────────────┐
  │  ZOHO    │    │  TWILIO  │     │    ZOHO      │
  │ RECRUIT  │    │   API    │     │  WORKDRIVE   │
  │   API    │    │  (OTP)   │     │     API      │
  └──────────┘    └──────────┘     └──────────────┘
```

## Component Details

### 1. Frontend Layer (Zoho SalesIQ)
**Technology**: Zoho SalesIQ Codeless Bot Builder + Deluge Scripts

**Responsibilities**:
- Conversational UI management
- Display job listings in carousel format
- Collect user input (forms, buttons, text)
- Real-time visitor interaction
- Error handling and user feedback

**Features**:
- Card-based UI components
- Multi-step conversation flows
- Context management across sessions
- Mobile-responsive interface

### 2. AI Intelligence Layer (Zia)
**Technology**: Zia Skills Platform

**Responsibilities**:
- Skill-to-job matching algorithm
- Resume quality analysis
- Intent detection from natural language
- Smart job recommendations
- Conversational context understanding

**AI Models**:
- NLU (Natural Language Understanding)
- Entity extraction
- Sentiment analysis
- Pattern matching

### 3. Workflow Automation (Zoho Flow)
**Technology**: Zoho Flow + Webhooks

**Responsibilities**:
- OTP workflow orchestration
- Third-party API integration
- Event-driven automation
- Error handling and retries
- Logging and monitoring

**Key Workflows**:
1. OTP Generation → Twilio SMS → Validation
2. Resume Upload → WorkDrive → Link to Candidate
3. Application Submit → Create Candidate → Link to Job
4. Status Update → Notify Bot → Update UI

### 4. Security & Authentication (Zoho Vault)
**Technology**: Zoho Vault + OAuth 2.0

**Responsibilities**:
- Secure credential storage
- API token management
- Token refresh automation
- Access control
- Audit logging

**Stored Credentials**:
- Zoho Recruit OAuth tokens
- Twilio API credentials
- WorkDrive access keys
- Zia AI API keys

### 5. HR Backend (Zoho Recruit)
**Technology**: Zoho Recruit REST API v2

**Responsibilities**:
- Job posting management
- Candidate profile storage
- Application tracking
- Interview scheduling
- Status management

**Key API Endpoints**:
```
GET    /recruit/v2/Jobs/search
GET    /recruit/v2/Jobs/{job_id}
POST   /recruit/v2/Candidates
GET    /recruit/v2/Candidates/search
POST   /recruit/v2/JobOpenings/{job_id}/Actions/associate
GET    /recruit/v2/Events?candidateId={id}
```

### 6. File Storage (Zoho WorkDrive)
**Technology**: Zoho WorkDrive API

**Responsibilities**:
- Resume file storage
- File access management
- Shareable link generation
- File organization by candidate

**Storage Structure**:
```
WorkDrive/
└── HR-Recruitment-Bot/
    └── Resumes/
        ├── 2025/
        │   ├── Jan/
        │   ├── Feb/
        │   └── ...
        └── Archives/
```

### 7. Communication (Twilio)
**Technology**: Twilio SMS API

**Responsibilities**:
- OTP SMS delivery
- Phone number verification
- Delivery status tracking
- Rate limiting

## Data Flow

### Flow 1: Job Application
```
1. Visitor clicks "Apply" on job
   ↓
2. Bot collects: Name, Email, Phone, Resume
   ↓
3. Zoho Flow triggers Twilio OTP
   ↓
4. Visitor enters OTP code
   ↓
5. Bot validates OTP
   ↓
6. Resume uploaded to WorkDrive
   ↓
7. Candidate created in Zoho Recruit
   ↓
8. Candidate associated with Job
   ↓
9. Confirmation sent to visitor
```

### Flow 2: Smart Job Search
```
1. Bot asks for: Skills, Experience, Salary
   ↓
2. Input sent to Zia AI for processing
   ↓
3. Zia extracts entities & intent
   ↓
4. Zoho Recruit API queried with criteria
   ↓
5. Zia ranks jobs by match score
   ↓
6. Top matches displayed in carousel
```

### Flow 3: Application Status Check
```
1. Visitor provides email
   ↓
2. Bot queries Recruit API for candidate
   ↓
3. Fetch all applications
   ↓
4. For each application:
   - Get job details
   - Get current status
   - Check for scheduled events
   ↓
5. Display in interactive list
   ↓
6. If rejected → Zia suggests similar jobs
```

## Security Considerations

### 1. Authentication
- OAuth 2.0 for all API calls
- Token rotation every 1 hour
- Refresh tokens stored in Vault
- HTTPS only communication

### 2. Data Privacy
- PII encrypted in transit
- Resume files access-controlled
- Phone numbers hashed for storage
- GDPR compliant data handling

### 3. Rate Limiting
- Twilio: 100 SMS/hour per number
- Recruit API: 2000 calls/day
- WorkDrive: 1000 uploads/day
- Bot handles rate limit gracefully

### 4. Error Handling
- Try-catch blocks in all API calls
- Fallback responses for failures
- User-friendly error messages
- Automatic retry with exponential backoff

## Scalability

### Horizontal Scaling
- SalesIQ supports 100+ concurrent bots
- Stateless bot design allows scaling
- API rate limits managed per account

### Performance Optimization
- Caching frequently accessed jobs
- Lazy loading in carousels
- Pagination for large result sets
- Async API calls where possible

### Monitoring
- API call logging
- Response time tracking
- Error rate monitoring
- User session analytics

## Technology Stack Summary

| Component | Technology | Purpose |
|-----------|-----------|----------|
| Frontend Bot | Zoho SalesIQ | User interface |
| Scripting | Deluge | Bot logic |
| AI | Zia Skills Platform | Intelligent matching |
| Workflow | Zoho Flow | Automation |
| HR System | Zoho Recruit API | Job & candidate mgmt |
| Storage | Zoho WorkDrive API | Resume files |
| Auth | Zoho Vault + OAuth 2.0 | Security |
| SMS | Twilio API | OTP delivery |

## Deployment Architecture

### Production Environment
```
Zoho SalesIQ Bot (Hosted by Zoho)
├── Connected to: Zoho Recruit
├── Connected to: Zoho WorkDrive  
├── Connected to: Zoho Vault
├── Connected to: Zoho Flow
│   └── Webhook to: Twilio
└── AI powered by: Zia
```

### Development Environment
- Zoho SalesIQ Sandbox
- Test Recruit account
- Separate WorkDrive folder
- Development Twilio number

## Integration Points

### Zoho Recruit
- **Authentication**: OAuth 2.0
- **Rate Limit**: 2000 API calls/day
- **Data Format**: JSON
- **Webhook Support**: Yes (for status updates)

### Twilio
- **Authentication**: API Key + Secret
- **Rate Limit**: Based on account tier
- **Delivery Time**: < 30 seconds
- **Webhook**: Delivery status callbacks

### WorkDrive
- **Authentication**: OAuth 2.0
- **File Size Limit**: 50 MB/file
- **Storage Quota**: Based on plan
- **Access**: Shareable links with expiry

## Future Enhancements

1. **Video Interview Integration**
   - Integrate Zoho Meeting for video calls
   - Schedule interviews directly from bot

2. **Advanced Analytics**
   - Candidate source tracking
   - Conversion rate analysis
   - Time-to-hire metrics

3. **Multi-language Support**
   - Zia NLP for multiple languages
   - Auto-detect visitor language

4. **Chatbot Voice**
   - Voice input for applications
   - Text-to-speech responses

---

**Last Updated**: November 2025  
**Version**: 1.0  
**Author**: Abishek P
