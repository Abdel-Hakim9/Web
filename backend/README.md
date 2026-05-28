# INSAT Events — Backend API

## Stack
- **PHP 8.1+** (no framework, no Composer required)
- **MySQL / MariaDB** (via XAMPP)
- **Apache** with `mod_rewrite`

---

## Requirements

- [XAMPP](https://www.apachefriends.org/download.html) installed
- Apache and MySQL both **running** in the XAMPP Control Panel

---

## First-Time Setup (Fresh Clone)

### 1. Place the project in the right folder

Copy the entire project folder into XAMPP's web root:
```
C:\xampp\htdocs\Web
```
So the backend is at `C:\xampp\htdocs\Web\backend`.

---

### 2. Create the database

**Option A — via phpMyAdmin (easiest)**
1. Open your browser and go to `http://localhost/phpmyadmin`
2. Click **New** in the left sidebar
3. Name it `event_management` → click **Create**
4. Click the `event_management` database → go to the **Import** tab
5. Click **Choose File** → select `backend/config/schema.sql`
6. Click **Go**

**Option B — via command line**
```bash
mysql -u root -p < C:\xampp\htdocs\Web\backend\config\schema.sql
```
Leave the password blank if you haven't set one.

---

### 3. Check database credentials

Open `backend/config/database.php` and verify these match your XAMPP setup:

```php
'host'     => 'localhost',
'dbname'   => 'event_management',
'username' => 'root',
'password' => '',        // ← add your MySQL password here if you set one
```

By default XAMPP uses `root` with no password — if that's your setup, no changes needed.

---

### 4. Enable Apache mod_rewrite

1. Open XAMPP Control Panel → click **Config** next to Apache → open `httpd.conf`
2. Find this line and make sure it is **not** commented out (no `#` at the start):
```
LoadModule rewrite_module modules/mod_rewrite.so
```
3. Save and **restart Apache** in XAMPP.

---

### 5. Open the app

```
http://localhost/Web/frontend/pages/index.html
```

---

## Common Issues

**"Access denied for user root"**
→ Your MySQL root has a password. Add it to `backend/config/database.php` in the `password` field.

**"Database event_management does not exist"**
→ You skipped step 2. Create the database and import `schema.sql`.

**API returns 404 / routes not working**
→ mod_rewrite is not enabled. Follow step 4 above.

**Events list is empty**
→ The database was created successfully but has no data yet. You need to insert events manually via phpMyAdmin into the `events` table, or register an account and add them through the app.

---

## Folder Structure

```
backend/
├── config/
│   ├── database.php        ← PDO singleton connection (edit credentials here)
│   └── schema.sql          ← Import this once to create all tables
├── controllers/
│   ├── AuthController.php
│   ├── EventController.php
│   └── RegistrationController.php
├── middleware/
│   └── AuthMiddleware.php  ← JWT guard for protected routes
├── models/
│   ├── User.php
│   ├── event.php
│   └── Registration.php
├── routes/
│   └── api.php             ← URL → controller dispatcher
├── utils/
│   ├── JWT.php             ← HS256 JWT (generate / verify)
│   ├── Response.php        ← JSON response helper
│   └── Validator.php       ← Validation engine
├── .htaccess               ← Apache URL rewrite rules
└── index.php               ← Entry point + CORS
```

---

## API Endpoints

### POST `/auth/register`
Create a new account.

**Request body (JSON)**
```json
{
  "first_name": "Amine",
  "last_name": "Hamed",
  "email": "amine.hamed@insat.ucar.tn",
  "birthdate": "2005-12-16",
  "academic_year": "1st Year",
  "password": "SecurePass123",
  "password_confirmation": "SecurePass123"
}
```

---

### POST `/auth/login`
Sign in with email + password.

**Request body (JSON)**
```json
{
  "email": "amine.hamed@insat.ucar.tn",
  "password": "SecurePass123"
}
```

---

### GET `/auth/me` *(protected)*
Returns the authenticated user's profile.
**Header:** `Authorization: Bearer <JWT>`

---

### GET `/events`
Returns all events. Optional query params: `?category=&search=`

### GET `/events/{id}`
Returns a single event by ID.

### POST `/events/{id}/register` *(protected)*
Register the authenticated user for an event.

### DELETE `/events/{id}/cancel` *(protected)*
Cancel the authenticated user's registration.

### GET `/users/me/events` *(protected)*
Returns all events the authenticated user is registered for.

---

## Error Responses

| HTTP Code | Meaning |
|---|---|
| 400 | Bad request / invalid JSON |
| 401 | Unauthorized — missing or expired token |
| 409 | Conflict — email already exists / already registered |
| 422 | Validation error |
| 500 | Server error |
