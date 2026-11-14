Hotel Management System Design : 



1. System Overview :

This project specifies the structural architecture for a robust Hotel 
Reservation Management System, focusing on entity relationships, data 
constraints, and lifecycle workflows for modern hospitality 
operations. The core design establishes clear boundaries for managing 
guests, room inventories, staff responsibilities, and booking 
transactions while maintaining strict data consistency through unique
constraints and validation rules. 



2. Database Design & Schemas :

The database consists of 4 primary collections designed for a MongoDB/Mongoose environment:
Guests: (fullName, email, phone, nationalId)
Rooms: (roomNumber, type, pricePerNight, maxCapacity, status)
Employees: (fullName, role, phone)
Bookings: (guestId, employeeId, rooms: [{room, pricePerNight}], bookingDate, checkInDate, checkOutDate, status, totalAmount)

A. Guests Collection
Field Name | Data Type | Constraints | Classification

fullName | String | Required, Trimmed | Data
email | String | Required, Unique, Lowercase | Data / Problem-Solving (Unique constraint prevents duplicate registrations)
phoneNumber | String | Required | Data
nationalIdOrPassportNumber | String | Required, Unique | Data / Problem-Solving (Ensures legal uniqueness per guest)

B. Rooms Collection
Field Name | Data Type | Constraints | Classification

roomNumber | String | Required, Unique | Data / Problem-Solving (Prevents physical room duplication)
type | String | Required, Enum: ['Single', 'Double', 'Suite', 'Deluxe'] | Data
pricePerNight | Number | Required, Min: 0 | Data
maximumCapacity | Number | Required, Min: 1 | Data

C. Employees Collection
Field Name | Data Type | Constraints | Classification

fullName | String | Required, Trimmed | Data
role| String | Required, Enum: ['Receptionist', 'Manager', 'Supervisor'] | Data
phoneNumber | String | Required | Data

D. Bookings Collection
Field Name | Data Type | Constraints | Classification

guestId | ObjectId | Required, Ref: 'Guest' | Relationship
employeeId | ObjectId | Required, Ref: 'Employee' | Relationship
rooms | Array of ObjectIds | Required, Ref: 'Room' | Relationship
bookingDate | Date | Default: Date.now | Data
checkInDate | Date | Required | Data / Business Rule
checkOutDate | Date | Required | Data / Business Rule
status | String | Required, Enum: ['Pending', 'Confirmed', 'CheckedIn', 'CheckedOut', 'Cancelled'], Default: 'Pending' | Data
totalAmount | Number | Required, Min: 0 | Problem-Solving / Business Rule (Calculated via formula)



3. Mongoose Schema Snippets :

const mongoose = require('mongoose');

// 1. Guest Schema
const guestSchema = new mongoose.Schema({
  fullName: { type: String, required: true, trim: true },
  email: { type: String, required: true, unique: true, lowercase: true },
  phone: { type: String, required: true },
  nationalIdOrPassport: { type: String, required: true, unique: true }
}, { timestamps: true });

// 2. Room Schema
const roomSchema = new mongoose.Schema({
  roomNumber: { type: String, required: true, unique: true },
  type: { type: String, required: true, enum: ['Single', 'Double', 'Suite', 'Deluxe'] },
  pricePerNight: { type: Number, required: true, min: 0 },
  maxCapacity: { type: Number, required: true, min: 1 } ,
  status : { type : string, required: true, enum:[ 'Available'
  'Occupied ' , ' Maintenance' ] , default : 'Available' }
}, { timestamps: true });

// 3. Employee Schema
const employeeSchema = new mongoose.Schema({
  fullName: { type: String, required: true, trim: true },
  role: { type: String, required: true, enum: ['Receptionist', 'Manager', 'Supervisor'] },
  phone: { type: String, required: true }
}, { timestamps: true });

// 4. Booking Schema
const bookingSchema = new mongoose.Schema({
  guestId: { type: mongoose.Schema.Types.ObjectId, ref: 'Guest', required: true },
  employeeId: { type: mongoose.Schema.Types.ObjectId, ref: 'Employee', required: true },
  rooms: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Room', required: true }],
  bookingDate: { type: Date, default: Date.now },
  checkInDate: { type: Date, required: true },
  checkOutDate: { type: Date, required: true },
  status: { 
    type: String, 
    required: true, 
    enum: ['Pending', 'Confirmed', 'CheckedIn', 'CheckedOut', 'Cancelled'], 
    default: 'Pending' 
  },
  totalAmount: { type: Number, required: true, min: 0 }
  , { timestamps: true } });

module.exports = {
  Guest: mongoose.model('Guest', guestSchema),
  Room: mongoose.model('Room', roomSchema),
  Employee: mongoose.model('Employee', employeeSchema),
  Booking: mongoose.model('Booking', bookingSchema)
};



4. Relationships & ERD

Cardinality Breakdown
Hotel to Rooms (1 : N): One hotel instance contains multiple individual room entities.  

Guest to Bookings (1 : N): A single guest can place multiple bookings over time, but each booking belongs to exactly one guest.  

Booking to Rooms (N : M): A booking can include multiple rooms, and a room can appear in multiple historical bookings across non-overlapping date periods.  

Employee to Bookings (1 : N): Each booking is handled/monitored by one
employee, while an employee can manage multiple bookings.



5. Business Rules & Problem-Solving Logic

Stay Nights Calculation:
Nights = ceil((checkOutDate - checkInDate) / (1000 * 60 * 60 * 24))

Explanation: Calculates the total duration of stay in days by subtracting the check-in timestamp from the check-out timestamp and converting milliseconds into days.

Total Amount Calculation Formula:
Total Amount = Nights * sum(Price per Night of all booked rooms)

Explanation: Multiplies the total number of calculated stay nights by the cumulative sum of price-per-night values for all rooms included in the booking.

Double-Booking Conflict Prevention:
Condition: NewCheckIn < ExistingCheckOut AND NewCheckOut > ExistingCheckIn

Explanation: To prevent double-booking a physical room, the system queries active bookings (excluding Cancelled or CheckedOut statuses) for the requested rooms. If an overlapping date range is detected for any requested room, the booking transaction is rejected to ensure data integrity.



6. Predicted REST APIs

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| POST | /api/guests | Register a new guest profile |
| GET | /api/guests | Retrieve all guests list |
| POST | /api/rooms | Add a new room to inventory |
| GET | /api/rooms/available | Query available rooms by date range & type |
| POST | /api/employees | Register a new employee/staff member |
| POST | /api/bookings | Create a new multi-room booking with validation |
| GET | /api/bookings/:id | Get detailed booking info including populated guest, rooms, and employee |
| PATCH | /api/bookings/:id/status | Update booking status (e.g., Confirmed, CheckedIn, Cancelled) | 



7. Planned Technology Stack :

Node.js: JavaScript runtime environment chosen for its asynchronous event-driven architecture and unified language syntax across backend development.

Express.js: Minimalist and flexible web framework for building RESTful routing infrastructure.

MongoDB: NoSQL document database chosen for its flexible schema design, perfect for handling nested room references and scalable booking collections.

Mongoose: ODM library for MongoDB providing robust schema validation, casting, and query composition tools.

Nodemon: Development utility tool that automatically restarts the Node.js application whenever file changes in the directory are detected, boosting developer productivity.

Dotenv: Zero-dependency module that loads environment variables from a .env file into process.env, securing sensitive configuration data like database URIs.

Morgan: HTTP request logger middleware for Node.js, used to log incoming requests and responses to the console for easier debugging and monitoring.



8. Recommended Project Architecture (MVC Structure):

hotel-management-system/
    src/
        config/
          db.js
        controllers/
          guestController.js
          roomController.js
          employeeController.js
          bookingController.js
        models/
          Guest.js
          Room.js
          Employee.js
          Booking.js
        routes/
          guestRoutes.js
          roomRoutes.js
          employeeRoutes.js
          bookingRoutes.js
        app.js

    .env
    .gitignore
    package.json
    DATABASE_DESIGN.md