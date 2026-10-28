# Testing Checklist and Report

This file separates what was **actually run** while the project was being built from what **you must verify on your own computer** with a real MongoDB. Nothing is marked as passed unless it was executed.

> **Important limitation:** the environment where this project was generated had **no MongoDB server** (its download was blocked). So every test that needs a live database - saving, reading, updating, deleting, duplicate detection against stored data, seed script insertion - was **not run** here. The code is written for a standard local MongoDB; Part B is your checklist to confirm it end to end.

---

## Part A - Tests that were actually run (no database needed)

| # | Check | Result |
|---|-------|--------|
| A1 | `npm install` in `backend` | Passed |
| A2 | `npm install` in `frontend` | Passed |
| A3 | `node --check` on every backend `.js` file | Passed, no syntax errors |
| A4 | `npm run build` in `frontend` (Vite production build, 1655 modules) | Passed |
| A5 | Both `package.json` files parse as valid JSON and install cleanly | Passed |
| A6 | Starting `node server.js` with MongoDB not running | Printed a clear "MongoDB connection failed (ECONNREFUSED)" message and exited with code 1 (API does not start without a database) |
| A7 | Running `node scripts/seed.js` with MongoDB not running | Printed "Seed failed: connect ECONNREFUSED" and exited with code 1 |

### API behaviour with the database disconnected (HTTP tests against the Express app)

| Check | Result |
|-------|--------|
| `GET /api/health` returns `503` with `database: "disconnected"` and no connection string in the body | Passed |
| `GET/POST /api/courses`, `GET/POST /api/enrollments`, `GET /api/dashboard/stats` return `503` JSON `{success:false,...}` | Passed |
| Unknown route returns `404` JSON | Passed |
| Malformed JSON body returns `400 "Invalid JSON in request body"` | Passed |
| Request body over 100 KB returns `413` | Passed |
| CORS header present for `http://localhost:5173` | Passed |
| CORS header absent for an unlisted origin | Passed |

### Schema validation (Mongoose, no connection required) - 17 checks, all passed
Valid course accepted; empty required fields rejected; negative price rejected; non-numeric price rejected; duration 0 and negative duration rejected; invalid level rejected; invalid image URL rejected and valid URL accepted; valid enrollment accepted; email lowercased; default status is `Enrolled`; invalid email rejected; invalid phone rejected; missing course rejected; invalid status rejected; unique `(email, course)` index is defined on the enrollment schema.

### Controller validation paths that fail before any database call - 14 checks, all passed
(The connection guard was bypassed **in the test only**, and only requests rejected before any query were asserted.)

| Check | Result |
|-------|--------|
| Invalid ID on GET/PUT/DELETE for courses and enrollments returns `400` | Passed |
| `GET /api/enrollments?courseId=bad` returns `400` | Passed |
| `POST /api/courses` with negative price returns `400 "Price cannot be negative"` | Passed |
| `POST /api/courses` with non-numeric price / blank duration returns `400` with a clear message | Passed |
| `PUT /api/courses/:id` with blank price returns `400` | Passed |
| `POST /api/enrollments` with an empty body returns `400` listing all missing fields | Passed |
| `POST /api/enrollments` with bad email and phone returns `400` with both messages | Passed |
| `POST /api/enrollments` with an invalid course ID returns `400 "Invalid course ID"` | Passed |

### Not tested in Part A
- The React pages were **built** but not opened or clicked in a browser, so layout, navigation, and form behaviour are unverified visually.
- No live database operations (see Part B).

---

## Part B - Verify on your computer (MongoDB running, backend and frontend started)

Tick each box as you test. Use the website unless a different method is named.

### Setup
- [ ] `npm install` succeeds in `backend` and `frontend`
- [ ] Backend prints `MongoDB connected: database "onlineCourseDB"`
- [ ] http://localhost:5000/api/health shows `"database": "connected"`
- [ ] http://localhost:5173 loads with no errors in the browser console (F12)
- [ ] `npm run seed` inserts 6 courses; running it a second time inserts 0 (no duplicates)
- [ ] With an empty database the Home, Courses, Enrollments, and Manage pages show friendly empty states

### Course CRUD
- [ ] Add a course through **Add Course** - success message appears on Manage Courses
- [ ] The course appears in **Courses**, **Manage Courses**, and in Compass (`onlineCourseDB` -> `courses`)
- [ ] **View Details** opens `/courses/<id>` with all fields
- [ ] **Edit** the course, change the price and description, save - success message and new values shown
- [ ] Refresh the page; the updated values persist; Compass shows them too
- [ ] **Delete** the course (no enrollments) - confirmation dialog appears, then the course disappears
- [ ] Search by title/instructor works; the category filter works; the level filter works on Courses

### Enrollment CRUD
- [ ] Enroll a student through **Enroll** - success appears only after saving; link to enrollments works
- [ ] The enrollment appears in **My Enrollments** (course name, date, status) and in Compass (`enrollments`)
- [ ] **Edit** the enrollment (change status to In Progress) - list refreshes
- [ ] Search (name/email/phone/course) and the course/status filters work
- [ ] **Delete** the enrollment - confirmation, then it is removed from the list and from Compass
- [ ] Course Details and the Manage table show an updated "Enrolled" count

### Validation and error handling
- [ ] Submit the enrollment form empty - each required field shows an error and nothing is sent
- [ ] Invalid email (`abc`) is rejected; invalid phone (`abc`) is rejected
- [ ] Enroll the same email in the same course twice - second attempt shows "already enrolled" (HTTP 409)
- [ ] Add Course with a negative price - blocked on the form
- [ ] Add Course with duration `0` or a decimal - blocked on the form
- [ ] Add Course with image URL `not-a-url` - blocked
- [ ] Open `http://localhost:5173/courses/123` - "Course not found" (invalid ID handled)
- [ ] Open `/courses/507f1f77bcf86cd799439011` - "Course not found" (valid but nonexistent ID)
- [ ] Open `/manage-courses/507f1f77bcf86cd799439011/edit` - "Course not found"
- [ ] Edit a course in one tab, delete it in another, then Save in the first - handled as not found
- [ ] Try to delete a course that has enrollments - an error explains it cannot be deleted (HTTP 409)
- [ ] Open `http://localhost:5173/anything` - 404 page; its button returns Home
- [ ] `GET http://localhost:5000/api/enrollments/abc` (browser) returns a `400` JSON error
- [ ] `GET http://localhost:5000/api/enrollments/507f1f77bcf86cd799439011` returns `404`

### Failure scenarios
- [ ] **MongoDB stopped:** stop the MongoDB service while the backend runs - the site shows a "Database is not connected" error and `/api/health` returns 503; start the service again, restart the backend, and data is back
- [ ] **Backend stopped:** stop the backend - the site shows "Cannot reach the server..." with a **Try again** button; restart the backend and click it
- [ ] Mobile layout: narrow the browser window (or F12 -> device toolbar); the menu collapses and tables become cards

### Optional API checks with curl.exe / Postman
- [ ] `POST /api/courses` with valid JSON returns `201`
- [ ] `PUT /api/courses/<id>` returns the updated course
- [ ] `DELETE /api/courses/<id>` returns `200`, then `GET` returns `404`
- [ ] `GET /api/dashboard/stats` totals match what you see in Compass

---

## Notes for your lab report
- Record the date, your Node and MongoDB versions, and a screenshot for each ticked area (Compass collections, the website pages, and a validation error).
- If a box fails, copy the message from the browser console (F12) or the backend terminal; the README troubleshooting section covers the common causes.
