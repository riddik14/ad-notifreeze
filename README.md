# NotiFreeze 🥶 🥵

[![python_badge](https://img.shields.io/static/v1?label=python&message=3.8%20|%203.9&color=blue&style=flat)](https://www.python.org) [![hacs_badge](https://img.shields.io/badge/HACS-Default-orange.svg)](https://github.com/hacs/integration)

> News ✌️ **NotiFreeze** **v0.5.0** contains **new features** and **breaking changes!** 🥶 Check below for more info about new config format!

---

[NotiFreeze](https://github.com/riddik14/ad-notifreeze) is an [AppDaemon](https://github.com/appdaemon/appdaemon) app which reminds to close windows if temperature difference between inside/outside exceeds a specified threshold.*  

This works for every **`room`** separately e.g. an open window in the bathroom checks outside temperate against the bathroom temperature sensor. Useful in winter to remind you to close the bathroom windows after airing 🥶 but also in the summer when you do not want that hot outside air inside 🥵

> **Note:** In **NotiFreeze** you configure just **one App for all your rooms** in contrast to separate apps/configurations per room like in [AutoMoLi](https://github.com/riddik14/ad-automoli).

## Installation

Use [HACS](https://github.com/hacs/integration) or [download](https://github.com/riddik14/ad-notifreeze/releases) the `notifreeze` directory from inside the `apps` directory here to your local `apps` directory, then add the configuration to enable the `notifreeze` module.

## Auto-Discovery of Entities/Sensors

If sensors entities have an ***entity id*** matching:

* ***binary_sensor.door_window_`*`***  
  **or**
* ***sensor.temperature_`*`***

**and**

* an ***entity id*** or ***friendly name*** containing the **`room`**/**`room`** name

**NotiFreeze** will detect them automatically. (Manually configured entities will take precedence.)

## Configuration Example

```yaml
notifreeze:
  module: notifreeze
  class: NotiFreeze
  locale: it_IT
  notify_service: script.my_notify #notify.notify #notify.mobile_app_iphone_*
  always_notify: true
  outdoor: sensor.temperatura_esterna #your external temperature sensor
  max_difference: 1
  delays:
    initial: 2
    reminder: 5
  message:
    - since: true
    - change: true
  rooms:
    - name: cucina
      door_window: binary_sensor.cucina_windows_sensor
      alexa_entity_id: "media_player.ovunque"
      google_entity_id: "mediaplayer.tutti"
      indoor:
        - sensor.cucina_temperatura
    - name: salone
      door_window: binary_sensor.salone_windows_sensor
      alexa_entity_id: "media_player.ovunque"
      google_entity_id: "mediaplayer.tutti"
      indoor:
        - sensor.salonetemperatura
```


in code notifreeze.py line 414 change with your google media_player.entity
### Available Options

key | optional | type | default | description
-- | -- | -- | -- | --
`module` | False | string | notifreeze | The module name of the app.
`class` | False | string | Notifreeze | The name of the Class.
`class` | True | string | en_US | Language! Available `en_US`, `de_DE` - contribute your language! 🤓 check below the code in [`notifreeze.py`](apps/notifreeze/notifreeze.py)!
`notify_service` | False | string | | Home Assistant notification service
`always_notify` | True | bool | false | Send notifications even when the indoor temperature is unchanged (compared to before the door/windows was open)
`outdoor` | False | string | | Sensor for outside temperature 🥵 🥶
`max_difference` | True | float | 5 | Maximum tolerated tmperature difference
`rooms` | False | list<string, [**room**](#room)> | | List of [**rooms**](#room) or simple *room* names NotiFreeze will monitor. Users of the famous [AutoMoLi](https://github.com/benleb/ad-automoli) may already by familiar with the *rooms* concept.
`delays` | True | [**delay**](#delays) | [**see below**](#delays) | Delays NotiFreeze will use.
`messages` | True | [**message**](#messages) | default english | Custom notification messages
~~`locale`~~ | ~~True~~ | ~~string~~ | ~~`en_US`~~ | **replaced by `messages`** ~~Locale for notifications in native language. See bottom of [`notifreeze.py`](apps/notifreeze/notifreeze.py) for available ones or add one yourself~~

## room

key | optional | type | default | description
-- | -- | -- | -- | --
`name` | True | string | | Name of the room (used for auto-discovery if no *alias* is set)
`alias` | True | string | | Alias used for auto-discovery of sensors (if your entity IDs not contain your *room*, this *alias* can be used)
`indoor` | True | string, list[string] | | Temperature sensor Entity ID(s)
`door_window` | True | string, list[string] | | Door/Windows sensor Entity ID(s)

## messages

key | optional | type | default | description
-- | -- | -- | -- | --
`since` | True | string | {room_name} {entity_name} open since {open_since}: {initial}°C | sent when temperature **did not change** since the door/windows was opened
`change` | True | string | {room_name} {entity_name} open since {open_since}: {initial}°C → {indoor}°C ({indoor_difference}°C) | sent when temperature **has changed** since the door/windows was opened

### variables

var | description | not available in message
-- | -- | --
`room_name` | name of the room
`entity_name` | name of the door/windows
`open_since` | time since opened
`initial` | indoor temperature when door/windows was opened
`indoor` | current indoor temperature | **since**, use `initial` for indoor temperate

## delays

key | optional | type | default | description
-- | -- | -- | -- | --
`initial` | True | integer | 5 | Time in minutes before sending first notification
`reminder` | True | integer | 3 | Time in minutes until next notification is send
