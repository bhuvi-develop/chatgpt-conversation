# Backend database overview

**User:** Anonymous  
**Created:** 8/18/2026 10:12:59  
**Updated:** 8/18/2026 10:24:26  
**Exported:** 9/28/2026 12:47:42  
**Link:** [https://chatgpt.com/g/g-p-6a58952e4f7c8191a6db46b8eb8f2dee/c/6a83e2b8-8c30-83e8-b787-15128f8db350](https://chatgpt.com/g/g-p-6a58952e4f7c8191a6db46b8eb8f2dee/c/6a83e2b8-8c30-83e8-b787-15128f8db350)  

## Prompt:
8/18/2026, 10:12:58 AM

enakuh ethula enna ennah backend errukuh ethula erruka database la ennah ennah tableahh errukuh ethukula use aguthuhh

[Attachment: CK-26(1).zip]

## Response:
8/18/2026, 10:12:59 AM · gpt-5-6

> **Inspecting ZIP archive contents**
>
>
> ****
>
> 
>
> **Inspecting backend routers and database table references**
>
> 
>
> Worked for 1m 4s

ஆம். இந்த `CK-26` project-ஐ பார்த்ததில் **backend + database architecture fairly பெரியது**. உனக்கு easy-ஆ புரியுற மாதிரி பிரிச்சு சொல்றேன்.

### 1. Backend என்ன use பண்ணிருக்காங்க?

இந்த project-ன் main backend:

**Python + FastAPI**

இதுக்குள் முக்கியமான பகுதிகள்:

- **FastAPI** → REST APIs
- **MQTT** → IoT device-லிருந்து live data receive
- **Kafka** → telemetry/message pipeline
- **PostgreSQL + TimescaleDB** → time-series IoT data
- **MySQL / existing Caleido DB** → legacy/application data
- **WebSocket** → dashboard-க்கு real-time data
- **RBAC** → user/role/permission management
- **Rule Engine** → threshold-based alerts
- **BHI Engine** → Building Health Index calculation

Dashboard API itself `08_dashboard_api`-ஆக அமைக்கப்பட்டுள்ளது; அதில் alerts, devices, charts, historical data, RBAC, BHI, telemetry, authentication, users, stakeholders, customer/site போன்ற routers இருக்கின்றன.

---

# 2. Database-ல் முக்கியமாக என்ன இருக்கு?

Project-ல் database ஒரே simple DB இல்லை.

Architecture-ல்:

```text
IoT Devices
     ↓
    MQTT
     ↓
   Kafka
     ↓
Python Backend
     ↓
PostgreSQL / TimescaleDB
     ↓
Dashboard APIs
     ↓
Frontend
```

அதே நேரத்தில் existing **Caleido application DB**-வும் பயன்படுத்தப்படுகிறது.

Project documentation படி முக்கிய database areas:

| Database / Schema | Purpose |
|---|---|
| `insp_iot` / PostgreSQL | New IoT platform data |
| `caleido` / MySQL | Existing application/device data |
| `caleido_notification` | Notifications |
| `caleido_scheduler` | Scheduled jobs |

---

# 3. `insp_iot` PostgreSQL tables

இந்த project-ன் புதிய PostgreSQL/TimescaleDB schema-ல் முக்கிய tables:

### User / Authentication

| Table | என்ன use? |
|---|---|
| `insp_user` | Users login/account information |
| `insp_role` | User roles |
| `insp_access_right` | Permissions/access rights |
| `insp_role_access_right` | Which role has which permission |
| `insp_support_tier` | Support-level information |

Example:

```text
User
 ↓
Role
 ↓
Access Rights
```

இதுதான் **RBAC**.

---

# 4. Customer / Site management

| Table | Purpose |
|---|---|
| `insp_customer` | Customer information |
| `insp_site` | Customer site/location |
| `insp_device_assignment` | Device எந்த customer/site-க்கு assigned என்று maintain பண்ணும் |

Relationship roughly:

```text
Customer
   ↓
 Site
   ↓
Device Assignment
   ↓
 Device
```

---

# 5. Device master tables

இது IoT project-க்கு ரொம்ப முக்கியம்.

### `insp_device`

இதுதான் device master/registry.

இதில்:

- `device_id`
- `device_uid`
- `serial_number`
- `device_name`
- `device_type`
- `node_type`
- `parent_device_id`
- `is_gateway`
- `appliance`
- `manufacturer`
- `model`
- `hardware_version`
- `firmware_version`

போன்ற information இருக்கும்.

அதாவது:

```text
insp_device
     │
     ├── device identity
     ├── appliance
     ├── manufacturer
     ├── model
     ├── firmware
     └── gateway relationship
```

**Infrastructure/device identity-க்கு இது core table.**

---

# 6. Device state

### `insp_device_state_type`

Device state என்ன?

Example:

```text
Active
Inactive
Maintenance
Fault
```

போன்ற state definitions.

### `insp_device_state`

ஒரு particular device எப்போது எந்த state-ல் இருந்தது என்பதை store செய்யும்.

```text
insp_device
     ↓
insp_device_state
     ↓
insp_device_state_type
```

---

# 7. Device inventory

### `insp_device_inventory`

Device physical inventory information.

Example:

- received date
- PO number
- lot number
- warehouse
- bin location
- condition
- quantity

இது **IoT telemetry table கிடையாது**.

இது device lifecycle/inventory management.

---

# 8. மிக முக்கியமான IoT telemetry tables

இதுதான் project-ன் actual sensor data side.

### `insp_device_health`

Device health information.

Store செய்யும் things:

- `device_uid`
- health status
- temperature
- timestamp
- message ID
- source timestamp
- device ID

Flow:

```text
MQTT
 ↓
HDT message
 ↓
insp_device_health
```

---

### `insp_device_performance`

**முக்கியமான telemetry table.**

இதில் device parameter/performance data store ஆகும்.

Example:

```text
Voltage
Current
Power
Energy
Frequency
Power Factor
Temperature
...
```

Project-ல் dashboard charts மற்றும் historical telemetry இந்த table-ஐ heavily use செய்கிறது.

Flow:

```text
MQTT PDT
   ↓
Kafka
   ↓
DB Writer
   ↓
insp_device_performance
   ↓
Dashboard
```

---

### `insp_device_connection`

Device connection/disconnection events.

MQTT **LWT (Last Will and Testament)** data இதற்குள் வருகிறது.

Example:

```text
Device connected
Device disconnected
```

இதன் purpose:

**Device online/offline history maintain செய்வது.**

---

# 9. Alerts

### `insp_device_alert`

Device-லிருந்து / rule engine-லிருந்து உருவாகும் alerts.

Fields include:

- device
- alert type
- severity
- alert data
- timestamp
- message information

Flow:

```text
Telemetry
   ↓
Rule Engine
   ↓
Threshold / anomaly condition
   ↓
insp_device_alert
   ↓
Dashboard
```

---

# 10. Building Health

### `insp_building_health`

இந்த project-ல் **BHI — Building Health Index** calculation இருக்கிறது.

Table-ல்:

- IEI
- PQI
- DSI
- BHI
- status
- details
- site

போன்ற values store செய்யப்படுகின்றன.

Architecture:

```text
Device telemetry
      ↓
   BHI Engine
      ↓
 ┌────┼────┐
 IEI  PQI  DSI
      ↓
     BHI
      ↓
insp_building_health
```

---

# 11. Service Request / Ticket management

இந்த tables dashboard/application workflow-க்கு:

| Table | Purpose |
|---|---|
| `insp_service_request` | Service request |
| `insp_issue_type` | Issue classification |
| `insp_priority` | Priority |
| `insp_ticket_status` | Ticket status |
| `insp_sr_status_history` | Status change history |

Example:

```text
Device Problem
      ↓
Service Request
      ↓
Priority
      ↓
Ticket Status
      ↓
Status History
```

---

# 12. Pipeline monitoring

### `insp_pipeline_metric`

Backend data pipeline healthy-ஆ இருக்கா என்று track செய்யும்.

For example:

- messages received
- messages processed
- DB writes
- failures
- processing window

So:

```text
MQTT/Kafka
    ↓
Consumers
    ↓
Pipeline metrics
    ↓
insp_pipeline_metric
```

---

# 13. Database-ஐ overall-ஆ பார்த்தால்

உனக்கு easiest mental model இது:

```text
                    ┌──────────────────┐
                    │   insp_customer  │
                    └────────┬─────────┘
                             ↓
                       ┌───────────┐
                       │ insp_site │
                       └─────┬─────┘
                             ↓
                    ┌─────────────────┐
                    │   insp_device   │
                    └────────┬────────┘
                             │
             ┌───────────────┼────────────────┐
             ↓               ↓                ↓
     device_health     device_performance  device_connection
             │               │                │
             └───────────────┼────────────────┘
                             ↓
                      Dashboard / APIs
                             │
                    ┌────────┴────────┐
                    ↓                 ↓
              device_alert       BHI Engine
                    ↓                 ↓
             Alert Dashboard    building_health
```

அதுக்கு side-ல:

```text
insp_user
    ↓
insp_role
    ↓
insp_role_access_right
    ↓
insp_access_right
```

இது authentication + authorization.

---

# 14. Backend folder-ஐ simple-ஆ புரிஞ்சுக்கணும்னா

### `iotBackend/core`

Core infrastructure:

- MQTT client
- Kafka producer
- configuration
- metrics
- settings
- threshold state

### `iotBackend/consumers`

Incoming data process பண்ணும்:

```text
MQTT/Kafka
   ↓
Consumers
   ├── dashboard
   ├── db_writer
   ├── ML pipeline
   └── threshold alert
```

### `iotBackend/services`

Actual processing services:

```text
01_data_extractor
        ↓
02_data_transformer
        ↓
...
08_dashboard_api
```

### `08_dashboard_api/routers`

Frontend call பண்ணும் API endpoints:

- devices
- alerts
- charts
- historical
- telemetry
- BHI
- users
- RBAC
- service requests
- authentication
- customers/sites
- stakeholders

---

## 15. Most important thing you should understand

இந்த project-ஐ நீ study பண்ணும்போது **ஒவ்வொரு table-ஐ தனித்தனியா memorize பண்ணாதே.**

இந்த 6 groups-ஆ புரிஞ்சுக்கோ:

```text
1. IDENTITY
   └── users / roles / permissions

2. ORGANIZATION
   └── customer / site / assignment

3. DEVICE
   └── device / state / inventory

4. TELEMETRY
   └── health / performance / connection

5. INTELLIGENCE
   └── alerts / BHI / thresholds

6. OPERATIONS
   └── service requests / tickets / pipeline metrics
```

**இதுதான் CK-26 backend + DB-யின் actual structure.**

மேலும் ஒரு important point: project-ல் **old Caleido MySQL tables-மும் new PostgreSQL/TimescaleDB tables-மும் coexist ஆகின்றன**. அதனால் `devices`, `device_stats`, `device_alerts` மாதிரி பெயர்கள் இரண்டு architecture layers-ல காணப்படலாம். அவற்றை ஒன்றே table என்று assume பண்ணக்கூடாது. `db_analysis.md` இதையே explicitly document செய்கிறது.

## Prompt:
8/18/2026, 10:16:28 AM

![3bf75f18-5179-43a6-a32f-7c526eb66f2a.png](https://chatgpt.com/backend-api/estuary/content?id=file_00000000e6888211972854b46b577fe8&fn=3bf75f18-5179-43a6-a32f-7c526eb66f2a.png&cd=attachment&ts=497383&p=fs&cid=1&sig=3c81ecb1798e8c4c3a41ae578f364a7b1b49c2c96c51ba3338c5f96b0bb87239&v=0)

![12830f58-b8d6-4422-bbcb-29c873bf95a1.png](https://chatgpt.com/backend-api/estuary/content?id=file_000000008f1882118675f139e88375da&fn=12830f58-b8d6-4422-bbcb-29c873bf95a1.png&cd=attachment&ts=497383&p=fs&cid=1&sig=deeed3c60c0c8f7c47b1fab32d0a92b1b64ca9947fd34ae109c39b98613ce521&v=0)

ethula erruka database ellameyy ethukuh la use aguthunuh solluh

## Response:
8/18/2026, 10:16:36 AM · gpt-5-6

> **Explained the purpose of all 24 database tables**
>
> 
>
> Worked for 6s

ஆம். நீ காட்டிய screenshot-ல **`insp_iot` PostgreSQL database → `public` schema → 24 tables** இருக்கு. இப்போ ஒவ்வொரு table-ம் **எதற்காக, என்ன data வைத்திருக்கும், project-ல் எங்கே use ஆகும்** என்று clear-ஆ சொல்றேன்.

> **Important:** Table name பார்த்து purpose-ஐ identify பண்ண முடியும். ஆனால் exact columns / foreign keys / API usage-ஐ 100% confirm பண்ண table definition பார்க்க வேண்டும். கீழே நான் screenshot + project structure அடிப்படையில் சொல்றேன்.

---

# 1. `insp_user` 👤

**Purpose:** Application users.

இதில்:

- username
- email
- password/auth-related data
- user status
- role relationship

போன்ற user information இருக்கும்.

### எங்கே use?

Login செய்யும்போது:

```text
User
 ↓
insp_user
 ↓
Authentication
 ↓
Role
 ↓
Permissions
```

Screenshot-ல் **12 records** இருக்கு.

---

# 2. `insp_role` 🔐

**Purpose:** User role definition.

Example:

```text
Admin
Engineer
Manager
Viewer
Support
```

### Use:

ஒரு user என்ன level access வைத்திருக்கிறார் என்பதை determine பண்ண.

```text
insp_user
    ↓
insp_role
```

---

# 3. `insp_access_right` 🛡️

**Purpose:** Individual permissions.

Example:

```text
VIEW_DEVICE
VIEW_ALERT
EDIT_DEVICE
VIEW_REPORT
MANAGE_USER
```

இது role இல்லை.

**Role = யார்?**

**Access Right = அவர் என்ன செய்யலாம்?**

---

# 4. `insp_role_access_right` 🔗

இது ஒரு **mapping/junction table**.

இதன் வேலை:

```text
Role  ↔  Access Right
```

Example:

```text
Admin
 ├── VIEW_DEVICE
 ├── EDIT_DEVICE
 ├── VIEW_ALERT
 └── MANAGE_USER

Viewer
 ├── VIEW_DEVICE
 └── VIEW_ALERT
```

Screenshot-ல் **132 records**.

### இதுதான் RBAC-ன் core table.

---

# 5. `insp_customer` 🏢

**Purpose:** Customer information.

Example:

```text
Customer A
Customer B
Customer C
```

Customer-க்கு:

- customer name
- contact
- organization details
- status

போன்ற information.

### Relationship:

```text
Customer
   ↓
Site
   ↓
Devices
```

---

# 6. `insp_site` 📍

**Purpose:** Customer-ன் physical/site location.

Example:

```text
Customer: ABC
    ↓
Site: Chennai Plant
```

ஒரு customer-க்கு multiple sites இருக்கலாம்.

```text
Customer
 ├── Chennai Site
 ├── Bangalore Site
 └── Coimbatore Site
```

---

# 7. `insp_device` 📡

**இது மிகவும் முக்கியமான table.**

இது **device master/identity table**.

Device-ஐ identify செய்யும் information இங்கே இருக்கும்.

Example:

```text
HUB-01
AirQ
Mikos-03
0102110000000001
```

Possible information:

- device ID
- device UID
- serial number
- device type
- appliance
- manufacturer
- model
- firmware
- gateway relationship

### மற்ற tables பல இதை reference செய்யும்.

```text
                insp_device
                    │
        ┌───────────┼────────────┐
        ↓           ↓            ↓
     health    performance    alert
        ↓           ↓            ↓
    connection   state       assignment
```

**இந்த table-ஐ infrastructure/device identity-க்கு பயன்படுத்துகிறார்கள்.**

---

# 8. `insp_device_assignment` 🔗

**Purpose:** எந்த device யாருக்கு / எந்த site-க்கு assigned?

Example:

```text
Device: HUB-01
       ↓
Customer: ABC
       ↓
Site: Chennai Plant
```

அதாவது device-ஐ organization/site-க்கு map பண்ணும் table.

---

# 9. `insp_device_inventory` 📦

**Purpose:** Physical device inventory/lifecycle information.

Example:

```text
Device received
Device stored
Device deployed
Device replaced
Device retired
```

இது **live sensor telemetry இல்லை**.

Device physical inventory management.

---

# 10. `insp_device_state_type` ⚙️

இது **state definitions/master table**.

Example:

```text
ONLINE
OFFLINE
ACTIVE
INACTIVE
MAINTENANCE
FAULT
```

---

# 11. `insp_device_state` 🔄

ஒரு particular device-ன் actual state.

Example:

```text
HUB-01
  ↓
ACTIVE
  ↓
2026-08-18 10:00

HUB-01
  ↓
FAULT
  ↓
2026-08-18 10:30
```

Relationship:

```text
insp_device_state_type
          ↓
   insp_device_state
          ↓
      insp_device
```

---

# 12. `insp_device_connection` 🌐

**Purpose:** Device online/offline connection history.

Example:

```text
10:00 → Connected
10:30 → Disconnected
10:35 → Connected
```

Screenshot-ல் **55,383 records** இருக்கிறது.

### Dashboard-ல் இதைப் பயன்படுத்தி:

```text
Device Online
Device Offline
Last Seen
Connection History
```

போன்ற information காட்டலாம்.

---

# 13. `insp_device_health` ❤️‍🩹

**Purpose:** Device health-related telemetry/status.

Screenshot-ல் இது **859,236 records** உடன் மிகப்பெரிய tables-ல் ஒன்று.

இதிலிருந்து device health data track பண்ண முடியும்.

Example:

```text
Device
 ↓
Temperature
Voltage
Current
Status
Timestamp
```

### Dashboard use:

```text
Device Health
Health status
Health trend
```

---

# 14. `insp_device_performance` 📊

**இது இன்னொரு முக்கியமான telemetry table.**

Device எப்படி perform செய்கிறது என்பதை store செய்யும்.

Example:

```text
Voltage
Current
Power
Energy
Frequency
Power Factor
Temperature
Timestamp
```

Screenshot-ல் **14,543 records**.

### Example:

```text
HUB-01
10:00
Voltage = 230V
Current = 4.2A
Power = 966W
```

Dashboard-ன்:

- charts
- graphs
- historical performance
- telemetry

போன்றவற்றுக்கு இது பயன்படுத்தப்படும்.

---

# 15. `insp_device_alert` 🚨

**Purpose:** Device alerts.

Screenshot-ல் **125 records**.

Example:

```text
High Voltage
Low Voltage
High Temperature
Communication Failure
Device Fault
```

Flow:

```text
Device Telemetry
       ↓
Threshold / Rule
       ↓
Condition detected
       ↓
insp_device_alert
       ↓
Dashboard Alert
```

Screenshot-ல் நீ பார்த்த **Device Alerts** screen-க்கு இது முக்கிய table.

---

# 16. `insp_limit_config` 🎚️

**Purpose:** Threshold / limit configuration.**

Example:

```text
Voltage:
Min = 200V
Max = 250V
```

```text
Temperature:
Max = 80°C
```

அதாவது:

```text
Actual value
     ↓
Compare with limit
     ↓
Limit exceeded?
     ↓
Alert
```

### Example

```text
Voltage = 270V
Configured max = 250V

270 > 250
   ↓
Alert
```

இதுதான் `insp_device_alert` உருவாகும் logic-க்கு பயன்படுத்தப்படக்கூடிய configuration table.

---

# 17. `insp_issue_type` 🐛

**Purpose:** Service/maintenance issue classification.**

Example:

```text
Electrical Issue
Mechanical Issue
Communication Issue
Device Failure
Installation Issue
```

Service request create செய்யும்போது:

```text
Issue Type = Electrical
```

என்று categorize செய்யலாம்.

---

# 18. `insp_priority` 🔴🟡🟢

**Purpose:** Service request priority.

Example:

```text
LOW
MEDIUM
HIGH
CRITICAL
```

ஒரு issue வந்தால் அதன் importance என்ன என்பதை define பண்ணும்.

```text
Device Failure
      ↓
Priority = CRITICAL
```

---

# 19. `insp_service_request` 🎫

**Purpose:** Customer/device maintenance service request.**

Example:

```text
Customer reports:
"AC not working"

        ↓

Service Request
        ↓
Issue Type
        ↓
Priority
        ↓
Engineer
        ↓
Resolution
```

இதுதான் actual service ticket/request record.

---

# 20. `insp_ticket_status` 🔄

**Purpose:** Ticket-ன் possible statuses.**

Example:

```text
OPEN
ASSIGNED
IN_PROGRESS
ON_HOLD
RESOLVED
CLOSED
```

---

# 21. `insp_sr_status_history` 📜

`SR` = **Service Request**

இந்த table service request-ன் **status history** maintain பண்ணும்.

Example:

```text
SR-1001

10:00 → OPEN
10:15 → ASSIGNED
11:00 → IN_PROGRESS
12:30 → RESOLVED
13:00 → CLOSED
```

இதனால் current status மட்டும் இல்லாமல் **எப்போது என்ன change நடந்தது** என்று தெரியும்.

---

# 22. `insp_support_tier` 👨‍🔧

**Purpose:** Support level / escalation level.**

Example:

```text
Tier 1 → Basic support
Tier 2 → Technical support
Tier 3 → Expert support
```

ஒரு service request எந்த support level-க்கு போக வேண்டும் என்பதை manage செய்ய உதவும்.

---

# 23. `insp_building_health` 🏢❤️

**Purpose:** Building/site overall health.**

Screenshot-ல் **5,945 records**.

Individual device health மட்டும் இல்லாமல் **building/site level health** represent செய்யும்.

Conceptually:

```text
Device 1 ──┐
Device 2 ──┤
Device 3 ──┼──> Building Health
Device 4 ──┘
```

Example:

```text
Building Health = 87%
Status = Healthy
```

Dashboard-ல் overall health score காட்ட இதைப் பயன்படுத்தலாம்.

---

# 24. `insp_pipeline_metric` ⚙️📈

**Purpose:** Backend data pipeline monitoring.**

Screenshot-ல் **11,957 records**.

IoT data flow:

```text
Device
  ↓
MQTT
  ↓
Kafka
  ↓
Backend Consumer
  ↓
Database
```

இந்த pipeline சரியாக வேலை செய்கிறதா என்று metrics store செய்யும்.

Example:

```text
Messages received
Messages processed
Messages failed
Processing latency
```

### இது device telemetry இல்லை.

**Backend itself healthy-ஆ இருக்கிறதா என்பதை monitor செய்யும் data.**

---

# இப்போ 24 tables-ஐ ஒரே picture-ஆ புரிஞ்சுக்கோ

```text
                    CUSTOMER
                       │
                       ↓
                      SITE
                       │
                       ↓
                    DEVICE
                       │
        ┌──────────────┼────────────────┐
        ↓              ↓                ↓
    ASSIGNMENT     INVENTORY          STATE
        │                              │
        │                         STATE_TYPE
        │
        ├───────────────┬──────────────────┐
        ↓               ↓                  ↓
   CONNECTION        HEALTH           PERFORMANCE
        │               │                  │
        └───────────────┼──────────────────┘
                        ↓
                  LIMIT_CONFIG
                        ↓
                      ALERT
                        ↓
                BUILDING_HEALTH
```

Separate application management:

```text
USER
 ↓
ROLE
 ↓
ROLE_ACCESS_RIGHT
 ↓
ACCESS_RIGHT
```

Service management:

```text
SERVICE_REQUEST
      │
      ├── ISSUE_TYPE
      ├── PRIORITY
      ├── TICKET_STATUS
      ├── SUPPORT_TIER
      └── SR_STATUS_HISTORY
```

Backend monitoring:

```text
MQTT
 ↓
Kafka
 ↓
Consumers
 ↓
PIPELINE_METRIC
```

---

# 🔥 Most important tables — priority order

நீ இந்த project-ஐ புரிஞ்சிக்கணும்னா **24 tables-யும் ஒரே நேரத்தில் படிக்க வேண்டாம்.**

இந்த order-ல் படி:

### Level 1 — Core

1. `insp_device`
2. `insp_customer`
3. `insp_site`
4. `insp_device_assignment`

### Level 2 — IoT data

5. `insp_device_performance`
6. `insp_device_health`
7. `insp_device_connection`
8. `insp_device_state`
9. `insp_device_alert`

### Level 3 — Intelligence

10. `insp_limit_config`
11. `insp_building_health`

### Level 4 — Users/RBAC

12. `insp_user`
13. `insp_role`
14. `insp_access_right`
15. `insp_role_access_right`

### Level 5 — Service

16. `insp_service_request`
17. `insp_issue_type`
18. `insp_priority`
19. `insp_ticket_status`
20. `insp_sr_status_history`
21. `insp_support_tier`

### Level 6 — Supporting

22. `insp_device_inventory`
23. `insp_device_state_type`
24. `insp_pipeline_metric`

---

## One critical distinction

**`insp_device` ≠ `insp_device_performance` ≠ `insp_device_health` ≠ `insp_device_alert`**

இதுதான் நீ முதலில் strong-ஆ புரிஞ்சிக்க வேண்டியது.

```text
insp_device
    ↓
"Who/what is this device?"

insp_device_performance
    ↓
"How is this device performing?"

insp_device_health
    ↓
"What is the health condition?"

insp_device_alert
    ↓
"Is something wrong / needs attention?"

insp_device_connection
    ↓
"Is it connected or disconnected?"
```

இதைக் clear-ஆ புரிஞ்சிட்டா, இந்த database architecture பாதி புரிஞ்ச மாதிரி.

## Prompt:
8/18/2026, 10:22:10 AM

ck project oda full explanation enakuh venum

## Response:
8/18/2026, 10:22:11 AM · gpt-5-6

> **Remembering**
>
>
> ****
>
> 
>
> **Reviewed and mapped your CK-26 architecture, tables, and data flow**
>
> 
>
> Worked for 1m 39s

ஆம். இப்போ நான் **CK-26 ZIP-ஐ actual project structure/code பார்த்து**, database screenshot-ஐயும் சேர்த்து, **“இந்த project என்ன? ஏன்? எப்படி வேலை செய்கிறது? frontend → backend → MQTT → Kafka → database → alert → dashboard வரை என்ன நடக்குது?”** என்று முழுசா explain பண்ணுறேன்.

ஒரு முக்கியமான விஷயம் முதலில்:

> **CK-26-ல் documentation-க்கும் current code-க்கும் சில இடங்களில் mismatch இருக்கு.**  
> குறிப்பாக பழைய `ARCHITECTURE.md` ஒரு 6-stage ML pipeline (`03_feature_engineer → 04_model_trainer → 05_anomaly_detector → 06_metrics_exporter`) பற்றி சொல்கிறது. ஆனால் தற்போதைய codebase-ல் அந்த services இல்லை; current implementation `MlPipelineConsumer`, `ThresholdAlertConsumer`, `DbWriterConsumer`, FastAPI dashboard API போன்றவற்றை பயன்படுத்துகிறது. அதனால் கீழே **current code-ஐ source of truth**-ஆ எடுத்துக்கொள்கிறேன்.

---

# 1. முதலில் CK-26 project என்ன?

CK-26 என்பது ஒரு **IoT device monitoring + management + anomaly/alert + dashboard platform**.

Simple-ஆ சொன்னா:

> IoT devices data அனுப்பும் → backend அதை receive/process பண்ணும் → database-ல் store பண்ணும் → rules/ML check பண்ணும் → problem இருந்தால் alert உருவாகும் → frontend dashboard-ல் user பார்க்கிறார்.

Overall:

```text
        IoT DEVICES
             │
             │ MQTT
             ▼
      ┌──────────────┐
      │ MQTT Broker  │
      └──────┬───────┘
             │
             ▼
      MQTT Payload Parser
             │
             ▼
          KAFKA
             │
      ┌──────┼──────────────┐
      │      │              │
      ▼      ▼              ▼
   DB Writer  ML        Threshold
      │      Pipeline      Engine
      │        │              │
      │        │              ▼
      │        │         Alert → Kafka
      │        │              │
      ▼        ▼              ▼
 PostgreSQL  Anomaly       DB Writer
 Timescale   detection         │
      │                         ▼
      └──────────────► DATABASE
                             │
                             ▼
                       FastAPI Backend
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
                 REST API         WebSocket
                    │                 │
                    ▼                 ▼
                  React Dashboard
```

இதுதான் project-ன் backbone.

---

# 2. Project-ல் முக்கியமான technologies

| Layer | Technology |
|---|---|
| Frontend | React 18 |
| Frontend build | Vite |
| Backend API | Python FastAPI |
| IoT protocol | MQTT |
| Message streaming | Apache Kafka |
| Database | PostgreSQL / TimescaleDB |
| Real-time browser stream | WebSocket |
| Monitoring | Prometheus |
| Visualization/infra monitoring | Grafana |
| Containerization | Docker Compose |
| Authentication | JWT / cookie-based session |
| Authorization | RBAC |
| Charts | Chart.js |
| State management | Zustand |
| Reports | jsPDF / XLSX |

Frontend dependency setup itself confirms React, React Router, Chart.js, Zustand, Vite, XLSX and jsPDF usage.

---

# 3. Project structure

Main project:

```text
CK-26/
└── caleido-kombos/
    │
    ├── frontend/
    │
    ├── iotBackend/
    │
    ├── infra/
    │
    ├── config/
    │
    ├── docs/
    │
    └── deploy/
```

இதில் முக்கியமான இரண்டு:

```text
frontend/
    ↓
User Interface

iotBackend/
    ↓
Actual IoT + API backend
```

---

# 4. FRONTEND என்ன செய்கிறது?

Frontend React application.

Structure:

```text
frontend/src/
│
├── App.jsx
├── api/
├── components/
├── context/
├── pages/
├── services/
├── store/
└── utils/
```

### `App.jsx`

Application routing.

Main flow:

```text
/login
   ↓
LoginPage

/change-password
   ↓
ChangePasswordPage

/*
   ↓
Authentication required
   ↓
MainLayout
```

Unauthenticated user dashboard-க்கு போக முடியாது.

---

# 5. Frontend pages என்னென்ன?

Project-ல் பல business screens இருக்கின்றன:

### Dashboard / monitoring

- `OverviewPage`
- `FleetPage`
- `RealtimePage`
- `ChartsPage`
- `HistoricalReportPage`
- `HistoricalDeviceDrillDown`
- `DeviceAlertsPage`
- `BHIPage`

### Device management

- `DeviceOnboardingPage`
- `CommissionPage`
- `DecommissionPage`
- `FirmwareManagementPage`

### Customer/site

- `ManageCustomersSitesPage`
- `ManageStakeholdersPage`
- `SiteListViewerPage`
- `NewClientEngagementInitializationPage`

### User/security

- `AllUsersPage`
- `AddUserPage`
- `EditUserPage`
- `RBACPage`
- `ChangePasswordPage`

### Service

- `ServiceRequestPage`
- `ServiceRequestHistoryPage`
- `TroubleshootingPage`

இதனால் இது வெறும் “IoT graph dashboard” இல்லை.

**Device lifecycle + customer management + RBAC + monitoring + alerting + service management** எல்லாம் சேர்ந்து இருக்கிறது.

---

# 6. FRONTEND → BACKEND எப்படி பேசுகிறது?

React directly database-ஐ access பண்ணாது.

Wrong:

```text
React → PostgreSQL ❌
```

Correct:

```text
React
  ↓
HTTP API
  ↓
FastAPI
  ↓
PostgreSQL
```

Realtime data:

```text
IoT
 ↓
MQTT
 ↓
Backend
 ↓
WebSocket
 ↓
React
```

---

# 7. Backend என்ன?

Backend main folder:

```text
iotBackend/
```

இதற்குள்:

```text
core/
common/
commons/
consumers/
realtime/
services/
monitoring/
conf/
scripts/
containers/
```

---

# 8. `core/` என்ன செய்கிறது?

இது backend infrastructure layer.

Important files:

```text
MqttClient.py
MqttPayloadParser.py
KafkaProducerClient.py
ConfigurationManager.py
Settings.py
VaultReader.py
ThresholdState.py
AlertRuleEngine.py
MetricsCollector.py
```

இதன் job:

- MQTT connect
- MQTT message receive
- payload parse
- Kafka publish
- configuration
- credentials
- threshold logic
- metrics

---

# 9. MQTT என்ன?

IoT device-கள் lightweight protocol-ஆக MQTT பயன்படுத்துகிறது.

இந்த project-ல் topics:

```text
hdt
pdt
lwt
at
```

Meaning roughly:

```text
HDT → Health data
PDT → Performance/parameter data
LWT → Connection lifecycle
AT  → Alert
```

---

# 10. Example: ஒரு device data அனுப்புகிறது

Suppose Mikos device:

```text
Device UID:
0302110000000001
```

ஒரு reading:

```text
temperature = 45
voltage = 230
current = 4.2
power_factor = 0.95
active_power = 966
```

Device:

```text
Mikos
   │
   │ MQTT
   ▼
hdt / pdt
```

---

# 11. `MqttPayloadParser`

Raw MQTT payload usually nested format.

Backend அதை normalize பண்ணும்.

Conceptually:

```text
RAW MQTT
{
   payload:
   {
      sender_uid,
      data: [
         {
            device_uid,
            device_data: [...]
         }
      ]
   }
}
```

இதிலிருந்து backend:

```text
device_uid
timestamp
payload
source_topic
message_id
source_timestamp
...
```

போன்ற normalized message உருவாக்கும்.

---

# 12. MQTT → Kafka

இதுதான் project-ல் மிகவும் முக்கியமான architecture.

```text
MQTT
 ↓
Parser
 ↓
Kafka Producer
 ↓
Kafka Topic
```

Kafka topics:

```text
iot.sensor.hdt
iot.sensor.pdt
iot.sensor.lwt
iot.sensor.at
```

---

# 13. ஏன் Kafka?

நேரடியாக:

```text
MQTT → Database
```

என்று செய்யலாம்.

ஆனால் project அதைப் பயன்படுத்தவில்லை.

Instead:

```text
MQTT
 ↓
Kafka
 ↓
Multiple consumers
```

இதனால் ஒரே message-ஐ multiple systems independently process செய்ய முடியும்.

Example:

```text
HDT message
    │
    ▼
Kafka
 ┌──┼──────────────┐
 ▼  ▼              ▼
DB  ML       Dashboard/other
```

ஒரு consumer fail ஆனாலும் மற்ற consumer architecture-ஆக independent-ஆக இருக்க முடியும்.

---

# 14. Kafka consumers

Current codebase-ல் முக்கியமான consumers:

### 1. `DbWriterConsumer`

Database write.

### 2. `DashboardConsumer`

Realtime/dashboard message pipeline.

### 3. `MlPipelineConsumer`

Anomaly detection.

### 4. `ThresholdAlertConsumer`

Configured limit breach detection.

---

# 15. `DbWriterConsumer` - மிக முக்கியம்

இதன் job:

```text
Kafka
 ↓
DbWriterConsumer
 ↓
PostgreSQL
```

Topic அடிப்படையில் dispatch:

```text
hdt → insp_device_health

pdt → insp_device_performance

lwt → insp_device_connection

at → insp_device_alert
```

இதுதான் நீ screenshot-ல் பார்த்த tables-க்கு data வருவதற்கான முக்கிய route.

---

# 16. Database - `insp_iot`

நீ VS Code-ல் பார்த்தது:

```text
192.168.0.6:5432
        ↓
    insp_iot
        ↓
      public
        ↓
    Tables (24)
```

இந்த PostgreSQL database project-ன் main new IoT database.

---

# 17. Database tables - பெரிய picture

24 tables-ஐ 6 groups-ஆ நினைச்சுக்கோ:

```text
1. ORGANIZATION
2. USER / RBAC
3. DEVICE LIFECYCLE
4. TELEMETRY
5. ALERT / INTELLIGENCE
6. SERVICE MANAGEMENT
```

---

# 18. Organization tables

### `insp_support_tier`

Customer support level.

```text
Tier 1
Tier 2
Tier 3
```

---

### `insp_customer`

Customer/company.

```text
Customer
   ↓
ABC Company
```

---

### `insp_site`

Customer-ன் physical site.

```text
Customer
   ↓
Site
```

Relationship:

```text
Customer
 ├── Site 1
 ├── Site 2
 └── Site 3
```

---

# 19. User + RBAC

### `insp_user`

Application users.

```text
User
 ↓
Role
 ↓
Permissions
```

---

### `insp_role`

Roles.

Project seeded roles include:

```text
Client Site User
Client Site Manager
Client Admin
INSP Back Office
INSP Technician
INSP Admin
```

---

### `insp_access_right`

Individual permission.

Example:

```text
VIEW_DEVICE
VIEW_ALERT
EDIT_DEVICE
MANAGE_USER
```

---

### `insp_role_access_right`

Mapping:

```text
Role ↔ Permission
```

Example:

```text
Admin
 ├── View device
 ├── Edit device
 ├── Manage user
 └── View alerts
```

---

# 20. Device management

### `insp_device`

**Master device table.**

இதுதான் device identity.

Fields include things such as:

```text
device_id
device_uid
serial_number
device_name
device_type
node_type
parent_device_id
is_gateway
appliance
manufacturer
model
firmware_version
mac_address
ip_address
comm_protocol
network_type
site_id
status
```

இதில் மிக முக்கியமான concept:

```text
device_id
```

vs

```text
device_uid
```

`device_id` = internal PostgreSQL identity.

`device_uid` = IoT device identity coming from MQTT.

---

# 21. Hub hierarchy

`insp_device` itself supports parent-child relationship.

```text
IntelliHub
   │
   ├── AirQ
   ├── Mikos
   └── Kleio
```

`parent_device_id` இதற்காக.

---

# 22. `insp_device_assignment`

Device எந்த customer/site-க்கு assigned?

```text
Device
   ↓
Customer
   ↓
Site
```

இதுதான் mapping.

---

# 23. `insp_device_inventory`

Physical device inventory.

Example:

```text
Received
Warehouse
PO
Lot
Bin
Condition
Quantity
```

இது sensor telemetry இல்லை.

---

# 24. Device state

### `insp_device_state_type`

Possible lifecycle states:

```text
In inventory
Onboarded
Commissioned
Maintenance
Decommissioned
Retired
```

### `insp_device_state`

ஒரு actual device அந்த state-ல் எப்போது இருந்தது.

```text
Device
 ↓
Commissioned
 ↓
Maintenance
 ↓
Decommissioned
```

---

# 25. Telemetry tables

இவை **actual IoT data**.

### `insp_device_health`

HDT data.

Example:

```text
device_uid
health_status
temperature
recorded_at
received_at
source_timestamp
message_id
```

---

### `insp_device_performance`

PDT data.

இந்த table-ல் `param_data` JSONB.

Example:

```json
{
  "voltage": 230,
  "current": 4.2,
  "active_power": 966,
  "frequency": 50,
  "power_factor": 0.95
}
```

இதனால் different device types-க்கு different parameters flexible-ஆ store செய்ய முடியும்.

---

### `insp_device_connection`

LWT / connection events.

```text
CONNECTED
DISCONNECTED
```

Device online/offline history.

---

# 26. Why TimescaleDB?

Telemetry:

```text
10:00:01
10:00:02
10:00:03
10:00:04
...
```

இப்படி millions of time-based records வரலாம்.

அதனால் PostgreSQL + TimescaleDB style time-series storage use பண்ணுவது logical.

இந்த tables-ல் `time` column முக்கியமானது.

---

# 27. `insp_device_alert`

Alert records.

இதில்:

```text
device
alert_type
severity
alert_data
timestamp
message_id
```

போன்ற data இருக்கும்.

Flow:

```text
Sensor data
    ↓
Rule/ML
    ↓
Alert
    ↓
insp_device_alert
```

---

# 28. `insp_limit_config`

இதுதான் **configured threshold**.

Example:

```text
Device: Mikos
Metric: voltage

LOW  = 200
HIGH = 250
```

Actual value:

```text
270V
```

Then:

```text
270 > 250
   ↓
High threshold breach
```

---

# 29. Threshold Alert Engine

Current implementation-ல் இது real functionality.

Flow:

```text
HDT/PDT
   ↓
ThresholdAlertConsumer
   ↓
insp_limit_config
   ↓
Compare value
   ↓
Breach?
   ↓
Alert
   ↓
Kafka AT topic
   ↓
DbWriterConsumer
   ↓
insp_device_alert
```

இதில் ஒரு நல்ல engineering detail இருக்கு:

**ஒவ்வொரு reading-க்கும் repeated alert generate ஆகாமல் hysteresis/state tracking பயன்படுத்துகிறது.**

Example:

```text
Voltage = 270
Limit = 250

270 → alert
271 → no duplicate alert
272 → no duplicate alert
...
240 → normal
270 → new alert
```

---

# 30. ML pipeline

Current `MlPipelineConsumer` actual sophisticated trained ML model இல்லை.

இது:

### Rolling window + Z-score

பயன்படுத்துகிறது.

Example:

```text
Previous temperatures:

40
41
42
41
40
42
41
40
...
```

Current:

```text
70
```

Calculate:

```text
mean
std
z-score
```

If:

```text
abs(z-score) > 3.0
```

then anomaly log செய்யும்.

Config:

```text
windowSize = 100
minWindow = 20
anomalyThreshold = 3.0
```

---

# 31. Important distinction: Threshold ≠ ML anomaly

இரண்டும் வேறு.

### Threshold

Known limit:

```text
Voltage > 250
```

Straight rule.

### Anomaly detection

Historical behavior:

```text
Normally:
220, 221, 219, 223...

Suddenly:
248
```

Even if 248 is below configured max, it can still be statistically unusual.

So:

```text
Threshold
   ↓
"Did it cross a known limit?"

ML anomaly
   ↓
"Is this behavior unusual compared to recent behavior?"
```

---

# 32. BHI - Building Health Index

இந்த project-ல் building-level health score இருக்கிறது.

Three components:

```text
IEI
PQI
DSI
```

### IEI

AirQ side.

Environmental health.

Uses things like:

- AQI
- humidity
- pressure

---

### PQI

Mikos side.

Power/electrical quality.

Uses:

- voltage
- frequency
- power factor
- active power
- temperature

---

### DSI

Kleio/device/security side.

Uses:

- battery
- lock status
- temperature

---

# 33. BHI calculation

Current code:

```text
BHI =
    IEI × 40%
  + PQI × 35%
  + DSI × 25%
```

Status:

```text
80-100 → Excellent
60-79  → Good
40-59  → Moderate
20-39  → Poor
0-19   → Critical
```

Then:

```text
insp_building_health
```

table-ல் store ஆகும்.

---

# 34. Dashboard API

Backend API:

```text
FastAPI
```

Important router groups:

```text
/api/auth
/api/users
/api/rbac
/api/devices
/api/alerts
/api/telemetry
/api/charts
/api/historical
/api/bhi
/api/limit-configs
/api/service-requests
/api/customers
/api/sites
/api/stakeholders
```

---

# 35. Device API

Examples:

```text
GET /api/devices
```

→ all devices.

```text
GET /api/devices/hubs
```

→ hubs.

```text
POST /api/devices
```

→ commission new device.

```text
PUT /api/devices/{id}/decommission
```

→ decommission.

So frontend:

```text
FleetPage
    ↓
deviceApi.js
    ↓
GET /api/devices
    ↓
FastAPI
    ↓
insp_device
```

---

# 36. Telemetry API

```text
GET /api/telemetry/latest
```

Latest readings.

```text
GET /api/telemetry/device
```

Specific device telemetry.

Flow:

```text
React
 ↓
FastAPI
 ↓
insp_device_performance
 ↓
JSON
 ↓
React chart/card
```

---

# 37. Historical API

Historical pages can query:

```text
/api/historical
/api/historical/range
/api/historical/alerts
/api/historical/drilldown
```

இதனால் user:

```text
Today
Yesterday
Last 7 days
Custom range
```

போன்ற historical data பார்க்க முடியும்.

---

# 38. Chart APIs

Different device types:

```text
/api/charts/air-quality
/api/charts/power
/api/charts/lock
/api/charts/hub
```

இதனால் frontend Chart.js components data பெறும்.

---

# 39. WebSocket realtime

இந்த project-ன் realtime feature:

```text
IoT device
 ↓
MQTT
 ↓
FastAPI MQTT listener
 ↓
WebSocket
 ↓
Browser
```

Endpoint:

```text
/api/stream
```

இதனால் page refresh இல்லாமல் live data dashboard-ல் update ஆக முடியும்.

---

# 40. Authentication

Login:

```text
React LoginPage
      ↓
POST /api/token
      ↓
FastAPI
      ↓
insp_user
      ↓
Password verification
      ↓
Token/session
      ↓
React AuthContext
```

After login:

```text
/api/me
```

current user information.

---

# 41. RBAC

Frontend-ல் user features fetch செய்கிறது:

```text
/api/rbac/my-features
```

Admin changes permissions:

```text
/api/rbac/grant
/api/rbac/revoke
```

மேலும் frontend roughly every **60 seconds** feature grants refresh செய்கிறது.

அதனால் admin permission change செய்தால் user logout/login இல்லாமலும் UI permissions update ஆகும்.

---

# 42. Service Request system

Problem:

```text
Device problem
    ↓
Service Request
```

Tables:

```text
insp_service_request
insp_issue_type
insp_priority
insp_ticket_status
insp_sr_status_history
insp_support_tier
```

Example:

```text
Device failure
    ↓
Issue Type = Electrical
    ↓
Priority = Critical
    ↓
Ticket = Open
    ↓
Assigned
    ↓
In Progress
    ↓
Resolved
    ↓
Closed
```

---

# 43. `insp_pipeline_metric`

இது IoT device metric இல்லை.

Backend pipeline monitoring.

Example:

```text
Consumer
Messages received
Messages processed
DB writes
Failures
```

இதனால்:

> “நம்ம backend data processing pipeline itself healthy-ஆ இருக்கா?”

என்று பார்க்க முடியும்.

---

# 44. Monitoring infrastructure

Docker Compose-ல்:

```text
Zookeeper
Kafka
Kafka UI
pgAdmin
Prometheus
Pushgateway
Grafana
```

### Kafka UI

Kafka topics/messages inspect.

### pgAdmin

PostgreSQL database inspect.

### Prometheus

Metrics collect.

### Grafana

Metrics visualization.

---

# 45. Docker architecture

Conceptually:

```text
Docker Compose
│
├── Zookeeper
│
├── Kafka
│
├── Kafka UI
│
├── pgAdmin
│
├── Prometheus
│
├── Pushgateway
│
└── Grafana
```

Backend/frontend are separately launched according to the project setup.

---

# 46. Security

Project-ல் secrets directly source code-ல் வைத்திருக்காமல்:

```text
KeePass vault
```

use செய்யும் architecture இருக்கு.

Examples:

```text
POSTGRES_IOT
KAFKA_IOT
INSP_MYSQL
```

போன்ற credentials vault மூலம் load செய்யப்படுகின்றன.

Authentication side:

```text
password
 ↓
hash
 ↓
verify
 ↓
JWT/session
```

Authorization:

```text
User
 ↓
Role
 ↓
Access Rights
```

---

# 47. UI architecture

UI itself intentionally **enterprise monitoring dashboard** style.

Visual system:

```text
Dark Navy
+
Glass surfaces
+
Indigo
+
Cyan
+
Emerald / Amber / Rose status colors
+
Inter font
+
Compact typography
+
Thin borders
+
Soft glow
```

The extracted UI reference confirms Inter as primary font, dark navy backgrounds, glass surfaces, indigo/cyan accents and semantic status colors. `reusable_ui_design_system.md`

Sidebar active state uses indigo, subtle glow, rounded navigation items, while hover interactions are deliberately small rather than flashy. `reusable_ui_design_system.md`

Cards use translucent surfaces, thin borders and a small `translateY(-2px)` hover movement with indigo glow. `reusable_ui_design_system.md`

---

# 48. Full data flow - இதை memorize பண்ணு

ஒரு real device reading எடுத்துக்கலாம்.

```text
                    DEVICE
                      │
                      │ MQTT
                      ▼
                MQTT BROKER
                      │
                      ▼
             MQTT PAYLOAD PARSER
                      │
                      ▼
                    KAFKA
                      │
        ┌─────────────┼──────────────┐
        │             │              │
        ▼             ▼              ▼
    DB WRITER      ML PIPELINE   THRESHOLD ENGINE
        │             │              │
        │             │              │
        ▼             ▼              ▼
  PostgreSQL       Z-score       limit_config
        │          anomaly            │
        │                              │
        │                         breach?
        │                              │
        │                              ▼
        │                         ALERT MESSAGE
        │                              │
        │                              ▼
        │                            KAFKA
        │                              │
        │                              ▼
        │                         DB WRITER
        │                              │
        └───────────────┬──────────────┘
                        ▼
                  PostgreSQL
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
        FastAPI                 WebSocket
            │                       │
            ▼                       ▼
         REST API              Live stream
            │                       │
            └──────────┬────────────┘
                       ▼
                 REACT DASHBOARD
```

**இதுதான் CK-26.**

---

# 49. ஒரு real example

Suppose:

```text
Mikos
Voltage = 280V
Configured limit = 250V
```

### Step 1

Mikos sends:

```text
PDT
voltage = 280
```

### Step 2

MQTT receives.

### Step 3

Parser normalize செய்கிறது.

### Step 4

Kafka:

```text
iot.sensor.pdt
```

### Step 5

Two things happen parallel:

```text
DB Writer
    ↓
insp_device_performance
```

and:

```text
Threshold Engine
    ↓
280 > 250
    ↓
HIGH_THRESHOLD_BREACH
```

### Step 6

Alert Kafka:

```text
iot.sensor.at
```

### Step 7

DB Writer:

```text
insp_device_alert
```

### Step 8

FastAPI:

```text
/api/alerts
```

### Step 9

React:

```text
Device Alerts
   ↓
CRITICAL / HIGH
Voltage
280V
Limit 250V
```

User dashboard-ல் problem பார்க்கிறார்.

---

# 50. இன்னொரு example - ML anomaly

Suppose Mikos normally:

```text
229
231
230
232
229
231
230
```

Suddenly:

```text
255
```

But configured threshold:

```text
LOW  = 200
HIGH = 260
```

So threshold engine says:

```text
255 < 260
→ No threshold breach
```

But ML rolling statistics:

```text
255 is unusual
→ high z-score
→ anomaly
```

இதுதான் **rule-based detection vs statistical anomaly detection** difference.

---

# 51. Database relationships - final mental model

```text
CUSTOMER
   │
   ▼
 SITE
   │
   ▼
 DEVICE
   │
   ├──────────────► ASSIGNMENT
   │
   ├──────────────► INVENTORY
   │
   ├──────────────► STATE
   │
   ├──────────────► HEALTH
   │
   ├──────────────► PERFORMANCE
   │
   ├──────────────► CONNECTION
   │
   └──────────────► ALERT
                         │
                         ▼
                    BUILDING HEALTH
```

Separate:

```text
USER
 ↓
ROLE
 ↓
ACCESS RIGHT
```

Service:

```text
SERVICE REQUEST
 ├── ISSUE TYPE
 ├── PRIORITY
 ├── TICKET STATUS
 ├── SUPPORT TIER
 └── STATUS HISTORY
```

---

# 52. The 24 PostgreSQL tables - one-line meaning

| Table | Simple meaning |
|---|---|
| `insp_support_tier` | Support levels |
| `insp_customer` | Customers |
| `insp_role` | User roles |
| `insp_site` | Customer sites |
| `insp_user` | Application users |
| `insp_device` | Device master/identity |
| `insp_device_state_type` | Possible lifecycle states |
| `insp_device_state` | Device state history |
| `insp_device_inventory` | Physical inventory |
| `insp_device_assignment` | Device ↔ customer/site |
| `insp_device_health` | HDT health data |
| `insp_device_performance` | PDT sensor/performance data |
| `insp_device_connection` | LWT connection events |
| `insp_device_alert` | Generated alerts |
| `insp_pipeline_metric` | Backend pipeline metrics |
| `insp_building_health` | IEI/PQI/DSI/BHI |
| `insp_access_right` | Permissions |
| `insp_role_access_right` | Role ↔ permission |
| `insp_issue_type` | Service issue categories |
| `insp_priority` | Ticket priority |
| `insp_ticket_status` | Ticket states |
| `insp_service_request` | Service tickets |
| `insp_sr_status_history` | Ticket status history |

---

# 53. One important thing I would NOT misunderstand

There are **two different architectural generations/documentation layers** inside this project.

### Older documented ML architecture

```text
Extract
 ↓
Transform
 ↓
Feature Engineer
 ↓
Model Train
 ↓
Anomaly Detect
 ↓
Prometheus
 ↓
Grafana
```

That is what the older architecture documentation describes.

### Current implementation

```text
MQTT
 ↓
Kafka
 ├── DB Writer
 ├── Threshold Alert
 ├── ML Pipeline
 └── Dashboard
       ↓
   PostgreSQL
       ↓
    FastAPI
       ↓
     React
```

And current ML consumer is **rolling Z-score**, not the old documented Isolation Forest service.

So if somebody asks you in an interview:

> “Does this project currently use Isolation Forest?”

**Don't blindly say yes.**

The old documentation mentions Isolation Forest, but the current `MlPipelineConsumer` implementation you have is doing rolling statistical/Z-score anomaly detection.

That distinction matters.

---

# 54. What is the actual purpose of CK-26?

In one sentence:

> **CK-26 is an enterprise IoT platform that ingests live device telemetry through MQTT/Kafka, persists it in PostgreSQL/TimescaleDB, performs rule-based threshold and statistical anomaly detection, calculates building health, exposes the information through FastAPI APIs/WebSockets, and presents it through a React monitoring and device-management dashboard.**

If you understand these **8 blocks**, you understand the project:

```text
1. IoT Devices
       ↓
2. MQTT
       ↓
3. Kafka
       ↓
4. Consumers
       ↓
5. PostgreSQL / TimescaleDB
       ↓
6. FastAPI
       ↓
7. WebSocket / REST
       ↓
8. React Dashboard
```

And around that core:

```text
Authentication
RBAC
Device Lifecycle
Customer/Site Management
Alerts
BHI
Service Requests
Monitoring
```

That is the **full CK-26 mental model**.



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)