# Wishaw Frontend – `read.md`

## Overview
This frontend is a **React + TypeScript + Vite** application for two audiences:
- **Standard users/volunteers** (signup, login, profile, calendar)
- **Admins** (admin login, staff management, event creation, calendar)

It communicates with the backend through `/api` endpoints using a shared Axios instance that injects JWT tokens from localStorage.

Tech stack:
- React 19
- React Router
- Axios
- TypeScript
- Vite

---

## Project Structure

```text
src/
  layouts/
    DashboardLayout.tsx         # user shell
    AdminDashboardLayout.tsx    # admin shell
  pages/
    LoginPage.tsx
    SignupPage.tsx
    DashboardHome.tsx
    EditProfilePage.tsx
    EventCalendar.tsx
    admin/
      AdminLogin.tsx
      StaffManagement.tsx
      EventCreation.tsx
  services/
    api.ts                      # axios instance + auth header interceptor
  context/
    ThemeContext.tsx            # dark/light theme state
  App.tsx                       # route map
```

---

## Setup & Run
From `wishaw-frontend/`:

```bash
npm install
npm run dev
```

By default Vite runs on `http://localhost:5173` and proxies API calls:
- `/api/*` → `http://localhost:5001`

(See `vite.config.ts` proxy config.)

Build + preview:

```bash
npm run build
npm run preview
```

---

## Routing Map

### Public
- `/login` – user login
- `/signup` – user registration
- `/admin/login` – admin login

### User Area (`/dashboard`)
- `/dashboard` – dashboard summary
- `/dashboard/edit-profile` – qualification/availability profile editor
- `/dashboard/calendar` – event calendar view

### Admin Area (`/admin/dashboard`)
- `/admin/dashboard/staff` – staff review, approval/rejection, event assignment
- `/admin/dashboard/events` – event creation and fulfillment tracking
- `/admin/dashboard/calendar` – calendar with admin staffing badges

---

## Auth & Session Behavior
- Login stores both `token` and `user` in `localStorage`.
- Axios interceptor adds `Authorization: Bearer <token>` for API calls.
- Layout components gate access by role:
  - User layout redirects admins to admin dashboard.
  - Admin layout redirects non-admins to admin login.
- Logout clears local storage and hard redirects to login route.

---

## Key UI Workflows

### User Flow
1. Sign up (`/signup`).
2. Log in (`/login`).
3. Update profile in `/dashboard/edit-profile`:
   - qualifications,
   - weekly hours,
   - date availability,
   - volunteer willingness.
4. View event calendar in `/dashboard/calendar`.

### Admin Flow
1. Log in (`/admin/login`) with admin account.
2. Open `/admin/dashboard/staff` to:
   - filter staff by current-week/day availability,
   - review pending users,
   - approve/reject users,
   - assign events during approval.
3. Use `/admin/dashboard/events` to:
   - create events,
   - inspect fulfillment progress (staff count + hour coverage),
   - review per-event details.
4. Use `/admin/dashboard/calendar` for date-centric event visibility.

---

## Data/State Notes
- Most server data is fetched per-page with `useEffect` and local component state.
- No global server-state library is used (e.g., React Query); fetching/refresh logic is embedded in pages.
- Profile page updates localStorage user snapshot after successful profile save.

---

## Styling & UX Notes
- Theme toggling is provided via `ThemeContext`.
- Layouts are responsive with mobile sidebar behavior.
- Staff management and event creation pages include dense operational admin UI (filtering, badges, progress indicators, modal assignment).

---

## Gaps / Improvements (Optional Next Steps)
- Add route-level guards using wrapper components instead of imperative `navigate` in render path.
- Add React Query (or equivalent) for better caching/revalidation.
- Add form schema validation with reusable validators.
- Add component-level test coverage (RTL/Vitest).
- Add role-aware nav constants to avoid route duplication.
