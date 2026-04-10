

## The World

Rahul is the IT head at LiftGrid Systems. His job is to onboard buildings onto the platform and make sure the whole system runs. Today he's onboarding **Infinity Tower**, a 40 floor corporate building in Andheri, Mumbai.

---

## `buildings`

First thing Rahul does is register the building itself.

**Purpose:** Master record. Every other table in the system ultimately traces back to a building. Without this, you have elevators and floors floating with no context of where they belong.

**Important fields:**
- `id` — every other table references this. The anchor of the whole system.
- `total_floors` — tells the system the maximum floor count. Useful for validation. You can't register a floor numbered 50 in a 40 floor building.
- `city / country` — LiftGrid operates across India. Chennai buildings, Delhi buildings, Pune buildings, all in one platform. These fields let Rahul filter by region on the dashboard.

Rahul enters: name = "Infinity Tower", city = "Mumbai", total_floors = 40. One row. Building is live.

---

## `floors`

Now Rahul registers every floor of Infinity Tower.

**Purpose:** Floors are first class entities in this system. Requests come FROM floors. Elevators are assigned TO floors. Rides go FROM one floor TO another. If floors are just numbers scattered across tables, you lose the ability to attach metadata to them, like labeling floor 1 as "Lobby" or floor 40 as "Terrace", or deactivating a floor under renovation.

**Important fields:**
- `building_id` — ties this floor to Infinity Tower specifically.
- `floor_number` — the actual number. 1, 2, 3... 40.
- `label` — human readable name. Floor 0 = "Basement", Floor 1 = "Lobby", Floor 22 = "Executive Wing". Not every floor is just a number in real buildings.
- `is_active` — Floor 17 is under renovation this month. Rahul flips this to false. System stops accepting requests from or to Floor 17 without deleting any historical data.
- Unique index on `(building_id, floor_number)` — you can't have two Floor 15s in the same building.

40 rows created. Infinity Tower is fully mapped floor by floor.

---

## `elevator_shafts`

Infinity Tower has 8 physical shafts built into the structure.

**Purpose:** The shaft is the civil engineering reality. It's a hole in the building. It exists before any elevator is installed. It has physical boundaries, how deep it goes, how high it reaches. This is infrastructure data, not operational data. It never changes unless the building is physically modified.

**Important fields:**
- `building_id` — which building this shaft belongs to.
- `shaft_code` — "Shaft-A", "Shaft-B" etc. Human readable identifier for maintenance teams.
- `min_floor` — the lowest floor this shaft physically reaches. Shaft A goes from Floor 1. Shaft E goes from Basement 2.
- `max_floor` — the highest floor this shaft reaches. Shafts 1-4 go up to Floor 20. Shafts 5-8 go up to Floor 40.

Unique index on `(building_id, shaft_code)` — two shafts in the same building can't have the same code.

8 rows created. The physical skeleton of the elevator system is registered.

---

## `elevators`

Inside each shaft, one elevator is installed.

**Purpose:** The elevator is the operational unit. It moves, it carries passengers, it gets assigned rides, it goes under maintenance. Separating it from the shaft means the shaft record stays as permanent civil infrastructure while the elevator record tracks operational state.

**Important fields:**
- `shaft_id` — one-to-one. This elevator lives in exactly this shaft. No two elevators share a shaft.
- `elevator_code` — "ELV-A1", "ELV-B2". Operations team uses this code when reporting issues or checking status.
- `capacity` — maximum persons this elevator can carry. Important for future load balancing logic.
- `current_floor` — where is this elevator right now. Stored here as a quick lookup field so the assignment algorithm doesn't have to scan `elevator_status` every time it needs to find the nearest elevator.
- `is_active` — master kill switch. If false, this elevator is completely out of service regardless of what `elevator_status` says.
- `commissioned_at` — when was this elevator officially put into service. Used for warranty tracking and lifespan calculations.

8 rows created. Each elevator linked to its shaft.

---

## `elevator_floor_service`

Now Rahul configures which floors each elevator actually stops at.

**Purpose:** The shaft defines the physical hole in the building. This table defines the programmed stop list. Elevator 3's shaft runs from Floor 1 to Floor 20, but Floor 7 is a restricted server room. Nobody should be able to ride to Floor 7. So Elevator 3 has junction rows for floors 1, 2, 3, 4, 5, 6, 8, 9... 20. Floor 7 is simply absent from its stop list.

This is a many-to-many junction. One elevator serves many floors. One floor is served by many elevators.

**Important fields:**
- `elevator_id` — which elevator
- `floor_id` — which floor it stops at
- Unique index on `(elevator_id, floor_id)` — same elevator can't be registered to the same floor twice.

This table answers two critical system questions:
- *Which elevators can I dispatch to Floor 15?* — query by `floor_id`
- *Which floors does Elevator 3 serve?* — query by `elevator_id`

Without this table, the assignment algorithm has no way to know which elevators are even eligible to respond to a request.

---

## `elevator_status`

The elevator is installed. Now the system needs to track what it's doing right now.

**Purpose:** This is the real-time state of each elevator. Completely separate from the elevator's configuration. An elevator's capacity, shaft, code, those never change mid-operation. But its status changes dozens of times per hour. Mixing dynamic state into the `elevators` table would make the configuration row noisy and hard to audit.

One row per elevator. Always. The unique constraint on `elevator_id` enforces this. It's not a history log, it's a live snapshot.

**Important fields:**
- `elevator_id` — which elevator this status belongs to. Unique, one active status row per elevator always.
- `status` — the enum: `idle`, `moving_up`, `moving_down`, `maintenance`, `disabled`. The assignment algorithm checks this before dispatching. It will never send a ride to an elevator in `maintenance` or `disabled` state.
- `current_floor` — where the elevator physically is right now. Updates every time the elevator moves a floor. The assignment algorithm uses this to find the nearest idle elevator to a request.
- `direction` — is it going up, down, or stationary. Used to optimize assignments. If an elevator is already moving up past Floor 10 and someone on Floor 12 wants to go up, that elevator is a strong candidate.
- `updated_at` — timestamp of last status change. If this hasn't updated in 5 minutes and status is `moving`, something is wrong. Monitoring alerts use this field.

8 rows created at setup, all defaulting to `idle` at Floor 0.

---

## `users`

**Priya** works at a tech firm on Floor 28. She has the Infinity Tower building app on her phone.

**Purpose:** Not every request has a user, hall button presses are anonymous. But when a user is logged in via the building app, the system can attach the request to them. This enables personalization, access control, and analytics per person. Which floor does Priya usually go to? Is she authorized to access Floor 35?

`user_id` on `floor_requests` is nullable, anonymous requests are perfectly valid.

**Important fields:**
- `id` — referenced by `floor_requests`
- `email / phone` — contact and identity. Unique email enforced.
- `created_at` — when did this person register on the platform.

---

## `floor_requests`

Monday 9 AM. Priya walks to the lobby on Floor 1 and presses UP.

**Purpose:** Every single button press in the building creates a row here. This is the demand side of the system. The request exists the moment someone presses a button, before any elevator is assigned, before any ride happens. Keeping requests as their own entity means the system can track pending demand, measure wait times, and analyze patterns independently of what elevators did.

**Important fields:**
- `origin_floor_id` — Floor 1. Where Priya is standing.
- `destination_floor_id` — NULL right now. She'll press 28 inside the cabin. This gets filled in after boarding. Nullable by design because hall buttons don't ask for destination.
- `user_id` — Priya's ID since she's on the app. NULL for anonymous presses.
- `direction` — UP. The system needs this immediately to dispatch correctly. You don't send a downward-traveling elevator to someone going up.
- `status` — starts as `pending`. Moves to `assigned` when an elevator is dispatched. Moves to `completed` when the ride finishes. `cancelled` if Priya takes the stairs.
- `requested_at` — exact timestamp. Used to calculate how long Priya waited. SLA monitoring.

One row created. Status: pending.

---

## `ride_assignments`

The system scans `elevator_floor_service`, finds elevators serving Floor 1. Checks `elevator_status`, Elevator 2 is idle at Floor 3, closest to Floor 1. Dispatches it.

**Purpose:** This is the decision record. The moment the system decides which elevator handles which request. Separate from the request itself because the assignment has its own lifecycle, it can fail, it can be in progress, it completes. Also separating it means one request maps to exactly one assignment. The one-to-one ref enforces this. You can't double-assign a request.

**Important fields:**
- `request_id` — which request is being fulfilled. One-to-one ref, same request can't be assigned twice.
- `elevator_id` — which elevator was chosen.
- `assigned_at` — when the decision was made. Gap between `floor_requests.requested_at` and this field = algorithm response time.
- `assignment_status` — `in_progress` while the elevator is en route and during the ride. `completed` when Priya exits. `failed` if the elevator broke down mid-assignment.

One row created. Elevator 2 is now committed to Priya's request.

---

## `ride_logs`

Elevator 2 arrives at Floor 1. Priya boards. She presses 28. Elevator moves. Doors open at Floor 28. Priya walks out.

**Purpose:** The permanent historical record of every completed trip. This is the analytics engine. Everything about what actually happened, which floors, how many passengers, how long it took. This data never gets updated after the ride completes. It's an immutable log. Management queries this table for performance reports, capacity planning, and billing.

Notice: this table records what actually happened, not what was requested. Priya requested from Floor 1 with no destination. The ride log records the actual trip, Floor 1 to Floor 28.

**Important fields:**
- `assignment_id` — one-to-one. Each assignment produces exactly one ride log entry.
- `origin_floor_id` / `destination_floor_id` — the actual floors of the completed trip. Both non-nullable here unlike `floor_requests` because by the time the ride is logged, both are known.
- `passenger_count` — how many people were in the elevator during this trip. Used for load analysis.
- `started_at` / `completed_at` — exact timestamps. Gap between them is the ride duration.
- `duration_seconds` — yes, this is derivable from the two timestamps. Stored anyway for fast analytics queries without computing diffs. Intentional denormalization.

One row created. Trip from Floor 1 to Floor 28, 1 passenger, 41 seconds.

---

## `maintenance_records`

Two hours later, **Suresh** the building technician gets an alert. Elevator 5 is making a grinding sound.

**Purpose:** Maintenance history must never overwrite or mix with operational data. If you store maintenance info directly on the `elevators` table, you lose history, only the latest issue is visible. This table keeps every maintenance event as a separate row, timestamped, with full details. An elevator can have 50 maintenance records over its lifetime. None of them disturb the elevator's configuration data.

**Important fields:**
- `elevator_id` — which elevator is being serviced.
- `reported_by` — "Suresh". Accountability. Who flagged this issue.
- `issue_type` — "mechanical noise", "door fault", "sensor failure". Categorized for pattern analysis. If Elevator 5 has had "door fault" 6 times this year, that's a replacement signal.
- `status` — `scheduled` when logged. `in_progress` when Suresh starts working. `resolved` when fixed.
- `scheduled_at` / `started_at` / `resolved_at` — three separate timestamps tracking the full lifecycle of the maintenance event.
- `notes` — free text. Suresh writes "replaced left door motor, tested 10 cycles". Full context preserved.
- `created_at` — when the record was first created. Different from `scheduled_at`, the issue might be logged today but scheduled for repair next week.

When this row's status goes to `in_progress`, Elevator 5's row in `elevator_status` flips to `maintenance`. System stops dispatching rides to it. Other elevators absorb the load.

When `resolved_at` gets a timestamp, `elevator_status` flips back to `idle`. Elevator 5 re-enters rotation.

---

## End of day

Rahul opens the dashboard. Queries `ride_logs`, 2,847 rides completed today across 8 elevators. Elevator 6 handled 412 rides, highest load. Elevator 5 was offline from 11 AM to 3 PM, 0 rides in that window. Average ride duration: 38 seconds. Zero pending requests in `floor_requests`. Zero open assignments in `ride_assignments`.

Clean day. System worked exactly as designed.

---

Every table has one job. Requests don't know about elevators. Elevators don't know about rides. Maintenance doesn't touch history. Status is always live. Logs are permanent. That separation is the whole point of the design.