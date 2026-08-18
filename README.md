# 🔔 Reolink Doorbell PoE Push Notifications for Home Assistant

A Home Assistant automation for the **Reolink Video Doorbell PoE** that sends an instant high-priority Android notification when the physical doorbell button is pressed.

The notification wakes the phone, displays a current camera snapshot, plays a dedicated doorbell sound, and opens the live camera feed in Home Assistant when tapped.

## 🎯 Features

* 🔔 Uses the physical Reolink doorbell button as the Home Assistant trigger
* 📱 Sends an immediate high-priority Android push notification
* 📸 Displays a current camera snapshot directly in the notification
* 🎥 Opens the live camera feed in Home Assistant when the notification is tapped
* 🔊 Uses a dedicated Android `deurbel` notification channel
* 🎵 Supports a custom doorbell sound (ding-dong)
* 📲 Supports multiple Android devices
* 🔒 No Reolink credentials or IP addresses required in the automation
* ✅ Tested with OPPO Find X9 Ultra / Android 16 / ColorOS 16

## 🛠️ Requirements

* Home Assistant
* Reolink Video Doorbell PoE
* Official Reolink integration for Home Assistant
* Home Assistant Companion App for Android
* Reolink `Visitor` binary sensor
* Reolink camera entity

## ⚙️ Home Assistant configuration

This example uses the following entities provided by the Reolink integration:

* `binary_sensor.doorbell_visitor`
* `camera.doorbell_fluent`

The `Visitor` binary sensor changes from `off` to `on` when the physical doorbell button is pressed.

Replace:

* `notify.mobile_app_phone_1`
* `notify.mobile_app_phone_2`

with the notification services of your own Android devices.

```yaml
alias: Bezoeker voordeur
description: ""

triggers:
  - trigger: state
    entity_id:
      - binary_sensor.doorbell_visitor
    from: "off"
    to: "on"

conditions: []

actions:
  - action: notify.mobile_app_phone_1
    data:
      title: "🔔 Deurbel"
      message: "Er staat iemand voor de deur"
      data:
        ttl: 0
        priority: high
        channel: deurbel
        image: "/api/camera_proxy/camera.doorbell_fluent"
        clickAction: "entityId:camera.doorbell_fluent"

  - action: notify.mobile_app_phone_2
    data:
      title: "🔔 Deurbel"
      message: "Er staat iemand voor de deur"
      data:
        ttl: 0
        priority: high
        channel: deurbel
        image: "/api/camera_proxy/camera.doorbell_fluent"
        clickAction: "entityId:camera.doorbell_fluent"

mode: single
```

> **Note:** Entity IDs and mobile notification service names may be different in your Home Assistant installation.

## 📱 Android / ColorOS configuration

The Home Assistant automation alone may not be enough to make the notification behave like a traditional doorbell.

During testing on an OPPO device, additional Android / ColorOS settings were required to make the notification arrive immediately, wake the display, show the camera preview, and play a dedicated doorbell sound.

### 1. Allow Home Assistant notifications

Open:

**Android Settings → Apps → Home Assistant → Notifications**

Enable:

* **Allow notifications**
* **Lock screen**
* **Banner**
* **Ringtone**
* **Vibration** if desired

Do not configure the Home Assistant notification as silent.

### 2. Allow notifications on the lock screen

The Home Assistant notification must be allowed to appear while the phone is locked.

On the tested OPPO / ColorOS device:

* **Lock screen notifications:** enabled
* **Hide notification details on lock screen:** disabled if you want the notification information and camera preview to be visible

### 3. Wake the screen for incoming notifications

This setting was essential on the tested OPPO device.

During initial testing, Home Assistant notifications were received, but the phone display remained black until the device was manually awakened.

Enable the ColorOS setting:

**Wake screen when a notification is received**

The exact wording or location of this setting may vary between Android devices and ColorOS versions.

### 4. Use a high-priority push notification

The automation uses:

```yaml
ttl: 0
priority: high
```

This is important for a doorbell notification, where delayed delivery is undesirable.

During testing, normal notifications could remain queued while the phone was asleep and appear only after the device was awakened.

Using `ttl: 0` together with `priority: high` made the notification arrive immediately.

### 5. Create a dedicated doorbell notification channel

The automation uses:

```yaml
channel: deurbel
```

The first notification using this channel creates a separate Android notification category named **deurbel**.

After the channel has been created, open:

**Android Settings → Apps → Home Assistant → Notifications → deurbel**

Configure this channel separately:

* Allow notifications
* Show notifications on the lock screen
* Enable **Banner**
* Enable vibration if desired
* Select a dedicated ringtone

This allows the Reolink doorbell notification to have its own behaviour and sound without changing other Home Assistant notifications.

### 6. Add a custom ding-dong sound

The tested OPPO device did not include a convincing traditional doorbell notification sound.

A custom **ding-dong** audio file was therefore added to the phone and assigned specifically to the Home Assistant `deurbel` notification channel.

On the tested ColorOS device, the sound can be selected through:

**Home Assistant → Notifications → deurbel → Ringtone / Beltoon → On this device**

Using a dedicated notification channel means the custom ding-dong sound is used only for the doorbell notification. Other Home Assistant notifications retain their normal notification sounds.

### 7. Verify background notification delivery

If notifications still arrive late, verify that Android / ColorOS is not restricting Home Assistant in the background.

Check that:

* Home Assistant is allowed to run in the background
* Battery optimization is not aggressively suspending Home Assistant
* Background data is allowed
* Home Assistant notifications are not blocked by a power-saving mode

## 🔄 Notification flow

With the configuration above, the complete notification flow is:

**Doorbell button pressed**
→ **Reolink `Visitor` sensor changes to `on`**
→ **Home Assistant automation triggers**
→ **High-priority Android push is sent**
→ **Phone screen wakes**
→ **Custom ding-dong sound plays**
→ **Current camera snapshot is displayed**
→ **Tap notification to open the live camera feed**

## 🧪 Tested configuration

This setup has been tested with:

| Component      | Configuration                               |
| -------------- | ------------------------------------------- |
| Doorbell       | Reolink Video Doorbell PoE                  |
| Integration    | Official Reolink Home Assistant integration |
| Home Assistant | 2026.8.x                                    |
| Mobile app     | Home Assistant Companion App for Android    |
| Mobile device  | OPPO Find X9 Ultra                          |
| Android        | Android 16                                  |
| OPPO software  | ColorOS 16                                  |

Other Android devices should also work, but notification, lock-screen, battery optimization, and screen-wake settings may differ between manufacturers.

## 🔧 Troubleshooting

### Notification only appears after waking the phone

Make sure the automation contains:

```yaml
ttl: 0
priority: high
```

Also verify that Home Assistant is not restricted by Android's background or battery optimization settings.

### Notification arrives but the screen stays black

Enable:

**Wake screen when a notification is received**

in the Android / ColorOS notification settings.

### Notification works but has the wrong sound

Open the Android notification settings for the Home Assistant **`deurbel`** channel and assign the desired ringtone.

For a traditional doorbell effect, a custom ding-dong audio file can be added to the phone and selected as the ringtone for this channel.

### Notification has no camera image

Verify that your camera entity exists and replace:

```text
camera.doorbell_fluent
```

with the camera entity ID used by your own Reolink integration.

### Tapping the notification does not open the camera

Verify that the `clickAction` references the same camera entity:

```yaml
clickAction: "entityId:camera.doorbell_fluent"
```

## 🔒 Privacy & security

This automation does not require Reolink credentials, camera passwords, IP addresses, or Home Assistant secrets to be stored in the automation.

Before publishing or sharing your configuration, replace personal device names and other installation-specific entity IDs where appropriate.

## 📄 License

This project is intended to be freely reusable and adaptable for other Home Assistant installations.

If this repository is published under the MIT License, see the `LICENSE` file for details.
