# 📘 Parking Management System – Table Relationships & Reasoning

This document explains the relationships between tables in the Comic-Con India Parking System database and the reason behind each relationship.

---

## 1. Vehicle → Vehicle Category

**Relationship:**  
Many vehicle → One vehicle_category

**Reason:**  
- Each vehicle must have a type (Bike, Car, SUV, EV) to enforce compatibility rules, calculate fees, and assign suitable parking spots.  
- Avoids hardcoding vehicle types in the vehicle table.

---

## 2. Vehicle ↔ Access Category (via vehicle_access)

**Relationship:**  
Many-to-many (vehicle ↔ access_category)

**Reason:**  
- A vehicle can have multiple access roles (VIP + Staff).  
- An access category can apply to many vehicles.  
- Ensures flexibility for assigning special permissions.

---

## 3. Vehicle → Parking Ticket

**Relationship:**  
One vehicle → Many parking_ticket

**Reason:**  
- A vehicle can enter the parking facility multiple times.  
- Each entry requires a new ticket for tracking.

---

## 4. Ticket → Parking Session

**Relationship:**  
One parking_ticket → One parking_session

**Reason:**  
- Ticket is the entry event; session tracks the full lifecycle including exit and payment.  
- Keeps entry and lifecycle separate for clarity.

---

## 5. Vehicle → Parking Session

**Relationship:**  
One vehicle → Many parking_session

**Reason:**  
- Vehicles can visit multiple times, even across multiple event days.  
- Sessions track entry/exit for each visit.

---

## 6. Parking Spot → Parking Session

**Relationship:**  
One parking_spot → Many parking_session

**Reason:**  
- Same parking spot can be used by different vehicles over time.  
- Tracks spot occupancy across sessions.

---

## 7. Spot ↔ Access Category (via spot_access_restriction)

**Relationship:**  
Many-to-many (parking_spot ↔ access_category)

**Reason:**  
- Restricts which access categories can use a particular spot.  
- Example: VIP-only, EV-only, or Cosplayer reserved spots.

---

## 8. Vehicle Category ↔ Spot Category (via vehicle_spot_category_rule)

**Relationship:**  
Many-to-many (vehicle_category ↔ spot_category)

**Reason:**  
- Ensures only compatible vehicles park in certain spots.  
- Example: SUVs cannot use small bike spots; EVs must use EV charging spots.

---

## 9. Parking Zone → Parking Level

**Relationship:**  
One parking_zone → Many parking_level

**Reason:**  
- Organizes parking hierarchically: Zone → Level → Spot.  
- Simplifies navigation and spot allocation.

---

## 10. Parking Level → Parking Spot

**Relationship:**  
One parking_level → Many parking_spot

**Reason:**  
- Each level contains multiple parking spots.  
- Supports spot lookup and availability tracking by level.

---

## 11. Parking Session → Event Day

**Relationship:**  
Many parking_session → One event_day

**Reason:**  
- Supports multi-day events.  
- Allows reporting and analytics per day.

---

## 12. Parking Fee Rate → Vehicle Category & Access Category

**Relationship:**  
Many parking_fee_rate → One vehicle_category  
Optional: Many parking_fee_rate → One access_category

**Reason:**  
- Pricing depends on vehicle type and access role.  
- Allows free parking for VIP/staff while charging normal vehicles.

---

## 13. Payment → Parking Session & Fee Rate

**Relationship:**  
One payment → One parking_session  
One payment → One parking_fee_rate

**Reason:**  
- Tracks payment for a specific session.  
- Indicates which rate was applied.  
- Separates financial data from session for clarity.

---

## 🔗 Summary Diagram of Relationships (Text Form)
vehicle_category ↔ vehicle ↔ parking_ticket → parking_session → payment
vehicle_category ↔ spot_category ↔ vehicle_spot_category_rule
parking_spot ↔ parking_level ↔ parking_zone
parking_spot ↔ spot_access_restriction ↔ access_category
vehicle ↔ vehicle_access ↔ access_category
parking_session → event_day
parking_fee_rate → vehicle_category & access_category
parking_session → payment → parking_fee_rate