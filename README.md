# General Information

This Readme contains information about using the Brennenstuhl Connect Wifi Eco Line (WS EL01 DE) power strip with Tasmota
Tasmota related informations about the [Brennenstuhl Connect Wifi Eco Line WS EL01 DE](https://www.brennenstuhl.com/de-DE/produkte/smart-home/brennenstuhl-connect-eco-line-wifi-steckdosenleiste-1-5m-h05vv-f3g1-5-schwarz) power strip.

The device can be flashed to tasmota (as of 2020-10-16) with [tuya-convert](https://github.com/ct-Open-Source/tuya-convert)

<font color=Red>**WARNING**</font> There is no easy way to go back to the original firmware after flashing to Tasmota! The only way to flash the original firmware is to use a serial connection, which requires disassembling the device and soldering! So keep that in mind!

# GPIO List
| GPIO | Type | function |
|------|------|------------------|
| GPIO0 | Button | All On/Off |
| GPIO1 | Button | On/Off 2 |
| GPIO3 | Button | On/Off 1 |
| GPIO4 | Power toggle | Switchable socket group 1 |
| GPIO5 | Power toggle | Switchable socket group 2 |
| GPIO12 | LED | LED 1 RED |
| GPIO13 | LED | LED 2 GREEN |
| GPIO15 | LED | LED 2 RED |
| GPIO14 | Button | Learn |
| GPIO16 | LED | LED 1 GREEN |

# Templates
## Basic 1
Basic template, Learn button enabled. Use with ruleset. This is the **recommended** template.

`{"NAME":"WS EL01 DE","GPIO":[19,18,0,17,21,22,0,0,52,0,20,53,158],"FLAG":0,"BASE":18}`

In case you rather like the connection LED to be ***off*** when the socket is connected to Wifi, use this template:

`{"NAME":"WS EL01 DE","GPIO":[19,18,0,17,21,22,0,0,52,0,20,53,157],"FLAG":0,"BASE":18}`

To disable all LED's use this template. There will be ***no indicator*** if a switchable socket group is ***on*** or ***off***.

`{"NAME":"WS EL01 DE","GPIO":[19,18,0,17,21,22,0,0,0,0,20,0,0],"FLAG":0,"BASE":18}`

## Basic 2
Basic template, Learn button disabled. Use with ruleset.

`{"NAME":"WS EL01 DE","GPIO":[19,18,0,17,21,22,0,0,52,0,0,53,158],"FLAG":0,"BASE":18}`

In case you rather like the connection LED to be ***off*** when the socket is connected to Wifi, use this template:

`{"NAME":"WS EL01 DE","GPIO":[19,18,0,17,21,22,0,0,52,0,0,53,157],"FLAG":0,"BASE":18}`


# Rules
To make full use of the device templates, use rules. Rules are set using the web console.

## Ruleset 1
Add rule to toggle power on both switchable socket groups on the *All On/Off* button. Also contains an MQTT publish to a state subtopic `Learn` when the *Learn* button is pressed. 

The MQTT will only be sent if you use a Template where the *Learn* button is enabled, see above. The rule works with all templates.

`Backlog Rule1 ON Button3#state DO Power0 toggle ENDON ON Button4#state DO Publish stat/%topic%/Learn {"timestamp":"%timestamp%"} ENDON; Rule1 ON;`

## Ruleset 2

Rules for making the *All On/Off* button switch all switchable socket groups ***on*** when one socket group is ***on***, and ***off*** if both socket groups are ***on***. In my opinion this is better than [#Ruleset 1](#Ruleset%201)

The MQTT will only be sent if you use a Template where the *Learn* button is enabled, see above. The rule works with all templates.

This long version is just for reference/readability, use the single line Backlog command below to actually set the rules (""-lines are comments) .
```
Rule1
    "On boot, reset state variables to defined values. The #state rules below only trigger on change, so this is needed to init state."
    ON Power1#Boot DO Add1 %value% ENDON 
    ON Power2#Boot DO Add1 %value% ENDON 
    "Rules that sets var1 to 1 or 2 depending on how the #state of each relay are on. This part only gets executed on change."
    ON Power1#state=1 DO Add1 1 ENDON
    ON Power1#state=0 DO Sub1 1 ENDON
    ON Power2#state=1 DO Add1 1 ENDON
    ON Power2#state=0 DO Sub1 1 ENDON
    "Helper rule to switch what Button3 does on toggle"
    ON Var1#state>1 DO Var2 0 ENDON
    ON Var1#state<2 DO Var2 1 ENDON
    "Rule that toggles power for all relays on or off depending on how many relays are already on"
    ON Button3#state DO Power0 %var2% ENDON
    "Learn button publish rule"
     ON Button4#state DO Publish stat/%topic%/Learn {"timestamp":"%timestamp%"} ENDON
"Enable rule"
Rule1 ON
```

Use this to Copy&Paste to the Tasmota web console

`Backlog Rule1 ON Power1#Boot DO Add1 %value% ENDON ON Power2#Boot DO Add1 %value% ENDON ON Power1#state=1 DO Add1 1 ENDON ON Power1#state=0 DO Sub1 1 ENDON ON Power2#state=1 DO Add1 1 ENDON ON Power2#state=0 DO Sub1 1 ENDON ON Var1#state>1 DO Var2 0 ENDON ON Var1#state<2 DO Var2 1 ENDON ON Button3#state DO Power0 %var2% ENDON ON Button4#state DO Publish stat/%topic%/Learn {"timestamp":"%timestamp%"} ENDON; Rule1 ON`

If you have suggestions or comments feel free to submit an issue. I will try to update this.