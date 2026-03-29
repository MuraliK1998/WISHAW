# Wishaw Backend – `read.md`

## Overview
This backend is a **Node.js + Express + TypeScript** API for the Wishaw volunteer/staff platform.
It handles:
- user registration and login with JWT authentication,
- volunteer profile management,
- admin-only staff approval/rejection,
- admin-only event creation and event listing,
- approval email notifications.

Tech stack:
- Express 5
- TypeScript
- MongoDB + Mongoose
- JWT + bcrypt
- Nodemailer

---

## Project Structure

```text
src/
  config/
    db.ts                # MongoDB connection
  controllers/
    authController.ts    # register/login handlers
    userController.ts    # profile + admin staff actions
    eventController.ts   # event creation + listing
  middleware/
    authMiddleware.ts    # protect/admin guards
  models/
    User.ts
    Event.ts
  routes/
    authRoutes.ts
    userRoutes.ts
    eventRoutes.ts
  services/
    authService.ts       # auth business logic
    emailService.ts      # approval email transport/template
  scripts/
    seedAdmin.ts         # one-off admin bootstrap
  index.ts               # app bootstrap + middleware + routes
```

---

## Environment Variables
Create a `.env` file in `wishaw-backend/`.

Required:
- `MONGODB_URI` – Mongo connection string
- `JWT_SECRET` – secret used to sign auth tokens

Optional:
- `PORT` – API server port (default: `5001`)
- `NODE_ENV` – `production` hides stack traces in errors

Email-related (optional but recommended for real email delivery):
- `SMTP_HOST`
- `SMTP_PORT`
- `SMTP_USER`
- `SMTP_PASS`

> If SMTP credentials are not configured, the app automatically uses an Ethereal test account for email preview links.

---

## Install & Run
From `wishaw-backend/`:

```bash
npm install
npm run dev
```

Backend starts on:
- `http://localhost:5001` (unless overridden by `PORT`)

Production start script expects compiled output at `dist/index.js`:

```bash
npm start
```

---

## Seed Admin Account
Run once to create a default admin user:

```bash
npm run seed
```

Default credentials created by the script:
- Email: `admin@wishaw.com`
- Password: `admin123`

> Change this immediately in real deployments.

---

## API Summary
All routes are mounted under `/api`.

### Auth Routes
- `POST /api/auth/register` – create user account
- `POST /api/auth/login` – authenticate and return token + user payload

### User Routes
- `GET /api/users/profile` – get current user profile (**protected**)
- `PUT /api/users/profile` – update user qualifications/availability/volunteer flag (**protected**)
- `GET /api/users` – list users (**protected + admin**)
- `PATCH /api/users/:id/status` – set `PENDING | APPROVED | REJECTED` and optionally assign events (**protected + admin**)

### Event Routes
- `POST /api/events` – create event (**protected + admin**)
- `GET /api/events` – list events (**protected + admin**)

Authentication uses `Authorization: Bearer <token>`.

---

## Data Model Notes

### `User`
Core fields:
- identity/contact: `emailId`, `firstName`, `middleName`, `lastName`, `phoneNumber`, `gender`
- auth: `password` (hashed)
- profile: `qualifications[]`, `hoursPerWeek`, `availableFrom`, `availableTo`, `willingToVolunteer`
- workflow: `status` (`PENDING`, `APPROVED`, `REJECTED`)
- authorization: `role` (`user`, `admin`)
- assignment: `assignedEvents[]`

### `Event`
Core fields:
- `eventName`, `location`
- staffing requirements: `hours`, `personsNeeded`, `qualifications[]`
- schedule: `startDate`, `endDate`
- assignment tracking: `assignedStaff[]`

---

## Business Workflow
1. User registers and logs in.
2. User updates profile with qualifications + availability and marks willingness to volunteer.
3. Admin reviews users in staff management.
4. Admin approves/rejects and can assign one or more events.
5. On approval, backend attempts to send an email containing assignment details.

---

## Validation & Security Notes
- Password hashing via `bcryptjs`.
- JWT token generated with 7-day expiry.
- Protected routes enforced via middleware (`protect`), admin routes via (`admin`).
- Qualification values are whitelist-filtered in profile updates.
- Error responses return stack traces only outside production mode.

---

## Gaps / Improvements (Optional Next Steps)
- Add unit/integration tests (currently no real `npm test` workflow).
- Add request schema validation (e.g., Zod/Joi) for stricter payload handling.
- Add refresh-token/session invalidation strategy.
- Add rate limiting and login brute-force protection.
- Add API docs (OpenAPI/Swagger).
