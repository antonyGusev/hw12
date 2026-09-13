# **PRD — IoT Smart Climate Controller**

## **1\. Загальний опис продукту**

**IoT Smart Climate Controller** — IoT-пристрій на базі ESP32 для автоматичного підтримання заданої температури та вологості в приміщенні.

Пристрій керує:

* кондиціонером у режимах `HEAT`, `COOL`, `DRY`, `OFF`;  
* системою зволоження;  
* автоматичним наповненням резервуара для води.

Система підтримує автоматичний і ручний режими, конфігуровані пороги, аварійні стани та віддалений моніторинг і керування.

## **2\. Призначення продукту**

Основне призначення системи — автоматично підтримувати комфортний мікроклімат у приміщенні без постійного втручання користувача.

Пристрій контролює температуру та вологість, керує виконавчими механізмами, дозволяє змінювати параметри роботи віддалено та використовується як цільова система для HIL/E2E автоматизованого тестування.

## **3\. Технічний стек**

* Main Controller:				ESP32  
* Firmware:					MicroPython  
* Temperature / Humidity Sensor:		SHT31  
* Sensor Interface:				I²C  
* Water Level Sensors:				LOW Level Sensor, FULL Level Sensor  
* Water Sensor Interface:			Digital GPIO  
* AC Control:					Infrared (IR)  
* Pump / Valve Control:				GPIO \+ Relay/MOSFET Driver  
* Wireless:					Wi-Fi 2.4 GHz  
* IoT Protocol:					MQTT  
* MQTT Client:					MicroPython `umqtt`  
* Configuration Storage:			ESP32 Non-Volatile Storage  
* Diagnostics:					UART / USB Serial

## **4\. Functional Requirements**

### **4.1 Default Configuration**

* Min Temperature:				20°C  
* Max Temperature:				25°C  
* Target Temperature:				22.5°C  
* Min Humidity:					40%RH  
* Max Humidity:					60%RH  
* Target Humidity:				50%RH  
* Measurement Interval:			5 s  
* WARNING Low Temperature:			\<16°C  
* WARNING High Temperature:		\>30°C  
* CRITICAL Low Temperature:			\<10°C  
* CRITICAL High Temperature:			\>35°C  
* WARNING Low Humidity:			\<30%RH  
* WARNING High Humidity:			\>70%RH  
* CRITICAL Low Humidity:			\<20%RH  
* CRITICAL High Humidity:			\>85%RH  
* Max Refill Time:				120 s

Target values are calculated automatically:

`Target Temperature = (Min Temperature + Max Temperature) / 2`

`Target Humidity = (Min Humidity + Max Humidity) / 2`

### **4.2 Climate Control Rules**

Temperature and humidity regulation are handled as two related but partially independent control channels.

#### **Temperature Control**

The air conditioner is responsible for temperature regulation.

**HEAT**

* Active only in `AUTO`.  
* Starts when `Temperature < Min Temperature`.  
* Continues until `Temperature >= Target Temperature`.  
* Default behavior: `<20°C → HEAT`, `>=22.5°C → HEAT OFF`.

**COOL**

* Active only in `AUTO`.  
* Starts when `Temperature > Max Temperature`.  
* Continues until `Temperature <= Target Temperature`.  
* Default behavior: `>25°C → COOL`, `<=22.5°C → COOL OFF`.

The air conditioner cannot operate in `HEAT`, `COOL`, and `DRY` simultaneously.

#### **Humidification Control**

The humidifier is controlled independently from `HEAT` and `COOL`.

**HUMIDIFY**

* Active only in `AUTO`.  
* Starts when `Humidity < Min Humidity`.  
* Continues until `Humidity >= Target Humidity`.  
* Default behavior: `<40%RH → HUMIDIFY`, `>=50%RH → HUMIDIFY OFF`.  
* May operate simultaneously with `HEAT`.  
* May operate simultaneously with `COOL`.  
* Must stop immediately when:  
  * `WATER_LOW` is active;  
  * reservoir refill is active;  
  * Safe State is active.

#### **Drying Control**

**DRY**

* Active only in `AUTO`.  
* Starts when `Humidity > Max Humidity` and neither `HEAT` nor `COOL` is required.  
* Continues until `Humidity <= Target Humidity`.  
* Default behavior: `>60%RH → DRY`, `<=50%RH → DRY OFF`.  
* `DRY` and `HUMIDIFY` must never operate simultaneously.  
* If temperature leaves the allowed range while `DRY` is active:  
  * stop `DRY`;  
  * re-evaluate temperature;  
  * start `HEAT` or `COOL` if required.

#### **Valid Combined States**

The following combinations are allowed:

* `HEAT + HUMIDIFY`  
* `COOL + HUMIDIFY`  
* `HEAT`  
* `COOL`  
* `DRY`  
* `HUMIDIFY`  
* `IDLE`

The following combination is not allowed:

* `DRY + HUMIDIFY`

#### **Priority Rules**

1. Safe State has the highest priority.  
2. When selecting an air-conditioner mode, temperature regulation (`HEAT` / `COOL`) has priority over `DRY`.  
3. `HUMIDIFY` is controlled independently and may run together with `HEAT` or `COOL`.  
4. `DRY` and `HUMIDIFY` are mutually exclusive.  
5. Reservoir refill may operate independently from climate regulation but blocks the humidifier while refill is active.

### **FR-01 — Temperature and Humidity Monitoring**

The system shall read temperature and humidity from SHT31.

* Default measurement interval: 5 seconds.  
* Configurable range: 1–60 seconds.  
* A valid interval change shall apply without reboot.  
* The configured interval shall be persisted in non-volatile storage.  
* Invalid values shall be rejected while preserving the previous valid value.

### **FR-02 — AUTO Mode**

In `AUTO` mode, the system shall automatically control temperature, humidity, and reservoir refill according to Section 4.2.

### **FR-03 — Heating**

When temperature is below `Min Temperature`, the system shall activate `HEAT` and keep it active until `Target Temperature` is reached.

Humidification may operate simultaneously when required.

### **FR-04 — Cooling**

When temperature is above `Max Temperature`, the system shall activate `COOL` and keep it active until `Target Temperature` is reached.

Humidification may operate simultaneously when required.

### **FR-05 — Drying**

When humidity is above `Max Humidity` and neither heating nor cooling is required, the system shall activate `DRY`.

`DRY` shall remain active until `Target Humidity` is reached.

If heating or cooling becomes necessary, `DRY` shall stop and the appropriate temperature-control mode shall start.

`DRY` and `HUMIDIFY` shall never be active simultaneously.

### **FR-06 — Humidification**

When humidity is below `Min Humidity`, the system shall activate the humidifier and keep it active until `Target Humidity` is reached.

Humidification may operate simultaneously with `HEAT` or `COOL`.

Humidification shall be blocked when:

* `WATER_LOW` is active;  
* refill is active;  
* Safe State is active.

### **FR-07 — Climate Control Coordination**

The system shall coordinate temperature and humidity control according to the following rules:

* `HEAT`, `COOL`, and `DRY` are mutually exclusive AC modes.  
* `HEAT` and `COOL` have priority over `DRY`.  
* `HUMIDIFY` may operate together with `HEAT` or `COOL`.  
* `DRY` and `HUMIDIFY` shall never operate simultaneously.  
* All climate-control outputs shall be disabled in Safe State.

### **FR-08 — Automatic Reservoir Refill**

When `LOW Level Sensor` is active:

* set `WATER_LOW`;  
* stop and block humidification;  
* start reservoir refill.

When `FULL Level Sensor` becomes active:

* stop reservoir refill;  
* clear `WATER_LOW`;  
* set `WATER_FULL`;  
* allow humidification again.

### **FR-09 — Humidifier Protection**

The humidifier shall not operate when:

* `WATER_LOW` is active;  
* reservoir refill is active;  
* Safe State is active.

### **FR-10 — Refill Protection**

If `FULL Level Sensor` does not become active within 120 seconds:

* stop refill;  
* set `REFILL_TIMEOUT`;  
* block automatic refill;  
* publish the fault through MQTT;  
* log the fault through diagnostics.

If `LOW Level Sensor` and `FULL Level Sensor` are active simultaneously:

* treat the sensor state as invalid;  
* stop refill;  
* block humidification;  
* set `WATER_LEVEL_SENSOR_FAULT`;  
* publish the fault through MQTT;  
* log the fault through diagnostics.

### **FR-11 — MANUAL Mode**

In `MANUAL` mode, the user may control:

* AC: `OFF`, `HEAT`, `COOL`, `DRY`;  
* humidifier;  
* reservoir refill.

Automatic climate-control logic shall not change actuator states in MANUAL mode.

Safety restrictions and Safe State shall override manual commands.

### **FR-12 — Parameter Configuration**

The user shall be able to remotely configure:

* Min Temperature;  
* Max Temperature;  
* Min Humidity;  
* Max Humidity;  
* Measurement Interval;  
* WARNING thresholds;  
* CRITICAL thresholds.

Each valid parameter change shall:

* apply immediately without reboot;  
* be stored in non-volatile memory;  
* survive reboot;  
* be confirmed through MQTT.

Target Temperature and Target Humidity shall be recalculated automatically after corresponding Min/Max changes.

### **FR-13 — Parameter Validation**

Each parameter shall be validated independently before it is applied or saved.

If one parameter is invalid:

* reject only that parameter;  
* keep all other valid parameters;  
* preserve the previous valid value for the rejected parameter;  
* notify the user immediately;  
* provide the rejection reason.

Validation rules:

* `Min < Max`  
* `0%RH <= Humidity <= 100%RH`  
* `1 s <= Measurement Interval <= 60 s`  
* `CRITICAL_LOW < WARNING_LOW < CONTROL_MIN < CONTROL_MAX < WARNING_HIGH < CRITICAL_HIGH`  
* Wrong data type or format shall be rejected.

### **FR-14 — Parameter Recommendations**

After the user enters one primary control boundary, the system shall suggest a related boundary.

Examples:

* Max Temperature entered → recommend `Min Temperature = Max Temperature - 5°C`  
* Min Temperature entered → recommend `Max Temperature = Min Temperature + 5°C`  
* Max Humidity entered → recommend `Min Humidity = Max Humidity - 20%RH`  
* Min Humidity entered → recommend `Max Humidity = Min Humidity + 20%RH`

After both boundaries are available, the corresponding target value shall be displayed.

Recommendations:

* shall not be applied automatically;  
* require user confirmation;  
* shall pass normal validation;  
* shall be adjusted to the nearest valid value if necessary.

### **FR-15 — Remote Monitoring**

The system shall publish through MQTT:

* temperature;  
* humidity;  
* `AUTO` / `MANUAL`;  
* current AC mode;  
* humidifier state;  
* refill state;  
* water-level sensor states;  
* current configuration;  
* Target Temperature;  
* Target Humidity;  
* `NORMAL` / `WARNING` / `CRITICAL`;  
* active faults.

Changes in actuator state, operating mode, or fault state shall be published independently of the next sensor-measurement cycle.

### **FR-16 — Remote Control**

The system shall accept MQTT commands for:

* switching between `AUTO` and `MANUAL`;  
* changing configuration;  
* manually controlling AC;  
* manually controlling humidifier;  
* manually controlling refill.

Manual actuator commands shall be accepted only in `MANUAL` mode.

Commands that violate safety restrictions shall be rejected.

### **FR-17 — SHT31 Fault Handling**

If SHT31 data is unavailable or unreliable, the system shall enter Safe State.

Safe State shall:

* set AC to `OFF`;  
* turn humidifier `OFF`;  
* stop reservoir refill;  
* block automatic actuator control;  
* block manual actuator control;  
* publish the fault through MQTT;  
* log the fault through diagnostics.

Safe State has priority over `AUTO` and `MANUAL`.

### **FR-18 — Environmental Warning Levels**

The system shall classify environmental values as:

* `NORMAL`;  
* `WARNING`;  
* `CRITICAL`.

WARNING and CRITICAL conditions shall:

* be published through MQTT;  
* generate a diagnostic event;  
* not automatically trigger Safe State by themselves.

Normal climate regulation shall continue unless another fault requires Safe State.

## **5\. Non-Functional Requirements**

### **NFR-01 — Response Time**

After receiving a new valid temperature, humidity, or water-level state, the system shall make the required control decision and update outputs within 1 second.

### **NFR-02 — Measurement Accuracy**

The system shall support:

* temperature accuracy: ±0.5°C;  
* humidity accuracy: ±3%RH.

Software processing shall not introduce additional significant measurement distortion.

### **NFR-03 — Startup Time**

After power-on or reboot, the system shall reach an operational state within 10 seconds.

All actuators shall remain OFF during initialization.

### **NFR-04 — Reboot Recovery**

After reboot, the system shall:

* restore the last valid configuration;  
* never restore partial or corrupted configuration;  
* start with actuators in a safe state;  
* initialize sensors;  
* restore Wi-Fi and MQTT connectivity;  
* re-evaluate current sensor values before activating actuators.

Active actuator states shall not be restored blindly.

### **NFR-05 — Network Recovery**

Loss of Wi-Fi or MQTT shall not stop local AUTO climate regulation.

The system shall:

* begin reconnect attempts within 5 seconds;  
* automatically restore Wi-Fi and MQTT;  
* resume telemetry after reconnect;  
* not enter Safe State solely because of network loss.

### **NFR-06 — MQTT Event Latency**

With an active MQTT connection, important state changes shall be published within 2 seconds.

This includes:

* AC mode;  
* `AUTO` / `MANUAL`;  
* humidifier state;  
* `WATER_LOW`;  
* `WATER_FULL`;  
* `WARNING`;  
* `CRITICAL`;  
* `REFILL_TIMEOUT`;  
* `WATER_LEVEL_SENSOR_FAULT`;  
* Safe State.

### **NFR-07 — Reliability**

The system shall operate continuously for 24 hours without:

* hanging;  
* unexpected reboot;  
* configuration loss;  
* uncontrolled actuator state.

Measurement, climate control, and MQTT communication shall remain operational.

### **NFR-08 — Configuration Integrity**

Configuration shall survive:

* software reboot;  
* hardware reboot;  
* power cycle.

If configuration is missing or corrupted:

* load default values;  
* generate a diagnostic event;  
* do not use corrupted values.

Changing one parameter shall not modify unrelated parameters, except automatically calculated dependent target values.

## **6\. Critical Scenarios**

### **CS-01 — Combined Cooling and Humidification**

Initial state:

* Temperature: 30°C  
* Humidity: 30%RH  
* Mode: `AUTO`

Expected behavior:

1. Start `COOL` because temperature is above Max Temperature.  
2. Start `HUMIDIFY` because humidity is below Min Humidity.  
3. `COOL` and `HUMIDIFY` operate simultaneously.  
4. When temperature reaches Target Temperature, stop `COOL`.  
5. If humidity is still below Target Humidity, continue `HUMIDIFY`.  
6. When humidity reaches Target Humidity, stop `HUMIDIFY`.  
7. Enter `IDLE` if no other regulation is required.

Related requirements: FR-01, FR-02, FR-04, FR-06, FR-07, NFR-01.

### **CS-02 — Low Water During Humidification**

1. `HUMIDIFY` is active.  
2. `LOW Level Sensor` becomes active.  
3. Stop humidifier immediately.  
4. Set `WATER_LOW`.  
5. Start reservoir refill.  
6. `FULL Level Sensor` becomes active.  
7. Stop refill.  
8. Clear `WATER_LOW`.  
9. Re-evaluate humidity.  
10. Resume `HUMIDIFY` if humidity is still below Target Humidity.

Related requirements: FR-06, FR-08, FR-09, NFR-01.

### **CS-03 — Invalid Configuration Parameter**

1. User changes a valid configuration parameter.  
2. User enters an invalid value for another parameter.  
3. Reject only the invalid parameter.  
4. Preserve all previously accepted valid values.  
5. Preserve the previous valid value for the rejected parameter.  
6. Do not persist the invalid value.  
7. Return an immediate validation response.

Related requirements: FR-12, FR-13, FR-14, NFR-08.

### **CS-04 — SHT31 Failure During Active Regulation**

1. Climate regulation is active.  
2. SHT31 stops responding or returns invalid data.  
3. Enter Safe State.  
4. Turn AC OFF.  
5. Turn humidifier OFF.  
6. Stop refill.  
7. Block manual and automatic actuator control.  
8. Publish fault through MQTT.  
9. Log diagnostic event.

Related requirements: FR-17, NFR-01, NFR-06.

### **CS-05 — Wi-Fi / MQTT Loss**

1. Device operates in AUTO mode.  
2. Wi-Fi or MQTT connection is lost.  
3. Local climate control continues.  
4. Device starts reconnect attempts.  
5. Connection is restored automatically.  
6. Current actual state is published after reconnect.

Related requirements: FR-01, FR-02, FR-15, NFR-05, NFR-06.

### **CS-06 — Reboot During Active Regulation**

1. `HEAT`, `COOL`, `DRY`, or `HUMIDIFY` is active.  
2. Device reboots.  
3. All actuators remain OFF during initialization.  
4. Restore the last valid configuration.  
5. Initialize sensors and water-level inputs.  
6. Read current environmental state.  
7. Re-evaluate required actuator states.  
8. Do not blindly restore the actuator state that was active before reboot.

Related requirements: FR-01, FR-02, FR-12, NFR-03, NFR-04, NFR-08.

### **CS-07 — Abnormally Fast Temperature Rise**

A thermal anomaly is detected when:

* Temperature increases by at least 10°C within 60 seconds; or  
* Temperature reaches at least 45°C.

Expected behavior:

1. Classify the event as `THERMAL_ANOMALY`.  
2. Turn AC OFF.  
3. Turn humidifier OFF.  
4. Stop refill.  
5. Block automatic and manual actuator control.  
6. Publish a critical MQTT event.  
7. Log the diagnostic event.

`THERMAL_ANOMALY` is treated as a safety condition and has higher priority than normal climate regulation.

This scenario does not represent fire detection, and the device is not a fire-alarm or fire-suppression system.

