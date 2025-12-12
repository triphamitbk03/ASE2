# 🎬 AUTHENTICATION SYSTEM - 2 MINUTE PRESENTATION

## 📋 Overview

Analysis of Authentication system (Login/Register/Logout) across 4 quality attributes:

- **Availability** - System readiness and accessibility
- **Safety** - Safe data handling and validation
- **Security** - Protection against threats
- **Reliability** - System dependability and consistency

**Structure:** 6 parts (Introduction + 4 attributes + Demo)

---

## 🎤 DETAILED SCRIPT

### PART 1: INTRODUCTION (15 seconds)

> **"Hello everyone! Today I will present the Authentication system of the ASE-251 project.**
>
> **The system includes three main features:**
>
> - **Register** - Create new user accounts
> - **Login** - Authenticate with email/password
> - **Logout** - End user session
>
> **I will analyze four quality attributes: Availability, Safety, Security, and Reliability."**

**🎬 Action:** Show slide or code structure overview

---

### PART 2: AVAILABILITY 

> **"First, Availability - the system's ability to be ready and accessible.**
>
>
> **✅ FEATURES IMPLEMENTED:**
>
> - **Deployed on Render** - Production environment accessible 24/7
> - **Async MongoDB operations** with Motor driver for concurrent request handling
> - **Auto-reconnect mechanism** - Automatically reconnects to database when connection is lost
> - **CORS enabled** - Easy frontend integration
>
> **Here's the code in db_client.py"**

**🎬 Action:** Show code `BE/app/database/db_client.py` lines 27-36

```python
async def get_database():
    """Get database instance. Auto-connect if not connected."""
    global client
    if client is None:
        await connect_to_mongo()  # ← Auto-reconnect!
    return client[DATABASE_NAME]

async def get_users_collection():
    """Get users collection with async client."""
    db = await get_database()
    return db["users"]  # ← Async operations
```

**🎬 Optional:** Show deployment proof

- Render dashboard or
- Access live API: `https://your-app.onrender.com/docs`
- Show API response time

---

### PART 3: SAFETY 

> **"Next is Safety - ensuring safe data handling and validation.**
>
>
> **✅ FEATURES IMPLEMENTED:**
>
> - **Input validation** - Email format checking with regex
> - **Role validation** - Only accepts 'lecturer' and 'student' roles
> - **Required fields check** - Mandatory fullname, email, and password
> - **Unique email constraint** - Prevents duplicate accounts
> - **Blacklist mechanism** - Blocks banned users with is_blacklisted flag
>
> **Here's the validation code in auth.py**
>
> **Now let me run a test to demonstrate the safety features."**

**🎬 Action:** Show code `BE/app/routers/auth.py` lines 66-112

```python
# Email format validation
if not _EMAIL_REGEX.match(request.email):
    return JSONResponse(status_code=400, ...)

# Role validation - chỉ 2 role
if role not in ("lecturer", "student"):
    return JSONResponse(status_code=400, ...)

# Unique email check
existing_email = await users_collection.find_one({"email": request.email})
if existing_email:
    return JSONResponse(status_code=409, ...)  # Conflict

# Blacklist check
if user.get("is_blacklisted"):
    return JSONResponse(status_code=403, ...)  # Forbidden
```

**🎬 Action:** Run test in terminal

```bash
pytest tests/test_auth.py::test_register_rejects_duplicate_email -v
```

> **"As you can see, the test passes - our system successfully prevents duplicate email registration."**

---

### PART 4: SECURITY 

> **"Now, Security - the most critical attribute.**
>
>
> **✅ FEATURES IMPLEMENTED:**
>
> - **Password hashing with bcrypt** - No plaintext passwords stored in database
> - **Secure password verification** - Uses bcrypt.verify to prevent timing attacks
> - **Proper HTTP status codes** - 401 Unauthorized, 403 Forbidden, 409 Conflict
>
> **Let me show you the password hashing code.**
>
> **This is GOOD - we hash passwords with bcrypt. Let me run a test to verify."**

**🎬 Action:** Show code `BE/app/routers/auth.py` lines 22-42

```python
# ✅ GOOD: Password hashing with bcrypt
_pwd_context = CryptContext(
    schemes=["bcrypt"],
    deprecated="auto",
    bcrypt__truncate_error=False,
)

def _hash_password(password: str) -> str:
    if len(password.encode("utf-8")) > _MAX_PASSWORD_BYTES:
        raise HTTPException(status_code=400, ...)
    return _pwd_context.hash(password)  # ← Secure hashing

def _verify_password(plain_password: str, hashed_password: str) -> bool:
    return _pwd_context.verify(plain_password, hashed_password)  # ← Safe verify
```

**🎬 Action:** Run test in terminal

```bash
pytest tests/test_auth_security.py::test_password_is_hashed_not_plaintext -v
```

> **"Perfect! The test confirms passwords are properly hashed with bcrypt, not stored as plaintext."**

---

### PART 5: RELIABILITY 

> **"Finally, Reliability - system dependability and consistency.**
>
>
> **✅ FEATURES IMPLEMENTED:**
>
> - **Type hints with Pydantic** - Automatic request/response validation
> - **Consistent error format** - All errors have standardized code and message structure
> - **UUID-based user_id** - Guarantees uniqueness, NO RACE CONDITION
> - **Datetime tracking** - Records created_at timestamp for each user
>
> **Here's how we FIXED the race condition using UUID."**

**🎬 Action:** Show code `BE/app/routers/auth.py` lines 48-51

```python
async def _generate_user_id(users_collection) -> str:
    """Generate unique user_id using UUID to prevent race condition."""
    # ✅ FIXED: Use UUID4 for guaranteed uniqueness (no race condition)
    return f"U{uuid.uuid4().hex[:12].upper()}"
```

**Explanation:**

- ✅ **BEFORE:** Used count + 1 → NOT ATOMIC → race condition possible
- ✅ **NOW:** Uses UUID4 → ALWAYS UNIQUE → no race condition
- Format: "U" + 12 hex characters from UUID (example: "U8F3A7B2E9D4C")

---

### PHẦN 6: DEMO 

> **"Now I will demonstrate the test suite to verify all features.**
>
> **We have 42 test cases covering 4 quality attributes."**

**🎬 Action:** Run terminal commands

```bash
# Terminal: Run all tests
cd /Users/phamnguyenviettri/Ses251/ASE-251/BE
pytest tests/test_auth*.py -v --tb=short

# Expected result:
# ✓ 42 passed in 2.5s
```

> **"Summary:**
>
> - ✅ **Availability:** - Deployed on Render with 24/7 accessibility, async operations for concurrent users
> - ✅ **Safety:** - Email validation, role validation, duplicate prevention, blacklist protection
> - ✅ **Security:** - Bcrypt password hashing with salt
> - ✅ **Reliability** - UUID-based unique IDs (race condition fixed), Pydantic validation, async error handling
>
> **Overall: System successfully implements core authentication with significant reliability improvements.**
>
> **Thank you for watching!"**

---

## 📂 FILES TO OPEN BEFORE RECORDING

### 1. Authentication Router

**File:** `BE/app/routers/auth.py`

**Key sections:**

- Lines 21-28: Password hashing setup (bcrypt)
- Lines 30-42: `_hash_password()` and `_verify_password()`
- Lines 48-54: `_generate_user_id()` - UUID implementation
- Lines 66-79: Email and required fields validation
- Lines 89-99: Role validation
- Lines 101-112: Unique email check
- Lines 173-180: Blacklist check
- Lines 213-216: Logout endpoint

### 2. Database Client

**File:** `BE/app/database/db_client.py`

**Key sections:**

- Lines 13-22: `connect_to_mongo()` - connection setup
- Lines 27-36: `get_database()` - auto-reconnect capability
- Lines 38-42: `get_users_collection()` - async collection

### 3. Schemas

**File:** `BE/app/schemas/auth.py`

**Key sections:**

- Lines 5-11: `RegisterRequest` - Pydantic validation
- Lines 14-16: `LoginRequest`
- Lines 19-26: `LoginResponse`

### 4. Settings

**File:** `BE/app/config/settings.py`

**Key sections:**

- Lines 1-3: MongoDB URI configuration

### 5. Test Files

**Files:**

- `BE/tests/test_auth.py` (7 tests)
- `BE/tests/test_auth_security.py` (15 tests)
- `BE/tests/test_auth_safety.py` (20 tests)

---

## 🎬 DETAILED TIMELINE (120 seconds)

| Time      | Section      | Content                              | Action                  |
| --------- | ------------ | ------------------------------------ | ----------------------- |
| 0:00-0:15 | Intro        | Introduce 3 functions + 4 attributes | Show overview           |
| 0:15-0:35 | Availability | Async ops, auto-reconnect, Render    | Show `db_client.py`     |
| 0:35-1:00 | Safety       | Validation rules, blacklist          | Show validation code    |
| 1:00-1:30 | Security     | Password hashing with bcrypt         | Show hashing code       |
| 1:30-1:55 | Reliability  | Pydantic validation, UUID fix        | Show user_id generation |
| 1:55-2:00 | Demo         | Run tests, summary                   | Run pytest command      |

---

## 🧪 COMMANDS TO RUN

### Before recording:

```bash
# 1. Ensure correct branch
git checkout ASE-RateLimit

# 2. Navigate to BE directory
cd /Users/phamnguyenviettri/Ses251/ASE-251/BE

# 3. Verify server runs
python3 -m app.main
# Ctrl+C to stop

# 4. Test pytest works
pytest tests/test_auth.py -v
```

### During video:

**Command 1: Chạy tất cả auth tests**

```bash
pytest tests/test_auth*.py -v
```

**Command 2: Chạy với coverage (optional)**

```bash
pytest tests/test_auth*.py --cov=app.routers.auth --cov-report=term-missing
```

**Command 3: Chạy specific test file**

```bash
# Security tests
pytest tests/test_auth_security.py -v

# Safety tests
pytest tests/test_auth_safety.py -v
```

---
# SWAGGER UI TEST CASES

## 🎯 Overview
Each quality attribute has **1 test case** to demonstrate on Swagger UI.

---

## 📍 Access Swagger UI

**Local:** http://localhost:8000/docs  
**Render:** https://ase-251.onrender.com/docs

---

## 1️⃣ AVAILABILITY Test Case

**Test:** Register endpoint is accessible and responds quickly

**Endpoint:** `POST /auth/register`

**Request Body:**
```json
{
  "full_name": "John Doe",
  "email": "john.doe@example.com",
  "password": "SecurePass123",
  "role": "customer"
}
```

**Expected Response:** `201 Created`
```json
{
  "data": {
    "user_id": "U1A2B3C4D5E6",
    "full_name": "John Doe",
    "email": "john.doe@example.com",
    "role": "customer"
  },
  "meta": {
    "message": "User registered successfully"
  }
}
```

**What to verify:**
- ✅ Endpoint responds within 2 seconds
- ✅ Returns 201 status code
- ✅ System is accessible on Render (24/7 availability)

---

## 2️⃣ SAFETY Test Case

**Test:** Duplicate email registration is rejected

**Endpoint:** `POST /auth/register`

**Step 1:** Register first user
```json
{
  "full_name": "Jane Smith",
  "email": "jane.smith@test.com",
  "password": "Password123",
  "role": "customer"
}
```

**Step 2:** Try to register with same email
```json
{
  "full_name": "Jane Duplicate",
  "email": "jane.smith@test.com",
  "password": "DifferentPass456",
  "role": "customer"
}
```

**Expected Response:** `409 Conflict`
```json
{
  "detail": "Email already registered"
}
```

**What to verify:**
- ✅ First registration succeeds (201)
- ✅ Second registration fails (409)
- ✅ System prevents duplicate emails (data integrity)

**Pytest command to verify:**
```bash
pytest tests/test_auth.py::test_register_rejects_duplicate_email -v
```

---

## 3️⃣ SECURITY Test Case

**Test:** Password is hashed (not stored in plaintext)

**Endpoint:** `POST /auth/register` → `POST /auth/login`

**Step 1:** Register user
```json
{
  "full_name": "Alice Secure",
  "email": "alice.secure@test.com",
  "password": "MySecretPassword",
  "role": "customer"
}
```

**Step 2:** Login with correct password
```json
{
  "email": "alice.secure@test.com",
  "password": "MySecretPassword"
}
```

**Expected Response:** `200 OK`
```json
{
  "access_token": "user_alice.secure@test.com",
  "token_type": "bearer",
  "user_id": "U1234567890AB",
  "email": "alice.secure@test.com",
  "full_name": "Alice Secure",
  "role": "customer"
}
```

**Step 3:** Verify in MongoDB
- Check database: password is hashed (starts with `$2b$`)
- Password "MySecretPassword" is NOT visible in plaintext

**What to verify:**
- ✅ Login succeeds with correct password
- ✅ Password is stored as bcrypt hash (not plaintext)
- ✅ Different users with same password have different hashes

**Pytest command to verify:**
```bash
pytest tests/test_auth_security.py::test_password_is_hashed_not_plaintext -v
```

---

## 4️⃣ RELIABILITY Test Case

**Test:** Multiple registrations generate unique user IDs (no race condition)

**Endpoint:** `POST /auth/register`

**Step 1:** Register user 1
```json
{
  "full_name": "User One",
  "email": "user1@test.com",
  "password": "Pass123",
  "role": "customer"
}
```

**Response:** User ID = `U1A2B3C4D5E6`

**Step 2:** Register user 2
```json
{
  "full_name": "User Two",
  "email": "user2@test.com",
  "password": "Pass456",
  "role": "customer"
}
```

**Response:** User ID = `U7F8G9H0I1J2` (different from User One)

**Step 3:** Register user 3
```json
{
  "full_name": "User Three",
  "email": "user3@test.com",
  "password": "Pass789",
  "role": "customer"
}
```

**Response:** User ID = `U3K4L5M6N7O8` (different from both previous)

**What to verify:**
- ✅ Each user gets unique user_id (UUID-based)
- ✅ No duplicate IDs even with concurrent requests
- ✅ System handles multiple registrations reliably

**Code to show:**
```python
# BE/app/routers/auth.py line 48-51
async def _generate_user_id(users_collection) -> str:
    """Generate unique user_id using UUID to prevent race condition."""
    return f"U{uuid.uuid4().hex[:12].upper()}"
```

---

## 🎬 Video Demonstration Flow

### Timeline (30 seconds per attribute):

**0:00-0:30 - AVAILABILITY**
1. Open Swagger UI at https://ase-251.onrender.com/docs
2. Expand `POST /auth/register`
3. Click "Try it out"
4. Paste test case 1 JSON
5. Click "Execute"
6. Show 201 response within 2 seconds
7. **Say:** "System is deployed on Render with 24/7 availability and async operations"

**0:30-1:00 - SAFETY**
1. Use same Swagger UI
2. Register first user with jane.smith@test.com
3. Show 201 success
4. Try to register again with same email
5. Show 409 Conflict error
6. **Say:** "Duplicate email prevention ensures data integrity"
7. Optional: Run pytest command in terminal

**1:00-1:30 - SECURITY**
1. Register alice.secure@test.com
2. Show 201 success
3. Login with same credentials
4. Show 200 success with token
5. Optional: Open MongoDB Compass to show hashed password ($2b$...)
6. **Say:** "Bcrypt hashing with salt keeps passwords secure"
7. Optional: Run pytest command in terminal

**1:30-2:00 - RELIABILITY**
1. Register 3 users quickly
2. Show each response with different user_id
3. Open `auth.py` and highlight UUID generation code (line 48-51)
4. **Say:** "UUID-based IDs prevent race conditions and ensure uniqueness"
5. Show summary scores

---

## 📊 Summary Scores

| Quality Attribute | Score | Test Case Demonstrated |
|------------------|-------|------------------------|
| **Availability** | 6/10 | Register endpoint accessible on Render |
| **Safety** | 6.5/10 | Duplicate email prevention |
| **Security** | 4/10 | Password hashing with bcrypt |
| **Reliability** | 7/10 | UUID-based unique IDs (no race condition) |

**Overall: 6/10** - Core authentication implemented successfully

---

## 🚀 Quick Start Commands

### Start local server:
```bash
cd /Users/phamnguyenviettri/Ses251/ASE-251/BE
python3 -m app.main
```

### Access Swagger UI:
```
http://localhost:8000/docs
```

### Run all tests:
```bash
pytest tests/test_auth*.py -v
```

### Run specific test:
```bash
# Safety test
pytest tests/test_auth.py::test_register_rejects_duplicate_email -v

# Security test
pytest tests/test_auth_security.py::test_password_is_hashed_not_plaintext -v
```

---

## 💡 Tips for Video Recording

1. **Use Render deployment** - Shows real availability (not just localhost)
2. **Keep Swagger UI zoomed** - 125-150% for visibility
3. **Show response times** - Highlights async performance
4. **Demonstrate failures** - 409 error shows safety works
5. **Point to code** - Line 48-51 in auth.py for UUID
6. **Terminal side-by-side** - Run pytest while showing Swagger

---

**Good luck with your demonstration! 🎬**

