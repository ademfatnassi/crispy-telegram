# Mergington High School Activities API

A super simple FastAPI application that allows students to view and sign up for extracurricular activities.

## Features

- View all available extracurricular activities
- Teacher-only registration and unregistration of students
- Public viewing of activities and participants
- Teacher sign-in with credentials stored as password hashes

## Getting Started

1. Install the dependencies:

   ```
   pip install fastapi uvicorn
   ```

2. Run the application:

   ```
   python app.py
   ```

3. Open your browser and go to:
   - API documentation: http://localhost:8000/docs
   - Alternative documentation: http://localhost:8000/redoc

## Configure teacher accounts

Teacher accounts are read from `teachers.json` in this directory. Generate a
PBKDF2 password hash and store only that hash in the file:

```python
import getpass
import hashlib
import secrets

password = getpass.getpass("Teacher password: ")
salt = secrets.token_hex(16)
password_hash = hashlib.pbkdf2_hmac(
    "sha256", password.encode(), salt.encode(), 310_000
).hex()
print(f"pbkdf2_sha256$310000${salt}${password_hash}")
```

Add the generated hash to the `teachers` list:

```json
{
  "teachers": [
    {
      "username": "teacher1",
      "password_hash": "pbkdf2_sha256$310000$..."
    }
  ]
}
```

Do not store or commit teacher passwords in plaintext. The application reads this
file when teachers sign in. For deployments, set `SESSION_SECRET` to a stable,
random secret and set `SESSION_COOKIE_SECURE=true` when served over HTTPS.
Without `SESSION_SECRET`, the app generates a temporary secret on startup, so
sessions are invalidated when it restarts. Sessions expire after eight hours.

## API Endpoints

| Method | Endpoint                                                          | Description                                                         |
| ------ | ----------------------------------------------------------------- | ------------------------------------------------------------------- |
| GET    | `/activities`                                                     | Get all activities with their details and current participant count |
| GET    | `/auth/me`                                                        | Check the current teacher session                                   |
| POST   | `/auth/login`                                                     | Sign in with a teacher username and password                        |
| POST   | `/auth/logout`                                                    | Sign out the current teacher                                        |
| POST   | `/activities/{activity_name}/signup?email=student@mergington.edu` | Teacher-only student registration                                   |
| DELETE | `/activities/{activity_name}/unregister?email=student@mergington.edu` | Teacher-only student unregistration |

## Data Model

The application uses a simple data model with meaningful identifiers:

1. **Activities** - Uses activity name as identifier:

   - Description
   - Schedule
   - Maximum number of participants allowed
   - List of student emails who are signed up

2. **Students** - Uses email as identifier:
   - Name
   - Grade level

All data is stored in memory, which means data will be reset when the server restarts.
