# MIKOS Smart Energy Monitoring System -- Complete A to Z Explanation (Tanglish)

> **Based on the uploaded MIKOS Complete Master Reference Guide.**

## 1. What is MIKOS?

MIKOS (Smart Energy Monitoring System) is an AI + IoT based platform
that continuously monitors electrical appliances like Air Conditioners,
Water Heaters, and Water Pumps.

Its main goals are:

-   Automatically identify appliances
-   Detect faults instantly
-   Predict failures before they happen
-   Schedule preventive maintenance
-   Recommend energy optimization
-   Reduce electricity cost
-   Increase equipment lifespan

Instead of only showing energy consumption, MIKOS converts raw
electrical data into actionable intelligence.

------------------------------------------------------------------------

# 2. Core Technologies

-   IoT Sensors
-   Machine Learning
-   Data Analytics
-   Cloud Database
-   Dashboard & Alerts

------------------------------------------------------------------------

# 3. Sensor Parameters

MIKOS continuously monitors:

1.  Voltage
2.  Current
3.  Active Power
4.  Reactive Power
5.  Apparent Power
6.  Power Factor
7.  Frequency
8.  Relay Status
9.  Relay Operations
10. Temperature

These parameters are used by every intelligence layer.

------------------------------------------------------------------------

# 4. Overall Working Flow

Appliance ON

↓

MIKOS Sensor collects electrical parameters

↓

IoT sends data to server

↓

Machine Learning identifies appliance

↓

Four Intelligence Layers analyze data

↓

Dashboard displays alerts, predictions and optimization recommendations.

------------------------------------------------------------------------

# 5. Seven Layer Architecture

## Layer 1 -- Application Layer

Dashboard, Mobile App, Reports, Notifications.

## Layer 2 -- Presentation Layer

Mind Maps, Flow Diagrams, UX representation.

## Layer 3 -- Session Layer

Software classes like Appliance, Device, Alert, Prediction.

## Layer 4 -- Sequence Layer

Sensor → Kafka → ML → Database → Dashboard.

## Layer 5 -- Network Layer

Database entities and ER Diagram.

## Layer 6 -- Data Link Layer

Sensor → Classification → Detection → Prediction → Optimization.

## Layer 7 -- Physical Layer

OFF → ON → IDLE → FAULT → MAINTENANCE.

------------------------------------------------------------------------

# 6. Four Layer Intelligence

## Layer 1 -- Anomaly Detection

Detects immediate faults like: - Overcurrent - Voltage fluctuation -
Overheating - Short Cycling

## Layer 2 -- Predictive Maintenance

Studies long-term trends.

Example: Current increases every week → Compressor likely to fail in 10
days.

## Layer 3 -- Preventive Maintenance

Schedules maintenance based on runtime.

Example: AC runtime \>1000 hours → Filter cleaning reminder.

## Layer 4 -- Prescriptive Optimization

Suggests best operating settings.

Example: Increase AC setpoint from 22°C to 24°C → \~18% energy savings.

------------------------------------------------------------------------

# 7. Air Conditioner Intelligence

## Anomaly Detection

### Overcurrent

Current \> Rated × 1.25

Possible causes: - Compressor jam - Gas pressure issue - Voltage
imbalance

### Low Power Factor

Power Factor \< 0.7

Usually indicates capacitor failure.

### Short Cycling

Repeated ON/OFF within a few minutes.

Causes: - Thermostat issue - Refrigerant issue - Oversized AC

### Voltage Instability

Voltage \<187V or \>253V.

------------------------------------------------------------------------

## Predictive Maintenance

Monitors: - Current increasing - Power Factor decreasing - Reactive
Power increasing

Predicts compressor failure before it happens.

------------------------------------------------------------------------

## Preventive Maintenance

Runtime \>1000 hours

→ Filter cleaning recommendation.

------------------------------------------------------------------------

## Prescriptive Optimization

Recommend increasing temperature setpoint to reduce energy consumption.

------------------------------------------------------------------------

# 8. Water Heater Intelligence

## Fault Detection

-   Overcurrent
-   Continuous heating (\>3 hours)

## Prediction

Heating time increases every month

→ Heating element degradation predicted.

------------------------------------------------------------------------

# 9. Water Pump Intelligence

## Dry Run Detection

Current \<60% of normal while relay is ON.

Immediate auto shutdown recommended.

## Blockage Detection

Current \>125% of rated.

## Predictive Maintenance

Current ↑

Temperature ↑

Power Factor ↓

→ Motor winding failure predicted.

## Optimization

Demand-based pump scheduling reduces runtime and energy usage.

------------------------------------------------------------------------

# 10. Business Benefits

-   Lower electricity consumption
-   Reduced maintenance cost
-   Increased equipment lifespan
-   Better operational reliability
-   Automated fault detection
-   Predictive maintenance
-   Intelligent optimization

------------------------------------------------------------------------

# 11. One-Line Summary

MIKOS is an AI-driven IoT platform that monitors electrical appliances
using real-time sensor data, detects faults instantly, predicts
failures, schedules maintenance, and optimizes energy consumption
through a seven-layer architecture and four-layer intelligence
framework.
