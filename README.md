# Django Models Documentation

## Table of Contents
1. [User Model](#user-model)
2. [Profile Models](#profile-models)
   - [Sacco](#sacco)
   - [Customer](#customer)
   - [System_Admin](#system_admin)
   - [Station_Admin](#station_admin)
   - [VehicleManager](#vehiclemanager)
   - [Owner](#owner)
3. [Location Models](#location-models)
   - [Region](#region)
   - [Station](#station)
4. [Route Models](#route-models)
   - [Route](#route)
   - [RouteStop](#routestop)
   - [RoutePricing](#routepricing)
5. [Vehicle Models](#vehicle-models)
   - [Vehicle](#vehicle)
   - [Ownership](#ownership)
   - [SaccoOwner](#saccoowner)
   - [Route_Change_Request](#route_change_request)
6. [Queue System](#queue-system)
   - [Queue](#queue)
7. [Notification System](#notification-system)
   - [Notification](#notification)
   - [Announcement](#announcement)
8. [Payment System](#payment-system)
   - [Payment](#payment)
   - [PaymentDetail](#paymentdetail)
9. [Booking System](#booking-system)
   - [Boooking](#boooking)
10. [Feedback System](#feedback-system)
    - [Complaint](#complaint)
    - [SaccoRating](#saccorating)
    - [ReportSacco](#reportsacco)
11. [Content Management](#content-management)
    - [Blogspot](#blogspot)

---

## User Model

### `User` (AbstractBaseUser, PermissionsMixin)
- **Fields**:
  - Basic info: `firstname`, `lastname`, `username`, `email`
  - Authentication: `password`, `otp`, `otp_expiration`
  - Role: `role` (choices: CUSTOMER, STATION_ADMIN, SACCO, etc.)
  - Payment info: `payment_status`, `payment_date`, `plan`, `invoice`, `api_ref`
  - Status flags: `is_staff`, `is_superuser`, `is_active`
  - Timestamps: `created_at`, `updated_at`

- **Relationships**:
  - One-to-one with all profile models (Sacco, Customer, etc.) through signals
  - ForeignKey in: Notification, Payment, Booking, etc.

- **Special Features**:
  - Custom user manager (`CustomUserManager`)
  - Automatic profile creation via `post_save` signal

---

## Profile Models

### `Sacco`
- **Fields**: National ID, contact, description, location (JSON), logo, etc.
- **Relationships**:
  - One-to-one with User
  - Many-to-many with Region (operating_regions)
  - ForeignKey in: Station, Vehicle, Route, etc.

### `Customer`
- **Fields**: National ID, contact, profile_pic
- **Relationships**: One-to-one with User

### `System_Admin`
- **Fields**: National ID, contact
- **Relationships**: One-to-one with User

### `Station_Admin`
- **Fields**: National ID, contact, profile_pic, current_location
- **Relationships**:
  - One-to-one with User
  - ForeignKey to Station

### `VehicleManager`
- **Fields**: National ID, contact, role_type, license info
- **Relationships**:
  - One-to-one with User
  - ForeignKey to Vehicle

### `Owner`
- **Fields**: National ID, contact, profile_pic
- **Relationships**:
  - One-to-one with User
  - Many-to-many with Sacco (through SaccoOwner)
  - Many-to-many with Vehicle (through Ownership)

---

## Location Models

### `Region`
- **Fields**: name, country, county, sub_county
- **Relationships**: Many-to-many with Sacco

### `Station`
- **Fields**: name, location (JSON), county info, profile_pic
- **Relationships**:
  - ForeignKey to Sacco
  - ForeignKey in: Route (origin/destination), Queue, etc.

---

## Route Models

### `Route`
- **Fields**: name, distance, status
- **Relationships**:
  - ForeignKeys to Station (origin/destination)
  - ForeignKey to Sacco
  - Many-to-many with Vehicle (through VehicleRouteAssignment)

### `RouteStop`
- **Fields**: name, flags (is_start_point, is_end_point)
- **Relationships**: ForeignKey to Route

### `RoutePricing`
- **Fields**: price, period type
- **Relationships**: ForeignKeys to Route and RouteStops

---

## Vehicle Models

### `Vehicle`
- **Fields**: plate_number, vehicle_type, make/model/year, capacity, etc.
- **Relationships**:
  - ForeignKey to Sacco
  - Many-to-many with Route
  - Many-to-many with Owner (through Ownership)

### `Ownership`
- **Fields**: ownership_percentage, co_owner_role
- **Relationships**: ForeignKeys to Owner and Vehicle

### `Route_Change_Request`
- **Fields**: status, remarks
- **Relationships**: ForeignKeys to Route (current/requested), Vehicle, User

---

## Queue System

### `Queue`
- **Fields**: position, status, timestamps, capacity info
- **Relationships**: ForeignKeys to Vehicle, Station, Route
- **Special Features**:
  - Custom manager (`QueueManager`)
  - Position management methods
  - Automated daily reset

---

## Notification System

### `Notification`
- **Fields**: message, is_read
- **Relationships**: ForeignKeys to User, Sacco, Station

### `Announcement`
- **Fields**: title, message, is_active
- **Relationships**: ForeignKeys to Sacco, Station

---

## Payment System

### `Payment`
- **Fields**: transaction details, amount, status
- **Relationships**: ForeignKey to User
- **Special Features**: post_save signal for status handling

### `PaymentDetail`
- **Fields**: payment credentials
- **Relationships**: ForeignKey to Sacco

---

## Booking System

### `Boooking`
- **Fields**: status
- **Relationships**: ForeignKeys to User, Vehicle, Route, Payment

---

## Feedback System

### `Complaint`
- **Fields**: message
- **Relationships**: ForeignKeys to Sacco, Vehicle, Station

### `SaccoRating`
- **Fields**: rating, review
- **Relationships**: ForeignKey to Sacco

### `ReportSacco`
- **Fields**: report content and type
- **Relationships**: ForeignKey to Sacco

---

## Content Management

### `Blogspot`
- **Fields**: title, content (CKEditor), image
- **Relationships**: ForeignKey to User (author)

---

## Entity Relationship Diagram (Summary)

```
User
├── One-to-One ── Sacco
├── One-to-One ── Customer
├── One-to-One ── System_Admin
├── One-to-One ── Station_Admin
├── One-to-One ── VehicleManager
└── One-to-One ── Owner

Sacco
├── Many-to-Many ── Region (operating_regions)
├── One-to-Many ── Station
├── One-to-Many ── Vehicle
└── One-to-Many ── Route

Vehicle
├── Many-to-Many ── Route (through VehicleRouteAssignment)
└── Many-to-Many ── Owner (through Ownership)

Route
├── One-to-Many ── RouteStop
└── One-to-Many ── RoutePricing

Station
├── One-to-Many ── Queue
└── One-to-Many ── Station_Admin
```
