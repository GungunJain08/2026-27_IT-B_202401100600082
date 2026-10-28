# Session 7 — UML Sequence Diagram

## Scenario
A patient signs in, views diagnostic tests recommended by hospital staff, selects an available slot, and books the test. The system validates the request, saves the booking, and sends a confirmation.

## Participants
- Patient
- Patient Portal
- Amazon Cognito
- API Gateway
- Test Booking Service (AWS Lambda)
- Amazon RDS
- Notification Service (SNS/SES)

## Sequence diagram

```mermaid
sequenceDiagram
    actor Patient
    participant Portal as Patient Portal
    participant Auth as Amazon Cognito
    participant API as API Gateway
    participant Booking as Test Booking Service
    participant DB as Amazon RDS
    participant Notify as SNS/SES Notification Service

    Patient->>Portal: Sign in
    Portal->>Auth: Authenticate credentials
    Auth-->>Portal: Access token / authentication result
    Portal-->>Patient: Show patient dashboard

    Patient->>Portal: View recommended tests
    Portal->>API: GET /patients/me/recommended-tests + token
    API->>Booking: Invoke authorized booking service
    Booking->>DB: Fetch tests for authenticated patient
    DB-->>Booking: Recommended tests and status
    Booking-->>API: Test list
    API-->>Portal: 200 OK + test list
    Portal-->>Patient: Display recommended tests

    Patient->>Portal: Select test and time slot
    Portal->>API: POST /bookings {testId, slotId} + token
    API->>Booking: Validate request and patient identity
    Booking->>DB: Check recommendation and slot availability
    DB-->>Booking: Validation result

    alt Test is recommended and slot is available
        Booking->>DB: Create booking with status = Booked
        DB-->>Booking: Booking ID and saved record
        Booking->>Notify: Send booking confirmation event
        Notify-->>Patient: Confirmation by configured channel
        Booking-->>API: Booking created
        API-->>Portal: 201 Created + booking details
        Portal-->>Patient: Show booking confirmation
    else Test not recommended or slot unavailable
        Booking-->>API: Validation error
        API-->>Portal: 400/409 error response
        Portal-->>Patient: Explain issue and allow another slot
    end
```

## Main message flow
1. The patient authenticates and receives a token.
2. The portal requests only the current patient's recommended tests.
3. The API authorizer validates the token before the booking service is invoked.
4. The booking service validates recommendation ownership and slot availability.
5. A valid booking is persisted before a confirmation is sent.
6. If validation fails, no booking is created and the patient can choose another slot.

## Design note
The diagram models the architecture described in the project README. Actual endpoint names and response codes should be aligned with the implementation when the backend API is finalized.
