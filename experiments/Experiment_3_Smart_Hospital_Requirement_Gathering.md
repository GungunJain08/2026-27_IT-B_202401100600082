# Experiment 3: Software Design Principles

## Experiment Statement
Identify stakeholders and perform requirement elicitation for the selected software project using suitable techniques such as interviews, questionnaires, observation, or user personas; prepare user stories and the Requirement Gathering Report.

## Aim
To identify stakeholders and perform requirement elicitation for the Smart Hospital Management System using suitable elicitation techniques, prepare user stories, and develop a Requirement Gathering Report.

## Objectives
- Identify key stakeholders of the Smart Hospital Management System.
- Apply suitable requirement elicitation techniques.
- Gather functional and non-functional requirements.
- Prepare user personas and user stories with acceptance criteria.
- Document the Requirement Gathering Report for the proposed system.

## Introduction
Requirement elicitation is the process of discovering, understanding, and documenting the needs and expectations of stakeholders. For a hospital management system, requirements must cover patient services, staff workflows, administration, privacy, security, reliability, and timely communication. Interviews, questionnaires, observation, and user personas can be combined to reduce ambiguity and improve requirement completeness.

## Selected Project
**Project Title:** Smart Hospital Management System  
**Domain:** Healthcare / Hospital Management

The proposed system is a cloud-based hospital management system that allows patients to manage their medical history, view recommended diagnostic tests, book test slots online, and receive medicine/treatment reminders. Hospital staff and administrators use role-based access to maintain records and manage permitted operations. The planned architecture uses AWS services such as Amazon Cognito, API Gateway, Lambda, RDS, S3, EventBridge, SNS and SES.

## Stakeholder Identification

| Stakeholder | Role | Key Requirements / Expectations |
|---|---|---|
| Patient | Uses the patient portal | Secure access to own records, view history, book recommended tests, receive reminders. |
| Doctor / Clinical Staff | Maintains patient medical information | Create/update records, add diagnoses and notes, recommend tests, prescribe treatments. |
| Hospital Administrator | Manages hospital operations and access | Role-based control, monitoring, reliable records, administrative visibility. |
| System / IT Administrator | Maintains application and infrastructure | Availability, diagnostics, deployment, secure configuration, auditability. |
| Diagnostic/Test Staff | Handles recommended tests and booking workflow | Accurate test/slot information and booking status tracking. |
| Notification Services | Delivers automated reminders/confirmations | Reliable and timely SMS/email delivery. |

## Requirement Elicitation Techniques

| Technique | Purpose | Expected Outcome |
|---|---|---|
| Interview | Discuss workflows and pain points with patients, clinical staff and administrators. | Detailed functional requirements and constraints. |
| Questionnaire | Collect feedback from multiple patients/users efficiently. | User expectations, preferences and usability priorities. |
| Observation | Observe how hospital staff currently handle records, test coordination and follow-ups. | Workflow, usability and process requirements. |
| User Persona | Represent typical users and their goals/pain points. | User-centred requirements and priorities. |

## Sample Requirement Gathering / Elicitation Findings
> **Note:** These are project-aligned sample findings. Replace them with actual findings if your team conducts real interviews/questionnaires.

### Interview Findings
- Patients want to view their medical history without repeatedly visiting the hospital for basic information.
- Clinical staff need a simple way to update diagnoses, notes, recommended tests and prescriptions.
- Patients want to book recommended diagnostic tests online and see the booking status.
- Users expect medicine and treatment reminders to be delivered at the scheduled time.
- Hospital administrators require role-based access so users only see information permitted for their role.

### Observation Findings
- Manual record handling can require repeated coordination between patients and staff.
- Patients benefit from a simple portal with clearly separated services such as medical history, test booking and reminders.
- Staff workflows should minimize repeated data entry and make patient records easy to retrieve.
- Booking status should be visible to reduce uncertainty about whether a test is pending, booked or completed.

### Questionnaire Findings
- Security and privacy are high-priority expectations for medical information.
- Users prefer online test booking over unnecessary in-person coordination when slots are available.
- Patients value timely reminders for medicines and treatments.
- Users expect the system to be available and responsive when they need to access records or bookings.

## Functional Requirements

| ID | Requirement | Description |
|---|---|---|
| FR1 | User Authentication | Authenticate patients, staff and administrators using role-based access. |
| FR2 | Patient Records | Staff can create/update patient profiles, diagnoses, notes and medical history. |
| FR3 | Document Management | Staff can attach medical documents; patients can view permitted documents. |
| FR4 | Recommended Tests | Staff can recommend diagnostic tests against a patient's record. |
| FR5 | Online Test Booking | Patients can view recommended tests and book an available slot. |
| FR6 | Booking Status | Track booking status as pending, booked or completed. |
| FR7 | Prescriptions / Treatment | Staff can add prescriptions or treatment plans to a patient's record. |
| FR8 | Reminders | Generate scheduled medicine/treatment reminders. |
| FR9 | Notifications | Send booking confirmations and reminders through configured channels. |
| FR10 | Access Control | Enforce permissions according to the authenticated user's role. |

## Non-Functional Requirements

| ID | Category | Requirement |
|---|---|---|
| NFR1 | Security | Protect patient data through authentication, authorization and secure communication. |
| NFR2 | Privacy | Patients shall only access their own permitted medical information. |
| NFR3 | Reliability | Bookings and record updates should be processed consistently without data loss. |
| NFR4 | Availability | The cloud-based system should be available whenever hospital services require it. |
| NFR5 | Performance | Common patient and staff operations should provide a reasonably fast response. |
| NFR6 | Usability | Interfaces should use simple navigation and clear labels. |
| NFR7 | Scalability | Architecture should support growth in patients, bookings and notifications. |
| NFR8 | Maintainability | Services should be modular for easier maintenance and deployment. |
| NFR9 | Auditability | Important access and record-changing actions should be traceable. |

## User Personas

| Persona | Age / Profile | Goal | Pain Point |
|---|---|---|---|
| Regular Patient | 35 | View history and book recommended tests quickly. | Repeated hospital visits and waiting for coordination. |
| Senior Patient | 65 | Access records and receive simple treatment reminders. | Complex navigation and difficulty remembering schedules. |
| Clinical Staff | 40 | Maintain records and recommend tests/treatments efficiently. | Manual record updates and repeated coordination. |
| Hospital Administrator | 45 | Manage access and monitor system operations. | Need for controlled access and reliable operational visibility. |

## User Stories and Acceptance Criteria

| ID | User Story | Acceptance Criteria |
|---|---|---|
| US1 | As a patient, I want to log in securely so that only I can access my permitted medical information. | Valid credentials authenticate; invalid credentials are rejected; role permissions are enforced. |
| US2 | As a patient, I want to view my medical history so that I can understand previous diagnoses and treatment information. | Patient can view only permitted record information and it is displayed correctly. |
| US3 | As clinical staff, I want to update a patient's medical record so that the latest clinical information is stored. | Authorized staff can update permitted fields; unauthorized users cannot modify the record. |
| US4 | As clinical staff, I want to recommend a diagnostic test so that the patient can arrange the required test. | Recommendation is linked to the correct patient and appears in the patient portal. |
| US5 | As a patient, I want to book an available recommended test slot so that I can schedule my diagnostic test online. | Only recommended tests are bookable; available slots can be selected; booking status is recorded. |
| US6 | As a patient, I want to see my test booking status so that I know whether it is pending, booked or completed. | Correct status is displayed and changes when the booking state changes. |
| US7 | As clinical staff, I want to add a prescription/treatment plan so that reminders can be generated from the schedule. | Authorized staff can save the schedule and link it to the correct patient. |
| US8 | As a patient, I want to receive medicine/treatment reminders so that I do not miss scheduled care. | Reminder is generated and sent through the configured notification channel. |
| US9 | As an administrator, I want role-based access control so that users can only perform permitted actions. | Patient, staff and admin permissions are enforced at the API level. |
| US10 | As a hospital administrator, I want reliable system operations so that patient and booking data remain consistent. | Updates complete successfully or fail safely without inconsistent data. |

## Requirement Gathering Report
The elicitation process for the Smart Hospital Management System identified patient record management, online diagnostic test booking, medicine/treatment reminders, secure authentication, role-based access, notifications and reliable data handling as the primary requirements. Stakeholder expectations emphasize privacy, ease of use, timely communication, accurate records and dependable service availability.

## Observation
Using multiple elicitation techniques improves requirement completeness and reduces ambiguity. Combining interviews, questionnaires, observation and user personas helps capture both technical requirements and user-centred needs across patients, clinical staff and administrators.

## Result
Stakeholders were identified, suitable requirement elicitation techniques were selected, functional and non-functional requirements were documented, user personas and user stories with acceptance criteria were prepared, and the Requirement Gathering Report for the Smart Hospital Management System was successfully completed.

## Team Contribution Note
This experiment focuses on the requirement elicitation and software-design documentation contribution handled by the assigned team member. The overall project is a three-member team project, with implementation responsibilities divided across authentication/patient records, test booking, and reminders/admin functionality.
