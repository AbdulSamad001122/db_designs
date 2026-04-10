# Smart Elevator Control System — ERD Explanation

## Overview

This document explains the relationships between all entities in the database design. It focuses on:

* relationship types (1:1, 1:N, M:N)
* why each relationship exists
* how it supports real-world system behavior

---

# 1. Buildings → Floors

### Relationship

**One-to-Many (1:N)**
One building → Many floors

### Implementation

* `floors.building_id → buildings.id`

### Reason

A building naturally contains multiple floors.
Floors cannot exist independently without a building.

### Why this matters

* Helps answer: *Which floors belong to which building?*
* Prevents floors from floating without context

---

# 2. Buildings → Elevator Shafts

### Relationship

**One-to-Many (1:N)**
One building → Many shafts

### Implementation

* `elevator_shafts.building_id → buildings.id`

### Reason

Each building contains multiple shafts where elevators operate.

### Why this matters

* Models real infrastructure
* Separates physical structure (shaft) from machine (elevator)

---

# 3. Elevator Shafts → Elevators

### Relationship

**One-to-One (1:1)**

### Implementation

* `elevators.shaft_id → elevator_shafts.id`

### Reason

Each shaft holds exactly one elevator.

### Why this matters

* Prevents multiple elevators being incorrectly assigned to the same shaft
* Keeps system physically accurate

---

# 4. Buildings → Elevators

### Relationship

**One-to-Many (1:N)**

### Implementation

* `elevators.building_id → buildings.id`

### Reason

A building contains multiple elevators.

### Why this matters

* Enables queries like:

  * *How many elevators are in a building?*
  * *Which elevators belong to a building?*

---

# 5. Elevators ↔ Floors

### Relationship

**Many-to-Many (M:N)**

### Implementation

* Junction table: `elevator_floor_service`

  * `elevator_id → elevators.id`
  * `floor_id → floors.id`

### Reason

* One elevator serves multiple floors
* One floor can be served by multiple elevators

### Why this matters

* Critical for real-world elevator systems
* Enables:

  * zoning logic
  * restricted floor access
  * multiple elevator coverage per floor

---

# 6. Elevators → Elevator Status

### Relationship

**One-to-One (1:1)**

### Implementation

* `elevator_status.elevator_id → elevators.id` (unique)

### Reason

Status is dynamic and frequently changing.

### Why separate it?

* Avoids mixing real-time data with static configuration
* Keeps elevator table clean

### Why this matters

* Supports:

  * real-time monitoring
  * dashboards
  * system decisions (dispatch logic)

---

# 7. Users → Floor Requests

### Relationship

**One-to-Many (1:N)**

### Implementation

* `floor_requests.user_id → users.id`

### Reason

A user can generate multiple requests.

### Why this matters

* Enables user-based analytics
* Optional but adds realism

---

# 8. Buildings → Floor Requests

### Relationship

**One-to-Many (1:N)**

### Implementation

* `floor_requests.building_id → buildings.id`

### Reason

Requests are generated within a building context.

### Why this matters

* Helps filter requests per building
* Useful for multi-building platform queries

---

# 9. Floors → Floor Requests

### Relationship

**One-to-Many (1:N)**

### Implementation

* `origin_floor_id → floors.id`
* `destination_floor_id → floors.id`

### Reason

Each request has:

* origin floor
* destination floor

### Why this matters

* Core to elevator logic
* Enables:

  * direction calculation
  * route optimization

---

# 10. Floor Requests → Ride Assignments

### Relationship

**One-to-One (1:1)**

### Implementation

* `ride_assignments.request_id → floor_requests.id`

### Reason

Each request results in exactly one assignment.

### Why this matters

* Separates:

  * request creation
  * elevator allocation

* Allows tracking:

  * assignment success/failure
  * reassignment scenarios (if extended)

---

# 11. Elevators → Ride Assignments

### Relationship

**One-to-Many (1:N)**

### Implementation

* `ride_assignments.elevator_id → elevators.id`

### Reason

One elevator handles many requests.

### Why this matters

* Enables:

  * load tracking
  * performance analysis
  * “which elevator handled most requests?”

---

# 12. Ride Assignments → Ride Logs

### Relationship

**One-to-One (1:1)**

### Implementation

* `ride_logs.assignment_id → ride_assignments.id`

### Reason

Each assignment results in one completed ride log.

### Why this matters

* Separates:

  * allocation phase
  * execution phase

---

# 13. Elevators → Ride Logs

### Relationship

**One-to-Many (1:N)**

### Implementation

* `ride_logs.elevator_id → elevators.id`

### Reason

An elevator completes many rides.

### Why this matters

* Enables:

  * daily ride count
  * usage analytics
  * performance insights

---

# 14. Floors → Ride Logs

### Relationship

**One-to-Many (1:N)**

### Implementation

* `origin_floor_id → floors.id`
* `destination_floor_id → floors.id`

### Reason

Each ride has:

* starting floor
* ending floor

### Why this matters

* Supports:

  * traffic analysis
  * peak floor detection

---

# 15. Elevators → Maintenance Records

### Relationship

**One-to-Many (1:N)**

### Implementation

* `maintenance_records.elevator_id → elevators.id`

### Reason

An elevator can have multiple maintenance events over time.

### Why this matters

* Preserves history (no overwriting)
* Enables:

  * maintenance tracking
  * downtime analysis

---

**End of Document**
