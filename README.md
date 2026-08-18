# 🔔 Reolink Doorbell PoE Push Notifications for Home Assistant

Home Assistant automation for a **Reolink Video Doorbell PoE** that sends instant high-priority Android push notifications when the physical doorbell button is pressed.

The notification wakes the phone, displays a current camera snapshot, plays a dedicated doorbell sound and opens the live camera feed in Home Assistant when tapped.

## 🎯 Features

* 🔔 Uses the Reolink physical doorbell button as the Home Assistant trigger
* 📱 Immediate high-priority Android push notification
* 📸 Current camera snapshot directly in the notification
* 🎥 Tap the notification to open the live camera feed
* 🔊 Dedicated Android `deurbel` notification channel
* 🎵 Custom doorbell sound (ding-dong)
* 📲 Supports multiple mobile devices
* ✅ Tested with OPPO Find X9 Ultra / Android 16 / ColorOS 16

## 🛠️ Requirements

* Home Assistant
* Reolink Video Doorbell PoE
* Official Reolink Home Assistant integration
* Home Assistant Companion App for Android
* Reolink `Visitor` binary sensor
* Reolink camera entity

## ⚙️ Home Assistant configuration

This example uses the following Reolink entities:

* `binary_sensor.doorbell_visitor`
* `camera.doorbell_fluent`

Replace the `notify.mobile_app_phone_1` and `notify.mobile_app_phone_2` actions with the notification services of your own Android devices.

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

## 📱 Android / ColorOS configuration

The Home Assistant automation alone is **not enough** to make the doorbell behave like a real mobile doorbell notification.

On the tested OPPO device, several Android / ColorOS notification settings had to be changed before notifications arrived immediately and woke the screen.

### 1. Allow Home Assistant notifications

Open:

**Android Settings → Apps → Home Assistant → Notifications**

Enable:

* **Allow notifications**
* **Lock screen**
* **Banner**
* **Ringtone**
* **Vibration** if desired

Do **not** configure the Home Assistant notifications as silent.

### 2. Allow notifications on the lock screen

The Home Assistant notification must be allowed to appear while the phone is locked.

On the tested OPPO / ColorOS device:

* **Lock screen notifications:** enabled
* **Hide notification details on lock screen:** disabled if you want the camera/message information visible

### 3. Wake the screen for incoming notifications

This setting was required on the tested OPPO device.

Without it, Home Assistant received the notification immediately, but the display stayed black until the phone was manually awakened.

Enable the ColorOS option similar to:

**Wake screen when a notification is received**

The exact wording may vary slightly between ColorOS versions.

### 4. Use a high-priority Home Assistant push

The automation uses:

```yaml
ttl: 0
priority: high
```

This is important for doorbell notifications.

During testing, normal notifications were sometimes held back while the phone was asleep and only appeared after the device was awakened.

Using:

```yaml
ttl: 0
priority: high
```

made the push arrive immediately.

### 5. Create a dedicated `deurbel` notification channel

The automation uses:

```yaml
channel: deurbel
```

The first notification creates a separate Android notification category/channel named **deurbel**.

After it has been created, open:

**Android Settings → Apps → Home Assistant → Notifications → deurbel**

Configure this channel separately:

* Allow notifications
* Show on lock screen
* Enable Banner if desired
* Enable vibration if desired
* Select a dedicated ringtone

### 6. Add a custom doorbell sound

The tested OPPO device did not include a convincing traditional doorbell sound.

A custom **ding-dong** audio file was therefore added to the phone and selected as the ringtone for the Home Assistant `deurbel` notification channel.

On ColorOS this can be selected via:

**Home Assistant → Notifications → deurbel → Ringtone / Beltoon → On this device**

Using a dedicated notification channel means the custom doorbell sound is only used for Reolink doorbell notifications and does not affect other Home Assistant notifications.

### 7. Verify background notification delivery

If notifications still arrive late, check that Android / ColorOS is not restricting Home Assistant in the background.

Verify:

* Home Assistant is allowed to run in the background
* Battery optimization is not aggressively suspending Home Assistant
* Background data is allowed
* Home Assistant notifications are not blocked by a power-saving mode

## ✅ Tested behaviour

With the above settings, pressing the Reolink Video Doorbell PoE results in:

**Doorbell button pressed → Home Assistant Visitor trigger → immediate high-priority Android push → phone screen wakes → custom ding-dong sound → camera snapshot shown → tap notification to open live camera**

Tested with:

* **Reolink Video Doorbell PoE**
* **Home Assistant 2026.8.x**
* **Home Assistant Companion App for Android**
* **OPPO Find X9 Ultra**
* **Android 16**
* **ColorOS 16**


## 🔒 Privacy & security

No Reolink credentials, IP addresses or Home Assistant secrets are required in this automation.

Replace entity IDs and mobile notification service names with those from your own Home Assistant installation.

## 🏷️ Suggested GitHub Topics

`home-assistant` `home-automation` `reolink` `reolink-doorbell` `doorbell` `android` `automation` `yaml` `notifications`
