# Project Structure

```text
iot-smart-climate-hil/
│
├── lib/
│   ├── __init__.py
│   ├── serial_driver.py
│   └── mqtt_driver.py
│
├── framework/
│   ├── __init__.py
│   ├── hil_agent.py
│   └── dut.py
│
├── tests/
│   ├── smoke/
│   │   ├── conftest.py
│   │   ├── test_dut_connection.py
│   │   ├── test_hil_agent_connection.py
│   │   ├── test_mqtt_connection.py
│   │   └── test_basic_climate_control.py
│   │
│   ├── functional/
│   │   ├── conftest.py
│   │   ├── test_temperature_control.py
│   │   ├── test_humidity_control.py
│   │   ├── test_mode_switching.py
│   │   ├── test_water_refill.py
│   │   ├── test_manual_mode.py
│   │   ├── test_configuration.py
│   │   ├── test_remote_monitoring.py
│   │   ├── test_remote_control.py
│   │   └── test_fault_handling.py
│   │
│   └── non_functional/
│       ├── conftest.py
│       ├── test_response_time.py
│       ├── test_sensor_accuracy.py
│       ├── test_reboot_recovery.py
│       ├── test_network_recovery.py
│       ├── test_mqtt_latency.py
│       ├── test_configuration_persistence.py
│       └── test_stability.py
│
├── test_data/
│   ├── default_config.yaml
│   ├── climate_scenarios.yaml
│   └── invalid_config.yaml
│
├── config/
│   └── test_config.yaml
│
├── reports/
│
├── pyproject.toml
└── README.md
```

## `lib/`

`lib/` містить низькорівневі перевикористовувані драйвери, які не залежать від бізнес-логіки конкретного IoT-продукту.

- `serial_driver.py` — відповідає за Serial-з'єднання: відкриття та закриття COM-порту, надсилання команд, читання відповідей і перевірку стану з'єднання.
- `mqtt_driver.py` — відповідає за MQTT-з'єднання: підключення до broker, publish, subscribe, отримання повідомлень і reconnect.

Драйвери винесені в `lib/`, тому що транспортний рівень можна повторно використовувати в інших embedded/HIL-проєктах без прив'язки до IoT Smart Climate Controller.

## `framework/`

`framework/` містить product-specific логіку тестового фреймворку для IoT Smart Climate Controller.

- `dut.py` — представляє Device Under Test і надає високорівневі методи для взаємодії з пристроєм через MQTT/Serial: зміна режиму, конфігурації, надсилання команд і отримання стану DUT.
- `hil_agent.py` — представляє HIL Agent і надає високорівневі методи для керування hardware simulation: встановлення температури/вологості, станів water-level sensors, fault injection та читання вихідних сигналів DUT.

`framework/` використовує драйвери з `lib/`, але приховує від тестів низькорівневі деталі Serial і MQTT. Завдяки цьому тести працюють із предметними командами на кшталт `set_temperature()` або `set_mode()`, а не напряму з COM-портами, байтами чи MQTT topic-ами.
