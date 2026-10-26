# Notify Open Sensors When Away – Home Assistant Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fsanftleben%2Fhomeassistant%2Frefs%2Fheads%2Fmain%2Fautomations%2Fopen_window_notification_on_leave.yaml)

A Home Assistant automation blueprint that sends a notification for every open door or window sensor the moment all tracked persons have left home.

---

## Features

- **Multiple persons** – select any number of `person.*` entities; the automation only triggers when **all** of them are away
- **Multiple sensors** – select any number of binary sensors (door, window, garage door, opening); each open sensor sends its own notification
- **Configurable notification service** – works with any `notify.*` service (mobile app, persistent notification, Telegram, etc.)
- **Customisable message** – set your own title and message prefix
- **Trigger delay** – optional grace period to avoid false alerts for very short absences

---

## Requirements

| Requirement | Details |
|---|---|
| Home Assistant | 2023.4 or newer (for `repeat: for_each`) |
| Person entities | At least one `person.*` entity with a linked device tracker |
| Binary sensors | Contact sensors with device class `door`, `window`, `garage_door`, or `opening` |
| Notify service | Any service under the `notify` domain |

---

## Installation

### Option A – Manual

1. Copy `notify_open_sensors_when_away.yaml` into your blueprints folder:
   ```
   config/blueprints/automation/custom/notify_open_sensors_when_away.yaml
   ```
2. In Home Assistant go to **Settings → Automations & Scenes → Blueprints**.
3. Click **Reload blueprints** (top-right menu).
4. Find *Notify Open Sensors When Away* and click **Create Automation**.

### Option B – My Home Assistant (HACS-style link)

Replace `<YOUR_HA_URL>` with your local Home Assistant address:

```
<YOUR_HA_URL>/config/blueprint/import?blueprint_url=<raw_file_url>
```

---

## Configuration

| Input | Description | Default |
|---|---|---|
| **Persons / Device Trackers** | One or more `person.*` entities to monitor | – |
| **Door / Window Sensors** | One or more `binary_sensor.*` entities to check | – |
| **Notification Service** | e.g. `notify.mobile_app_my_phone` | `notify.persistent_notification` |
| **Notification Title** | Title shown in the notification | `🏠 Left home – sensor still open!` |
| **Message Prefix** | Text before the sensor name in the body | `Warning: sensor is still open: ` |
| **Trigger Delay (seconds)** | Grace period before checking sensors (0–600 s) | `30` |

---

## How It Works

```
Person changes state to "not_home"
        │
        ▼
Are ALL persons away?  ──No──► Stop
        │ Yes
        ▼
Wait <trigger_delay> seconds
        │
        ▼
Are ALL persons still away?  ──No──► Stop
        │ Yes
        ▼
For each sensor in the list:
  Is sensor open (state == "on")?
    Yes ──► Send notification via <notify_service>
    No  ──► Skip
```

Each open sensor produces its own notification so you can see at a glance which specific door or window was left open.

---

## Example Notification

**Title:** `🏠 Left home – sensor still open!`  
**Message:** `Warning: sensor is still open: Front Door`

---

## Troubleshooting

**No notification received**
- Confirm all persons are actually in state `not_home` (check Developer Tools → States).
- Verify the notify service name is spelled correctly (case-sensitive).
- Check the Home Assistant automation trace for the exact condition that failed.

**Too many false alerts**
- Increase the *Trigger Delay* to give yourself more time to close doors/windows before leaving.

**Sensor not listed in selector**
- Make sure the sensor's `device_class` is set to `door`, `window`, `garage_door`, or `opening` in its entity settings.

---

## License

MIT – free to use and modify.
