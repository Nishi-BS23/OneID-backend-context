# 🚀 RVL One ID Backend - সম্পূর্ণ Codebase Context (বাংলা)

## 📑 সূচিপত্র
1. [প্রজেক্ট ওভারভিউ](#প্রজেক্ট-ওভারভিউ)
2. [সিস্টেম আর্কিটেকচার](#সিস্টেম-আর্কিটেকচার)
3. [টেকনোলজি স্ট্যাক](#টেকনোলজি-স্ট্যাক)
4. [ডিরেক্টরি স্ট্রাকচার](#ডিরেক্টরি-স্ট্রাকচার)
5. [মূল মডিউল এবং তাদের কাজ](#মূল-মডিউল-এবং-তাদের-কাজ)
6. [ডাটাবেস স্কিমা](#ডাটাবেস-স্কিমা)
7. [API এন্ডপয়েন্ট সামারি](#api-এন্ডপয়েন্ট-সামারি)
8. [Authentication & Authorization](#authentication--authorization)
9. [Payment Integration](#payment-integration)
10. [কিভাবে নতুন কাজ শুরু করবেন](#কিভাবে-নতুন-কাজ-শুরু-করবেন)
11. [Development Workflow](#development-workflow)
12. [Troubleshooting Guide](#troubleshooting-guide)

---

## 📖 প্রজেক্ট ওভারভিউ

**RVL One ID Backend** হলো একটি Identity and Access Management (IAM) সিস্টেম যা FastAPI দিয়ে তৈরি করা হয়েছে। এটি একটি ডিজিটাল আইডেন্টিটি সিস্টেম যেখানে:

- ✅ **User Registration & Authentication** - Keycloak দিয়ে নিরাপদ
- ✅ **NID Verification** - জাতীয় পরিচয়পত্র যাচাইকরণ
- ✅ **Document Management** - ডকুমেন্ট আপলোড, ভেরিফিকেশন, VDS QR কোড জেনারেশন
- ✅ **Payment Integration** - DGPay/bKash/Nagad QR payment
- ✅ **Service Provider Portal** - সার্ভিস প্রোভাইডারদের জন্য আলাদা সিস্টেম
- ✅ **OTP System** - SMS এবং Email OTP (AWS SES)
- ✅ **Media Storage** - AWS S3 integration
- ✅ **Background Jobs** - Celery দিয়ে async task processing

### 🎯 মূল উদ্দেশ্য
একটি সেন্ট্রালাইজড ডিজিটাল আইডেন্টিটি সিস্টেম তৈরি করা যেখানে ব্যবহারকারীরা তাদের ডকুমেন্ট নিরাপদে সংরক্ষণ করতে পারবে এবং বিভিন্ন সার্ভিস প্রোভাইডারের সাথে শেয়ার করতে পারবে।

---

## 🏗️ সিস্টেম আর্কিটেকচার

### আর্কিটেকচার ডায়াগ্রাম (লজিক্যাল ভিউ)

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Client Applications                         │
│  (Mobile App, Web Portal, Service Provider Dashboard)              │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             │ HTTPS/REST API
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      FastAPI Application                            │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  API Layer (Controllers/Routes)                               │  │
│  │  - Rate Limiting (SlowAPI)                                    │  │
│  │  - CORS Middleware                                            │  │
│  │  - Exception Handling                                         │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Business Logic Layer (Services)                              │  │
│  │  - Auth Service                                               │  │
│  │  - Document Service                                           │  │
│  │  - Payment Service                                            │  │
│  │  - User Service                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Core Components                                              │  │
│  │  - Keycloak Integration                                       │  │
│  │  - Redis Cache                                                │  │
│  │  - Celery Task Queue                                          │  │
│  └──────────────────────────────────────────────────────────────┘  │
└──────────┬────────────┬────────────┬────────────┬─────────────────┘
           │            │            │            │
           ▼            ▼            ▼            ▼
    ┌──────────┐  ┌─────────┐  ┌─────────┐  ┌──────────┐
    │ Keycloak │  │  Redis  │  │PostgreSQL│  │ AWS S3   │
    │  (IAM)   │  │ (Cache) │  │   (DB)  │  │ (Storage)│
    └──────────┘  └─────────┘  └─────────┘  └──────────┘
           │                                        │
           ▼                                        ▼
    ┌──────────────────┐                   ┌──────────────┐
    │ External Services │                   │   AWS SES    │
    │ - NID Verify API  │                   │ (Email OTP)  │
    │ - DGPay API       │                   │   AWS KMS    │
    │ - VDS TSP API     │                   │ (Encryption) │
    └──────────────────┘                   └──────────────┘
```

### Request Flow Example (Document Upload)

```
1. User → [POST /documents/upload] → FastAPI
2. FastAPI → JWT Token Validation → Keycloak
3. Keycloak → Returns User Info
4. FastAPI → Upload to S3 → AWS S3
5. AWS S3 → Returns File URL
6. FastAPI → Save Metadata → PostgreSQL
7. FastAPI → Generate VDS QR → TSP API
8. TSP API → Returns Signed QR Code
9. FastAPI → Update Document Record → PostgreSQL
10. FastAPI → Return Response → User
```

---

## 🛠️ টেকনোলজি স্ট্যাক

### Backend Core
| Technology | Version | Purpose |
|-----------|---------|---------|
| **Python** | 3.13.5 | Programming Language |
| **FastAPI** | 0.104.1+ | Web Framework for REST APIs |
| **Uvicorn** | 0.27.0+ | ASGI Server |
| **Pydantic** | 2.5.0+ | Data Validation & Settings |
| **SQLAlchemy** | 2.0.30+ | ORM for Database |
| **Alembic** | 1.12.1+ | Database Migrations |

### Infrastructure & Services
| Technology | Purpose |
|-----------|---------|
| **Keycloak** | Identity & Access Management (IAM) |
| **PostgreSQL 17** | Primary Database |
| **Redis 7.4** | Caching & Session Management |
| **Celery** | Async Task Queue |
| **Docker** | Containerization |
| **Poetry** | Dependency Management |

### Cloud & External Services
| Service | Purpose |
|---------|---------|
| **AWS S3** | File/Document Storage |
| **AWS SES** | Email OTP Delivery |
| **AWS KMS** | Asymmetric Encryption |
| **SSL Wireless** | SMS OTP Delivery |
| **NID Verification API** | National ID Verification |
| **DGPay** | Payment Gateway |
| **TSP (VDS)** | Digital Signature for QR Codes |

### Development & Quality Tools
- **pytest** - Testing Framework
- **black** - Code Formatting
- **flake8** - Linting
- **mypy** - Type Checking
- **bandit** - Security Checks

---

## 📂 ডিরেক্টরি স্ট্রাকচার

```
rvl-one-id-backend/
│
├── app/                          # মূল অ্যাপ্লিকেশন কোড
│   ├── main.py                   # FastAPI app initialization
│   ├── routes.py                 # সব routes register করা হয়
│   │
│   ├── core/                     # Core functionality & configs
│   │   ├── config.py            # Environment settings
│   │   ├── database.py          # DB connection & session management
│   │   ├── redis.py             # Redis connection
│   │   ├── keycloak.py          # Keycloak configuration
│   │   ├── celery.py            # Celery configuration
│   │   ├── response.py          # Standard API response format
│   │   ├── exceptions/          # Custom exception handlers
│   │   ├── logging.py           # Logging setup
│   │   ├── limiter.py           # Rate limiting config
│   │   ├── payment.py           # Payment gateway config
│   │   ├── dgpay.py             # DGPay specific settings
│   │   ├── vds.py               # VDS/QR code settings
│   │   ├── nid.py               # NID verification settings
│   │   └── recaptcha.py         # reCAPTCHA settings
│   │
│   ├── modules/                 # Feature modules (মূল features)
│   │   │
│   │   ├── auth/               # Authentication module
│   │   │   ├── controller.py   # API endpoints (login, register, etc.)
│   │   │   ├── service.py      # Business logic
│   │   │   └── models.py       # Request/Response models
│   │   │
│   │   ├── user/               # User management
│   │   │   ├── controller.py   # User CRUD endpoints
│   │   │   ├── service.py      # User business logic
│   │   │   └── models.py       # User data models
│   │   │
│   │   ├── document/           # Document management
│   │   │   ├── controller.py   # Document upload/download endpoints
│   │   │   ├── service.py      # Document processing logic
│   │   │   ├── models.py       # Document request/response models
│   │   │   ├── orm_models.py   # Database models
│   │   │   └── s3_utils.py     # S3 upload utilities
│   │   │
│   │   ├── payment/            # Payment processing
│   │   │   ├── controller.py   # Payment endpoints
│   │   │   ├── service.py      # Payment logic
│   │   │   ├── dgpay_client.py # DGPay API client
│   │   │   ├── models.py       # Payment models
│   │   │   ├── orm_models.py   # Payment DB models
│   │   │   └── enums.py        # Payment status enums
│   │   │
│   │   ├── otp/                # OTP management
│   │   │   ├── controller.py   # OTP send/verify endpoints
│   │   │   └── service.py      # OTP generation & verification
│   │   │
│   │   ├── nid/                # NID verification
│   │   │   ├── controller.py   # NID verify endpoint
│   │   │   ├── service.py      # NID API integration
│   │   │   └── models.py       # NID data models
│   │   │
│   │   ├── vds/                # VDS/QR code generation
│   │   │   ├── controller.py   # VDS endpoints
│   │   │   ├── service.py      # VDS signing logic
│   │   │   └── models.py       # VDS models
│   │   │
│   │   ├── media/              # Media file handling
│   │   │   ├── controller.py   # File upload endpoints
│   │   │   ├── service.py      # S3 integration
│   │   │   └── models.py       # Media models
│   │   │
│   │   ├── admin/              # Admin & Service Provider management
│   │   │   ├── controller.py   # Admin endpoints
│   │   │   ├── service.py      # Admin logic
│   │   │   ├── models.py       # Admin models
│   │   │   └── orm_models.py   # Service Provider, Categories tables
│   │   │
│   │   └── derived_documents/  # Derived document types
│   │       ├── controller.py   # Derived document endpoints
│   │       ├── service.py      # Derived document logic
│   │       └── orm_models.py   # DB models
│   │
│   └── utils/                  # Utility functions
│       ├── keycloak.py         # Keycloak helper functions
│       ├── limiter.py          # Rate limiter instance
│       ├── recaptcha.py        # reCAPTCHA verification
│       └── provider_auth.py    # Service provider auth helpers
│
├── alembic/                    # Database migrations
│   ├── versions/               # Migration files
│   ├── env.py                  # Alembic environment config
│   └── script.py.mako          # Migration template
│
├── tests/                      # Test files
│   ├── unit/                   # Unit tests
│   ├── integration/            # Integration tests
│   └── conftest.py             # Test fixtures
│
├── scripts/                    # Utility scripts
│   ├── celery.sh              # Celery management script
│   ├── build-and-push.sh      # Docker build script
│   └── lint.sh                # Code quality check script
│
├── examples/                   # Example code & demos
│   ├── payment_example.py     # Payment integration examples
│   └── crypto_examples.py     # Encryption examples
│
├── context-docs/              # Additional documentation
│
├── docker-compose.dev.yml     # Development Docker setup
├── docker-compose.yml         # Production Docker setup
├── Dockerfile                 # App container definition
├── celery.Dockerfile         # Celery worker container
├── pyproject.toml            # Poetry dependencies & config
├── alembic.ini               # Alembic configuration
├── .env                      # Environment variables (not in git)
├── .env.example              # Environment template
│
└── Documentation Files:
    ├── README.md                      # Main documentation
    ├── KEYCLOAK_SETUP.md             # Keycloak configuration guide
    ├── PAYMENT_API_EXAMPLES.md       # Payment API usage
    ├── CELERY_SETUP.md               # Celery setup guide
    ├── NID_IMPLEMENTATION_COMPLETE.md
    ├── DOCUMENT_CATEGORIES_IMPLEMENTATION.md
    └── SERVICE_PROVIDER_CATEGORIES_IMPLEMENTATION.md
```

---

## 🎯 মূল মডিউল এবং তাদের কাজ

### 1. **Auth Module** (`app/modules/auth/`)

**কাজ:** User authentication, registration, login, logout, password management

**প্রধান endpoints:**
- `POST /api/v1/auth/register` - নতুন user registration
- `POST /api/v1/auth/login` - User login (JWT token return করে)
- `POST /api/v1/auth/logout` - Logout
- `POST /api/v1/auth/refresh` - Token refresh
- `POST /api/v1/auth/forgot-password` - Password reset request
- `POST /api/v1/auth/change-password` - Password change
- `POST /api/v1/auth/validate-unique-identity` - NID/Phone uniqueness check

**কিভাবে কাজ করে:**
1. User credentials receive করে
2. Keycloak এর সাথে communicate করে authentication এর জন্য
3. JWT token generate করে return করে
4. reCAPTCHA validation করে (security)

**Important files:**
- `controller.py` - API routes define করা
- `service.py` - Keycloak API calls, token management
- `models.py` - Request/response schemas

---

### 2. **User Module** (`app/modules/user/`)

**কাজ:** User profile management, user data CRUD operations

**প্রধান endpoints:**
- `GET /api/v1/users/me` - Current user profile
- `PUT /api/v1/users/me` - Update profile
- `GET /api/v1/users/{user_id}` - Get user by ID (admin)
- `DELETE /api/v1/users/me` - Account deactivation
- `POST /api/v1/users/me/mfa/enable` - Enable MFA
- `POST /api/v1/users/me/mfa/verify` - Verify MFA

**User Attributes (Keycloak এ stored):**
- `nid`, `one_id`, `phone_number`, `dob`, `passport_number`
- `mfa_verified`, `mfa_enabled`, `mfa_method`
- `profilePhotoPath`, `idCardFrontPath`, `idCardBackPath`

---

### 3. **Document Module** (`app/modules/document/`)

**কাজ:** Document upload, storage, verification, sharing with service providers

**প্রধান endpoints:**
- `POST /api/v1/documents/upload` - Upload document (Service Provider)
- `GET /api/v1/documents/` - List all documents (paginated)
- `GET /api/v1/documents/{document_id}` - Get document details
- `GET /api/v1/documents/categories` - Get document categories
- `POST /api/v1/documents/{document_id}/request` - Request document
- `POST /api/v1/documents/requests/{request_id}/approve` - Approve request
- `POST /api/v1/documents/requests/{request_id}/reject` - Reject request

**Document Flow:**
```
1. Service Provider uploads document → S3
2. Document metadata saved → PostgreSQL
3. VDS QR code generated → TSP API
4. User receives notification
5. User can view/share document
6. Other providers can request access
7. User approves/rejects request
```

**Database Tables:**
- `documents` - Document metadata
- `document_categories` - Document types (NID, Passport, etc.)
- `document_requests` - Access requests from service providers

---

### 4. **Payment Module** (`app/modules/payment/`)

**কাজ:** QR code based payment processing (DGPay, bKash, Nagad)

**প্রধান endpoints:**
- `POST /api/v1/payments/initiate` - Start payment
- `POST /api/v1/payments/callback` - Payment callback (webhook)
- `GET /api/v1/payments/{transaction_id}` - Payment status
- `GET /api/v1/payments/history` - Payment history

**Payment Flow:**
```
1. User scans QR code → App extracts payment info
2. App calls /initiate → Backend validates QR
3. Backend calls DGPay API → Payment initiated
4. DGPay processes payment → Sends callback
5. Backend updates transaction status
6. User receives confirmation
```

**Database Tables:**
- `payment_transactions` - Payment records with status tracking

**Payment Statuses:**
- `PENDING`, `PROCESSING`, `SUCCESS`, `FAILED`, `CANCELLED`

---

### 5. **OTP Module** (`app/modules/otp/`)

**কাজ:** OTP generation, sending (SMS/Email), verification

**প্রধান endpoints:**
- `POST /api/v1/otp/send` - Send OTP
- `POST /api/v1/otp/verify` - Verify OTP

**OTP Flow:**
```
1. User requests OTP (phone/email)
2. Backend generates 6-digit OTP
3. Stores in Redis with 5 min expiry
4. Sends via SMS (SSL Wireless) or Email (AWS SES)
5. User enters OTP
6. Backend verifies from Redis
7. Returns success/failure
```

**Configuration:**
- SMS: SSL Wireless API
- Email: AWS SES
- OTP expiry: 5 minutes
- Rate limiting: Max 3 requests per 15 minutes

---

### 6. **NID Module** (`app/modules/nid/`)

**কাজ:** National ID verification with government API

**প্রধান endpoints:**
- `POST /api/v1/nid/verify` - Verify NID

**NID Verification Flow:**
```
1. User provides NID number + DOB
2. Backend calls government NID API
3. API returns verification result
4. Backend saves verification status
5. Returns user data if verified
```

---

### 7. **VDS Module** (`app/modules/vds/`)

**কাজ:** Visible Digital Seal (VDS) QR code generation for documents

**প্রধান endpoints:**
- `POST /api/v1/vds/sign` - Sign and generate VDS
- `GET /api/v1/vds/status/{ticket_id}` - Check signing status
- `POST /api/v1/vds/extract-qr` - Extract data from QR code

**VDS Flow:**
```
1. Document uploaded
2. Backend prepares VDS payload
3. Calls TSP (Trust Service Provider) API
4. TSP digitally signs data
5. Generates QR code with signed data
6. QR code stored with document
```

---

### 8. **Admin Module** (`app/modules/admin/`)

**কাজ:** Service provider management, categories, services

**প্রধান endpoints:**
- `POST /api/v1/admin/service-providers` - Create service provider
- `GET /api/v1/admin/service-providers` - List providers
- `POST /api/v1/admin/services` - Create service
- `GET /api/v1/admin/services` - List services
- `POST /api/v1/admin/categories` - Create category

**Database Tables:**
- `service_providers` - Provider organizations
- `service_provider_categories` - Provider types
- `services` - Services offered
- `provider_user_associations` - Provider-User mapping

---

### 9. **Media Module** (`app/modules/media/`)

**কাজ:** File upload to S3, presigned URL generation

**প্রধান endpoints:**
- `POST /api/v1/media/upload` - Upload file to S3
- `GET /api/v1/media/presigned-url` - Get presigned download URL

---

## 🗄️ ডাটাবেস স্কিমা

### Core Tables

#### 1. **users** (Managed by Keycloak)
Keycloak database এ থাকে, custom attributes আছে:
- `id`, `username`, `email`, `first_name`, `last_name`
- `nid`, `one_id`, `phone_number`, `dob`
- `mfa_enabled`, `mfa_verified`, `status`

#### 2. **service_provider_categories**
```sql
id                  UUID PRIMARY KEY
name                VARCHAR(255) UNIQUE
is_active           BOOLEAN
created_at          TIMESTAMP
updated_at          TIMESTAMP
```

#### 3. **service_providers**
```sql
id                  UUID PRIMARY KEY
name                VARCHAR(255) UNIQUE
bin_number          VARCHAR(50)
contact_email       VARCHAR(255)
contact_phone       VARCHAR(20)
category_id         UUID → service_provider_categories(id)
is_active           BOOLEAN
verification_status VARCHAR(50)
keycloak_user_id    UUID
created_at          TIMESTAMP
updated_at          TIMESTAMP
```

#### 4. **document_categories**
```sql
id                  UUID PRIMARY KEY
name                VARCHAR(255) UNIQUE
description         TEXT
is_active           BOOLEAN
is_system_reserved  BOOLEAN
created_at          TIMESTAMP
updated_at          TIMESTAMP
```

#### 5. **services**
```sql
id                             UUID PRIMARY KEY
name                           VARCHAR(255)
logo                           TEXT
document_category_id           UUID → document_categories(id)
service_provider_category_id   UUID → service_provider_categories(id)
is_active                      BOOLEAN
created_at                     TIMESTAMP
updated_at                     TIMESTAMP
```

#### 6. **documents**
```sql
id                      UUID PRIMARY KEY
user_id                 UUID (Keycloak user)
document_category_id    UUID → document_categories(id)
document_number         VARCHAR(100)
file_path               TEXT (S3 path)
issue_date              DATE
expiry_date             DATE
issuing_authority       VARCHAR(255)
verification_status     VARCHAR(50)
vds_qr_data             TEXT (QR code data)
created_at              TIMESTAMP
updated_at              TIMESTAMP
```

#### 7. **document_requests**
```sql
id                      UUID PRIMARY KEY
document_id             UUID → documents(id)
requester_user_id       UUID (Service provider)
requester_organization  VARCHAR(255)
request_status          VARCHAR(50)
purpose                 TEXT
requested_at            TIMESTAMP
approved_at             TIMESTAMP
rejected_at             TIMESTAMP
```

#### 8. **payment_transactions**
```sql
id                  UUID PRIMARY KEY
user_id             UUID
transaction_id      VARCHAR(255)
order_id            VARCHAR(255)
amount              DECIMAL(10,2)
currency            VARCHAR(3)
status              VARCHAR(50)
payment_method      VARCHAR(50)
qr_code_data        TEXT
callback_data       JSONB
created_at          TIMESTAMP
updated_at          TIMESTAMP
```

#### 9. **provider_user_associations**
```sql
id                      UUID PRIMARY KEY
service_provider_id     UUID → service_providers(id)
keycloak_user_id        UUID
role                    VARCHAR(50)
created_at              TIMESTAMP
```

### Migration Management
- Alembic দিয়ে database migrations handle করা হয়
- Migration files: `alembic/versions/`
- Run migrations: `alembic upgrade head`

---

## 🌐 API এন্ডপয়েন্ট সামারি

### Authentication APIs
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/auth/register` | নতুন user registration | ❌ |
| POST | `/api/v1/auth/login` | User login | ❌ |
| POST | `/api/v1/auth/logout` | User logout | ✅ |
| POST | `/api/v1/auth/refresh` | Refresh access token | ✅ |
| POST | `/api/v1/auth/forgot-password` | Password reset request | ❌ |
| POST | `/api/v1/auth/change-password` | Change password | ✅ |

### User APIs
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/v1/users/me` | Current user profile | ✅ |
| PUT | `/api/v1/users/me` | Update profile | ✅ |
| DELETE | `/api/v1/users/me` | Deactivate account | ✅ |
| POST | `/api/v1/users/me/mfa/enable` | Enable MFA | ✅ |

### Document APIs
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/documents/upload` | Upload document | ✅ (Provider) |
| GET | `/api/v1/documents/` | List documents | ✅ |
| GET | `/api/v1/documents/{id}` | Get document | ✅ |
| POST | `/api/v1/documents/{id}/request` | Request access | ✅ (Provider) |
| POST | `/api/v1/documents/requests/{id}/approve` | Approve request | ✅ |

### Payment APIs
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/payments/initiate` | Initiate payment | ✅ |
| POST | `/api/v1/payments/callback` | Payment webhook | ❌ |
| GET | `/api/v1/payments/{id}` | Payment status | ✅ |
| GET | `/api/v1/payments/history` | Payment history | ✅ |

### OTP APIs
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/otp/send` | Send OTP | ❌ |
| POST | `/api/v1/otp/verify` | Verify OTP | ❌ |

### NID APIs
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/nid/verify` | Verify NID | ✅ |

### Admin APIs
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/admin/service-providers` | Create provider | ✅ (Admin) |
| GET | `/api/v1/admin/service-providers` | List providers | ✅ (Admin) |
| POST | `/api/v1/admin/services` | Create service | ✅ (Admin) |

---

## 🔐 Authentication & Authorization

### Keycloak Integration

**Keycloak হলো** একটি Open Source Identity and Access Management (IAM) সিস্টেম।

**এখানে Keycloak এর কাজ:**
1. User registration এবং authentication
2. JWT token generation
3. User profile management
4. Role-based access control (RBAC)
5. Custom user attributes storage

### Token Flow

```
1. User logs in → Keycloak validates credentials
2. Keycloak returns JWT token (access_token + refresh_token)
3. Client stores token
4. Every API request → Token sent in Authorization header
5. Backend validates token with Keycloak public key
6. If valid → Request processed
7. If expired → Refresh token used to get new access token
```

### JWT Token Structure

```json
{
  "sub": "user-uuid",
  "email": "user@example.com",
  "preferred_username": "john_doe",
  "realm_access": {
    "roles": ["user", "provider", "admin"]
  },
  "nid": "1234567890",
  "one_id": "ONE123456",
  "phone_number": "+8801712345678",
  "provider": true,  // Service provider flag
  "exp": 1700000000,
  "iat": 1699999000
}
```

### Authorization Levels

1. **Public Endpoints** - কোনো auth লাগে না
   - `/auth/register`, `/auth/login`, `/otp/send`

2. **User Endpoints** - JWT token লাগে
   - `/users/me`, `/documents/`, `/payments/initiate`

3. **Provider Endpoints** - JWT + `provider` claim = true
   - `/documents/upload`, `/documents/{id}/request`

4. **Admin Endpoints** - JWT + `admin` role
   - `/admin/service-providers`, `/admin/services`

### Token Validation Process

```python
# app/utils/keycloak.py এ implemented

async def get_current_user(credentials: HTTPAuthorizationCredentials):
    token = credentials.credentials
    keycloak_openid = get_keycloak_openid()
    
    # Public key দিয়ে token verify করা
    pub_key = "-----BEGIN PUBLIC KEY-----\n" + 
              keycloak_openid.public_key() + 
              "\n-----END PUBLIC KEY-----\n"
    
    decoded_token = keycloak_openid.decode_token(
        token, 
        key=pub_key, 
        options={"verify_aud": False}
    )
    
    return decoded_token
```

---

## 💰 Payment Integration

### DGPay QR Payment System

**কিভাবে কাজ করে:**

1. **QR Code Scanning**
   - User scans merchant's QR code
   - App extracts payment information from QR

2. **QR Code Format (EMV QR)**
   ```
   00020101021226360010BDT0115DGPayMerchant5204000053030505802BD5910ABCStore6009Dhaka6304A13B
   ```

3. **Payment Initiation**
   ```json
   POST /api/v1/payments/initiate
   {
     "qr_code": "00020101...",
     "amount": 100.00
   }
   ```

4. **Backend Processing**
   - Parse QR code → Extract merchant info
   - Validate amount and merchant
   - Call DGPay API → Create transaction
   - Save transaction in DB
   - Return transaction ID

5. **DGPay Processing**
   - User completes payment in DGPay
   - DGPay sends callback to our webhook

6. **Callback Handling**
   ```json
   POST /api/v1/payments/callback
   {
     "transaction_id": "TXN123",
     "status": "SUCCESS",
     "amount": 100.00,
     ...
   }
   ```

7. **Update Status**
   - Verify callback signature
   - Update transaction status in DB
   - Notify user

### Payment States

```
PENDING → PROCESSING → SUCCESS
                    ↓
                  FAILED
                    ↓
                CANCELLED
```

---

## 🚀 কিভাবে নতুন কাজ শুরু করবেন

### যখন নতুন task assign হবে:

#### 1. **Task Understand করুন**
- Requirements পড়ুন
- কোন module এ কাজ করতে হবে identify করুন
- কোন endpoints/features add করতে হবে বুঝুন
- Database schema changes লাগবে কিনা check করুন

#### 2. **Codebase Explore করুন**
```bash
# Similar features খুঁজুন
grep -r "similar_function_name" app/

# Module structure দেখুন
ls -la app/modules/target_module/

# Existing models দেখুন
cat app/modules/target_module/models.py
cat app/modules/target_module/orm_models.py
```

#### 3. **Development Environment Setup**
```bash
# Docker containers start করুন
docker-compose -f docker-compose.dev.yml up -d

# Database migrations run করুন
docker-compose -f docker-compose.dev.yml exec app alembic upgrade head

# Logs check করুন
docker-compose -f docker-compose.dev.yml logs -f app
```

#### 4. **Coding Pattern Follow করুন**

**নতুন API endpoint add করার steps:**

1. **Model Define করুন** (`models.py`)
   ```python
   # Request model
   class NewFeatureRequest(BaseModel):
       field1: str
       field2: int
   
   # Response model
   class NewFeatureResponse(BaseModel):
       id: UUID
       field1: str
       created_at: datetime
   ```

2. **Database Model Add করুন** (`orm_models.py` - if needed)
   ```python
   class NewFeature(OrmBase):
       __tablename__ = "new_features"
       
       id: Mapped[uuid.UUID] = mapped_column(UUID(as_uuid=True), primary_key=True)
       field1: Mapped[str] = mapped_column(String(255))
       created_at: Mapped[datetime] = mapped_column(TIMESTAMP)
   ```

3. **Migration Create করুন**
   ```bash
   docker-compose -f docker-compose.dev.yml exec app \
     alembic revision -m "add new feature table" --autogenerate
   
   # Migration run করুন
   docker-compose -f docker-compose.dev.yml exec app \
     alembic upgrade head
   ```

4. **Service Logic Implement করুন** (`service.py`)
   ```python
   class NewFeatureService:
       def __init__(self, db: Session):
           self.db = db
       
       async def create_feature(self, data: NewFeatureRequest) -> NewFeature:
           # Business logic here
           feature = NewFeature(**data.dict())
           self.db.add(feature)
           self.db.commit()
           return feature
   ```

5. **Controller/Route Add করুন** (`controller.py`)
   ```python
   @router.post("/new-feature", response_model=BaseResponse[NewFeatureResponse])
   async def create_new_feature(
       request: NewFeatureRequest,
       credentials: HTTPAuthorizationCredentials = Security(security),
       service: NewFeatureService = Depends(get_service)
   ):
       user = await get_current_user(credentials)
       result = await service.create_feature(request)
       return BaseResponse.success_response(
           data=NewFeatureResponse.from_orm(result),
           message="Feature created successfully"
       )
   ```

6. **Routes Register করুন** (`app/routes.py`)
   ```python
   from app.modules.new_module.controller import router as new_router
   
   app.include_router(new_router, prefix=settings.API_V1_STR)
   ```

#### 5. **Testing করুন**
```bash
# Unit tests লিখুন
# tests/unit/test_new_feature.py

# Integration tests লিখুন  
# tests/integration/test_new_feature_api.py

# Tests run করুন
docker-compose -f docker-compose.dev.yml exec app \
  poetry run pytest tests/

# Specific test run করুন
docker-compose -f docker-compose.dev.yml exec app \
  poetry run pytest tests/unit/test_new_feature.py -v
```

#### 6. **Code Quality Check করুন**
```bash
# Format code
docker-compose -f docker-compose.dev.yml exec app \
  poetry run black app/

# Check linting
docker-compose -f docker-compose.dev.yml exec app \
  poetry run flake8 app/

# Type checking
docker-compose -f docker-compose.dev.yml exec app \
  poetry run mypy app/
```

---

## 🔄 Development Workflow

### Daily Development Process

```bash
# 1. Pull latest changes
git pull origin dev

# 2. Start development environment
docker-compose -f docker-compose.dev.yml up -d

# 3. Check if everything is running
docker-compose -f docker-compose.dev.yml ps

# 4. View logs
docker-compose -f docker-compose.dev.yml logs -f app

# 5. Access services
# API: http://localhost:8000
# Docs: http://localhost:8000/docs
# Keycloak: http://localhost:8080
# pgAdmin: http://localhost:8888
# Redis: localhost:6379

# 6. Make changes in code (hot reload enabled)

# 7. Run tests
docker-compose -f docker-compose.dev.yml exec app poetry run pytest

# 8. Commit changes
git add .
git commit -m "feat: add new feature"
git push origin feature-branch

# 9. Stop environment
docker-compose -f docker-compose.dev.yml down
```

### Database Operations

```bash
# Create new migration
docker-compose -f docker-compose.dev.yml exec app \
  alembic revision -m "description" --autogenerate

# Apply migrations
docker-compose -f docker-compose.dev.yml exec app \
  alembic upgrade head

# Rollback migration
docker-compose -f docker-compose.dev.yml exec app \
  alembic downgrade -1

# Check migration history
docker-compose -f docker-compose.dev.yml exec app \
  alembic history

# Access database directly
docker-compose -f docker-compose.dev.yml exec postgres \
  psql -U postgres -d oneid
```

### Celery Tasks

```bash
# Start Celery worker
docker-compose -f docker-compose.dev.yml up -d celery-worker

# View Celery logs
docker-compose -f docker-compose.dev.yml logs -f celery-worker

# Restart Celery
docker-compose -f docker-compose.dev.yml restart celery-worker
```

### Redis Operations

```bash
# Access Redis CLI
docker-compose -f docker-compose.dev.yml exec redis redis-cli

# Inside Redis CLI:
# List all keys
KEYS *

# Get value
GET key_name

# Delete key
DEL key_name

# Clear all
FLUSHALL
```

---

## 🐛 Troubleshooting Guide

### Common Issues এবং Solutions

#### 1. **Container Start হচ্ছে না**
```bash
# Logs check করুন
docker-compose -f docker-compose.dev.yml logs app

# Rebuild করুন
docker-compose -f docker-compose.dev.yml build --no-cache app

# Restart করুন
docker-compose -f docker-compose.dev.yml down -v
docker-compose -f docker-compose.dev.yml up -d
```

#### 2. **Database Connection Error**
```bash
# Database running check করুন
docker-compose -f docker-compose.dev.yml ps postgres

# Database logs check করুন
docker-compose -f docker-compose.dev.yml logs postgres

# .env file check করুন
cat .env | grep POSTGRES

# Database recreate করুন
docker-compose -f docker-compose.dev.yml down -v
docker-compose -f docker-compose.dev.yml up -d postgres
```

#### 3. **Migration Issues**
```bash
# Current revision check করুন
docker-compose -f docker-compose.dev.yml exec app \
  alembic current

# Downgrade এবং upgrade করুন
docker-compose -f docker-compose.dev.yml exec app \
  alembic downgrade base
docker-compose -f docker-compose.dev.yml exec app \
  alembic upgrade head

# Manual migration fix (if needed)
docker-compose -f docker-compose.dev.yml exec postgres \
  psql -U postgres -d oneid -c "DROP TABLE alembic_version;"
```

#### 4. **Keycloak Connection Error**
```bash
# Keycloak running check করুন
docker-compose -f docker-compose.dev.yml ps keycloak

# Keycloak URL check করুন
echo $KEYCLOAK_SERVER_URL

# Keycloak admin console access করুন
# http://localhost:8080
# Username: admin
# Password: .env এ দেখুন
```

#### 5. **Redis Connection Error**
```bash
# Redis running check করুন
docker-compose -f docker-compose.dev.yml ps redis

# Redis connection test করুন
docker-compose -f docker-compose.dev.yml exec redis redis-cli ping

# Redis restart করুন
docker-compose -f docker-compose.dev.yml restart redis
```

#### 6. **Module Import Errors**
```bash
# Dependencies reinstall করুন
docker-compose -f docker-compose.dev.yml exec app \
  poetry install

# Container rebuild করুন
docker-compose -f docker-compose.dev.yml build app
```

#### 7. **Port Already in Use**
```bash
# Running processes check করুন
sudo netstat -tulpn | grep :8000
sudo netstat -tulpn | grep :5432

# Kill process
sudo kill -9 PID

# অথবা docker-compose.dev.yml এ port change করুন
```

#### 8. **Permission Denied Errors**
```bash
# File permissions fix করুন
sudo chown -R $USER:$USER .

# Docker socket permission
sudo chmod 666 /var/run/docker.sock
```

---

## 📚 Additional Resources

### Important Documentation Files
1. `README.md` - Main documentation
2. `KEYCLOAK_SETUP.md` - Keycloak configuration
3. `PAYMENT_API_EXAMPLES.md` - Payment integration guide
4. `CELERY_SETUP.md` - Celery task queue setup
5. `NID_IMPLEMENTATION_COMPLETE.md` - NID verification
6. `DOCUMENT_CATEGORIES_IMPLEMENTATION.md` - Document types
7. `SERVICE_PROVIDER_CATEGORIES_IMPLEMENTATION.md` - Provider types

### Useful Commands Cheat Sheet

```bash
# Start everything
docker-compose -f docker-compose.dev.yml up -d

# Stop everything
docker-compose -f docker-compose.dev.yml down

# Stop and remove volumes (clean slate)
docker-compose -f docker-compose.dev.yml down -v

# View logs
docker-compose -f docker-compose.dev.yml logs -f [service_name]

# Execute command in container
docker-compose -f docker-compose.dev.yml exec app [command]

# Rebuild specific service
docker-compose -f docker-compose.dev.yml build --no-cache [service_name]

# Scale service
docker-compose -f docker-compose.dev.yml up -d --scale celery-worker=3

# Check service status
docker-compose -f docker-compose.dev.yml ps

# View resource usage
docker stats
```

### API Testing Tools

1. **Swagger UI** - http://localhost:8000/docs
2. **Postman** - Import `test_payment_postman.json`
3. **curl** - Terminal based testing
   ```bash
   # Login
   curl -X POST http://localhost:8000/api/v1/auth/login \
     -H "Content-Type: application/json" \
     -d '{"username": "test", "password": "test123"}'
   
   # Get profile
   curl -X GET http://localhost:8000/api/v1/users/me \
     -H "Authorization: Bearer YOUR_TOKEN"
   ```

### Environment Variables Reference

Key environment variables in `.env`:

```bash
# Database
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_password
POSTGRES_DB=oneid

# Keycloak
KEYCLOAK_SERVER_URL=http://keycloak:8080
KEYCLOAK_REALM=oneid
KEYCLOAK_CLIENT_ID=oneid
KEYCLOAK_CLIENT_SECRET=your_secret
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=admin_password

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# AWS
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
BUCKET_NAME=your_bucket
KMS_KEY_ID=your_kms_key

# SMS
SSL_SMS_API_BASE_URL=https://sms.sslwireless.com
SSL_SMS_API_TOKEN=your_token
SSL_SMS_API_SID=your_sid

# Payment
DGPAY_API_BASE_URL=https://api.dgpay.com
DGPAY_MERCHANT_ID=your_merchant_id
DGPAY_API_KEY=your_api_key

# VDS
VDS_TSP_API_URL=https://tsp.example.com
VDS_TSP_API_KEY=your_tsp_key

# Application
DEBUG=true
ENVIRONMENT=development
API_V1_STR=/api/v1
```

---

## 🎓 Learning Path for New Developers

### Week 1: Understanding the Basics
1. FastAPI fundamentals বুঝুন
2. Pydantic models এবং validation
3. JWT authentication flow
4. Database operations (SQLAlchemy)

### Week 2: Module Deep Dive
1. Auth module পড়ুন এবং বুঝুন
2. User module explore করুন
3. Keycloak integration বুঝুন
4. একটা simple endpoint create করুন

### Week 3: Advanced Features
1. Document module architecture বুঝুন
2. Payment flow implementation দেখুন
3. Celery tasks এবং async processing
4. S3 integration বুঝুন

### Week 4: Production Ready
1. Testing লিখুন
2. Error handling implement করুন
3. Logging এবং monitoring
4. Performance optimization

---

## 📞 Support & Contact

### যদি কোনো সমস্যা হয় বা প্রশ্ন থাকে:

1. **Code Review করুন** - Similar implementations খুঁজুন
2. **Documentation পড়ুন** - উপরের সব MD files
3. **Logs Check করুন** - Error messages পড়ুন
4. **Team Lead এর সাথে যোগাযোগ করুন** - Detailed explanation দিয়ে

### Common Questions to Ask:
- কোন module এ কাজ করতে হবে?
- Database schema change লাগবে কিনা?
- কোন external API integration লাগবে?
- Authentication/Authorization requirements কি?
- Testing requirements কি?
- Deadline কি?

---

## 🚀 Quick Reference - যখন কাজ শুরু করবেন

### Step-by-Step Checklist:

```bash
✅ 1. Task requirements পড়ুন
✅ 2. Related module identify করুন
✅ 3. Similar features খুঁজুন (grep/search)
✅ 4. Database changes plan করুন
✅ 5. Development environment start করুন
✅ 6. Models define করুন
✅ 7. Database migration create করুন (if needed)
✅ 8. Service logic implement করুন
✅ 9. Controller/API endpoint add করুন
✅ 10. Testing করুন (manual + automated)
✅ 11. Code quality check করুন
✅ 12. Documentation update করুন
✅ 13. Git commit & push করুন
✅ 14. Pull request create করুন
```

---

## 🎯 মনে রাখবেন

1. **Code consistency maintain করুন** - Existing patterns follow করুন
2. **Error handling properly করুন** - Custom exceptions use করুন
3. **Logging add করুন** - Debug করা easy হবে
4. **Tests লিখুন** - Future bugs prevent করবে
5. **Documentation update করুন** - Others বুঝতে পারবে
6. **Security check করুন** - Authentication/Authorization properly implement করুন
7. **Git best practices follow করুন** - Meaningful commit messages
8. **Code review করান** - Before merging

---

**এই document টা আপনার জন্য complete reference guide হিসেবে কাজ করবে। যেকোনো নতুন task এ এখান থেকে relevant information পাবেন এবং কিভাবে implement করতে হবে বুঝতে পারবেন। Happy Coding! 🚀**
