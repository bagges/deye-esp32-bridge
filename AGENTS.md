# AGENTS.md - deye-esp32-bridge

## Project Overview

This is an ESPHome project for bridging Deye/Sunsynk inverters with Battery Management Systems (SeplosBMS and PaceBMS). The configuration files are YAML-based with embedded C++ lambdas for custom logic.

## Build Commands

### Build and Upload to ESP32
```bash
# Build and flash a specific configuration
esphome run deye-esp32-bridge.yaml

# Validate without flashing (dry run)
esphome run deye-esp32-bridge.yaml --dry-run

# Compile only (no upload)
esphome compile deye-esp32-bridge.yaml

# Upload only (requires prior compilation)
esphome upload deye-esp32-bridge.yaml

# Clean build
esphome clean deye-esp32-bridge.yaml
```

### Configuration Files
| File | Description |
|------|-------------|
| `deye-esp32-bridge.yaml` | Main config for SeplosBMS with home automation |
| `deye-seplosV3.yaml` | Seplos V3 BMS only |
| `deye-esp32-bridge-pace.yaml` | PaceBMS variant |
| `deye-multi-seplosv3.yaml` | Multiple Seplos BMS support |
| `secrets.yaml` | WiFi credentials (never commit this) |

### Prerequisites
1. Install ESPHome: `pip install esphome`
2. Configure WiFi credentials in `secrets.yaml`:
   ```yaml
   wifi_ssid: "YourSSID"
   wifi_password: "YourPassword"
   ```
3. Connect ESP32 via USB

## Code Style Guidelines

### YAML Structure

#### Indentation
- Use 2 spaces for indentation (no tabs)
- Align related properties within their parent

```yaml
# Good
sensor:
  - platform: modbus_controller
    name: "${device_type}-PV1 Power"
    address: 672
    unit_of_measurement: "W"

# Bad - 4 spaces (inconsistent)
sensor:
    - platform: modbus_controller
        name: "..."
```

#### Substitutions
Use `substitutions` for reusable values at the top of files:
```yaml
substitutions:
  device_type: sun10k
  modbus_update_interval: "10s"
  query_throttle: "100ms"
```

#### Entity Naming
- Names use Title Case: `"${device_type}-PV1 Power"`
- IDs use snake_case: `${device_type}_PV1_Power`
- Use device_class, state_class, and unit_of_measurement for all sensors

### Modbus Register Definitions

#### Standard Sensor Pattern
```yaml
- platform: modbus_controller
  modbus_controller_id: ${modbus_controller_id}
  name: "${device_type}-Grid Voltage L1"
  address: 598
  register_type: holding
  unit_of_measurement: "V"
  state_class: "measurement"
  accuracy_decimals: 1
  filters:
    - multiply: 0.1
  value_type: U_WORD
```

#### Value Types
- `U_WORD` - Unsigned 16-bit
- `S_WORD` - Signed 16-bit
- `U_DWORD_R` - Unsigned 32-bit (reversed byte order)
- `S_DWORD_R` - Signed 32-bit (reversed byte order)

#### Bitmask Sensors
```yaml
- platform: modbus_controller
  modbus_controller_id: ${modbus_controller_id}
  name: ${device_type}-AC INV relay
  register_type: holding
  address: 552
  bitmask: 0x1
```

### Lambda Expressions

#### C++ Lambda Syntax
```yaml
# Reading lambda (sensor value transformation)
lambda: |-
  uint16_t value = modbus_controller::word_from_hex_str(x, 0);
  switch (value) {
    case 0: return std::string("standby");
    case 1: return std::string("selfcheck");
    default: return std::string("unknown");
  }
  return x;

# Write lambda (value transformation before sending)
write_lambda: "return x * 100;"

# Complex sensor with data access
lambda: |-
  if (data.size() < 4) {
    return NAN;
  }
  float current = (int16_t)(data[0] << 8 | data[1] << 0);
  float total_voltage = (uint16_t)(data[2] << 8 | data[3] << 0);
  return current * total_voltage * 0.0001f;
```

#### Accessing Entity States
```yaml
lambda: |-
  if (id(batterymode).state == "Normal") {
    return abs(id(helper_discharge_current));
  }
  return 5.0;
```

#### Logging
```yaml
ESP_LOGI("Component", "Message: %d", value);  // Info
ESP_LOGW("Component", "Warning: %s", msg);    // Warning
ESP_LOGE("Component", "Error: %d", code);     // Error
ESP_LOGD("Component", "Debug: %.2f", val);    // Debug
```

### Filters
```yaml
filters:
  - multiply: 0.1      # Scale value
  - offset: -1000       # Add offset (before multiply)
  - throttle: 10s       # Limit update frequency
  - skip_updates: 5     # Skip every N updates
  - lambda: "return x * 2.0;"  # Custom filter
```

### Time Configuration
```yaml
time:
  - platform: sntp
    id: sntp_time
    timezone: Europe/Berlin
    servers:
      - pool.ntp.org
```

### UART Configuration
```yaml
uart:
  - id: uartdeye
    tx_pin: 27
    rx_pin: 26
    baud_rate: 9600
    rx_buffer_size: 256
```

### Modbus Controller
```yaml
modbus_controller:
  - id: ${modbus_controller_id}
    address: 0x01
    modbus_id: modbus1
    setup_priority: -10
    update_interval: ${modbus_update_interval}
    command_throttle: ${query_throttle}
```

### Global Variables
```yaml
globals:
  - id: helper_discharge_current
    type: int
    restore_value: yes
    initial_value: '100'
  - id: max_soc
    type: float
    restore_value: yes
    initial_value: '0.0'
```

## Common Patterns

### Time-Based Actions
```yaml
on_time:
  - seconds: 0
    minutes: 0
    hours: 23
    days_of_week: MON-SUN
    then:
      - lambda: |-
          // Action code here
      - delay: 1s
      - number.set:
          id: some_number_id
          value: !lambda return id(target_soc);
```

### On Value Callbacks
```yaml
on_value:
  then:
    - delay: 200ms  # Prevent bus contention
    - lambda: |-
        // Handle value change
    - delay: 500ms  # Verification delay
    - lambda: |-
        // Verify write succeeded
```

### External Components
```yaml
external_components:
  - source: github://syssi/esphome-seplos-bms@main
    refresh: 0s
```

## Error Handling

### Null/Invalid Data Checks
```yaml
lambda: |-
  if (data.size() < expected_size) {
    return NAN;
  }
  // Process data safely
```

### Switch Statements for Status Codes
```yaml
lambda: |-
  uint16_t value = modbus_controller::word_from_hex_str(x, 0);
  switch (value) {
    case 0: return std::string("status0");
    case 1: return std::string("status1");
    // ... more cases
    default: return std::string("unknown");
  }
  return x;
```

## Hardware Pin Assignments

| UART | TX | RX | Purpose |
|------|----|----|---------|
| uartdeye | 27 | 26 | Deye Inverter |
| uartseplos | 17 | 16 | Seplos BMS |
| uartpace | 17 | 16 | Pace BMS |

## Known Issues / Gotchas

1. **Bus Contention**: Add delays (200-500ms) between modbus writes to prevent bus contention
2. **Buffer Sizes**: Adjust `rx_buffer_size` for high-traffic UART connections (384 for Seplos)
3. **Seplos Protocol**: Uses protocol_version 0x20 with send_wait_time of 200ms
4. **Byte Order**: Deye uses U_WORD, some values require U_DWORD_R (reversed)
5. **Temperature Offsets**: Seplos temperatures need `-2731.5` offset (Kelvin to Celsius)

## Documentation References

- [ESPHome Documentation](https://esphome.io/)
- [ESPHome Modbus Controller](https://esphome.io/components/modbus_controller.html)
- [Seplos BMS Component](https://github.com/syssi/esphome-seplos-bms)
