
# **Backend Requirement Specifications**

## 1. User Authentication

**Objective:** Securely handle user registration, login, and authentication using JWT.

### **API Endpoints**

* `POST /api/auth/register`
* `POST /api/auth/login`
* `GET /api/auth/profile` (JWT required)

### **Input/Output Specs**

* **Register (POST /api/auth/register)**
  **Input (JSON):**

  ```json
  {
    "email": "user@example.com",
    "password": "SecurePass123",
    "role": "guest"
  }
  ```

  **Output (Success 201):**

  ```json
  {
    "message": "User registered successfully",
    "userId": "uuid",
    "token": "jwt_token_here"
  }
  ```

  **Output (Error 400):**

  ```json
  {
    "error": "Email already exists"
  }
  ```

* **Login (POST /api/auth/login)**
  **Input (JSON):**

  ```json
  {
    "email": "user@example.com",
    "password": "SecurePass123"
  }
  ```

  **Output (Success 200):**

  ```json
  {
    "message": "Login successful",
    "token": "jwt_token_here"
  }
  ```

### **Validation Rules**

* Password ≥ 8 chars, must include letters and numbers.
* Valid email format.
* Role must be either `guest` or `host`.

### **Performance Criteria**

* Login requests should respond within **<200ms** under normal load.
* JWT must expire after **24 hours** (refresh option available).

---

## 2. Property Management

**Objective:** Allow hosts to create, update, delete, and fetch property listings.

### **API Endpoints**

* `POST /api/properties` (Host only)
* `PUT /api/properties/{id}` (Host only)
* `DELETE /api/properties/{id}` (Host only)
* `GET /api/properties/{id}` (Public)
* `GET /api/properties` (Search/filter, public)

### **Input/Output Specs**

* **Add Property (POST /api/properties)**
  **Input (JSON):**

  ```json
  {
    "title": "Cozy Apartment",
    "description": "A nice apartment downtown",
    "location": "New York",
    "price": 120,
    "amenities": ["WiFi", "Air Conditioning"],
    "maxGuests": 4,
    "availability": ["2025-09-20", "2025-09-30"]
  }
  ```

  **Output (201):**

  ```json
  {
    "message": "Property created successfully",
    "propertyId": "uuid"
  }
  ```

### **Validation Rules**

* Title ≤ 100 characters.
* Price must be positive integer/float.
* Availability dates must follow ISO format and cannot overlap with existing bookings.

### **Performance Criteria**

* Property search should return paginated results in **<500ms** for datasets up to **1M listings**.
* Images uploaded should be stored in cloud storage (e.g., S3/Cloudinary).

---

## 3. Booking System

**Objective:** Handle property reservations, cancellations, and status tracking.

### **API Endpoints**

* `POST /api/bookings` (Guest only)
* `GET /api/bookings/{id}` (Guest/Host/Admin)
* `DELETE /api/bookings/{id}` (Guest/Host with policy)
* `GET /api/bookings/user/{id}` (Guest only, view booking history)

### **Input/Output Specs**

* **Create Booking (POST /api/bookings)**
  **Input (JSON):**

  ```json
  {
    "propertyId": "uuid",
    "guestId": "uuid",
    "checkIn": "2025-09-25",
    "checkOut": "2025-09-28"
  }
  ```

  **Output (201):**

  ```json
  {
    "message": "Booking confirmed",
    "bookingId": "uuid",
    "status": "confirmed"
  }
  ```

### **Validation Rules**

* Check-in date < Check-out date.
* Dates must not overlap with existing bookings.
* Guest cannot book their own property.

### **Performance Criteria**

* Prevent **double-booking race conditions** with database transaction locks.
* Booking confirmation should occur within **<300ms**.
