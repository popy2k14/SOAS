# System Of Alarm Sound - A cheap full featured alarm clock


|                                                                     |                             |
| ------------------------------------------------------------------- | --------------------------- |
| <img src="images/SOASClocks.jpeg" alt="SOAS" style="width:700px;"/> | ![Wake UP](images/soas.png) |




## What?
ESPHome with Home Assistant integration?! "No shit, Sherlock". Well yes, all ESPHome has got HA integration, but SOAS has features that enables you to have HA automations based on your alarm time. So automations can be triggered based on the alarm time set on the alarm clock. There are 4 switches that will switch relative to the alarm time, you have the choice to enable the HA switch whether the alarm will sound or not. So the alarm does not have to sound for the HA automation to be triggered.

## Why?
This alarm clock is customizable, full featured and smart for under €35,-. It's a clock that can be managed through Home Assistant and (mostly) on the clock itself. The clock can be set while your partner is asleep, a tradional clock could be making noise. And, because of the smart features, you could for instance enable the heating in your home 15 minutes before the alarm sets off. Especially when your wake up schedule is not regular, this clock can make automations efficient by associating the triggers with your alarm time.

## Features
* Alarm based on time.
* Different display modes so there is more or less light emitted by the clock.
* Contrast based on day or night.
* 4 Home Assistant integrated switches that switch based on the alarm time, alarm does not have to sound. These switches be switched manually on the clock itself.
* Customizable sleep timer.
* Customizable snooze timer.
* Online radio streams.

## Requirements
* < €25,-
* 3d printer (not included in the price)
* ESP32 devkit 1 ~ €5,-
* Oled screen
  - [SH1106](https://nl.aliexpress.com/item/1005007253095259.html) (128x64) ~ €2,50
  - [SH1107](https://nl.aliexpress.com/item/1005005313150711.html) (128x128) ~ €6,-
* MAX98357a amplifier ~ €3,-
* [3W speaker](https://nl.aliexpress.com/item/32593991938.html) ~ €3,-
* [Rotary button](https://nl.aliexpress.com/item/1005001877184897.html) < €1,-
* [Flat head button](https://nl.aliexpress.com/item/1005003400929705.html) ~ €1,50
* Dupont cables
* PLA
* Glue

## Optional
* A bit of soldering is not required, but the ground has to be shared so it is nice to solder that one. Depending on your rotary button, you maybe also need to do a little bit of soldering
* Design a fit for your speaker, or glue the speaker to the case

## Installation

3d print [these](https://www.thingiverse.com/thing:6948471) files. There is an example for the speaker. You can design your own speaker holder.

Connect all dupont cables corresponding the schema's below:

| SH1106 / SH1107 | ESP32  |
| --------------- | ------ |
| VCC             | 3v     |
| GND             | GND    |
| SCL             | GPIO23 |
| SDA             | GPIO18 |

| Rotary Button | ESP32  |
| ------------- | ------ |
| A             | GPIO26 |
| B             | GPIO25 |
| C             | GND    |
| D             | GND    |
| E             | GPIO27 |

| MAX98357a | ESP32  |
| --------- | ------ |
| LRC       | GPIO33 |
| BLCK      | GPIO22 |
| DIN       | GPIO19 |
| GND       | GND    |
| Vin       | 5v     |

| Flat head button | ESP32  |
| ---------------- | ------ |
| Switch           | GPIO21 |
| GND              | GND    |

Include the config below in your YAML:

```yaml
substitutions:
  display_model: "SH1107 128x128" #use "SH1106 128x64" for SH1106
  display_rotation: "270" #use 0 for SH1106

packages:
  remote_package_shorthand: github://skons/soas/alarm-clock-soas.yaml@main
  sun_latitude: 52.37°
  sun_longitude: 4.89°

time:
  - id: !extend ntp
    timezone: Europe/Amsterdam
    servers:
      - 0.pool.ntp.org
      - 1.pool.ntp.org
      - 2.pool.ntp.org

select:
  - id: !extend alarm_stream_url
    options:
      - "mp3 or pls url to radio" #AAC seams to be making SOAS crash
  - id: !extend alarm_stream_name
    options:
      - "Friendly name of the radio station"

esphome:
  name: alarm-clock-coen
  friendly_name: Alarm Clock Coen
  on_boot:
    - priority: 800
      then:
        #sometimes the media_player starts with clicking sounds, playing media solves the problem
        - media_player.volume_set: 0
        - media_player.play_media: !lambda return id(alarm_stream_url).state.c_str();
        - wait_until:
            media_player.is_playing :
        - media_player.stop:
        - media_player.volume_set: !lambda "return (id(alarm_volume).state/100);"

```

Save the `fonts` folder into your ESPHome folder. The folder needs to be placed in the same directory as your YAML.

## Usage

The rotary button is the button for accessing and editing configuration. When on a page, and there is no blinking of a configuration, you will automatically be redirected to the time page after 5 seconds of inactivity. The edit mode, blinking of a configuration, needs to be exited to return back to the time page. Entering and exiting the edit mode is done by single clicking the rotary button.

### Time page

#### Flathead short press
When the alarm, sleep timer and snooze are off, single press will switch the alarm on. If the sleep timer is enabled, the sleep timer will also switch on.

When the alarm is on, snooze will switch on and the alarm will go to off.

When snooze is on, the snooze will be switched off on single press.

#### Flathead long press
When the alarm is on, the flathead button needs to be pressed at least 2 seconds to disable the alarm.

If snooze is on, press it at least two seconds for disabling the snooze. Your alarm won't go off again until the next day.

Press it at least 2 seconds when snooze is not on, and the alarm is not sounding, to switch on the alarm. If the sleep timer is enabled, the alarm will go silent after the defined time.

#### Rotary single click
Single click of the rotary button toggles the alarm.

#### Rotary double click
Double click will bring you to the contrast page.

#### Rotary triple click
Triple click will bring you to the reboot page.

#### Display
Blinking of the alarm icon (most left icon) means that the radio is on. This happens when it is switched on manually or by the time set as alarm. When it is displayed, but not blinking it means the alarm is enabled. If the radio is on. the rotary button does not allow to switch pages but instead it will interact with the volume. If settings must be changed, the radio must be switched off.

The second icon is the snooze icon and tells you if snooze is on.

The Home Assistant icon is cut into 4 pieces. Left top is HA 1, right top is HA 2, left bottom is HA 3 and right bottom is HA 4. Is that part visible?, then that HA switch is enabled and it will go off relative to the alarm time. If it is blinking, then the HA switch is on. The on switch will be on for 1 minute and after that it will be automatically switched off.

Blinking of the sleep timer icon (second to last icon) means that the sleep timer is on. When it is displayed, but not blinking, the sleep timer is enabled.

The SH1107 display contains more pixel space, therefor the day of the week and date is displayed. The alarm time is also displayed, if snooze is on, then the time when the alarm will sound is displayed.

### Alarm page

#### Rotary single click
The alarm can be edited by single clicking the rotary button. This will make the time blink that is in edit mode. Single again and it will bring you to the minute. The rotary button is used to edit the hour or minute.

#### Rotary double click
Double click will enable or disable the alarm.

### Home Assitant page 1,2,3 and 4

This page displays if the Home Assistant switch is enabled, if the Home Assistant switch is associated with the alarm enabled and it shows the time relative to the alarm time. If the Home Assistant switch is enabled, it will switch to on even if the alarm is disabled.

#### Rotary single click
Single click of the rotary button will enable editing of the relative time, the rotary button can then be used to change the time.

#### Rotary double click
Double click will toggle the Home Assistant enabled switch and alarmed switch. The enabled switch (right top icon) means the HA switch will switch on relative to the alarm time. The alarmed switch (right bottom icon) means that is associated with the alarm being enabled. If HA is enabled, alarmed off, then the HA switch will switched based on the alarm time but the alarm does not have to be enabled. If HA is enabled and alarmed is switched on, then the HA switch will switch on only when the alarm is also enabled.

First double click will enable the enabled switch. The second double click will enable both enabled and alarmed switch. The third will disable both switches.

#### Rotary triple click
Triple click will switch the Home Assistant switch to on. This way it is possible to trigger your Home Assistant automation anytime you want. Regardless of the enabled and alarmed switched, and regardless of the alarm time.

### Sleep timer page

#### Rotary single click
With a single click with the rotary button you can define how long the music will play until it goes off.

#### Rotary double click
Double click will enable or disable the sleep timer.

### Radio page

Enter edit mode with a rotary button single click to select the station that will be used for the radio. Switching to a radio station means the radio will go on.

### Volume and contrast pages

The pages that are accessible from the time page are editable by rotating the rotary button. The contrast is depending on the day or night time. There is a different setting for both, and that can only be configured during that time.

### Restart page

Use the rotary button to select the correct option, single click the rotary button to restart or cancel your action.

### Options not available on the alarm itself

A few options are not (yet) available on the alarm self:
* Snooze duration
* Display Mode

Use Home Assistant to configure these options.

## Display mode

There are 3 display modes:
* Full
* Minimum night only
* Minimum

The `Full` means that all is visible. The second indicator `:` will only blink during day time.

The `Minimum night only` will have a smaller font for less light. The wifi icon will not be displayed unless there is no wifi. The second indicator does not blink. This will only be displayed during night time.

`Minimum` is the same as `Minimum night only`, except that it is the mode also during daytime.

## ToDo

* Home Assistant file stream as fallback on internet failure
* Local file as fallback when internet failure and Home Assistant failure/travel clock (https://esphome.io/guides/audio_clips_for_i2s.html)
* Ability to save streamed url to local instead of having a list of streams (https://alshowto.com/home-assistant-and-esphome-how-to-series-1-step-3-make-a-simple-media-speaker/, see things that are quirky)