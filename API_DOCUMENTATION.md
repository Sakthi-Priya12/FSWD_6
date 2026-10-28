# API Documentation

**Base URL (development):** `http://localhost:5000/api`
**Format:** JSON requests and responses.

## Response format

Success:
```json
{ "success": true, "message": "Course created successfully", "data": { } }
```
List endpoints also return `"count"`.

Error:
```json
{ "success": false, "message": "Validation failed", "errors": ["Price cannot be negative"] }
```
`errors` is present only when there are several field-level messages.

## Status codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created |
| 400 | Validation error, malformed JSON, or invalid ID |
| 403 | Origin not allowed by CORS |
| 404 | Record or route not found |
| 409 | Conflict - duplicate enrollment, or deleting a course that has enrollments |
| 413 | Request body larger than 100 KB |
| 503 | Database not connected |
| 500 | Unexpected server error (details are not exposed) |

---

## Health

### GET `/api/health`
Returns API and database state. `200` when MongoDB is connected, `503` otherwise.
```json
{ "success": true, "message": "API is running",
  "data": { "api": "ok", "database": "connected", "timestamp": "2026-10-28T05:41:30.229Z" } }
```

---

## Courses

### GET `/api/courses`
Query parameters (all optional): `search` (title, instructor, description, category), `category` (exact), `level` (exact).
Each course includes `enrollmentCount`. Newest first.

### GET `/api/courses/:id`
Returns one course (with `enrollmentCount`). `400` for a malformed ID, `404` if it does not exist.

### POST `/api/courses`
```json
{
  "title": "Intro to Python",
  "description": "Learn Python from scratch.",
  "instructor": "Dr. A. Kumar",
  "category": "Data Science",
  "level": "Beginner",
  "durationWeeks": 6,
  "price": 1999,
  "imageUrl": "https://example.com/python.jpg",
  "prerequisites": "None",
  "learningOutcomes": ["Write Python programs", "Use lists and dictionaries"]
}
```
Required: `title`, `description`, `instructor`, `category`, `durationWeeks`, `price`.
Rules: `price >= 0`; `durationWeeks` between 1 and 520; `level` one of Beginner/Intermediate/Advanced; `imageUrl` must be http(s) if given. `learningOutcomes` may be an array or a newline-separated string. Returns `201`.

### PUT `/api/courses/:id`
Same fields as POST; send only the fields to change. Validators run on the update. Returns the updated course.

### DELETE `/api/courses/:id`
Deletes the course **only if it has no enrollments**; otherwise `409` with the number of enrollments.

---

## Enrollments

### GET `/api/enrollments`
Query parameters (all optional): `search` (name, email, phone, or course title), `status` (exact), `courseId` (a valid ObjectId). Each enrollment has its `course` populated (`title`, `instructor`, `category`, `level`, `durationWeeks`, `price`, `imageUrl`).

### GET `/api/enrollments/:id`
One enrollment, or `404`.

### POST `/api/enrollments`
```json
{
  "studentName": "Asha Rao",
  "email": "asha@example.com",
  "phone": "+91 98765 43210",
  "course": "<course _id>",
  "status": "Enrolled",
  "enrollmentDate": "2026-10-28"
}
```
Required: `studentName`, `email`, `phone`, `course`. `status` defaults to `Enrolled`; `enrollmentDate` defaults to now. The email is stored in lowercase. `404` if the course does not exist; `409` if this email is already enrolled in that course. Returns `201`.

### PUT `/api/enrollments/:id`
Any of the POST fields. Changing `course` or `email` re-checks the duplicate rule (`409`).

### DELETE `/api/enrollments/:id`
Deletes the enrollment, or `404`.

---

## Dashboard

### GET `/api/dashboard/stats`
All values are calculated from MongoDB.
```json
{ "success": true, "message": "Dashboard statistics retrieved successfully",
  "data": {
    "totalCourses": 6, "totalEnrollments": 3, "totalInstructors": 6, "totalCategories": 6,
    "categories": [{ "name": "Web Development", "count": 1 }],
    "enrollmentsByStatus": [{ "status": "Enrolled", "count": 3 }]
  } }
```

---

## Trying the API with curl (Command Prompt on Windows 10/11)

PowerShell aliases `curl` to another command, so use `curl.exe` there. In Command Prompt, JSON quotes must be escaped:

```
curl.exe http://localhost:5000/api/health
curl.exe http://localhost:5000/api/courses
curl.exe -X POST http://localhost:5000/api/courses -H "Content-Type: application/json" -d "{\"title\":\"Test\",\"description\":\"Demo\",\"instructor\":\"Me\",\"category\":\"Web Development\",\"durationWeeks\":4,\"price\":0}"
```
Or use a GUI such as Postman or the Thunder Client extension for VS Code.
