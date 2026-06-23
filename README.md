# HassBle Config

> **EN** — This repository stores the device configuration (`config.yaml`) for the [HassBle](https://github.com/eigger/hassble-android) Android app.  
> **KO** — 이 저장소는 [HassBle](https://github.com/eigger/hassble-android) 안드로이드 앱이 사용하는 기기 설정(`config.yaml`)을 보관합니다.

---

## Table of Contents / 목차

1. [How It Works / 동작 원리](#how-it-works--동작-원리)
2. [Quick Start / 빠른 시작](#quick-start--빠른-시작)
3. [Root Structure / 최상위 구조](#root-structure--최상위-구조)
4. [defaults — Publish Rules / 기본 발행 규칙](#defaults--publish-rules--기본-발행-규칙)
5. [Device Types / 기기 소스 타입](#device-types--기기-소스-타입)
6. [source: advertisement — BLE Passive Scan](#source-advertisement--ble-passive-scan)
   - [match — Matching Conditions / 매칭 조건](#match--matching-conditions--매칭-조건)
   - [instance_mode — Entity Strategy / 엔티티 전략](#instance_mode--entity-strategy--엔티티-전략)
7. [source: gatt_notify — Active GATT Connection](#source-gatt_notify--active-gatt-connection)
8. [source: obd — ELM327 OBD Polling](#source-obd--elm327-obd-polling)
9. [sensors — Sensor Config / 센서 설정](#sensors--sensor-config--센서-설정)
   - [decode — Binary Decoding / 바이너리 디코딩](#decode--binary-decoding--바이너리-디코딩)
   - [Data Types / 데이터 타입](#data-types--데이터-타입)
10. [controls — HA Controls / HA 제어 (GATT/OBD)](#controls--ha-controls--ha-제어)
11. [publish — Per-Sensor Filter / 센서별 발행 필터](#publish--per-sensor-filter--센서별-발행-필터)
12. [OBD Preset List / OBD 프리셋 목록](#obd-preset-list--obd-프리셋-목록)
13. [Full Examples / 전체 예제](#full-examples--전체-예제)
14. [Tips & Gotchas / 주의사항](#tips--gotchas--주의사항)

---

## How It Works / 동작 원리

**EN**  
The HassBle app downloads your `config.yaml` from a Git raw URL, decodes BLE data according to the rules you define, and automatically declares sensors in Home Assistant via the [ws_bridge](https://github.com/eigger/hass-ws-bridge) integration. No code changes are needed — just edit YAML and push.

**KO**  
HassBle 앱은 Git raw URL에서 `config.yaml`을 다운로드하고, 정의된 규칙대로 BLE 데이터를 디코딩하여 [ws_bridge](https://github.com/eigger/hass-ws-bridge) 통합을 통해 Home Assistant에 센서를 자동으로 선언합니다. 코드 수정 없이 YAML만 편집하고 푸시하면 됩니다.

```
Android Phone (BLE Scan)
       │  raw bytes
       ▼
  HassBle App  ──── config.yaml (this repo) ────▶  decode rules
       │  decoded value
       ▼
  HA WebSocket API  ──────────────────────────────▶  ws_bridge
       │
       ▼
  Home Assistant Entity (sensor / switch / number / …)
```

---

## Quick Start / 빠른 시작

**EN**
1. Fork or copy this repository to your GitHub account.
2. Edit `config.yaml` to describe your BLE devices.
3. Get the **Raw URL** of your file.  
   Example: `https://raw.githubusercontent.com/YOUR_NAME/hassble-config/main/config.yaml`
4. In the HassBle app → **Gateway tab** → paste the Raw URL → start the gateway.
5. In **Sensors tab**, enable the sensors you want and they will appear in Home Assistant.

> ⚠️ **Never put HA credentials (URL / Token) in this file.**  
> Enter them only in the app settings.

**KO**
1. 이 저장소를 Fork하거나 복사하여 자신의 GitHub 계정에 저장합니다.
2. `config.yaml`을 자신의 BLE 기기에 맞게 수정합니다.
3. 파일의 **Raw URL**을 복사합니다.  
   예: `https://raw.githubusercontent.com/YOUR_NAME/hassble-config/main/config.yaml`
4. HassBle 앱 → **Gateway 탭** → Raw URL 붙여넣기 → 게이트웨이 시작.
5. **Sensors 탭**에서 원하는 센서를 활성화하면 Home Assistant에 자동으로 나타납니다.

> ⚠️ **이 파일에 HA 자격증명(URL / 토큰)을 절대 입력하지 마세요.**  
> 자격증명은 앱 설정에서만 입력합니다.

---

## Root Structure / 최상위 구조

```yaml
version: 1          # 현재 항상 1 / Always 1

defaults:           # 전체 기기에 적용되는 기본 발행 규칙
  publish:
    on_change_only: true
    min_interval: 10s

devices:            # 기기 목록 (순서 무관)
  - id: ...
```

| Field | Type | Default | EN Description | KO 설명 |
|-------|------|---------|----------------|---------|
| `version` | int | `1` | Schema version. Always `1`. | 스키마 버전. 항상 `1`. |
| `defaults.publish` | object | see below | Default publish filter applied to all sensors. | 모든 센서에 기본 적용되는 발행 필터. |
| `devices` | list | `[]` | List of device configurations. | 기기 설정 목록. |

---

## defaults — Publish Rules / 기본 발행 규칙

**EN** — Controls how often and under what conditions sensor values are sent to HA. Can be overridden per-device or per-sensor.

**KO** — HA로 센서값을 전송하는 빈도와 조건을 제어합니다. 기기별·센서별로 덮어쓸 수 있습니다.

```yaml
defaults:
  publish:
    on_change_only: true   # 값이 변경될 때만 전송 (기본: true)
    min_interval: 10s      # 최소 전송 간격 (기본: 10s)
    heartbeat: 5m          # 변화 없어도 주기적 강제 전송 (선택)
    deadband: 0.5          # 이 값 이하의 변화는 무시 (선택, 숫자형 센서만)
```

| Field | Type | Default | EN | KO |
|-------|------|---------|----|----|
| `on_change_only` | bool | `true` | Only send when the value changes. | 값이 변경될 때만 전송. |
| `min_interval` | duration | `10s` | Minimum time between two sends. Formats: `500ms`, `30s`, `5m`. | 두 전송 사이 최소 시간. 형식: `500ms`, `30s`, `5m`. |
| `heartbeat` | duration | `null` | Even if unchanged, force a send at this interval. | 값이 동일해도 이 주기마다 강제 전송. |
| `deadband` | float | `null` | Ignore changes smaller than this value (numeric sensors). | 이 값보다 작은 변화는 무시 (숫자형 센서). |

---

## Device Types / 기기 소스 타입

**EN** — Every entry in `devices` must have a `source` field that selects the data path.

**KO** — `devices`의 모든 항목은 데이터 경로를 지정하는 `source` 필드가 있어야 합니다.

| `source` | EN | KO |
|----------|----|----|
| `advertisement` | Passive BLE scan — no connection, reads broadcast packets. | 수동 BLE 스캔 — 연결 없이 브로드캐스트 패킷을 읽습니다. |
| `gatt_notify` | Active GATT connection — connects to the device and subscribes to notifications. Supports write commands. | 능동 GATT 연결 — 기기에 연결하여 알림을 구독합니다. 쓰기 명령 지원. |
| `obd` | ELM327 OBD adapter over BLE — polls PID commands and parses ISO-TP responses. | BLE ELM327 OBD 어댑터 — PID 명령을 폴링하고 ISO-TP 응답을 파싱합니다. |

**Common fields shared by all device types / 모든 소스 공통 필드:**

```yaml
- id: my_device          # 고유 식별자 (영문 소문자·숫자·_) — HA entity ID의 일부가 됨
  name: "My Device"      # 표시 이름 (HA 장치 이름)
  source: advertisement  # advertisement | gatt_notify | obd
  publish:               # (선택) 이 기기의 모든 센서에 대한 발행 규칙 덮어쓰기
    on_change_only: true
    min_interval: 5s
  sensors: [...]         # 센서 목록
  controls: [...]        # 제어 목록 (gatt_notify / obd만)
```

| Field | Type | Required | EN | KO |
|-------|------|----------|----|----|
| `id` | string | ✅ | Unique identifier. Used in HA entity unique_id. Use lowercase letters, digits, `_`. | 고유 식별자. HA entity unique_id에 사용됩니다. 소문자·숫자·`_` 사용. |
| `name` | string | ✅ | Display name shown in HA device. | HA 장치에 표시되는 이름. |
| `source` | enum | ✅ | Data source type (see above). | 데이터 소스 타입 (위 표 참조). |
| `publish` | object | ❌ | Device-level publish rule override. | 기기 수준 발행 규칙 덮어쓰기. |
| `sensors` | list | ❌ | List of sensor definitions. | 센서 정의 목록. |
| `controls` | list | ❌ | List of control definitions. | 제어 정의 목록. |

---

## source: advertisement — BLE Passive Scan

**EN** — Listens for BLE advertisement packets without connecting. Use `match` to filter packets.

**KO** — 연결 없이 BLE 광고 패킷을 수신합니다. `match`로 패킷을 필터링합니다.

```yaml
- id: my_sensor
  name: "My BLE Sensor"
  source: advertisement
  instance_mode: mac      # mac | shared (아래 설명 참조)
  match:
    manufacturer_id: 76   # Apple = 76 (0x004C) — 반드시 10진수 사용
    manufacturer_min_length: 24
    manufacturer_hex_prefix: "0215"
  sensors:
    - key: temperature
      source_field: manufacturer_data   # manufacturer_data | service_data | raw
      decode:
        offset: 18
        length: 2
        type: uint16
        endian: big
        scale: 0.002670288
        offset_value: -45
```

### match — Matching Conditions / 매칭 조건

**EN** — All specified conditions must be satisfied simultaneously (AND logic). Unspecified fields are ignored.

**KO** — 지정된 모든 조건을 동시에 만족해야 합니다 (AND 논리). 미지정 필드는 무시합니다.

| Field | Type | EN | KO |
|-------|------|----|----|
| `mac` | string | Fixed MAC address filter. Format: `"AA:BB:CC:DD:EE:FF"`. If set, `instance_mode` is effectively `mac` with a single fixed instance. | 고정 MAC 주소 필터. 형식: `"AA:BB:CC:DD:EE:FF"`. 설정 시 해당 MAC만 수신. |
| `service_data_uuid` | string | 16-bit or 128-bit service UUID suffix. E.g. `"F51C"`, `"181A"`. | 서비스 데이터 UUID. 예: `"F51C"`, `"181A"`. |
| `manufacturer_id` | int | Manufacturer company ID. **Must use decimal** (not `0x...`). E.g. Apple=`76`, custom=`861`. | 제조사 ID. **반드시 10진수** 사용 (0x... 형식 사용 금지). 예: Apple=`76`. |
| `manufacturer_hex_prefix` | string | The manufacturer payload (hex string) must start with this prefix. E.g. iBeacon=`"0215"`. | 제조사 페이로드(hex)가 이 접두사로 시작해야 합니다. iBeacon=`"0215"`. |
| `manufacturer_min_length` | int | Minimum byte length of the manufacturer payload. | 제조사 페이로드 최소 바이트 수. |
| `name_prefix` | string | BLE device name must start with this string. | BLE 기기 이름이 이 문자열로 시작해야 합니다. |

> **⚠️ Hex literals warning / hex 리터럴 주의**  
> EN: The YAML parser (`kaml`) may silently fail with `0x...` hex values. Always use **decimal integers**.  
> KO: YAML 파서(`kaml`)는 `0x...` 형식을 잘못 파싱할 수 있습니다. **반드시 10진수** 정수를 사용하세요.  
> `0x004C` → `76`, `0x035D` → `861`

### instance_mode — Entity Strategy / 엔티티 전략

**EN** — Determines how HA entities are created when multiple devices of the same type are detected.

**KO** — 동일 타입의 기기가 여러 개 감지될 때 HA 엔티티를 어떻게 생성할지 결정합니다.

| Value | EN | KO |
|-------|----|----|
| `mac` (default) | Each unique MAC address creates its own set of HA entities. Entity unique_id: `{device_id}_{MAC}_temperature` etc. Bind a specific MAC in the app's Sensors tab to pre-declare entities before the device is seen. | MAC 주소마다 별도의 HA 엔티티 세트 생성. 앱의 Sensors 탭에서 MAC을 바인딩하면 기기가 감지되기 전에 엔티티가 미리 선언됩니다. |
| `shared` | All matching advertisers share a single HA entity. The last received packet overwrites the state. Useful for parking beacons, clickers, etc. | 모든 매칭 광고 기기가 하나의 HA 엔티티를 공유. 마지막 수신 패킷이 상태를 덮어씁니다. 주차 비콘 등에 유용. |

---

## source: gatt_notify — Active GATT Connection

**EN** — Connects to a specific BLE device, subscribes to a GATT characteristic, and receives notifications. Optionally sends hex commands back.

**KO** — 특정 BLE 기기에 연결하여 GATT 캐릭터리스틱을 구독하고 알림을 받습니다. 선택적으로 hex 명령을 전송할 수 있습니다.

```yaml
- id: custom_meter
  name: "Custom BLE Meter"
  source: gatt_notify
  gatt:
    mac: "AA:BB:CC:DD:EE:FF"   # 앱 Sensors 탭에서 바인딩하거나 여기에 직접 입력
    service_uuid: "ffe0"
    notify_char_uuid: "ffe1"
    write_char_uuid: "ffe2"    # 선택 — controls 사용 시 필요
    auto_connect: true
  sensors:
    - key: power
      unit: "W"
      device_class: power
      state_class: measurement
      decode: { offset: 0, length: 2, type: uint16, endian: little }
  controls:
    - key: relay
      type: switch
      command: { on: "A10100", off: "A10000" }
```

**gatt fields / gatt 필드:**

| Field | Type | Required | EN | KO |
|-------|------|----------|----|----|
| `mac` | string | ❌ | Device MAC address. Can also be set via app bind. | 기기 MAC 주소. 앱에서 바인딩으로 설정 가능. |
| `service_uuid` | string | ✅ | GATT service UUID (16-bit or full 128-bit). | GATT 서비스 UUID. |
| `notify_char_uuid` | string | ✅ | Characteristic to subscribe for notifications. | 알림을 구독할 캐릭터리스틱. |
| `write_char_uuid` | string | ❌ | Characteristic to write commands to (needed for controls). | 명령을 쓸 캐릭터리스틱 (controls 사용 시 필요). |
| `auto_connect` | bool | ❌ | Automatically reconnect on disconnect. Default `true`. | 연결 끊김 시 자동 재연결. 기본 `true`. |

> **EN** — If `mac` is not set in the YAML, bind the device in the app's **Sensors tab** (tap "Connect / Bind" next to the device).  
> **KO** — YAML에 `mac`이 없으면 앱 **Sensors 탭**에서 기기 옆 "Connect / Bind"를 탭해 바인딩합니다.

---

## source: obd — ELM327 OBD Polling

**EN** — Connects to an ELM327 BLE adapter, sends OBD-II (and GM mode 22) PID commands, and parses ISO-TP responses. Use built-in `preset`s or define custom `mode`/`pid`/`formula`.

**KO** — ELM327 BLE 어댑터에 연결하여 OBD-II (및 GM mode 22) PID 명령을 전송하고 ISO-TP 응답을 파싱합니다. 내장 `preset` 또는 직접 `mode`/`pid`/`formula`를 정의할 수 있습니다.

```yaml
- id: car_obd
  name: "Vehicle OBD"
  source: obd
  obd:
    # mac: "AA:BB:CC:DD:EE:FF"  # 앱에서 바인딩하거나 여기에 입력
    service_uuid: "18F0"
    tx_char_uuid: "2AF1"
    rx_char_uuid: "2AF0"
    tx_delay: 50ms
    init_commands:
      - "ATZ"
      - "ATE0"
      - "ATL0"
      - "ATS0"
      - "ATH0"
      - "ATSP6"     # 프로토콜 선택: ATSP0=auto, ATSP6=ISO 15765-4 CAN 500kbps
  sensors:
    # preset 사용 (권장)
    - { key: rpm,   preset: rpm,   update_interval: 1s }
    - { key: speed, preset: speed, update_interval: 1s }
    # 직접 정의
    - key: custom_pid
      mode: "01"
      pid: "0C"
      formula: "(a*256+b)/4"
      unit: "rpm"
      state_class: measurement
      update_interval: 1s
```

**obd fields / obd 필드:**

| Field | Type | Default | EN | KO |
|-------|------|---------|----|-----|
| `mac` | string | — | Adapter MAC. Set here or via app bind. | 어댑터 MAC. 여기 또는 앱 바인딩으로 설정. |
| `service_uuid` | string | `"FFF0"` | BLE GATT service UUID. | BLE GATT 서비스 UUID. |
| `tx_char_uuid` | string | `"FFF2"` | Write characteristic (send commands). | 쓰기 캐릭터리스틱 (명령 전송). |
| `rx_char_uuid` | string | `"FFF1"` | Notify characteristic (receive responses). | 알림 캐릭터리스틱 (응답 수신). |
| `tx_delay` | duration | `50ms` | Delay between consecutive commands. | 연속 명령 사이 딜레이. |
| `init_commands` | list | `[]` | Commands sent once on each connection before polling starts. | 폴링 시작 전 연결마다 한 번 전송하는 초기 명령. |
| `auto_connect` | bool | `true` | Automatically reconnect on disconnect. | 연결 끊김 시 자동 재연결. |

**OBD sensor-specific fields / OBD 센서 전용 필드:**

| Field | Type | EN | KO |
|-------|------|----|----|
| `preset` | string | Use a built-in preset (see table below). | 내장 프리셋 사용 (아래 표 참조). |
| `mode` | string | OBD mode. `"01"` = standard, `"22"` = manufacturer specific. Default `"01"`. | OBD 모드. `"01"`=표준, `"22"`=제조사 확장. 기본 `"01"`. |
| `pid` | string | PID hex code (without mode prefix). E.g. `"0C"`, `"199A"`. | PID hex 코드. 예: `"0C"`, `"199A"`. |
| `formula` | string | Math expression. Variables `a`,`b`,`c`,`d` are response bytes. E.g. `"(a*256+b)/4"`. | 수식. `a`,`b`,`c`,`d`는 응답 바이트. 예: `"(a*256+b)/4"`. |
| `update_interval` | duration | How often to poll this PID. Default `60s`. | 이 PID를 폴링하는 간격. 기본 `60s`. |
| `pre_commands` | list | Extra AT/OBD commands to send before this PID (e.g., header switch for GM). | 이 PID 전에 전송할 추가 명령 (예: GM 헤더 전환). |

---

## sensors — Sensor Config / 센서 설정

**EN** — Each item in the `sensors` list defines one HA entity.

**KO** — `sensors` 목록의 각 항목은 하나의 HA 엔티티를 정의합니다.

```yaml
sensors:
  - key: temperature           # 필수 — 고유 키 (같은 기기 내)
    platform: sensor           # sensor | binary_sensor | text_sensor
    device_class: temperature  # HA device_class (선택)
    unit: "°C"                 # 단위 (선택)
    state_class: measurement   # measurement | total | total_increasing (선택)
    icon: "mdi:thermometer"    # MDI 아이콘 (선택)
    entity_category: diagnostic # diagnostic | config (선택, 없으면 일반 엔티티)
    accuracy_decimals: 2       # 소수점 자리수 (선택)
    source_field: manufacturer_data  # advertisement만: 페이로드 소스
    min_length: 24             # 최소 페이로드 길이 (선택)
    decode:                    # 바이너리 디코딩 규칙
      offset: 18
      length: 2
      type: uint16
      endian: big
      scale: 0.002670288
      offset_value: -45
    publish:                   # 이 센서만의 발행 규칙 덮어쓰기 (선택)
      deadband: 0.1
      min_interval: 30s
```

| Field | Type | Required | EN | KO |
|-------|------|----------|----|----|
| `key` | string | ✅ | Unique key within the device. Becomes part of HA entity unique_id. | 기기 내 고유 키. HA entity unique_id의 일부. |
| `platform` | string | ❌ | HA entity platform. Default `sensor`. | HA 엔티티 플랫폼. 기본 `sensor`. |
| `device_class` | string | ❌ | HA device class (temperature, humidity, battery, voltage, …). | HA device class. |
| `unit` | string | ❌ | Unit of measurement shown in HA. | HA에 표시되는 단위. |
| `state_class` | string | ❌ | HA state class for statistics. | HA 통계용 state class. |
| `icon` | string | ❌ | MDI icon string, e.g. `"mdi:thermometer"`. | MDI 아이콘. 예: `"mdi:thermometer"`. |
| `entity_category` | string | ❌ | `diagnostic` or `config`. Omit for normal sensor. | `diagnostic` 또는 `config`. 일반 센서는 생략. |
| `accuracy_decimals` | int | ❌ | Round the decoded value to N decimal places. | 디코딩된 값을 소수 N자리로 반올림. |
| `source_field` | enum | ❌ | `manufacturer_data` / `service_data` / `raw`. Default `raw`. | 페이로드 소스. 기본 `raw`. |
| `min_length` | int | ❌ | Skip this sensor if the payload is shorter than N bytes. | 페이로드가 N바이트 미만이면 이 센서를 건너뜀. |
| `decode` | object | ❌ | Binary decoding rules. See below. | 바이너리 디코딩 규칙. 아래 참조. |
| `publish` | object | ❌ | Override publish rule for this sensor only. | 이 센서만의 발행 규칙 덮어쓰기. |

### decode — Binary Decoding / 바이너리 디코딩

**EN** — Extracts a numeric or string value from a raw byte array.

**KO** — raw 바이트 배열에서 숫자 또는 문자열 값을 추출합니다.

**Formula:** `final_value = (raw_integer * scale) + offset_value`

```yaml
decode:
  offset: 18          # 바이트 오프셋 (0-based)
  length: 2           # 읽을 바이트 수
  type: uint16        # 데이터 타입 (아래 표 참조)
  endian: big         # big | little
  bitmask: 0x0F       # (선택) AND 비트마스크 적용 후 디코딩
  scale: 0.1          # (선택) 곱할 계수. 기본 1.0
  offset_value: -40   # (선택) 더할 오프셋. 기본 0.0
  map:                # (선택) 정수값 → 문자열 매핑
    "0": "off"
    "1": "on"
    "2": "standby"
```

| Field | Type | Default | EN | KO |
|-------|------|---------|----|-----|
| `offset` | int | `0` | Start byte index (0-based). | 시작 바이트 인덱스 (0부터). |
| `length` | int | `1` | Number of bytes to read. | 읽을 바이트 수. |
| `type` | enum | `uint8` | Data type to interpret the bytes as. | 바이트를 해석할 데이터 타입. |
| `endian` | enum | `big` | Byte order. `big` = MSB first, `little` = LSB first. | 바이트 순서. `big`=MSB 먼저, `little`=LSB 먼저. |
| `bitmask` | int | `null` | Apply AND bitmask to the raw integer before scaling. | 스케일링 전 AND 비트마스크 적용. |
| `scale` | float | `1.0` | Multiply the raw integer by this value. | raw 정수에 곱할 계수. |
| `offset_value` | float | `0.0` | Add this after scaling: `(raw * scale) + offset_value`. | 스케일링 후 더하는 값. |
| `map` | object | `{}` | Map integer result to a string. Keys must be quoted strings. | 정수 결과를 문자열로 매핑. 키는 문자열로 따옴표 필요. |

### Data Types / 데이터 타입

| Type | Bytes | EN | KO |
|------|-------|----|-----|
| `uint8` | 1 | Unsigned 8-bit integer (0–255). | 부호 없는 8비트 정수 (0–255). |
| `int8` | 1 | Signed 8-bit integer (-128–127). | 부호 있는 8비트 정수 (-128–127). |
| `uint16` | 2 | Unsigned 16-bit integer (0–65535). | 부호 없는 16비트 정수. |
| `int16` | 2 | Signed 16-bit integer (-32768–32767). | 부호 있는 16비트 정수. |
| `uint32` | 4 | Unsigned 32-bit integer. | 부호 없는 32비트 정수. |
| `int32` | 4 | Signed 32-bit integer. | 부호 있는 32비트 정수. |
| `float32` | 4 | IEEE 754 single-precision float. | IEEE 754 단정밀도 부동소수점. |
| `timestamp` | 4 | Bytes interpreted as month/day/hour/minute (1 byte each). Returned as ISO 8601 string. | 각 바이트를 월/일/시/분으로 해석. ISO 8601 문자열 반환. |
| `string` | N | Bytes decoded as ASCII. Use with `platform: text_sensor`. | 바이트를 ASCII로 디코딩. `platform: text_sensor`와 함께 사용. |

---

## controls — HA Controls / HA 제어

**EN** — Define Home Assistant entities that send BLE write commands when triggered from HA. Only available for `gatt_notify` and `obd` sources.

**KO** — HA에서 제어 시 BLE 쓰기 명령을 전송하는 HA 엔티티를 정의합니다. `gatt_notify`와 `obd` 소스에서만 사용 가능합니다.

```yaml
controls:
  # Switch — on/off hex 명령
  - key: relay
    type: switch
    name: "Main Relay"           # (선택) HA 표시 이름. 없으면 key에서 자동 생성
    icon: "mdi:electric-switch"  # (선택)
    entity_category: config      # (선택)
    command:
      on:  "A10100"   # 켤 때 전송할 hex
      off: "A10000"   # 끌 때 전송할 hex

  # Number — 값을 hex에 삽입
  - key: brightness
    type: number
    min: 0
    max: 100
    step: 1
    command:
      template: "B2{value:02X}FF"  # {value} 또는 {value:형식} 사용

  # Select — 옵션별 hex 명령
  - key: mode
    type: select
    options: ["auto", "cool", "heat", "fan"]
    command:
      auto: "C000"
      cool: "C001"
      heat: "C002"
      fan:  "C003"

  # Button — 한 번 누르면 hex 전송
  - key: restart
    type: button
    command:
      press: "FFFE"
```

| Field | Type | Required | EN | KO |
|-------|------|----------|----|----|
| `key` | string | ✅ | Unique key within the device. | 기기 내 고유 키. |
| `type` | enum | ✅ | `switch` / `number` / `select` / `button`. | 제어 타입. |
| `name` | string | ❌ | Display name in HA. Auto-generated from key if omitted. | HA 표시 이름. 생략 시 key에서 자동 생성. |
| `icon` | string | ❌ | MDI icon string. | MDI 아이콘. |
| `entity_category` | string | ❌ | `config` or `diagnostic`. | `config` 또는 `diagnostic`. |
| `command` | object | ✅ | Command map. Keys depend on type (see examples above). | 명령 맵. type에 따라 키가 다릅니다 (위 예제 참조). |
| `options` | list | select만 | Options list for `select` type. | `select` 타입 옵션 목록. |
| `min` | float | number만 | Minimum value for `number` type. | `number` 타입 최솟값. |
| `max` | float | number만 | Maximum value for `number` type. | `number` 타입 최댓값. |
| `step` | float | ❌ | Step size for `number` type. | `number` 타입 단계. |

**command template syntax / command 템플릿 문법:**

| Placeholder | EN | KO |
|-------------|----|----|
| `{value}` | Integer value (decimal). | 정수 값 (10진수). |
| `{value:02X}` | Value as 2-digit uppercase hex (zero-padded). | 값을 2자리 대문자 hex로 (0 패딩). |
| `{value:04X}` | Value as 4-digit uppercase hex. | 4자리 대문자 hex. |

---

## publish — Per-Sensor Filter / 센서별 발행 필터

**EN** — Override the global `defaults.publish` for a specific sensor. Useful for sensors that change slowly (fuel level) or need frequent updates (RPM).

**KO** — 특정 센서에 대해 전역 `defaults.publish`를 덮어씁니다. 느리게 변하는 센서(연료량)나 빠른 업데이트가 필요한 센서(RPM)에 유용합니다.

```yaml
sensors:
  - key: fuel_level
    preset: fuel_level
    update_interval: 30s
    publish:
      on_change_only: true
      deadband: 0.5          # 0.5% 미만 변화는 무시
      min_interval: 60s      # 최소 1분 간격
      heartbeat: 30m         # 변화 없어도 30분마다 강제 전송
```

**Priority / 우선순위:** sensor `publish` > device `publish` > `defaults.publish`

---

## OBD Preset List / OBD 프리셋 목록

**EN** — Use `preset: <name>` in OBD sensors to use these built-in definitions. You can override individual fields (`unit`, `accuracy_decimals`, etc.) after specifying the preset.

**KO** — OBD 센서에서 `preset: <이름>`으로 내장 정의를 사용합니다. 프리셋 지정 후 개별 필드(`unit`, `accuracy_decimals` 등)를 덮어쓸 수 있습니다.

### Standard OBD-II (mode 01) / 표준 OBD-II

| Preset | PID | Formula | Unit | KO 설명 |
|--------|-----|---------|------|---------|
| `rpm` | `0C` | `(a*256+b)/4` | rpm | 엔진 RPM |
| `speed` | `0D` | `a` | km/h | 차속 |
| `engine_load` | `04` | `a/2.55` | % | 엔진 부하 |
| `throttle` | `11` | `a/2.55` | % | 스로틀 개도 |
| `coolant_temp` | `05` | `a-40` | °C | 냉각수 온도 |
| `intake_air_temp` | `0F` | `a-40` | °C | 흡기 온도 |
| `ambient_temp` | `46` | `a-40` | °C | 외기 온도 |
| `battery_voltage` | `42` | `(a*256+b)/1000` | V | 배터리 전압 |
| `run_time` | `1F` | `a*256+b` | s | 시동 후 경과 시간 |
| `fuel_level` | `2F` | `a/2.55` | % | 연료량 |
| `actual_torque` | `62` | `a-125` | % | 실제 토크 |
| `odometer` | `A6` | `(a*16777216+b*65536+c*256+d)/10` | km | 주행 거리 |

### GM Extensions (mode 22) / GM 확장

| Preset | PID | Formula | Unit | KO 설명 |
|--------|-----|---------|------|---------|
| `gm_current_gear` | `199A` | `a` | — | 현재 기어 |
| `gm_prnd_status_alt` | `2889` | `a` | — | PRND 상태 |
| `gm_oil_pressure` | `115C` | `(a*0.65)-17.5` | psi | 오일 압력 |
| `gm_trans_temp` | `1940` | `a-40` | °C | 변속기 온도 |
| `gm_fuel_level_liters` | `132A` | `(a*256+b)/64` | L | 연료량 (리터) |

---

## Full Examples / 전체 예제

### Example 1 — Jaalee JHT (iBeacon Temperature/Humidity)

```yaml
# Jaalee JHT 온도·습도 센서 — iBeacon 형식 (Apple 0x004C)
# instance_mode: mac → 각 기기 MAC마다 별도 엔티티 생성
- id: jaalee_jht
  name: "Jaalee JHT"
  source: advertisement
  instance_mode: mac
  match:
    manufacturer_id: 76          # Apple Inc. (반드시 10진수)
    manufacturer_min_length: 24
    manufacturer_hex_prefix: "0215"   # iBeacon 타입 마커
  sensors:
    - key: temperature
      device_class: temperature
      unit: "°C"
      state_class: measurement
      accuracy_decimals: 2
      source_field: manufacturer_data
      decode: { offset: 18, length: 2, type: uint16, endian: big, scale: 0.002670288, offset_value: -45 }
    - key: humidity
      device_class: humidity
      unit: "%"
      state_class: measurement
      accuracy_decimals: 2
      source_field: manufacturer_data
      decode: { offset: 20, length: 2, type: uint16, endian: big, scale: 0.001524504 }
    - key: battery
      device_class: battery
      unit: "%"
      state_class: measurement
      source_field: manufacturer_data
      decode: { offset: 23, length: 1, type: uint8 }
```

### Example 2 — Custom Service Data Sensor

```yaml
# 서비스 데이터에 센서 값이 포함된 커스텀 기기
- id: my_env_sensor
  name: "Environment Sensor"
  source: advertisement
  instance_mode: shared
  match:
    service_data_uuid: "181A"     # Environmental Sensing UUID
  sensors:
    - key: temperature
      device_class: temperature
      unit: "°C"
      state_class: measurement
      accuracy_decimals: 1
      source_field: service_data
      decode: { offset: 0, length: 2, type: int16, endian: little, scale: 0.01 }
    - key: humidity
      device_class: humidity
      unit: "%"
      state_class: measurement
      accuracy_decimals: 1
      source_field: service_data
      decode: { offset: 2, length: 2, type: uint16, endian: little, scale: 0.01 }
    - key: battery
      device_class: battery
      unit: "%"
      source_field: service_data
      decode: { offset: 9, length: 1, type: uint8 }
```

### Example 3 — GATT Sensor with Switch Control

```yaml
# GATT 기반 전력 모니터 + 릴레이 제어
- id: power_monitor
  name: "Power Monitor"
  source: gatt_notify
  gatt:
    service_uuid: "ffe0"
    notify_char_uuid: "ffe1"
    write_char_uuid: "ffe2"
  sensors:
    - key: power
      device_class: power
      unit: "W"
      state_class: measurement
      decode: { offset: 0, length: 2, type: uint16, endian: little }
    - key: voltage
      device_class: voltage
      unit: "V"
      state_class: measurement
      accuracy_decimals: 1
      decode: { offset: 2, length: 2, type: uint16, endian: little, scale: 0.1 }
    - key: current
      device_class: current
      unit: "A"
      state_class: measurement
      accuracy_decimals: 2
      decode: { offset: 4, length: 2, type: uint16, endian: little, scale: 0.01 }
  controls:
    - key: relay
      type: switch
      name: "Relay"
      icon: "mdi:electric-switch"
      command: { on: "A10100", off: "A10000" }
```

### Example 4 — OBD with Presets and Custom Publish

```yaml
# OBD 차량 센서 (프리셋 + 커스텀 발행 규칙)
- id: car_obd
  name: "Vehicle"
  source: obd
  obd:
    service_uuid: "18F0"
    tx_char_uuid: "2AF1"
    rx_char_uuid: "2AF0"
    tx_delay: 50ms
    init_commands: ["ATZ", "ATE0", "ATL0", "ATS0", "ATH0", "ATSP6"]
  sensors:
    - { key: rpm,          preset: rpm,          update_interval: 1s }
    - { key: speed,        preset: speed,        update_interval: 1s }
    - { key: coolant_temp, preset: coolant_temp, update_interval: 10s }
    - key: fuel_level
      preset: fuel_level
      update_interval: 30s
      publish:
        deadband: 1.0     # 1% 미만 변화 무시
        min_interval: 60s
        heartbeat: 10m
```

### Example 5 — Value Map (Status Codes)

```yaml
# 상태 코드를 문자열로 매핑하는 예제
- id: status_device
  name: "Status Device"
  source: advertisement
  instance_mode: shared
  match:
    manufacturer_id: 1234
  sensors:
    - key: mode
      platform: text_sensor
      source_field: manufacturer_data
      decode:
        offset: 5
        length: 1
        type: uint8
        map:
          "0": "idle"
          "1": "running"
          "2": "error"
          "3": "standby"
```

---

## Tips & Gotchas / 주의사항

### ⚠️ Always use decimal for manufacturer_id / manufacturer_id는 반드시 10진수

**EN** — The YAML parser (`kaml`) may silently parse `0x004C` as `0` or cause a crash. Always convert hex to decimal:

**KO** — YAML 파서(`kaml`)는 `0x004C`를 `0`으로 파싱하거나 오류를 낼 수 있습니다. 항상 hex를 10진수로 변환하세요:

```yaml
# ❌ 잘못된 방법 — 파싱 오류 가능
manufacturer_id: 0x004C

# ✅ 올바른 방법
manufacturer_id: 76       # 0x004C = 76
manufacturer_id: 861      # 0x035D = 861
```

### 📌 AND logic in match / match의 AND 논리

**EN** — All `match` conditions are ANDed together. If you specify both `service_data_uuid` and `manufacturer_id`, the packet must satisfy **both**. Remove a condition if your device doesn't always advertise it.

**KO** — 모든 `match` 조건은 AND로 결합됩니다. `service_data_uuid`와 `manufacturer_id`를 함께 지정하면 패킷이 **둘 다** 만족해야 합니다. 기기가 항상 특정 필드를 광고하지 않는다면 해당 조건을 제거하세요.

### 📌 instance_mode: mac — Bind for early entity creation / 조기 엔티티 생성을 위한 바인딩

**EN** — With `instance_mode: mac` and no fixed `match.mac`, HA entities only appear after the device is detected by BLE scan. To make entities appear immediately on gateway connect, bind the MAC in the app's **Sensors tab** → "Connect / Bind".

**KO** — `instance_mode: mac`이고 `match.mac`이 없으면 기기가 BLE 스캔에서 감지된 후에야 HA 엔티티가 나타납니다. 게이트웨이 연결 즉시 엔티티가 나타나도록 하려면 앱 **Sensors 탭** → "Connect / Bind"에서 MAC을 바인딩하세요.

### 📌 source_field selection / source_field 선택

**EN** — For advertisement sensors, choose the right payload source:
- `manufacturer_data` — data after the manufacturer company ID (most common for custom sensors)
- `service_data` — data in the service data AD type (common for standard profiles like 0x181A)
- `raw` — full scan record bytes (rarely needed)

**KO** — 광고 센서의 페이로드 소스 선택:
- `manufacturer_data` — 제조사 ID 이후 데이터 (커스텀 센서에서 가장 일반적)
- `service_data` — 서비스 데이터 AD 타입 데이터 (0x181A 등 표준 프로필에 일반적)
- `raw` — 전체 스캔 레코드 바이트 (거의 사용 안 함)

### 📌 Duration format / 시간 형식

**EN** — Durations support milliseconds (`ms`), seconds (`s`), and minutes (`m`).

**KO** — 시간은 밀리초(`ms`), 초(`s`), 분(`m`)을 지원합니다.

```yaml
tx_delay: 50ms
min_interval: 10s
heartbeat: 5m
update_interval: 1s
```

### 📌 Config is cached / 설정은 캐시됨

**EN** — The app caches the last successfully loaded config. If the network is unavailable on startup, it uses the cache. The app always shows "Using cached config" in the header when this happens.

**KO** — 앱은 마지막으로 성공적으로 불러온 설정을 캐시합니다. 시작 시 네트워크가 없으면 캐시를 사용합니다. 이 경우 앱 상단에 "캐시된 설정 사용 중"이 표시됩니다.

---

## License

MIT License — see [LICENSE](LICENSE).
