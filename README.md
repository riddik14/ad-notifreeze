# NotiFreeze 🥶 🥵

[![python_badge](https://img.shields.io/static/v1?label=python&message=3.8%20|%203.9&color=blue&style=flat)](https://www.python.org) [![hacs_badge](https://img.shields.io/badge/HACS-Default-orange.svg)](https://github.com/hacs/integration)

> News ✌️ **NotiFreeze** **v2024.01.01** contiene **nuove funzionalità** e **cambiamenti importanti!** 🥶 Controlla sotto per ulteriori informazioni sul nuovo formato di configurazione!

---

[NotiFreeze](https://github.com/riddik14/ad-notifreeze) è un'app di [AppDaemon](https://github.com/appdaemon/appdaemon) che ricorda di chiudere le finestre se la differenza di temperatura tra interno ed esterno supera una soglia specificata.*

Questo funziona per ogni **`stanza`** separatamente, ad esempio una finestra aperta in bagno controlla la temperatura esterna rispetto al sensore di temperatura del bagno. Utile in inverno per ricordarti di chiudere le finestre del bagno dopo l'aerazione 🥶 ma anche in estate quando non vuoi che l'aria calda esterna entri 🥵

## Installazione

Usa [HACS](https://github.com/hacs/integration) o [scarica](https://github.com/riddik14/ad-notifreeze/releases) la directory `notifreeze` dall'interno della directory `apps` qui nel tuo `apps` directory locale, quindi aggiungi la configurazione per abilitare il modulo `notifreeze`.

## Rilevamento automatico di Entità/Sensori

Se le entità dei sensori hanno un ***ID entità*** che corrisponde a:

* ***binary_sensor.door_window_`*`***  
  **oppure**
* ***sensor.temperature_`*`***

**e**

* un ***ID entità*** o ***nome amichevole*** contenente il nome della **`stanza`**/**`stanza`**

**NotiFreeze** le rileverà automaticamente. (Le entità configurate manualmente avranno la precedenza.)

## Esempio di Configurazione

```yaml
notifreeze:
  module: notifreeze
  class: NotiFreeze
  locale: it_IT #en_US es_ES fr_FR de_DE zh_CN ru_RU
  notify_service: script.my_notify #notify.notify #notify.mobile_app_tuo_dispositivo
  always_notify: true
  outdoor: sensor.temperatura_esterna #il tuo sensore di temperatura esterna
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
      alexa_entity_id: "media_player.ovunque" #opzionali se non si usa il centro notifiche
      google_entity_id: "mediaplayer.tutti" #opzionali se non si usa il centro notifiche
      indoor:
        - sensor.cucina_temperatura
    - name: salone
      door_window: binary_sensor.salone_windows_sensor
     # alexa_entity_id: "media_player.ovunque" #opzionali se non si usa il centro notifiche
     # google_entity_id: "mediaplayer.tutti" #opzionali se non si usa il centro notifiche
      indoor:
        - sensor.salonetemperatura

```

nel codice notifreeze.py, alla riga 414, sostituisci con il tuo media_player.entity di Google.

per convertire per Alexa mediaplayer cambia dalla riga 411 alla 416 con questo

```
                # Send custom Alexa TTS notification
                await self.call_service(
                    "notify/alexa_media",
                    data={
                        "data": {"type": "tts"},
                        "message": message,
                        "target": "media_player.echo",  # Replace with your specific Alexa device entity_id
                    },
                )


#oppure

                # Send custom cento notifiche
                    await self.call_service(
                                "script/my_notify",
                                title=" notifreeze",
                                message=message,
                                google="1",
                                saluto="0",
                                call_no_annuncio="1",
                            )
```


                
### opzioni disponibili

key | optional | type | default | description
-- | -- | -- | -- | --
`module` | False | string | notifreeze | The module name of the app.
`class` | False | string | Notifreeze | The name of the Class.
`class` | True | string | en_US | Language! Available `en_US`, `de_DE` - contribute your language! 🤓 check below the code in [`notifreeze.py`](apps/notifreeze/notifreeze.py)!
`notify_service` | False | string | | Home Assistant notification service
`always_notify` | True | bool | false | Send notifications even when the indoor temperature is unchanged (compared to before the door/windows was open)
`outdoor` | False | string | | Sensor for outside temperature 🥵 🥶
`max_difference` | True | float | 5 | Maximum tolerated tmperature difference
`rooms` | False | list<string, [**room**](#room)> | | List of [**rooms**](#room) or simple *room* names NotiFreeze will monitor. 
`delays` | True | [**delay**](#delays) | [**see below**](#delays) | Delays NotiFreeze will use.
`messages` | True | [**message**](#messages) | default english | Custom notification messages
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



<a href="https://www.buymeacoffee.com/T1Pqksy" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/arial-black.png" alt="Buy Me A Coffee" style="height: 51px !important;width: 217px !important;" ></a>
