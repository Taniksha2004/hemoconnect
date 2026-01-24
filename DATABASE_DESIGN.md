# HemoConnect Backend - Database Design Documentation

## Overview
HemoConnect uses MongoDB with Mongoose for data persistence. The database is designed to support a healthcare blood donation ecosystem with geospatial querying, role-based access, and healthcare-grade security.

---

## 📋 Entity Relationship Diagram

```
User (Base Entity)
├── Donor (1:1 relationship)
├── Patient (1:1 relationship)
├── Hospital (1:1 relationship)
└── Admin (implicit, based on role)

BloodRequest
├── Patient (Many:1)
└── Hospital (Many:1)
    └── matchedDonors (Donor references)

Donation
├── BloodRequest (Many:1)
├── Donor (Many:1)
└── Hospital (Many:1)

Campaign
├── Hospital/User (organizer)
├── registeredDonors (Donor array references)
└── partneredHospitals (Hospital array references)
```

---

## 🔐 Model Details

### 1. **User Model** (`User.js`)
**Purpose:** Base authentication model for all platform users

**Key Fields:**
- `name`, `email` (unique), `phone` (unique)
- `password` (hashed with bcrypt)
- `role`: donor | patient | hospital | admin
- `location`: GeoJSON Point with coordinates [longitude, latitude]
- `isVerified`: Email verification status
- `lastLogin`, `isActive`: Account management

**Security Features:**
- Password hashing before save
- Password comparison method for login
- Sensitive data removed in JSON conversion
- Unique email and phone validation

**Indexes:**
- Geospatial index on location for proximity searches

**Example Use Case:**
```
User registers → Email verification → Role assignment → Access to role-specific features
```

---

### 2. **Donor Model** (`Donor.js`)
**Purpose:** Extended profile for blood donors

**Key Fields:**
- `userId`: Reference to User (unique)
- `bloodGroup`: O+, O-, A+, A-, B+, B-, AB+, AB-
- `availabilityStatus`: available | unavailable | cooldown
- `lastDonationDate`: Timestamp of last donation
- `nextEligibleDonationDate`: Auto-calculated (56-day rule)
- `healthDeclaration`: Chronic/infectious disease status
- `emergencyContact`: Contact info for emergencies
- `totalDonations`: Cumulative donation count
- `rating`: 1-5 based on reliability
- `preferences`: Notification preferences

**Healthcare Rules:**
- Donors can donate every 56 days (auto-cooldown)
- Health screening before each donation
- Multiple conditions can prevent donation eligibility

**Indexes:**
- bloodGroup, availabilityStatus, userId

**Example Use Case:**
```
Donor registration → Blood group collection → Availability tracking → Real-time matching for requests
```

---

### 3. **Patient Model** (`Patient.js`)
**Purpose:** Profile for patients needing blood transfusions

**Key Fields:**
- `userId`: Reference to User (unique)
- `medicalRecordNumber`: Hospital-assigned ID
- `requiredBloodGroup`: Blood type needed
- `bloodGroupCompatibility`: Auto-calculated compatible types
- `urgencyLevel`: low | medium | high | critical
- `unitsRequired`: 1-10 units
- `currentHospital`: Reference to Hospital (required)
- `admissionDate`: Hospital admission timestamp
- `medicalDocumentation`: Prescription, labs, consent forms
- `guarantor`: Payment responsible person
- `requestHistory`: Array of BloodRequest references

**Healthcare Compliance:**
- Informed consent form requirement
- Doctor prescription mandatory
- Hospital verification required
- Guarantor contact for payment

**Blood Group Compatibility Logic:**
```
O+ accepts: O+, O-
O- accepts: O-
A+ accepts: A+, A-, O+, O-
A- accepts: A-, O-
B+ accepts: B+, B-, O+, O-
B- accepts: B-, O-
AB+ accepts: All groups (universal recipient)
AB- accepts: AB-, A-, B-, O-
```

**Indexes:**
- currentHospital, urgencyLevel, userId

**Example Use Case:**
```
Patient admitted → Request blood → Hospital initiates matching → Donors notified → Donation process
```

---

### 4. **Hospital Model** (`Hospital.js`)
**Purpose:** Blood bank/hospital profile and inventory management

**Key Fields:**
- `userId`: Reference to User (unique)
- `registrationNumber`: Government registration (unique)
- `address`: Street, city, state, postal code
- `location`: GeoJSON Point for proximity queries
- `contactInfo`: Main phone, emergency, email, website
- `bloodBankHead`: Contact person details
- `isVerified`: Admin verification status
- `accreditations`: ISO, AABB, etc. with expiry dates
- `inventory`: Real-time stock for each blood group
- `capacity`: Max donations/day and requests/month
- `operatingHours`: Monday-Sunday hours
- `services`: blood_collection, testing, storage, transfusion, etc.
- `stats`: Monthly donations, fulfillment rate, response time

**Healthcare Standards:**
- Inventory tracking by blood group
- Accreditation verification
- Operating hours and capacity limits
- Performance metrics (response time, success rate)

**Indexes:**
- Geospatial on location, isVerified, registrationNumber

**Example Use Case:**
```
Hospital registration → Inventory setup → Receives requests → Matches donors → Verifies donations → Updates inventory
```

---

### 5. **BloodRequest Model** (`BloodRequest.js`)
**Purpose:** Central request matching system

**Key Fields:**
- `patientId`: Reference to Patient
- `hospitalId`: Reference to Hospital
- `bloodGroup`: Required blood type
- `urgencyLevel`: low | medium | high | critical
- `requiredUnits`: 1-10 units
- `unitsMatched`, `unitsFulfilled`: Progress tracking
- `status`: pending | matched | partial_fulfilled | fulfilled | cancelled
- `matchedDonors`: Array with donor info, match scores, response times
- `requestingDoctor`: Doctor details and registration
- `deliveryDetails`: Time, address, instructions
- `estimatedFulfillmentDate`: Expected delivery date
- `actualFulfillmentDate`: Completion timestamp

**Auto-Status Logic:**
- 0 units fulfilled → pending
- 0 < units < required → partial_fulfilled
- units >= required → fulfilled

**Indexes:**
- hospitalId + status, patientId, bloodGroup, urgencyLevel, status, createdAt

**Example Use Case:**
```
Request created → Algorithm finds nearby donors → Sends notifications → Donors respond → Status updates → Fulfilled/Cancelled
```

---

### 6. **Donation Model** (`Donation.js`)
**Purpose:** Complete donation lifecycle tracking

**Key Fields:**
- `requestId`: Reference to BloodRequest
- `donorId`: Reference to Donor
- `hospitalId`: Reference to Hospital
- `bloodGroup`: Blood type collected
- `unitsCollected`: 0.5-1 unit (standard)
- `donationStatus`: 11 states from notified to expired
- `healthScreening`: Pre-donation health check
- `labTesting`: HIV, Hepatitis B/C, Syphilis, Blood Culture
- `verifiedByHospital`: Hospital verification and timestamp
- `bagNumber`: Unique blood bag ID
- `storageLocation`: Shelf/position and temperature
- `expiryDate`: Auto-calculated (42 days from collection)
- `transfusionInfo`: When/where/who transfused
- `compensation`: Donor compensation reference
- `feedbackFromDonor`, `feedbackFromHospital`: Ratings

**11-State Donation Workflow:**
1. **notified** → Donor notified of request
2. **accepted** → Donor accepts
3. **rejected** → Donor declines
4. **scheduled** → Appointment set
5. **collection_started** → Collection begins
6. **collection_completed** → Blood collected
7. **testing** → Lab testing in progress
8. **testing_passed** → All tests pass
9. **testing_failed** → Tests fail (unsafe)
10. **stored** → Ready for transfusion
11. **transfused** → Given to patient
12. **expired** → Beyond 42-day shelf life

**Healthcare Compliance:**
- Lab test results mandatory
- Hospital verification required
- Temperature-controlled storage tracking
- Expiry date auto-management

**Indexes:**
- donorId + status, hospitalId, donationStatus, bagNumber, expiryDate

**Example Use Case:**
```
Donor accepts → Health screening → Lab tests → Storage → Transfusion → Feedback
```

---

### 7. **Campaign Model** (`Campaign.js`)
**Purpose:** Blood donation drive coordination and awareness

**Key Fields:**
- `title`, `description`, `type`: emergency | planned | seasonal | community | corporate
- `organizer`: Reference to User (hospital/organization)
- `location`: GeoJSON Point
- `campaignDate`: startDate, endDate
- `targetBloodGroups`: Specific groups or 'all'
- `targetDonors`: Goal number
- `registeredDonors`: Array with donor info, slot times, attendance, donation status
- `successMetrics`: Total registered, attended, donated, units collected
- `status`: upcoming | active | completed | cancelled
- `images`: Campaign photos with captions
- `contact`: Campaign manager contact
- `incentives`: Optional donor incentives
- `partneredHospitals`: Multiple hospital references
- `visibility`: public | private | invited
- `socialMedia`: Hashtags and links

**Metrics Tracking:**
- Registration rate
- Attendance rate
- Donation completion rate
- Total units collected

**Indexes:**
- Geospatial on location, status, campaign dates, organizer

**Example Use Case:**
```
Campaign creation → Public announcement → Donor registration → Campaign active → Track attendance → Donations → Metrics analysis
```

---

## 🔑 Key Design Patterns

### 1. **Geospatial Indexing**
- All models with location use GeoJSON Point format
- 2dsphere indexes enable proximity-based queries
- Find nearest donors/hospitals within radius

```javascript
// Query: Find donors within 5km of patient
db.donors.find({
  location: {
    $near: {
      $geometry: { type: "Point", coordinates: [longitude, latitude] },
      $maxDistance: 5000 // meters
    }
  }
})
```

### 2. **Role-Based References**
- Single User model with role enum
- Role determines accessible models
- Clean separation of concerns

### 3. **Status State Machines**
- BloodRequest: 5 states
- Donation: 11 states
- Campaign: 4 states
- Auto-transitions based on data changes

### 4. **Audit Trail**
- All models include `timestamps: true`
- createdAt, updatedAt auto-managed
- Support for historical analysis

### 5. **Soft Deletions Ready**
- Can add `isDeleted` boolean field
- Preserve data integrity
- Support compliance audits

### 6. **Extensibility**
- Extra fields in nested objects
- Array fields for future features
- AI/ML ready (match scores, recommendations)

---

## 📊 Database Statistics

**Indexes Created:** 25+ across all models
**Unique Constraints:** 6 (email, phone, registrationNumber, medicalRecordNumber, bagNumber, userId)
**Geospatial Indexes:** 4 (User, Hospital, Campaign, and for future queries)

---

## 🔒 Security Measures

✅ Password hashing with bcryptjs
✅ JWT token authentication
✅ Sensitive data exclusion in JSON
✅ Email/phone uniqueness constraints
✅ Role-based field access
✅ Verification tokens for email confirmation
✅ Healthcare data privacy compliance
✅ Referential integrity with ObjectId

---

## 📚 Next Steps

After models are created, we will generate:
1. Middleware (authentication, validation, error handling)
2. Controllers (business logic for all operations)
3. Routes (API endpoints)
4. Utility functions (matching algorithm, notifications)
5. Tests (unit and integration)

---

## 📖 Database Setup Instructions

```bash
# Install dependencies
npm install

# Create .env file from .env.example
cp .env.example .env

# Update MongoDB URI in .env
MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net/hemoconnect

# Connect to MongoDB and create indexes
npm run seed  # (Will be created after seed script generation)
```

---

**Document Version:** 1.0
**Last Updated:** January 23, 2026
**Status:** Complete - Ready for Controller Generation
