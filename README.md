# ERD of clinic managment system

<img width="3055" height="2798" alt="diagram-export-4-8-2026-9_38_36-PM" src="https://github.com/user-attachments/assets/12befb12-823a-4465-bdde-23babd54615e" />

## Some Notes 

## Design Decision Taken

### 1. Appointment-Consultation Separation (1:0..1)
Appointments are reservations; consultations are clinical facts. Separating them allows tracking no-shows separately from actual patient volume, and enables walk-in consultations without dummy appointment records.

### 2. Test Order-Report Separation (1:1)
Orders capture the clinical decision (immutable), while reports capture technical results (mutable through amendments). This supports asynchronous lab processing and maintains audit trails for medical compliance.

### 3. Specialty as Junction Table (N:M)
Doctors often hold multiple specialties (e.g., Internal Medicine + Cardiology). A separate table prevents data duplication and enables analytics like "revenue by specialty" independent of doctor assignments.

## Entity Envolved 

| Entity                  | Business Role                                             | Key Attributes                                     | Domain       |
| ----------------------- | --------------------------------------------------------- | -------------------------------------------------- | ------------ |
| **patients**            | Master demographic registry for anyone receiving care     | patient\_code, blood\_group, emergency\_contacts   | Patient Data |
| **doctors**             | Licensed medical practitioners with operational status    | license\_number, consultation\_fee, status         | Staff        |
| **departments**         | Organizational units grouping doctors by function         | name, location, head\_doctor\_id                   | Reference    |
| **specialties**         | Medical expertise taxonomy (Cardiology, Orthopedics)      | name, category                                     | Reference    |
| **doctor\_specialties** | Junction enabling multiple specialties per doctor         | is\_primary flag                                   | Linking      |
| **appointments**        | Time-slot reservations (future or historical)             | scheduled\_datetime, status, reason                | Scheduling   |
| **consultations**       | Actual clinical encounters that occurred                  | diagnosis, examination\_notes, actual\_start\_time | Clinical     |
| **test\_catalogs**      | Master price list and definition of available diagnostics | cost, turnaround\_hours, reference\_range          | Reference    |
| **test\_orders**        | Prescription instance linking a test to a patient visit   | priority, status, clinical\_notes                  | Diagnostics  |
| **test\_reports**       | Final laboratory results with approval audit              | result\_values, abnormal\_flag, verified\_by       | Diagnostics  |
| **invoices**            | Financial header aggregating all charges per visit        | total\_amount, due\_date, status                   | Billing      |
| **invoice\_items**      | Line-item charges linking services to billing             | item\_type, reference\_id (polymorphic)            | Billing      |
| **payments**            | Transaction records settling invoice balances             | amount, method, transaction\_reference             | Billing      |


## Relationships 

| Parent         | Child               | Cardinality | Business Justification                                                        |
| -------------- | ------------------- | ----------- | ----------------------------------------------------------------------------- |
| patients       | appointments        | **1:N**     | One patient books multiple appointments over their lifetime                   |
| doctors        | appointments        | **1:N**     | One doctor hosts many appointment slots in their schedule                     |
| appointments   | consultations       | **1:0..1**  | Not every appointment results in a visit (no-shows/cancellations exist)       |
| patients       | consultations       | **1:N**     | One patient has multiple historical visits (medical history)                  |
| doctors        | consultations       | **1:N**     | One doctor attends to many patients over time                                 |
| consultations  | test\_orders        | **1:N**     | One clinical encounter can generate multiple diagnostic prescriptions         |
| test\_catalogs | test\_orders        | **1:N**     | One test type (CBC) can be ordered thousands of times                         |
| doctors        | test\_orders        | **1:N**     | One doctor can prescribe many tests across patients                           |
| patients       | test\_orders        | **1:N**     | One patient undergoes multiple tests over their lifetime                      |
| test\_orders   | test\_reports       | **1:1**     | Each ordered test produces exactly one final report (asynchronous generation) |
| doctors        | test\_reports       | **1:N**     | One pathologist verifies many lab reports                                     |
| doctors        | doctor\_specialties | **1:N**     | One doctor can hold multiple board certifications                             |
| specialties    | doctor\_specialties | **1:N**     | One specialty can be practiced by many doctors                                |
| departments    | doctors             | **1:N**     | One department employs many doctors                                           |
| departments    | doctors (self-ref)  | **1:1**     | Each department has one head doctor (nullable)                                |
| patients       | invoices            | **1:N**     | One patient receives multiple bills over time                                 |
| invoices       | invoice\_items      | **1:N**     | One bill aggregates multiple services (consultation + tests)                  |
| invoices       | payments            | **1:N**     | One invoice can be paid in installments or partial payments                   |
| test\_orders   | test\_orders        | **1:N**     | Parent orders can group child tests into panels (self-referencing)            |
| test\_reports  | test\_reports       | **1:N**     | Original reports can spawn amended versions (versioning)                      |

