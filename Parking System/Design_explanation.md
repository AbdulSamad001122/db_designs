# 🚗 Comic-Con India Parking Management System

A database design for managing a multi-zone parking system during a large-scale event like Comic-Con India.
It handles vehicle entry, parking allocation, reserved access categories, sessions, and payments across multiple days.

---

## 🚀 Key Features

* Multi-day event support
* Vehicle and access category management (VIP, Staff, Cosplayer, etc.)
* Smart parking spot allocation based on rules
* Reserved and restricted parking areas
* Real-time parking session tracking
* Flexible fee calculation and payment tracking

---

## 🎭 System Story & Design Explanation

During the annual Comic-Con India event in Mumbai, thousands of visitors arrive daily from casual attendees to VIP guests, cosplayers with large props, exhibitors with merchandise, and staff managing operations.

To handle this complexity, the venue adopts a structured parking management system designed to efficiently track vehicles, allocate spots, enforce access rules, and manage payments.

---

## 🚘 Vehicle Entry

Rahul arrives on his bike, Priya comes in her EV car, and Arjun, a VIP guest, enters in an SUV.

Each vehicle is registered in the **`vehicle`** table, storing:

* license plate
* owner name
* contact

Every vehicle is linked to a **`vehicle_category`** such as Bike, Car, SUV, or EV.

This allows the system to:

* identify vehicle type
* apply correct parking rules
* calculate appropriate fees

---

## 🎟️ Special Access Handling

Arjun is a VIP guest, while Sneha is a cosplayer carrying large props.

Their permissions are stored in **`access_category`** with values like VIP, Cosplayer, Staff, and linked through **`vehicle_access`**.

This ensures:

* VIP vehicles get priority parking
* Cosplayers get larger or special spots
* Staff vehicles may get free parking

---

## 🅿️ Parking Structure

The venue is divided into multiple zones like:

* Zone A (General)
* Zone B (VIP)
* EV Zone

These are stored in **`parking_zone`**.

Each zone has multiple levels such as Ground, P1, P2, stored in **`parking_level`**, forming a hierarchy:

```
Zone → Level → Spot
```

---

## 📍 Parking Spots & Categories

Each parking space is defined in **`parking_spot`**, which includes:

* level reference
* spot category
* spot number
* reservation flag

Spot types are defined in **`spot_category`** such as:

* General
* VIP
* Exhibitor
* EV Charging

Some spots are reserved, and restrictions are enforced using **`spot_access_restriction`**, ensuring only certain access categories can use them.

Example:

* EV spots only for EV vehicles
* VIP spots restricted to VIP guests

---

## ⚙️ Vehicle-Spot Compatibility

Not every vehicle can park everywhere.

Rules like:

* SUVs cannot park in small spots
* EVs must use EV charging spots

are handled through **`vehicle_spot_category_rule`**.

This ensures:

* proper utilization of space
* no invalid allocations

---

## 🎫 Ticket Generation

When a vehicle enters:

* A record is created in **`parking_ticket`**
* A parking spot is assigned
* Entry time is recorded

The ticket acts as:

* entry proof
* reference for the parking session

---

## ⏱️ Parking Sessions

Every visit is tracked in **`parking_session`**, which stores:

* vehicle
* assigned spot
* ticket reference
* event day
* entry time
* exit time

Important capabilities:

* One vehicle can visit multiple times across days
* One spot can be reused across multiple sessions
* Active sessions where exit_time is null indicate currently parked vehicles

---

## 📅 Multi-Day Event Support

Since Comic-Con runs for multiple days, each day is tracked in **`event_day`**.

This allows:

* day-wise analytics
* separate tracking for each event day

---

## 💰 Fee Calculation & Payment

Parking charges are defined in **`parking_fee_rate`**, based on:

* vehicle category
* access category optional

When a vehicle exits:

* total duration is calculated
* applicable rate is applied
* payment is recorded in **`payment`**

The **`payment`** table stores:

* amount paid
* payment method such as Cash, Card, Online
* payment status such as Paid, Pending, Waived

---

## 🎯 Conclusion

From Rahul’s bike to Arjun’s VIP SUV, the system ensures every vehicle is handled efficiently from entry to exit while maintaining order, fairness, and smooth operations throughout the event.

