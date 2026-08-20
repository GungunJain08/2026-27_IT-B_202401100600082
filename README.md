# Smart Hospital Management System

A cloud-based hospital management system that lets patients manage their medical history, book recommended diagnostic tests online, and receive medicine/treatment reminders — with role-based access for hospital staff, admins, and patients. Built on AWS.

## Overview

Hospitals still rely heavily on manual record-keeping and in-person coordination for test bookings and treatment follow-ups. This project digitizes that workflow into a single system where:

- Staff can maintain a patient's medical history and recommend tests or prescribe treatments directly against their record.
- Patients can log in, see the tests recommended for them, and book a slot online.
- Patients receive automated reminders for medicines and treatments based on the schedule staff set.
- Access is strictly role-based — patients only see their own data; staff/admins see what their role permits.

## Core Features

1. **Patient Records Management** — Staff can create and update patient profiles, log diagnoses and notes, and attach past medical documents (reports, prescriptions). Patients can view their own history read-only.
2. **Online Test Booking** — Patients see the list of tests recommended for them by staff and can book an available slot. Booking status (pending / booked / completed) is tracked against their record.
3. **Medicine & Treatment Reminders** — Reminders are generated from prescriptions/treatment plans staff add to a patient's record, and delivered automatically by SMS/email at the scheduled time.
4. **Role-Based Access (Staff/Admin & Patient login)** — Separate authenticated experiences for patients vs. hospital staff/admin, with permissions enforced at the API level, not just hidden in the UI.

## System Architecture

```text
Patient Portal          Staff / Admin Portal
       |                         |
       +----------+--------------+
                  |
            Amazon Cognito
   (patient / staff / admin groups)
                  |
          Amazon API Gateway
                  |
       +----------+--------------+
       |          |              |
    Patient  Test Booking  Reminder Service
    Records     Service   (EventBridge + SNS/SES)
    Service       |              |
       |          |              |
       +----------+--------------+
                  |
        Amazon RDS + Amazon S3
(patient data, bookings, prescriptions,
             documents)
```

- **Frontend:** React (or HTML/CSS/JS) — separate views for patient vs. staff/admin, hosted on Amazon S3 + served via CloudFront.
- **Auth:** Amazon Cognito User Pools with `Patient`, `Staff`, and `Admin` groups.
- **API:** Amazon API Gateway with a Cognito authorizer on every route.
- **Compute:** AWS Lambda, split into three services — patient records, test booking, reminders.
- **Database:** Amazon RDS (PostgreSQL/MySQL) for relational data — patients, staff, tests, bookings, prescriptions.
- **File storage:** Amazon S3 for uploaded documents (reports, scans, prescriptions).
- **Notifications:** Amazon EventBridge Scheduler + Amazon SNS (SMS) / Amazon SES (email) for reminders and booking confirmations.
- **Infrastructure as Code:** AWS SAM for deploying the backend.

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React.js / HTML, CSS, JS |
| Hosting | Amazon S3 + CloudFront |
| Authentication | Amazon Cognito |
| API | Amazon API Gateway |
| Compute | AWS Lambda |
| Database | Amazon RDS (PostgreSQL/MySQL) |
| File storage | Amazon S3 |
| Scheduling & notifications | Amazon EventBridge, Amazon SNS, Amazon SES |
| Infrastructure as Code | AWS SAM |
| Version control | Git / GitHub |

## Data Model (high level)

`User (role: patient/staff/admin) -> PatientProfile -> MedicalRecord -> RecommendedTest -> Booking`
`MedicalRecord -> Prescription -> Reminder`

## Team & Roles

| Member | Module | Responsibilities |
|---|---|---|
| [Name 1] | Auth & Patient Records | Cognito setup (user pools/groups), RDS schema, patient records API (CRUD, add diagnosis/recommend test/prescribe), staff-side records UI |
| [Name 2] | Test Booking | Booking API (view recommended tests, book/cancel slot), patient-side booking UI, booking confirmation notifications |
| [Name 3] | Reminders & Admin Dashboard | EventBridge scheduling, SNS/SES integration, reminder API, staff/admin dashboard UI |

## Project Structure

```text
smart-hospital-management-system/
├── frontend/
│   ├── patient-portal/
│   └── staff-admin-portal/
├── backend/
│   ├── patient-records-service/
│   ├── test-booking-service/
│   └── reminder-service/
├── infrastructure/
│   └── template.yaml      # AWS SAM template
├── docs/
│   └── architecture-diagram.png
└── README.md
```

## Getting Started

### Prerequisites

- Node.js (v18+) and npm
- AWS CLI, configured with credentials (`aws configure`)
- AWS SAM CLI
- Git

### Setup

```bash
git clone https://github.com/<org>/smart-hospital-management-system.git
cd smart-hospital-management-system

# Frontend
cd frontend/patient-portal
npm install
npm start

# Backend (per service)
cd backend/patient-records-service
sam build
sam deploy --guided
```

Environment-specific values (Cognito pool ID, API base URL, RDS endpoint) should be provided via a `.env` file — a `.env.example` will be added once the shared AWS foundation is provisioned.

## Branching Strategy

- `main` — stable, deployable code only
- `dev` — integration branch, merged from feature branches
- `feature/<module>-<short-description>` — one branch per task, e.g. `feature/records-add-test-recommendation`

Open a PR into `dev`, get at least one review from another team member, then merge.

## Status

- [ ] Shared AWS foundation (Cognito, RDS, S3)
- [ ] Patient records module
- [ ] Test booking module
- [ ] Reminder module
- [ ] Integration & testing
- [ ] Deployment

## License

To be decided by the team (e.g., MIT).
