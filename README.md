# HireLoop — Full-Stack Job Portal

A production-structured job portal where job seekers search and apply for jobs, and recruiters post
and manage job listings. Built with the MERN-adjacent stack: **React (Vite) + Tailwind CSS** on the
frontend, **Node.js + Express + MongoDB (Mongoose)** on the backend, secured with **JWT authentication**.

---

## 1. Tech Stack

| Layer      | Technology                                             |
|------------|---------------------------------------------------------|
| Frontend   | React 18, Vite, React Router v6, Tailwind CSS, Axios, react-hot-toast |
| Backend    | Node.js, Express.js, Mongoose (MongoDB)                |
| Auth       | JWT (jsonwebtoken), bcryptjs for password hashing       |
| Validation | express-validator (backend), inline validation (frontend) |
| Security   | helmet, cors, express-rate-limit                        |

---

## 2. Folder Structure

```
job-portal/
├── backend/
│   ├── config/
│   │   └── db.js                  # MongoDB connection
│   ├── controllers/
│   │   ├── authController.js      # register, login, getMe
│   │   ├── userController.js      # profile, dashboard stats, saved jobs
│   │   ├── jobController.js       # job CRUD, search/filter/pagination
│   │   ├── applicationController.js # apply, track, manage applicants
│   │   └── companyController.js   # company profiles
│   ├── middleware/
│   │   ├── authMiddleware.js      # protect (JWT verify), authorize (roles)
│   │   ├── errorMiddleware.js     # centralized error handler
│   │   └── validateRequest.js     # express-validator result handler
│   ├── models/
│   │   ├── User.js
│   │   ├── Job.js
│   │   ├── Application.js
│   │   └── Company.js
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── userRoutes.js
│   │   ├── jobRoutes.js
│   │   ├── applicationRoutes.js
│   │   └── companyRoutes.js
│   ├── utils/
│   │   └── seed.js                # demo data seeding script
│   ├── .env.example
│   ├── package.json
│   └── server.js                  # app entry point
│
└── frontend/
    ├── src/
    │   ├── components/             # Navbar, Footer, JobCard, Loader, etc.
    │   ├── pages/                  # Home, Login, Register, JobListings, JobDetails,
    │   │                           # PostJob, Dashboard, Profile, MyApplications, ApplicantsList
    │   ├── context/
    │   │   └── AuthContext.jsx     # global auth state
    │   ├── services/
    │   │   ├── api.js              # axios instance + interceptors
    │   │   └── jobService.js       # grouped API calls
    │   ├── utils/
    │   │   └── format.js           # salary formatting, time-ago, category colors
    │   ├── App.jsx                 # routes
    │   ├── main.jsx                # entry point
    │   └── index.css               # Tailwind + reusable component classes
    ├── index.html
    ├── tailwind.config.js
    ├── postcss.config.js
    ├── vite.config.js
    ├── .env.example
    └── package.json
```

---

## 3. Setup Instructions

### Prerequisites
- Node.js 18+
- MongoDB running locally, or a free MongoDB Atlas cluster

### Backend

```bash
cd backend
npm install
cp .env.example .env
# edit .env: set MONGO_URI and a strong JWT_SECRET
npm run dev          # starts on http://localhost:5000
```

Optional — seed the database with a demo recruiter, jobseeker, and sample jobs:

```bash
npm run seed
```
This creates:
- Recruiter login: `recruiter@demo.com` / `password123`
- Jobseeker login: `jobseeker@demo.com` / `password123`

### Frontend

```bash
cd frontend
npm install
cp .env.example .env
# edit .env if your backend runs on a different URL
npm run dev           # starts on http://localhost:5173
```

Open `http://localhost:5173` in your browser. The frontend expects the backend at
`http://localhost:5000/api` by default (set via `VITE_API_URL`).

### Production build

```bash
cd frontend
npm run build          # outputs static files to frontend/dist
```
Serve `frontend/dist` with any static host (Vercel, Netlify, Nginx, etc.), and deploy `backend/`
to a Node host (Render, Railway, Fly.io, etc.) with `NODE_ENV=production` and your production
`MONGO_URI` / `JWT_SECRET` / `CLIENT_URL` set.

---

## 4. API Documentation

Base URL: `http://localhost:5000/api`

All protected routes require an `Authorization: Bearer <token>` header.

### Auth
| Method | Endpoint            | Access  | Description                     |
|--------|----------------------|---------|----------------------------------|
| POST   | `/auth/register`     | Public  | Register a jobseeker or recruiter |
| POST   | `/auth/login`        | Public  | Log in, returns JWT + user       |
| GET    | `/auth/me`           | Private | Get the logged-in user           |

**Register body:**
```json
{ "name": "Ada Lovelace", "email": "ada@example.com", "password": "secret123", "role": "jobseeker" }
```

### Users
| Method | Endpoint                     | Access             | Description                    |
|--------|-------------------------------|--------------------|----------------------------------|
| PUT    | `/users/profile`              | Private            | Update your own profile         |
| GET    | `/users/dashboard`            | Private            | Role-aware dashboard stats      |
| PUT    | `/users/saved-jobs/:jobId`    | Private (jobseeker)| Toggle save/unsave a job        |
| GET    | `/users/:id`                  | Private            | Get a user's public profile     |

### Jobs
| Method | Endpoint                     | Access              | Description                                  |
|--------|-------------------------------|----------------------|-----------------------------------------------|
| GET    | `/jobs`                       | Public               | List jobs — supports query params below      |
| GET    | `/jobs/:id`                   | Public               | Get single job details                        |
| POST   | `/jobs`                       | Private (recruiter)  | Create a job posting                          |
| PUT    | `/jobs/:id`                   | Private (recruiter)  | Update your own job posting                   |
| DELETE | `/jobs/:id`                   | Private (recruiter)  | Delete your own job posting                   |
| GET    | `/jobs/recruiter/my-jobs`     | Private (recruiter)  | List jobs you've posted                       |

**`GET /jobs` query params:** `keyword`, `location`, `category`, `jobType`, `experienceLevel`,
`minSalary`, `sort` (`newest`|`oldest`|`salary-high`|`salary-low`), `page`, `limit`.

### Applications
| Method | Endpoint                          | Access               | Description                          |
|--------|-------------------------------------|-----------------------|-----------------------------------------|
| POST   | `/applications/:jobId`              | Private (jobseeker)   | Apply to a job                          |
| GET    | `/applications/my-applications`     | Private (jobseeker)   | List your applications                  |
| DELETE | `/applications/:id`                 | Private (jobseeker)   | Withdraw an application                 |
| GET    | `/applications/job/:jobId`          | Private (recruiter)   | List applicants for one of your jobs    |
| PUT    | `/applications/:id/status`          | Private (recruiter)   | Update an applicant's status            |

Application statuses: `applied`, `under review`, `shortlisted`, `rejected`, `hired`.

### Companies
| Method | Endpoint         | Access              | Description             |
|--------|-------------------|-----------------------|----------------------------|
| GET    | `/companies`       | Public                | List all companies         |
| GET    | `/companies/:id`   | Public                | Get one company profile    |
| POST   | `/companies`       | Private (recruiter)   | Create a company profile   |

### Error responses

All errors are returned as:
```json
{ "success": false, "message": "Human-readable error message" }
```
Validation errors additionally include an `errors` array with `{ field, message }` entries.

---

## 5. Key Design Decisions

- **JWT stored in `localStorage`** on the frontend, attached via an Axios request interceptor;
  a response interceptor auto-redirects to `/login` on a `401`.
- **Role-based access** is enforced both in the UI (`ProtectedRoute` with a `roles` prop) and on
  the backend (`authorize()` middleware) — never trust the client alone.
- **One application per job per user** is enforced with a unique compound index on
  `(job, applicant)` in the `Application` model.
- **Denormalized `companyName`** on `Job` keeps job search/listing fast without always joining
  against `Company`.
- **Text index** on `Job` (`title`, `companyName`, `skills`, `location`, `category`) backs the
  keyword search.

---

## 6. Next Steps / Ideas for Extension

- Add file uploads for resumes/avatars (e.g. via Multer + S3/Cloudinary) instead of URL fields.
- Add email notifications on application status changes.
- Add refresh tokens / shorter-lived access tokens for stronger session security.
- Add automated tests (Jest + Supertest for the API, Vitest + React Testing Library for the UI).
