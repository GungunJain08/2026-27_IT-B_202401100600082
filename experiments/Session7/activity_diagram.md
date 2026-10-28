# Session 7 — UML Activity Diagram

## Workflow
This activity diagram describes the diagnostic test booking workflow from sign-in through booking confirmation or recovery from an invalid/unavailable slot.

```mermaid
flowchart TD
    A([Start]) --> B[Open patient portal]
    B --> C[Enter login credentials]
    C --> D{Credentials valid?}
    D -- No --> E[Display login error]
    E --> C
    D -- Yes --> F[Load patient dashboard]
    F --> G[Fetch staff-recommended tests]
    G --> H{Recommended tests available?}
    H -- No --> I[Display no recommended tests]
    I --> Z([End])
    H -- Yes --> J[Display recommended tests]
    J --> K[Patient selects test and slot]
    K --> L{Test belongs to patient and slot available?}
    L -- No --> M[Display validation error]
    M --> N{Try another slot?}
    N -- Yes --> K
    N -- No --> Z
    L -- Yes --> O[Create booking record in database]
    O --> P[Set booking status to Booked]
    P --> Q[Trigger confirmation notification]
    Q --> R[Display booking confirmation]
    R --> Z([End])
```

## Activity notes
- Invalid credentials return the patient to the login step.
- Only tests recommended for the authenticated patient may be booked.
- An unavailable slot does not create a booking; the patient may retry.
- The booking is saved before the confirmation is sent.
- Notification delivery depends on the configured SNS/SES integration.
