# Student Management System

A full-stack CRUD web application built to the CRUD Web Application Development
SOP: **React** frontend, **Django REST Framework** backend, **SQLite** database.

## 1. Project Overview

Registrar-style app for managing student records: add, view, search, edit, and
delete students. Built as a reference implementation of every stage in the SOP
(requirement analysis → design → frontend/backend → REST API → validation →
testing → docs).

## 2. Problem Statement

Academic departments need a simple, reliable way to keep student records
(contact details, department, year) accurate and searchable without manual
spreadsheets, which are error-prone and hard to share.

## 3. Objectives

- Provide Create, Read, Update, Delete operations on student records
- Enforce validation on both client and server
- Expose a documented REST API
- Support search/filter by name, email, roll number, department, and year

## 4. Technology Stack

| Layer      | Technology                          |
|------------|--------------------------------------|
| Frontend   | React 18, plain CSS                  |
| Backend    | Django 5, Django REST Framework      |
| Database   | SQLite (dev) — swappable to PostgreSQL/MySQL |
| API Testing| Postman (collection notes below)     |
| Versioning | Git                                   |

## 5. System Architecture

```
User → React (frontend, port 3000)
         │  fetch() JSON over HTTP
         ▼
     Django REST Framework API (port 8000)
         │  Django ORM
         ▼
     SQLite database (db.sqlite3)
```

## 6. Database Design

**Table: `students_student`**

| Field        | Type          | Constraints              |
|--------------|---------------|---------------------------|
| id           | BigAutoField  | Primary key                |
| name         | CharField(100)| NOT NULL                   |
| email        | EmailField    | NOT NULL, UNIQUE           |
| roll_number  | CharField(20) | NOT NULL, UNIQUE           |
| department   | CharField(100)| NOT NULL                   |
| year         | SmallInteger  | NOT NULL, choices 1–4      |
| phone        | CharField(15) | NOT NULL, regex-validated  |
| created_at   | DateTime      | auto-set on create         |
| updated_at   | DateTime      | auto-set on update         |

Single-entity schema; no foreign keys required for this scope. Uniqueness on
`email` and `roll_number` is enforced at the database level and re-checked in
the serializer for clean error messages.

## 7. REST API Documentation

Base URL: `http://127.0.0.1:8000/api`

| Operation  | Method | Endpoint                | Body                          | Success  |
|------------|--------|--------------------------|--------------------------------|----------|
| Create     | POST   | `/students/`             | student JSON (see below)       | 201      |
| Read All   | GET    | `/students/`             | — (optional `?search=`, `?department=`, `?year=`) | 200 |
| Read One   | GET    | `/students/{id}/`        | —                               | 200      |
| Update     | PUT    | `/students/{id}/`        | full student JSON              | 200      |
| Update     | PATCH  | `/students/{id}/`        | partial student JSON           | 200      |
| Delete     | DELETE | `/students/{id}/`        | —                               | 204      |

**Sample student JSON:**
```json
{
  "name": "Asha Rao",
  "email": "asha.rao@example.com",
  "roll_number": "CS2024001",
  "department": "Computer Science",
  "year": 2,
  "phone": "+919876543210"
}
```

**Validation error example (400):**
```json
{
  "email": ["A student with this email already exists."]
}
```

**Not found (404):** returned automatically by DRF's `get_object_or_404` for
unknown `id` on GET/PUT/PATCH/DELETE.

## 8. Installation and Execution

### Backend (Django)

```bash
cd backend
python -m venv venv
# macOS/Linux:
source venv/bin/activate
# Windows:
venv\Scripts\activate

pip install -r requirements.txt

cp .env.example .env      # then edit DJANGO_SECRET_KEY

python manage.py makemigrations students
python manage.py migrate
python manage.py createsuperuser   # optional, for /admin/
python manage.py runserver
```

Backend runs at `http://127.0.0.1:8000`. Admin panel at `/admin/`.

### Frontend (React)

```bash
cd frontend
npm install
cp .env.example .env       # defaults already point at the local backend
npm start
```

Frontend runs at `http://localhost:3000`.

Make sure the backend is running first — the frontend will show a connection
error banner if it cannot reach the API.

## 9. Testing

Backend automated tests (covers valid/invalid/duplicate create, empty and
populated read, valid/invalid-id update and delete, and search — per SOP
section 10):

```bash
cd backend
python manage.py test students
```

**Manual Postman checklist:**
1. POST valid student → expect 201
2. POST missing required field → expect 400
3. POST duplicate email or roll number → expect 400
4. GET `/students/` on empty DB → expect 200, `[]`
5. GET `/students/` after inserts → expect 200, populated list
6. GET `/students/{id}/` valid and invalid id → 200 / 404
7. PUT valid id with changed data → 200, values updated
8. PUT invalid id → 404
9. DELETE valid id → 204, confirm row removed
10. DELETE invalid id → 404
11. GET `/students/?search=asha` → filtered results

## 10. Security Notes

- `SECRET_KEY` and `DEBUG` are read from environment variables (`.env`), never hard-coded
- CORS is restricted to the known frontend origin (`localhost:3000`)
- All queries go through the Django ORM (parameterized, no raw SQL)
- Server-side validation runs independently of client-side validation

## 11. Known Limitations / Future Enhancements

- No authentication/authorization layer yet (add DRF token or session auth
  before any multi-user or production deployment)
- No pagination controls in the UI (API paginates at 20/page by default)
- Could add bulk import/export (CSV) and audit logging of edits

## 12. Project Structure

```
student-management-system/
├── backend/
│   ├── config/            # Django project settings, urls, wsgi
│   ├── students/          # models, serializers, views, urls, tests, admin
│   ├── manage.py
│   ├── requirements.txt
│   └── .env.example
├── frontend/
│   ├── public/index.html
│   ├── src/
│   │   ├── components/    # StudentForm, StudentTable, ConfirmDialog
│   │   ├── api.js         # fetch wrapper for the REST API
│   │   ├── App.js / App.css
│   │   └── index.js / index.css
│   ├── package.json
│   └── .env.example
├── .gitignore
└── README.md
```
