# Session 8 — Software Design Principles Report

## Project
Smart Hospital Management System

## 1. Objective
This report evaluates the proposed design of the Smart Hospital Management System using modularization, coupling and cohesion, function-oriented versus object-oriented design, and top-down versus bottom-up design. The recommendations follow the architecture described in the project README.

## 2. System overview
The system supports staff-managed patient records, staff-recommended diagnostic tests, patient test booking, medicine/treatment reminders, and role-based access. The documented target architecture uses patient and staff/admin portals, Amazon Cognito, API Gateway, AWS Lambda services, Amazon RDS, Amazon S3, EventBridge, SNS and SES.

## 3. Modularization

### Proposed modules
| Module | Responsibilities | Main inputs/outputs |
|---|---|---|
| Authentication & Authorization | Authenticate users and enforce Patient/Staff/Admin permissions | Credentials/token; identity and claims |
| Patient Records | Maintain profiles, diagnoses, notes, prescriptions and recommendations | Patient/staff requests; patient record data |
| Test Booking | List recommended tests, validate slots, create/cancel bookings and track status | Patient ID, test ID, slot ID; booking result |
| Reminder Service | Schedule medicine/treatment reminders and dispatch notifications | Prescription schedule; reminder events |
| Document Storage | Store and retrieve reports/prescriptions using object storage | File metadata and authorized upload/download |
| Web Portals | Present role-appropriate workflows and validate user input | User actions; API responses |
| Infrastructure & Configuration | Define cloud resources, environment settings and deployment | Configuration; deployed resources |

### Benefits
- Each module has a clearly defined responsibility.
- Changes to reminder delivery should not require rewriting test-booking logic.
- Modules can be tested independently and deployed according to operational needs.
- API contracts establish clear boundaries between the UI and backend services.

### Recommendation
Keep business rules in backend services rather than relying on hidden UI controls. Reuse common authorization and validation patterns, but avoid creating a single oversized service that owns all features.

## 4. Coupling and cohesion analysis

**Coupling** measures how strongly modules depend on one another. **Cohesion** measures how closely related the responsibilities inside a module are.

| Module | Target cohesion | Target coupling | Rationale |
|---|---|---|---|
| Patient Records Service | High | Low to moderate | Owns patient profile and medical-history rules |
| Test Booking Service | High | Low to moderate | Owns booking validation and booking status |
| Reminder Service | High | Low | Consumes prescription/reminder events rather than modifying booking internals |
| Patient Portal | High | Low | Calls documented APIs; should not access database directly |
| Staff/Admin Portal | High | Low | Uses authorized staff/admin APIs |
| Notification adapter | High | Low | Hides SNS/SES provider details behind a small interface |

### Risks to avoid
- **Shared database assumptions:** one service directly changing another service's tables creates schema coupling.
- **UI-enforced permissions only:** users could bypass visual restrictions by calling an API directly.
- **Synchronous notification dependency:** a temporary SMS/email outage should not necessarily undo a saved booking.
- **Duplicated business rules:** separate copies of slot validation may drift and produce inconsistent results.

### Improvements
1. Enforce role and record-ownership checks in API/backend code.
2. Use stable request/response schemas and document API contracts.
3. Emit a booking-confirmed event after a successful database commit; process notifications separately.
4. Keep environment-specific endpoints and credentials in configuration/secrets, not source code.
5. Add unit tests for authorization, booking conflicts and reminder scheduling.

## 5. Function-oriented vs object-oriented design

| Aspect | Function-oriented design | Object-oriented design | Recommended use here |
|---|---|---|---|
| Main organizing unit | Functions and data transformations | Objects/classes combining state and behavior | Use both where appropriate |
| Typical example | `validate_slot(slot_id)` | `Booking` with status-transition rules | Keep validation functions small and testable; model domain rules clearly |
| Strength | Simple, direct processing flow | Encapsulation and explicit domain concepts | Use function-oriented handlers in Lambda services and domain objects/value types for complex rules |
| Risk | State/rules may become scattered | Excessive classes can overcomplicate simple handlers | Avoid both duplicated procedural rules and unnecessary class hierarchies |

### Example
A booking service can have a handler such as `create_booking(request, user_context)`. It delegates to small functions such as `validate_recommendation(...)` and `check_slot_availability(...)`, then persists a booking. A `Booking` domain object or equivalent validation layer can ensure that only valid status transitions are allowed.

The implementation language and existing code should determine whether these concepts are represented by classes, modules, or plain functions; the design goal is clear responsibility and testability.

## 6. Top-down vs bottom-up design

### Top-down approach
Start with major system capabilities, then decompose them:
1. Smart Hospital Management System
2. Authentication, patient records, test booking, reminders and portals
3. Service APIs, data models and workflows
4. Functions, validation rules, database operations and UI components

**Advantages:** aligns implementation with requirements, keeps scope visible, and helps establish service boundaries early.

### Bottom-up approach
Start by implementing and testing reusable building blocks:
1. Database access and data models
2. Authentication/authorization helpers
3. Notification adapter
4. Booking validation and scheduling utilities
5. Service handlers and portal workflows

**Advantages:** validates technical foundations early and encourages reuse. **Risk:** building utilities without clear requirements can create unnecessary abstractions.

### Recommendation
Use a hybrid approach. Define use cases and module boundaries top-down, then implement and test low-level components bottom-up. Integrate incrementally with automated tests.

## 7. Quality attributes and design decisions

| Quality attribute | Design decision | Verification |
|---|---|---|
| Security | Authenticate requests and authorize by role and patient ownership at the API | Test unauthorized and cross-patient access |
| Reliability | Persist bookings before dispatching notifications; retry transient notification failures | Simulate notification failure |
| Maintainability | Separate records, booking and reminder responsibilities | Review module boundaries and run unit tests |
| Scalability | Keep service handlers stateless where practical and scale independently | Load test APIs and inspect service metrics |
| Data integrity | Validate slot availability and prevent duplicate/conflicting bookings transactionally | Concurrent booking tests |
| Privacy | Restrict medical records and document access to authorized users | Access-control and audit tests |

## 8. Testing strategy
- **Unit tests:** booking validation, status transitions, reminder schedule calculations and permission checks.
- **Integration tests:** API authorization, service-to-database operations and notification event handling.
- **Workflow tests:** successful booking, invalid login, no recommended tests, unavailable slot and notification failure.
- **Security tests:** patient A cannot read or book against patient B's records; patient accounts cannot call staff/admin operations.
- **Operational tests:** retries, logging, monitoring and recovery from dependency failure.

## 9. Conclusion
A modular, service-oriented design is appropriate for the Smart Hospital Management System because records, booking and reminders have distinct business responsibilities. High cohesion, low coupling, backend-enforced authorization, stable API contracts and automated tests should improve maintainability and security. A hybrid top-down/bottom-up process is recommended to preserve alignment with requirements while validating reusable components early.

## 10. Scope and assumptions
This is a design analysis based on the architecture and feature descriptions in the repository README. It is not a code audit and does not claim that AWS resources, APIs, or integrations are already deployed. Update module names, endpoint details and test evidence to match the implementation before final submission.
