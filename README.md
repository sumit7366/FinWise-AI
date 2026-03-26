# FinWise AI 



## What's Working

| Feature | Status |
|---------|--------|
| Landing page | ✅ Full — SEO, mobile, PWA |
| Google OAuth | ✅ Real — users saved to Firestore |
| Email/password login | ✅ Real — bcrypt hashed in Firestore |
| FIRE Calculator | ✅ Working + saves to profile |
| Tax Wizard | ✅ FY2025-26, old+new regime |
| MF X-Ray | ✅ XIRR, overlap, expense ratio |
| Money Health Score | ✅ 6-dimension, saves to Firestore |
| Scenario Simulator | ✅ 6 life scenarios |
| AI Chatbot | ✅ Claude API + Hinglish + history saved |
| Goals CRUD | ✅ Real Firestore — create/update/delete |
| Dashboard | ✅ Real data from Firestore |
| Privacy Policy | ✅ Full 10-section legal page |
| Terms of Service | ✅ Full SEBI-compliant |
| Footer | ✅ App Store, Play Store, PWA, legal |
| Sitemap + robots.txt | ✅ SEO ready |
| Security headers | ✅ XSS, HSTS |

---

## Firebase Firestore Collections

Collections auto-created on first use:

```
users/          → { name, email, password, plan, provider, createdAt }
profiles/       → { userId, moneyScore, fireResult, taxResult, mfResult, ... }
goals/          → { userId, name, icon, targetAmount, savedAmount, ... }
chats/{userId}/messages/ → { role, content, createdAt }
---

## Database Design & ER Diagram

### Entities and Attributes

#### 1. **Users** (Primary Entity)
- **userId** (Primary Key, String, Auto-generated)
- name (String, Required)
- email (String, Required, Unique)
- password (String, Hashed, Optional - for credentials login)
- phone (String, Optional)
- plan (String, Default: 'FREE')
- provider (String, Enum: 'credentials', 'google')
- image (String, Optional - for Google OAuth)
- createdAt (Timestamp)
- updatedAt (Timestamp)

#### 2. **Profiles** (User Profile Data)
- **userId** (Primary Key & Foreign Key → Users.userId)
- moneyScore (Number, 0-100, Calculated)
- monthlyIncome (Number, Optional)
- monthlyExpenses (Number, Optional)
- emergencyFundMonths (Number, Optional)
- insuranceCoverLakhs (Number, Optional)
- fireResult (Object, Contains FIRE calculation results)
- taxResult (Object, Contains tax calculation results)
- mfResult (Object, Contains mutual fund analysis results)
- age (Number, Optional)
- retirementAge (Number, Optional)
- existingCorpus (Number, Optional)
- createdAt (Timestamp)
- updatedAt (Timestamp)

#### 3. **Goals** (Financial Goals)
- **goalId** (Primary Key, String, Auto-generated)
- **userId** (Foreign Key → Users.userId)
- name (String, Required)
- icon (String, Default: '🎯')
- targetAmount (Number, Required)
- savedAmount (Number, Default: 0)
- category (String, Default: 'OTHER')
- targetDate (Date, Optional)
- isCompleted (Boolean, Default: false)
- createdAt (Timestamp)
- updatedAt (Timestamp)

#### 4. **Messages** (Chat History)
- **messageId** (Primary Key, String, Auto-generated)
- **userId** (Foreign Key → Users.userId, Part of collection path)
- role (String, Enum: 'user', 'assistant')
- content (String, Required)
- createdAt (Timestamp)

#### 5. **Nudges** (User Notifications/Reminders)
- **nudgeId** (Primary Key, String, Auto-generated)
- **userId** (Foreign Key → Users.userId)
- type (String, e.g., 'CUSTOM')
- message (String, Required)
- read (Boolean, Default: false)
- createdAt (Timestamp)

### Relationships

```
Users (1) ──── (1) Profiles
    │
    ├── (N) Goals
    │
    ├── (N) Messages
    │
    └── (N) Nudges
```

- **Users ↔ Profiles**: One-to-One (1:1) - Each user has exactly one profile document
- **Users ↔ Goals**: One-to-Many (1:N) - One user can have multiple financial goals
- **Users ↔ Messages**: One-to-Many (1:N) - One user can have multiple chat messages
- **Users ↔ Nudges**: One-to-Many (1:N) - One user can have multiple notification nudges

### ER Diagram (Text Representation)

```
┌─────────────────┐       ┌──────────────────┐
│     Users       │       │    Profiles      │
├─────────────────┤       ├──────────────────┤
│ userId (PK)     │◄──────┤ userId (PK,FK)   │
│ name            │       │ moneyScore       │
│ email (UQ)      │       │ monthlyIncome    │
│ password        │       │ monthlyExpenses  │
│ phone           │       │ emergencyFund... │
│ plan            │       │ insuranceCover.. │
│ provider        │       │ fireResult       │
│ image           │       │ taxResult        │
│ createdAt       │       │ mfResult         │
│ updatedAt       │       │ age              │
└─────────────────┘       │ retirementAge    │
                          │ existingCorpus   │
                          │ createdAt        │
                          │ updatedAt        │
                          └──────────────────┘
                                   │
                                   │ 1:1
                                   │
┌─────────────────┐       ┌──────────────────┐
│     Goals       │       │    Messages      │
├─────────────────┤       ├──────────────────┤
│ goalId (PK)     │       │ messageId (PK)   │
│ userId (FK)     │◄──────┤ userId (FK)      │
│ name            │       │ role             │
│ icon            │       │ content          │
│ targetAmount    │       │ createdAt        │
│ savedAmount     │       └──────────────────┘
│ category        │
│ targetDate      │
│ isCompleted     │
│ createdAt       │
│ updatedAt       │
└─────────────────┘
        │
        │ 1:N
        │
┌─────────────────┐
│    Nudges       │
├─────────────────┤
│ nudgeId (PK)    │
│ userId (FK)     │
│ type            │
│ message         │
│ read            │
│ createdAt       │
└─────────────────┘
```

### ER Diagram (Mermaid DSL)

```mermaid
erDiagram
    USERS {
        string userId PK
        string name
        string email
        string password
        string phone
        string role
        string plan
        timestamp createdAt
    }

    PROFILES {
        string userId PK, FK
        number monthlyIncome
        number monthlyExpenses
        number moneyScore
        json fireResult
        json taxResult
        json mfResult
        timestamp createdAt
    }

    BUSINESS_PROFILE {
        string businessId PK
        string userId FK
        string businessName
        string type
        number revenue
        number expenses
        string gstNumber
        timestamp createdAt
    }

    BANK_ACCOUNT {
        string accountId PK
        string userId FK
        string bankName
        string accountType
        number balance
        string ifscCode
        timestamp createdAt
    }

    TRANSACTIONS {
        string transactionId PK
        string accountId FK
        string userId FK
        number amount
        string type
        string category
        string description
        timestamp createdAt
    }

    PAYMENTS {
        string paymentId PK
        string userId FK
        string orderId
        number amount
        string plan
        string status
        string paymentMethod
        timestamp createdAt
    }

    GOALS {
        string goalId PK
        string userId FK
        string name
        number targetAmount
        number savedAmount
        date deadline
        string status
    }

    MESSAGES {
        string messageId PK
        string userId FK
        string role
        string content
        timestamp createdAt
    }

    NUDGES {
        string nudgeId PK
        string userId FK
        string message
        string type
        boolean read
        timestamp createdAt
    }

    REPORTS {
        string reportId PK
        string userId FK
        string summary
        string scoreSnapshot
        timestamp createdAt
    }

    USERS ||--|| PROFILES : has
    USERS ||--o{ BUSINESS_PROFILE : owns
    USERS ||--o{ BANK_ACCOUNT : has
    BANK_ACCOUNT ||--o{ TRANSACTIONS : records
    USERS ||--o{ TRANSACTIONS : performs
    USERS ||--o{ PAYMENTS : makes
    USERS ||--o{ GOALS : sets
    USERS ||--o{ MESSAGES : chats
    USERS ||--o{ NUDGES : receives
    USERS ||--o{ REPORTS : generates
```

### Work artifacts
- attached: `FinWise_AI_Work_Report.pdf`
- attached: `FinWise_AI_Pitch_Deck.pptx`

### Project Overview Analysis

#### Project Overview in Simple Terms
**FinWise AI** is a comprehensive personal finance management web application built with Next.js and Firebase. It serves as an AI-powered financial advisor specifically designed for Indian users, offering tools to calculate FIRE (Financial Independence, Retire Early), tax optimization, mutual fund analysis, money health scoring, goal tracking, and AI chat support. The app uses Firebase Firestore for data storage and integrates with Anthropic's Claude AI for conversational financial advice.

#### Detailed Features and Functionalities

##### Core Financial Tools
1. **FIRE Calculator** - Calculates retirement corpus needed, monthly SIP requirements, and investment allocation based on age, income, expenses, and retirement goals
2. **Tax Wizard** - Compares old vs new tax regimes (FY 2025-26), identifies missing deductions (80C, 80D, NPS), and provides tax-saving recommendations
3. **MF X-Ray** - Analyzes mutual fund portfolio performance, calculates XIRR, identifies overlap risk, and suggests cost optimization
4. **Money Health Score** - 6-dimension assessment covering emergency funds, insurance, investments, debt, tax planning, and retirement
5. **Scenario Simulator** - Models financial impact of life events like job loss, medical emergencies, etc.
6. **Goals CRUD** - Create, track, and manage financial goals with progress monitoring

##### AI Features
- **AI Chatbot** - Powered by Claude AI, provides personalized financial advice in Hinglish/English, remembers conversation history
- **Smart Nudges** - Automated personalized notifications and reminders based on user profile and behavior

##### User Management
- **Dual Authentication** - Google OAuth and email/password login with bcrypt hashing
- **User Profiles** - Stores all calculation results and personal financial data
- **Dashboard** - Centralized view of money score, goals, and recent nudges

##### Technical Features
- **PWA Ready** - Progressive Web App with service worker, manifest, and offline capabilities
- **SEO Optimized** - Sitemap, robots.txt, meta tags for search engine visibility
- **Mobile Responsive** - Tailwind CSS for responsive design
- **Security Headers** - XSS protection, HSTS, CSP implemented

#### System Workflow

1. **User Registration/Login** → Creates user document and empty profile in Firestore
2. **Profile Setup** → User completes financial profile with income, expenses, goals
3. **Tool Usage** → User runs calculators (FIRE, Tax, MF, Health) → Results saved to profile
4. **AI Interaction** → Chat with AI advisor → Messages stored in subcollection
5. **Goal Management** → Create/update goals → Progress tracked in database
6. **Dashboard View** → Displays aggregated data from all tools and goals
7. **Nudge System** → Automated notifications based on profile data and inactivity

#### Database Design Reasoning

The database design uses **Firebase Firestore** (NoSQL document database) for the following reasons:

1. **Document-Oriented Structure** - Perfect for storing complex nested objects like calculation results (fireResult, taxResult, mfResult)
2. **Real-time Capabilities** - Firestore's real-time listeners enable live dashboard updates
3. **Scalability** - Serverless architecture handles variable user loads
4. **Security** - Firebase Authentication integrates seamlessly with Firestore security rules
5. **Subcollections** - Chat messages stored as subcollections under users for efficient querying
6. **Geographic Distribution** - Hosted in asia-south1 (Mumbai) for low latency in India

#### Assumptions
- All users are Indian residents (tax calculations, currency in INR, regional financial knowledge)
- Free tier users have basic access; premium features might be planned but not implemented
- Data persistence relies on Firebase's reliability; no local backups mentioned
- AI responses are generated in real-time without caching for freshness
- All calculations assume standard Indian financial parameters (FY 2025-26 tax slabs, etc.)

---

## Deploy to Vercel

```bash
npm i -g vercel
vercel --prod
```

Add all `.env.local` variables in Vercel Dashboard → Settings → Environment Variables.

---

## Hackathon Demo Script (5-7 minutes)

1. **Landing page** → scroll through features, show pricing
2. **Register** with Google or email → lands on dashboard
3. **Dashboard** → explain Money Score, nudges
4. **Tools → FIRE Planner** → age 28, income ₹80K, retire at 50 → show results
5. **Tools → Tax Wizard** → salary ₹12L → show old vs new regime, missing deductions
6. **AI Advisor** → type in Hindi: "Mera SIP kitna hona chahiye?"
7. **Goals** → create "Dream Home ₹40L" goal
8. **Scenario → Job Loss** → show runway calculation
9. Show **Firestore console** to prove data is really being saved

---

## Support

- Setup issues: sumitranjanhisu@gmail.com
- Project Demo questions: singhalmehak04@gmail.com
