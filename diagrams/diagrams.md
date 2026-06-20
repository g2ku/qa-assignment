# Booking Process Diagrams

## 1. Booking Flow (BPMN-style) — Happy Path + Exceptions

```mermaid
flowchart TD
    Start([User starts booking]) --> SelectRoom[Select room and dates]
    SelectRoom --> CheckAvail{Room available?}
    CheckAvail -- No --> RoomUnavailable[Show: Room not available]
    RoomUnavailable --> End1([End])
    CheckAvail -- Yes --> FillDetails[Fill guest details and payment]
    FillDetails --> Payment{Payment successful?}
    Payment -- No --> PaymentFailed[Show: Payment failed]
    PaymentFailed --> End2([End])
    Payment -- Yes --> CreateBooking[Create booking, status = pending]
    CreateBooking --> Confirm[Confirm booking, status = confirmed]
    Confirm --> SendEmail[Send confirmation email]
    SendEmail --> End3([End: Booking confirmed])

    User2([User cancels booking]) --> FindBooking[Find existing booking]
    FindBooking --> CancelCheck{Booking status?}
    CancelCheck -- confirmed/pending --> CancelBooking[Set status = cancelled]
    CancelBooking --> NotifyCancel[Send cancellation email]
    NotifyCancel --> End4([End: Booking cancelled])
    CancelCheck -- already cancelled --> AlreadyCancelled[Show: Already cancelled]
    AlreadyCancelled --> End5([End])
```

**Happy path:** Select room → fill details → pay → booking created (pending) → confirmed → email sent.

**Exception 1:** Room unavailable — process stops, user is notified before any data is created.

**Exception 2:** Payment fails — booking is not created, status never enters the system; this prevents orphaned bookings.

---

## 2. Sequence Diagram — Booking Creation

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant A as API
    participant D as Database
    participant E as Email Service

    U->>F: Submit booking form (room, dates, guest info)
    F->>A: POST /booking (booking data)
    A->>D: INSERT INTO Bookings (status = pending)
    D-->>A: booking_id
    A->>D: UPDATE Bookings SET status = confirmed
    D-->>A: OK
    A->>E: Send confirmation email request
    E-->>A: Email queued
    A-->>F: 200 OK + booking confirmation
    F-->>U: Show "Booking confirmed" screen
```

---

## 3. State Transition — Booking Object

```mermaid
stateDiagram-v2
    [*] --> pending: Booking created (payment authorized)
    pending --> confirmed: Payment captured successfully
    pending --> cancelled: User cancels before confirmation / payment fails
    confirmed --> cancelled: User or admin cancels
    cancelled --> [*]
    confirmed --> [*]: Stay completed (checkout passed)
```

### Transitions that MUST be covered by tests:

- **[*] → pending** — verify a new booking is created with the correct default status, and that required fields (user_id, room_id, checkin, checkout) are validated.
- **pending → confirmed** — verify that confirmation only happens after successful payment, and that `status` field updates correctly in the database.
- **pending → cancelled** — verify cancellation is possible before confirmation and that the room becomes available again for other users.
- **confirmed → cancelled** — verify cancellation of an already-confirmed booking is possible, the room is freed, and a cancellation email is triggered.
- **cancelled → (any)** — verify the system does NOT allow re-confirming a cancelled booking (this is the most likely place for bugs — boundary of an invalid transition).

This matters for QA because invalid transitions (e.g. confirming an already-cancelled booking, or double-confirming) are a very common source of real bugs in booking systems — explicit negative tests should target exactly these transitions.
