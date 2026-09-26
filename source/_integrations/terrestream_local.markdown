---
title: Terrestream
description: Connect a Terrestream Indoor Air Quality sensor over your local network.
ha_category:
  - Sensor
ha_iot_class: Local Polling
ha_release: "2026.10"
ha_config_flow: true
ha_zeroconf: true
ha_codeowners:
  - '@Xynergi'
ha_domain: terrestream_local
ha_platforms:
  - diagnostics
  - number
  - select
  - sensor
  - switch
  - text
  - time
ha_integration_type: device
ha_quality_scale: bronze
---

The **Terrestream** {% term integration %} connects a Terrestream Indoor Air Quality sensor to Home Assistant over your local network. It provides air quality and room readings for dashboards and automations, plus controls for the sensor’s display, sound, and quiet hours. No Terrestream account or MQTT broker is required.

## Supported devices

- Terrestream Indoor Air Quality sensor with firmware 4.1.0 or later

## Prerequisites

1. Connect your sensor to Wi-Fi and make sure its IP address or hostname is reachable from Home Assistant.
2. If your network restricts connections between devices, allow Home Assistant to reach the sensor on TCP port 6054 for pairing and TCP port 6053 for normal use. Keep these ports off the public internet.
3. On the sensor, open **Setup** > **Home Assistant** > **Pair**. Keep the displayed eight-digit code ready. It expires after two minutes.

Firmware updates are supplied by Terrestream and require internet access. Reading measurements through this integration does not require an internet connection.

{% include integrations/config_flow.md %}

On a network that supports multicast discovery, the sensor appears automatically in **Settings** > **Devices & services**. Select it to begin pairing. If discovery is unavailable, add **Terrestream** manually and enter its reachable local address.

Enter the sensor's local address and all eight digits of the displayed code, including any leading zeros. If the code expires, open a new pairing window on the sensor.

{% configuration_basic %}
IP address or hostname:
  description: "The sensor's reachable local address. Do not include a URL scheme or port."
Pairing code:
  description: "The eight-digit code displayed by the sensor."
{% endconfiguration_basic %}

## Supported functionality

### Sensors

The integration provides the following sensor entities:

- Carbon dioxide, in ppm
- PM1, PM2.5, PM4, and PM10, in µg/m³
- Temperature, in °C
- Humidity, in %
- Pressure, in hPa
- Illuminance, in lx
- VOC index
- NOx index
- Computed EPA particulate AQI

Home Assistant can convert supported measurement units for display. VOC and NOx indices are relative values, not gas concentrations. The computed particulate air quality index (AQI) uses instantaneous PM2.5 and PM10 readings with the United States Environmental Protection Agency's 2024 breakpoints. It is not a daily outdoor AQI or the sensor's optional AQHI+ display value.

### Device settings

Go to **Settings** > **Devices & services** > **Devices**, select your Terrestream sensor, and open its **Configuration** controls. Each sensor has its own settings. Changes are saved on the sensor and do not require a Terrestream account or internet access.

The integration exposes only settings advertised by your sensor. It does not change settings when you pair or reload it.

#### Display and sound switches

- **Dark theme**
- **Fahrenheit display**
- **24-hour clock**
- **Automatic brightness**
- **Quiet hours**
- **Touch sounds**
- **Notification sounds**

These controls affect the physical sensor. **Fahrenheit display** does not change Home Assistant’s temperature units. Sound controls affect the sensor’s speaker, not phone notifications.

#### Brightness and volume

Adjust levels with sliders using the sensor’s native scales, in whole-number steps:

- **Display brightness**: 10–255
- **Ring brightness**: 0–64; 0 turns the ring off
- **Speaker volume**: 0–10
- **Quiet display brightness**: 1–255
- **Quiet ring brightness**: 0–64
- **Quiet speaker volume**: 0–10

Automatic brightness and quiet hours can affect the active brightness. The sliders show saved preferences, not a measurement of current light output.

#### Quiet-hours schedule and time zone

Set **Quiet hours start** and **Quiet hours end** with the time controls, then enable **Quiet hours**. The times use the sensor’s **Device time zone**, which can differ from Home Assistant’s time zone. Use whole minutes, with seconds set to zero.

For **Device time zone**, enter an IANA name such as `Europe/London` or `America/New_York`. The sensor must support the selected time zone.

#### Display choices

- **Device language**: English or French (Canada)
- **Display air index**: EPA AQI or AQHI+
- **VOC presentation**: VOC index, WELL, or RESET

These choices affect the sensor’s display. They do not change the meaning of the Home Assistant measurement entities.

You can also use these controls in automations through the standard **Turn on switch**, **Turn off switch**, **Set number value**, **Select option**, **Set time**, and **Set text value** actions. Select the setting entity belonging to the sensor you want to control.

## Use cases

- Track carbon dioxide trends in a bedroom or home office.
- Compare particulate readings before and after running an air cleaner.
- Display temperature, humidity, and illuminance alongside other room sensors.

## Terrestream automation examples

### Automation: Notify when carbon dioxide stays above a threshold

Use the carbon dioxide threshold blueprint to show a Home Assistant notification when your selected reading stays above a threshold for a chosen duration. You choose the threshold; its initial value is an example, not a health limit or a recommendation to open windows regardless of outdoor conditions.

[Import the threshold notification blueprint](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fwww.home-assistant.io%2Fblueprints%2Fintegrations%2Fterrestream_local%2Fco2_notification.yaml)

The waiting period starts again after Home Assistant restarts or automations are reloaded. The automation uses standard numeric-state triggers and notification actions; this integration defines no custom actions, triggers, or conditions.

## Data updates

Home Assistant {% term polling polls %} the sensor every five seconds. Air quality, temperature, humidity, and pressure readings expire 60 seconds after acquisition. Illuminance readings expire after 20 seconds. Network delay counts toward these limits.

Settings are refreshed with each poll. After a setting change, Home Assistant immediately reads the device state back. Controls show the value reported by the sensor rather than assuming the requested value was applied. Changes made on the sensor or through Bluetooth appear after the next successful poll.

Missing, warming, cleaning, invalid, or stale readings appear as unavailable. Unavailable readings are never replaced with zero. Readings missed during a connection outage are not added to the history later.

## Known limitations

- Only one Home Assistant instance can maintain an active connection to a sensor at a time.
- Device setup, Bluetooth onboarding, and firmware update checks can temporarily interrupt the local connection.
- This integration does not expose firmware update controls, cloud-sharing controls, website situation recognition, or Pro intelligence.
- Pairing with Home Assistant does not change the sensor's existing cloud-sharing settings.

## Troubleshooting

### Cannot connect

Check the sensor's address and Wi-Fi connection. If you are pairing, open a new pairing window and try again. Make sure your network allows the connections listed under [Prerequisites](#prerequisites).

### Pairing failed

Enter all eight digits of the current code, including any leading zeros. If the code has expired, select **Pair** on the sensor again to display a new code.

### Readings are unavailable

Allow sensor warm-up, fan cleaning, or maintenance to finish. If you reload the integration during a firmware update check, leave the sensor powered on and allow up to two minutes for reconnection before troubleshooting further. Repeated reloads can delay recovery.

If another Home Assistant instance is connected to the sensor, stop that instance, wait one minute, and reload this integration.

### Renew pairing

If Home Assistant reports that pairing is no longer valid, keep the existing integration entry. On the original sensor, open **Setup** > **Home Assistant**, disconnect the old pairing, then select **Pair**. Open the Home Assistant repair notification and enter the current address and new code. This preserves the existing entry, entities, and their settings.

### Change the sensor address

Home Assistant can update the address when it discovers the original sensor at its new address and verifies its saved credentials and certificate. If discovery is unavailable, open the integration entry's menu and select **Reconfigure**. Enter the original sensor's new IP address or hostname. A different sensor cannot replace the original entry through this flow.

Reconfigure changes only the local address. There are no other integration options. Device settings and cloud-sharing preferences are not changed.

### A setting could not be changed

If settings changed on the sensor before your request arrived, wait for the controls to refresh and try again. If the sensor is busy, allow maintenance to finish first.

If a command was sent but its value could not be confirmed, wait for the sensor to reconnect and check the displayed value before retrying. A connection failure can occur after the sensor has already saved a change.

### Settings-storage warning

Let any firmware update finish, restart the sensor once, and check whether its settings persist. If the warning remains, contact support@terrestream.com. Avoid a factory reset as a first troubleshooting step. The warning clears after the sensor confirms that settings storage is working again.

### Download diagnostics

Open the integration entry's menu and select **Download diagnostics**. The Terrestream-specific data contains firmware version, connection status, and measurement-validity labels. It excludes sensor addresses, credentials, and measurement values. Home Assistant also includes system information and any repair issues, which can contain your integration entry name. Review the file before sharing it with support. Do not share complete Home Assistant backups or pairing codes.

## Removing the integration

{% include integrations/remove_device_service.md %}

After removing the integration, open **Setup** > **Home Assistant** > **Disconnect** on the sensor and confirm. This revokes the pairing credentials while preserving Wi-Fi settings and the Terrestream account binding.

Home Assistant backups contain pairing credentials. Keep backups private. Removing an integration entry alone does not revoke credentials held in an old backup; use **Disconnect** on the sensor to revoke them.
