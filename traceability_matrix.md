# Traceability Matrix

| PRD Requirement | Test File | Test Function |
|---|---|---|
| FR-01 — Monitoring temperature and humidity | `tests/functional/test_temperature_control.py` | `test_temperature_humidity_measurement()` |
| FR-01 — Configurable measurement interval (1–60 s, default 5 s) | `tests/functional/test_configuration.py` | `test_measurement_interval_configuration()` |
| FR-02 — AUTO mode | `tests/functional/test_mode_switching.py` | `test_auto_mode_enables_automatic_regulation()` |
| FR-03 — Heating below Min Temperature until Target Temperature | `tests/functional/test_temperature_control.py` | `test_heating_below_min_until_target()` |
| FR-04 — Cooling above Max Temperature until Target Temperature | `tests/functional/test_temperature_control.py` | `test_cooling_above_max_until_target()` |
| FR-05 — Drying above Max Humidity until Target Humidity | `tests/functional/test_humidity_control.py` | `test_dry_above_max_humidity_until_target()` |
| FR-06 — Humidifying below Min Humidity until Target Humidity | `tests/functional/test_humidity_control.py` | `test_humidify_below_min_humidity_until_target()` |
| FR-07 — Temperature control priority over humidity control | `tests/functional/test_mode_switching.py` | `test_temperature_control_has_priority_over_humidity()` |
| FR-08 — Automatic reservoir refill from LOW to FULL | `tests/functional/test_water_refill.py` | `test_refill_from_low_until_full()` |
| FR-09 — Humidifier protection when water is unavailable, refill is active, or Safe State is active | `tests/functional/test_water_refill.py` | `test_humidifier_blocked_when_water_unavailable()` |
| FR-10 — Refill timeout protection | `tests/functional/test_water_refill.py` | `test_refill_timeout_stops_refill_and_reports_fault()` |
| FR-10 — Invalid simultaneous LOW and FULL sensor state | `tests/functional/test_water_refill.py` | `test_invalid_water_level_sensor_combination_reports_fault()` |
| FR-11 — MANUAL mode actuator control without automatic override | `tests/functional/test_manual_mode.py` | `test_manual_mode_controls_actuators_without_auto_override()` |
| FR-11 — Safety rules override MANUAL commands | `tests/functional/test_manual_mode.py` | `test_safety_overrides_manual_commands()` |
| FR-12 — Valid parameter change applies immediately and persists | `tests/functional/test_configuration.py` | `test_valid_parameter_change_applies_and_persists()` |
| FR-12 — Target values are recalculated after Min/Max change | `tests/functional/test_configuration.py` | `test_target_recalculated_after_boundary_change()` |
| FR-13 — Invalid parameter is rejected independently while valid values are retained | `tests/functional/test_configuration.py` | `test_invalid_parameter_is_rejected_independently()` |
| FR-13 — WARNING/CRITICAL/control threshold sequence validation | `tests/functional/test_configuration.py` | `test_threshold_sequence_validation()` |
| FR-14 — Related parameter recommendation requires user confirmation | `tests/functional/test_configuration.py` | `test_related_parameter_recommendation_requires_confirmation()` |
| FR-14 — Invalid recommendation is adjusted to the nearest valid value | `tests/functional/test_configuration.py` | `test_recommendation_is_adjusted_to_valid_range()` |
| FR-15 — Remote MQTT monitoring publishes required device state and telemetry | `tests/functional/test_remote_monitoring.py` | `test_mqtt_monitoring_publishes_required_state()` |
| FR-15 — State, actuator, and fault changes are published independently of the measurement cycle | `tests/functional/test_remote_monitoring.py` | `test_state_change_is_published_without_waiting_for_measurement_cycle()` |
| FR-16 — Remote MQTT control of mode, configuration, and actuators | `tests/functional/test_remote_control.py` | `test_remote_control_commands()` |
| FR-16 — Manual actuator commands are rejected outside MANUAL mode or when conflicting with safety rules | `tests/functional/test_remote_control.py` | `test_unsafe_or_invalid_remote_command_is_rejected()` |
| FR-17 — SHT31 failure forces Safe State and blocks actuator control | `tests/functional/test_fault_handling.py` | `test_sht31_fault_enters_safe_state()` |
| FR-18 — NORMAL/WARNING/CRITICAL environmental classification without entering Safe State | `tests/functional/test_fault_handling.py` | `test_warning_and_critical_classification_without_safe_state()` |
| NFR-01 — Control response time ≤ 1 s | `tests/non_functional/test_response_time.py` | `test_control_response_within_one_second()` |
| NFR-02 — Temperature accuracy ±0.5 °C | `tests/non_functional/test_sensor_accuracy.py` | `test_temperature_accuracy()` |
| NFR-02 — Humidity accuracy ±3 %RH | `tests/non_functional/test_sensor_accuracy.py` | `test_humidity_accuracy()` |
| NFR-03 — Startup ≤ 10 s and actuators remain OFF during initialization | `tests/non_functional/test_reboot_recovery.py` | `test_startup_within_ten_seconds_and_outputs_off_during_init()` |
| NFR-04 — Reboot restores the last valid configuration and re-evaluates current sensor state | `tests/non_functional/test_reboot_recovery.py` | `test_reboot_restores_config_and_reevaluates_state()` |
| NFR-05 — Local AUTO control continues during Wi-Fi/MQTT loss | `tests/non_functional/test_network_recovery.py` | `test_local_auto_control_continues_without_network()` |
| NFR-05 — Wi-Fi/MQTT reconnects automatically and actual state is republished | `tests/non_functional/test_network_recovery.py` | `test_wifi_mqtt_reconnect_and_state_republish()` |
| NFR-06 — Important MQTT events are published within 2 s | `tests/non_functional/test_mqtt_latency.py` | `test_important_event_published_within_two_seconds()` |
| NFR-07 — 24-hour continuous operation without hangs, unexpected reboot, configuration loss, or uncontrolled outputs | `tests/non_functional/test_stability.py` | `test_24h_continuous_operation()` |
| NFR-08 — Configuration survives software reboot and power cycle | `tests/non_functional/test_configuration_persistence.py` | `test_configuration_survives_reboot_and_power_cycle()` |
| NFR-08 — Missing or corrupted configuration loads defaults and reports a diagnostic event | `tests/non_functional/test_configuration_persistence.py` | `test_corrupt_configuration_loads_defaults_and_reports_diagnostic()` |
| NFR-08 — Changing one parameter does not alter unrelated configuration values | `tests/non_functional/test_configuration_persistence.py` | `test_changing_parameter_does_not_modify_unrelated_values()` |

## Critical Scenarios

| Critical Scenario | Test File | Test Function |
|---|---|---|
| CS-01 — Automatic climate stabilization with temperature priority | `tests/functional/test_mode_switching.py` | `test_temperature_priority_then_humidity_stabilization()` |
| CS-02 — Low water detected during humidification and automatic refill until FULL | `tests/functional/test_water_refill.py` | `test_low_water_during_humidification_triggers_refill_and_resume()` |
| CS-03 — Invalid configuration input rejects only the invalid parameter and retains valid values | `tests/functional/test_configuration.py` | `test_invalid_parameter_does_not_discard_valid_configuration()` |
| CS-04 — SHT31 failure during active regulation forces Safe State | `tests/functional/test_fault_handling.py` | `test_sensor_failure_during_active_regulation_enters_safe_state()` |
| CS-05 — Wi-Fi/MQTT loss does not interrupt local AUTO control and state is restored after reconnect | `tests/non_functional/test_network_recovery.py` | `test_local_control_during_network_loss_and_recovery()` |
| CS-06 — Reboot during active regulation restores configuration and re-evaluates state without restoring actuator state blindly | `tests/non_functional/test_reboot_recovery.py` | `test_reboot_during_active_regulation_recovers_safely()` |
| CS-07 — Abnormally fast temperature rise triggers thermal anomaly handling | `tests/functional/test_fault_handling.py` | `test_thermal_anomaly_forces_all_outputs_off()` |
