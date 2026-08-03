I have rewritten the README with the shared Grill Low/High Offset variable behavior included. I also tightened a few sections so it matches the current code more closely (especially the Hub Variable behavior and alert reset behavior).

# FireBoard Monitor Companion App

## Hubitat Groovy Application

**Author:** Tray E.
**Namespace:** tweas
**Version:** 1.0.0

---

# Overview

The FireBoard Monitor Companion App provides advanced FireBoard temperature monitoring and notifications within Hubitat.

The app is designed to replace multiple individual rules by handling FireBoard monitoring, temperature alerts, notifications, and cook workflow management.

Current features:

* Grill temperature monitoring
* Multiple grill temperature probes
* Multiple food temperature probes
* Grill ready notifications
* Grill low temperature alerts
* Grill high temperature alerts
* Food flip/stall alerts
* Food target temperature alerts
* Automatic flip temperature calculation
* Manual flip/stall temperature override
* Hub Variable integration
* Editable notification messages
* Phone notifications
* Text-to-speech notifications
* FireBoard API-aware polling

---

# FireBoard Configuration Requirements

The app controls FireBoard polling directly.

The FireBoard driver should be configured as follows:

```
Refresh Rate:
Manual

Drive Logging:
Disabled
```

The app performs:

* Temperature updates using `GetTemperature()`
* Full refresh operations using `refresh()`

The app monitors FireBoard API usage and will warn if polling settings exceed recommended limits.

---

# Required Devices

## FireBoard Device

Required.

The device must be created by the FireBoard Hubitat driver.

Required capability:

```
switch
```

This device is used for FireBoard communication and refresh operations.

Example:

```
FireBoard 2 Pro
```

---

# Temperature Sensors

Temperature sensors must be created by the FireBoard driver.

Required capability:

```
temperatureMeasurement
```

The same sensor cannot be assigned to both Grill and Food monitoring.

---

# Grill Temperature Sensors

Optional.

Used for:

* Grill reached target temperature alert
* Grill low temperature alert
* Grill high temperature alert

Multiple grill sensors can be selected.

Examples:

```
Big Green Egg Dome
Smoker Chamber
Pizza Oven
```

---

# Food Temperature Sensors

Optional.

Used for:

* Food flip/stall alerts
* Food target temperature alerts

Multiple food probes can be selected.

Examples:

```
Brisket
Chicken
Pork Shoulder
Steak
```

---

# Required Virtual Switches

## Start Monitoring Switch

Required.

Capability:

```
switch
```

Purpose:

Starts and stops monitoring.

When turned ON:

* Initializes the cook
* Captures starting temperatures
* Calculates automatic flip temperatures
* Starts FireBoard polling

When turned OFF:

* Stops polling
* Cancels scheduled polling
* Clears active cook state

Example:

```
Green Egg Monitor FireBoard
```

---

## Calculate Flip Switch

Optional.

Capability:

```
switch
```

Purpose:

Manually recalculates food flip temperatures.

Typical workflow:

1. Start monitoring
2. Wait for grill temperature to stabilize
3. Place food on the grill
4. Turn ON Calculate Flip

The app calculates:

```
Flip Temperature =
Current Temperature +
((Target Temperature - Current Temperature) / 2)
```

Example:

```
Current food temperature:
40°F

Target:
205°F

Calculated flip temperature:
122.5°F
```

The switch automatically turns OFF after calculation.

---

## Clear Alerts Switch

Optional.

Capability:

```
switch
```

Purpose:

Resets alert history without stopping monitoring.

Clears:

* Grill target alert
* Grill low alert
* Grill high alert
* Food flip alert
* Food done alert

Useful when:

* Restarting a cook
* Moving probes
* Testing notifications
* Manually starting a new alert cycle

The monitoring process continues.

---

# Temperature Configuration

The app supports two methods for temperature settings.

## Option 1: App Settings

Temperature values are stored directly in the app.

Includes:

* Grill target temperature
* Grill low offset
* Grill high offset
* Food target temperature
* Food flip/stall temperature

---

## Option 2: Hub Variables

Hub Variables allow temperature settings to be changed from dashboards or rules without opening the app.

Supported Hub Variable types:

```
Integer

BigDecimal
```

Recommended:

Use numeric Hub Variables.

---

# Hub Variable Assignments

## Grill Target Temperature Variable

Required when using Hub Variables.

Type:

```
Integer
or
BigDecimal
```

Example:

```
Name:
Grill Target Temp

Value:
450
```

Used for:

* Grill ready alert
* Grill low/high calculations

---

## Grill Low Offset Variable

Optional.

Type:

```
Integer
or
BigDecimal
```

Defines how far below the grill target temperature the low alert occurs.

Example:

```
Target:
450°F

Low Offset:
50°F

Low Alert:
400°F
```

---

## Grill High Offset Variable

Optional.

Type:

```
Integer
or
BigDecimal
```

Defines how far above the grill target temperature the high alert occurs.

Example:

```
Target:
450°F

High Offset:
50°F

High Alert:
500°F
```

---

## Using the Same Variable for Grill Low and High Offset

The Grill Low Offset Variable and Grill High Offset Variable can use the same Hub Variable if the same offset value should always be used above and below the target temperature.

Example:

Hub Variable:

```
Name:
Grill Temperature Offset

Type:
Integer

Value:
50
```

Assigned as:

```
Grill Low Offset Variable:
Grill Temperature Offset

Grill High Offset Variable:
Grill Temperature Offset
```

Result:

```
Grill Target:
450°F

Low Alert:
400°F

High Alert:
500°F
```

This allows a single dashboard control to adjust both limits.

Separate variables can be used when different offsets are desired.

Example:

```
Low Offset:
25°F

High Offset:
75°F
```

Result:

```
Target:
450°F

Low Alert:
425°F

High Alert:
525°F
```

---

## Food Target Temperature Variable

Required when using Hub Variables.

Type:

```
Integer
or
BigDecimal
```

Example:

```
Name:
Brisket Target

Value:
205
```

Used for:

* Food complete alert

---

## Food Flip/Stall Temperature Variable

Optional.

Type:

```
Integer
or
BigDecimal
```

If provided:

The app uses this temperature.

If not provided:

The app automatically calculates the flip temperature.

Example:

```
Brisket Stall Temperature:

165°F
```

---

# Notifications

## Phone Notifications

Optional.

Required capability:

```
notification
```

Used for:

* Grill alerts
* Food alerts
* Flip alerts

---

## Text-to-Speech Speakers

Optional.

Required capability:

```
speechSynthesis
```

Used for spoken alerts.

Examples:

```
Alexa
Google Home
Hubitat compatible speakers
```

---

# Custom Notification Messages

All messages are editable.

Supported replacement fields:

## Grill Target

```
%sensor%
%current%
%target%
```

Example:

```
Big Green Egg has reached 450°.
```

---

## Grill Low

```
%sensor%
%current%
%target%
%low%
```

---

## Grill High

```
%sensor%
%current%
%target%
%high%
```

---

## Food Flip

```
%sensor%
%current%
%flip%
%target%
```

---

## Food Done

```
%sensor%
%current%
%target%
```

---

# Alert Behavior

## Grill Target Alert

The grill target alert is intended as a "grill ready" notification.

Example:

```
Grill Target:
450°F
```

When the grill reaches:

```
450°F or higher
```

The alert is sent once.

The alert resets when:

* Monitoring is restarted
* The grill target temperature changes
* Clear Alerts is pressed

The target alert does not continuously reset when temperature moves above or below the target because the purpose is to tell you when the grill is ready for food.

---

## Grill Low/High Alerts

These alerts use hysteresis.

Low alert:

Triggers:

```
Temperature <= Low Limit
```

Resets:

```
Temperature rises above Low Limit
```

High alert:

Triggers:

```
Temperature >= High Limit
```

Resets:

```
Temperature falls below High Limit
```

---

## Food Alerts

Food alerts are immediate.

Flip/stall alert:

Triggers when:

```
Current Temperature >= Flip Temperature
```

Done alert:

Triggers when:

```
Current Temperature >= Target Temperature
```

---

# Polling Configuration

## Poll Frequency

Controls how often temperature values are checked.

Example:

```
20 seconds
```

---

## Refresh Interval

Controls how often a full FireBoard refresh occurs.

Example:

```
5 minutes
```

The app calculates API usage based on these settings.

---

# Typical Cooking Workflow

Example:

## Brisket Cook

1. Set grill target:

```
250°F
```

2. Turn ON:

```
Green Egg Monitor FireBoard
```

3. Wait for:

```
Grill Ready Alert
```

4. Place brisket on grill.

5. Turn ON:

```
Calculate Flip
```

6. App calculates flip/stall temperature.

7. Receive:

```
Flip/Stall Alert
```

8. Continue cooking.

9. Receive:

```
Food Done Alert
```

---

# Troubleshooting

## No Alerts

Verify:

* Monitoring switch is ON
* Notification devices are selected
* Notifications are enabled
* Sensors are assigned correctly

---

## Polling Stops

Check:

* FireBoard API warning
* Poll interval
* Refresh interval
* FireBoard driver settings

---

## Hub Variable Changes Do Not Apply

Verify:

* Variable still exists
* Variable type is numeric
* App settings reference the correct variable

---

# Current Limitations

The current version does not include:

* Cook history logging
* Graphing
* Multiple independent cook profiles
* Estimated completion time
* Cloud synchronization

---

# Version History

## Version 1.0.0

Initial release.

Included:

* Grill monitoring
* Multiple food probes
* Flip calculations
* Hub Variables
* Custom notifications
* TTS support
* FireBoard API-aware polling
* Alert management
