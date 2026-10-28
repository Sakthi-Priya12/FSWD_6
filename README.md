# CourseHub - Online Course Management System

A full-stack web application for the FSWD lab activity **"AppTailor - Stitch Your App"**.

| Layer     | Technology                                              |
|-----------|---------------------------------------------------------|
| Frontend  | React 18, Vite, React Router, Axios, Lucide icons, CSS3 |
| Backend   | Node.js, Express.js (REST API), Mongoose, CORS, dotenv  |
| Database  | MongoDB Community Server (local)                        |

- **Database name:** `onlineCourseDB`
- **Connection URI:** `mongodb://127.0.0.1:27017/onlineCourseDB`
- **Frontend URL:** http://localhost:5173
- **API base URL:** http://localhost:5000/api
- **Health check:** http://localhost:5000/api/health

Other documents: [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) | [API_DOCUMENTATION.md](API_DOCUMENTATION.md) | [TESTING_CHECKLIST.md](TESTING_CHECKLIST.md)

---

## What you can do in the app

| Page | Route | What it does |
|------|-------|--------------|
| Home | `/` | Hero, live statistics, featured courses, categories, why-choose section |
| All Courses | `/courses` | Search, category + level filters, course cards |
| Course Details | `/courses/:id` | Full course information, enroll button, "not found" handling |
| Enroll | `/enroll` | Student enrollment form with validation and duplicate prevention |
| My Enrollments | `/enrollments` | Search, filter, edit, delete (with confirmation) |
| Manage Courses | `/manage-courses` | Stats, course table, search, category filter, edit, delete |
| Add Course | `/manage-courses/new` | Create a course |
| Edit Course | `/manage-courses/:id/edit` | Update a course |
| 404 | any other URL | Not-found page with a Home button |

---

## Setup on Windows (step by step)

### Step 1 - Install Node.js LTS
1. Download the **LTS** installer from https://nodejs.org and run it (keep the default options).
2. Open a **new** terminal and check:
   ```
   node -v
   npm -v
   ```
   Both should print a version number (Node 18 or newer).

### Step 2 - Install MongoDB Community Server
1. Download **MongoDB Community Server** (Windows, MSI) from https://www.mongodb.com/try/download/community.
2. In the installer choose **Complete** and keep **"Install MongoDB as a Service"** ticked (Run as Network Service).
3. Verify the service is running:
   - Press `Win + R`, type `services.msc`, press Enter.
   - Find **MongoDB Server (MongoDB)**. Its status should be **Running**.
   - If it is not running: right-click it and choose **Start**.
   - Or, from a terminal opened **as Administrator**:
     - Command Prompt or PowerShell: `net start MongoDB`
     - PowerShell check: `Get-Service MongoDB`

### Step 3 - (Optional) Install MongoDB Compass
Compass is a visual tool for looking at your data. It is offered at the end of the MongoDB installer, or download it from https://www.mongodb.com/products/tools/compass. Connect to:
```
mongodb://127.0.0.1:27017
```

### Step 4 - Extract the ZIP
Right-click `Online-Course-Management-System.zip` -> **Extract All...** and choose a simple location such as `C:\Projects\`.

### Step 5 - Open the project in VS Code
Open VS Code -> **File -> Open Folder...** -> select the `online-course-management-system` folder.

### Step 6 - Open a terminal in the project folder
In VS Code: **Terminal -> New Terminal**. It opens in the project folder.

### Step 7 - Install backend dependencies
```
cd backend
npm install
```

### Step 8 - Configure backend environment variables
Create the real `.env` file from the template.

Command Prompt:
```
copy .env.example .env
```
PowerShell:
```
Copy-Item .env.example .env
```
The default contents already work for a standard local MongoDB:
```
PORT=5000
MONGODB_URI=mongodb://127.0.0.1:27017/onlineCourseDB
CORS_ORIGIN=http://localhost:5173,http://127.0.0.1:5173
NODE_ENV=development
```
(The database `onlineCourseDB` is created automatically the first time data is saved.)

### Step 9 - Start the backend
```
npm run dev
```
You should see:
```
MongoDB connected: database "onlineCourseDB"
API running at http://localhost:5000/api
```
Leave this terminal open.

### Step 10 - Install frontend dependencies (second terminal)
Click the **+** icon in the VS Code terminal panel to open a second terminal in the project root, then:
```
cd frontend
npm install
```

### Step 11 - Configure frontend environment variables
Command Prompt:
```
copy .env.example .env
```
PowerShell:
```
Copy-Item .env.example .env
```
Contents:
```
VITE_API_URL=http://localhost:5000/api
```

### Step 12 - Start the frontend
```
npm run dev
```

### Step 13 - Open the website
Open the exact URL printed by Vite, normally **http://localhost:5173**.

### Step 14 - Test the API health endpoint
Open http://localhost:5000/api/health in the browser. You should see:
```json
{ "success": true, "message": "API is running", "data": { "api": "ok", "database": "connected", "timestamp": "..." } }
```

### Step 15 - Add data and try everything
1. **Seed sample courses (optional)** - in a terminal inside `backend`:
   ```
   npm run seed
   ```
   This inserts six sample courses. Running it again does **not** create duplicates.
2. Or add your own on the site: **Manage Courses -> Add Course**.
3. Open **Courses**, search and filter, open a course.
4. Click **Enroll Now**, submit the form, then go to **My Enrollments**.
5. Edit an enrollment (change status), then delete one (confirm the dialog).
6. Edit and delete a course. (A course that still has enrollments cannot be deleted - see below.)

### Step 16 - Inspect the data in MongoDB Compass
1. Open Compass and connect to `mongodb://127.0.0.1:27017`.
2. In the left sidebar open the **onlineCourseDB** database.
3. Open the **courses** and **enrollments** collections to see the documents the website saved. Add, edit, or delete something on the site and click the **refresh** button in Compass to see it change.

---

## Useful commands

| Where | Command | Purpose |
|-------|---------|---------|
| `backend` | `npm run dev` | Start API with auto-restart (nodemon) |
| `backend` | `npm start` | Start API without auto-restart |
| `backend` | `npm run seed` | Insert 6 sample courses (skips ones that already exist) |
| `backend` | `npm run seed:clear` | Remove **only** the sample courses (skips any that have enrollments) |
| `frontend` | `npm run dev` | Start the React dev server |
| `frontend` | `npm run build` | Create a production build in `frontend/dist` |

To clear sample data: `npm run seed:clear` in `backend`. To remove everything, open Compass, right-click the `onlineCourseDB` database, and choose **Drop Database**. The server never deletes data on its own.

---

## Business rules to know

- **Duplicate enrollments:** the same email cannot enroll in the same course twice (checked by the server and enforced by a unique MongoDB index). Emails are stored in lowercase.
- **Deleting a course:** a course that still has enrollments **cannot be deleted** (the API returns `409`). Delete or move those enrollments first. This keeps every enrollment pointing at a real course.
- **Duration** is stored in **whole weeks**; **price** is in Indian rupees (INR) and `0` shows as "Free".

---

## Troubleshooting

### MongoDB connection refused (`ECONNREFUSED 127.0.0.1:27017`)
- Open `services.msc` and make sure **MongoDB Server** is **Running** (right-click -> Start).
- Keep `127.0.0.1` in `MONGODB_URI` (not `localhost`); on newer Node versions `localhost` can resolve to IPv6 and fail.
- If you installed MongoDB without the service option, start it manually in a terminal: `"C:\Program Files\MongoDB\Server\<version>\bin\mongod.exe" --dbpath C:\data\db` (create `C:\data\db` first).

### `mongod` is not recognized
You only need this for manual starts; the Windows service works without it. To use it from any terminal, add MongoDB's `bin` folder to PATH: Start menu -> "Edit the system environment variables" -> Environment Variables -> Path -> New -> `C:\Program Files\MongoDB\Server\<version>\bin`. Open a new terminal afterwards.

### PowerShell says "npm.ps1 cannot be loaded because running scripts is disabled"
Do **not** turn off Windows security. Use either of these safe options:
- Run npm through its `.cmd` launcher in PowerShell: `npm.cmd install`, `npm.cmd run dev`
- Or open **Command Prompt** (in VS Code: terminal dropdown -> *Command Prompt*) and use normal `npm` commands.

### Port 5000 or 5173 is already in use
Find the process (Command Prompt or PowerShell):
```
netstat -ano | findstr :5000
```
The last column is the PID. Stop it with `taskkill /PID <PID> /F`, or change the port:
- Backend: change `PORT` in `backend/.env` **and** `VITE_API_URL` in `frontend/.env`, then restart both.
- Frontend: Vite automatically picks the next free port (for example 5174) and prints it. If the printed port is not 5173, add that origin (for example `http://localhost:5174`) to `CORS_ORIGIN` in `backend/.env` and restart the backend.

### CORS errors in the browser console
The message mentions "blocked by CORS policy". The page's origin (for example `http://localhost:5174`) must be listed in `CORS_ORIGIN` in `backend/.env`, separated by commas. Restart the backend after editing `.env`.

### The website shows "Cannot reach the server"
1. Is the backend terminal running and showing `API running at http://localhost:5000/api`?
2. Open http://localhost:5000/api/health - does it respond?
3. Does `frontend/.env` contain `VITE_API_URL=http://localhost:5000/api`? After editing `.env`, **stop and restart** `npm run dev` in the frontend.

### How to check for errors
- **Browser:** press `F12` -> **Console** tab (JavaScript errors) and **Network** tab (click a failed request to see its status and response).
- **Backend:** read the terminal running `npm run dev`; startup and server errors are printed there.
- **Health:** http://localhost:5000/api/health shows whether the database is connected.

### "Database is not connected" (503)
MongoDB stopped after the server started. Start the MongoDB service and restart the backend.

---

## Security note

This is a **college demonstration project**. It has **no login, authentication, or role-based authorization**. Anyone who can open the site can view and change courses and enrollment records. A production system would need authentication (for example JWT or sessions), authorization so students only see their own enrollments, and admin-only course management. See PROJECT_OVERVIEW.md for more.

## Project structure

```
online-course-management-system/
  README.md, PROJECT_OVERVIEW.md, API_DOCUMENTATION.md, TESTING_CHECKLIST.md, .gitignore
  backend/
    server.js            starts the server after MongoDB connects
    app.js               Express app (CORS, JSON, routes, errors)
    config/db.js         Mongoose connection
    models/              Course.js, Enrollment.js
    controllers/         courseController.js, enrollmentController.js, dashboardController.js
    routes/              courseRoutes.js, enrollmentRoutes.js, dashboardRoutes.js
    middleware/          errorMiddleware.js (async handler, ID validation, errors)
    scripts/seed.js      sample data
    .env.example
  frontend/
    index.html, vite.config.js, .env.example
    src/
      api/               axios.js + courseApi / enrollmentApi / dashboardApi
      components/        Navbar, Footer, CourseCard, CourseForm, Modal, ConfirmDialog, ...
      pages/             the 9 pages
      hooks/, utils/, styles/
```
