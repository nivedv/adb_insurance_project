# 📊 Insurance Claims Data Dictionary

## Preapred by Nived Varma / Microsoft Certified Trainer

## Dataset: `insurance_claims_data.csv`

| Column Name              | Data Type | Description                   | Example Values                                        | Nullable | Business Rules                            |
| ------------------------ | --------- | ----------------------------- | ----------------------------------------------------- | -------- | ----------------------------------------- |
| **claim_id**             | String    | Unique claim identifier       | CLM-2024-123456                                       | No       | Primary Key, Format: CLM-YYYY-NNNNNN      |
| **policy_id**            | String    | Policy reference number       | POL-12345678                                          | Yes\*    | Foreign Key, Format: POL-NNNNNNNN         |
| **customer_id**          | String    | Customer unique identifier    | CUST-100001                                           | Yes\*    | Foreign Key, Format: CUST-NNNNNN          |
| **customer_name**        | String    | Full customer name            | John Smith                                            | Yes\*    | First + Last name                         |
| **customer_email**       | String    | Customer email address        | john.smith@email.com                                  | Yes\*    | Must be valid email format                |
| **customer_phone**       | String    | Customer phone number         | (555) 234-5678                                        | Yes\*    | US phone format                           |
| **customer_state**       | String    | Customer residence state      | CA, NY, TX                                            | Yes\*    | 2-letter state codes                      |
| **customer_segment**     | String    | Customer tier classification  | Premium, Standard, Basic                              | Yes\*    | Business segmentation                     |
| **policy_type**          | String    | Type of insurance policy      | Auto, Home, Health, Life, Business, Travel            | Yes\*    | Policy category                           |
| **coverage_amount**      | Integer   | Policy coverage limit         | 250000, 500000                                        | No       | USD amount                                |
| **premium_amount**       | Integer   | Annual premium paid           | 2500, 1800                                            | No       | USD amount                                |
| **deductible**           | Integer   | Policy deductible amount      | 500, 1000, 2500                                       | No       | USD amount                                |
| **claim_date**           | Date      | Date when incident occurred   | 2024-01-15                                            | No       | Format: YYYY-MM-DD                        |
| **reported_date**        | Date      | Date claim was reported       | 2024-01-16                                            | No       | Must be >= claim_date                     |
| **claim_type**           | String    | Category of claim             | Collision, Theft, Fire, Water Damage                  | No       | Incident classification                   |
| **claim_amount**         | Integer   | Total claim amount requested  | 25000, 15000                                          | Yes\*    | USD amount, can be missing                |
| **status**               | String    | Current claim status          | Open, Under Review, Approved, Denied, Settled, Closed | No       | Workflow status                           |
| **priority**             | String    | Claim processing priority     | Low, Medium, High, Critical                           | No       | Business priority                         |
| **adjuster_id**          | String    | Assigned adjuster ID          | ADJ-1001                                              | No       | Format: ADJ-NNNN                          |
| **location_state**       | String    | State where incident occurred | CA, NY, TX                                            | No       | 2-letter state codes                      |
| **processing_days**      | Integer   | Days to process claim         | 35, 28, 42                                            | Yes      | NULL for open claims                      |
| **settlement_amount**    | Float     | Final settlement paid         | 23500.0, 14800.0                                      | Yes      | NULL for non-settled claims               |
| **is_fraud**             | Boolean   | Fraud indicator flag          | TRUE, FALSE                                           | No       | ML target variable                        |
| **fraud_score**          | Float     | Fraud risk score (0-1)        | 0.245, 0.890                                          | No       | ML model feature, 0=low risk, 1=high risk |
| **incident_description** | String    | Brief incident summary        | Vehicle collision at intersection                     | No       | Text description                          |
| **weather_conditions**   | String    | Weather during incident       | Clear, Rainy, Snowy, Foggy, Windy                     | No       | Environmental factor                      |
| **police_report_filed**  | Boolean   | Police report indicator       | TRUE, FALSE                                           | No       | Legal documentation flag                  |
| **witnesses**            | Integer   | Number of witnesses           | 0, 1, 2, 3+                                           | No       | Evidence factor                           |
| **payment_method**       | String    | Settlement payment method     | Check, Wire Transfer, ACH, Digital Wallet             | Yes      | NULL for unpaid claims                    |
| **created_at**           | Date      | Record creation timestamp     | 2024-08-07                                            | No       | System timestamp                          |

## 🎯 Key Data Quality Notes:

**Critical Fields** (should not be NULL):

- `claim_id`, `claim_date`, `reported_date`, `claim_type`, `status`, `priority`

**Conditional NULLs** (business logic):

- `processing_days` and `settlement_amount`: NULL for claims with status "Open" or "Under Review"
- `payment_method`: NULL for non-settled claims

**Data Validation Rules**:

- `reported_date` >= `claim_date`
- `settlement_amount` <= `claim_amount` (when both present)
- `fraud_score` between 0.0 and 1.0
- Email format validation required for `customer_email`

## 🔧 ML Model Targets:

**Fraud Detection Model:**

- **Target**: `is_fraud` (Boolean)
- **Features**: `fraud_score`, `claim_amount`, `processing_days`, `claim_type`, `customer_segment`

**Processing Time Prediction:**

- **Target**: `processing_days` (Integer)
- **Features**: `claim_amount`, `claim_type`, `priority`, `policy_type`, `location_state`

## 📈 Recommended Partitioning:

**Delta Lake Partitioning Strategy:**

- Primary: `claim_date` (year/month)
- Secondary: `location_state`
- Tertiary: `policy_type`

_(_) Nullable fields included for data cleaning practice
