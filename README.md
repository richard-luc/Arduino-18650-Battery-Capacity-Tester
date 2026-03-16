# Arduino 18650 Battery Capacity Tester #

This project is an Arduino Nano compatible 18650 battery capacity tester. It can discharge a cell, measure voltage and current, estimate capacity in mAh, and monitor temperature with thermistors for safer testing.

This repository preserves the original idea while documenting the updates made for Richard's version of the tester.

## Highlights

### v0.1
 - Arduino VCC is calculated automatically.
 - Settings are easier to adjust through variables.
 - Added discharge percentage reporting.
 - Added battery and power resistor temperature monitoring.

### v0.2
 - Added battery type selection.
 - Added prototype-board support.
 - Kept the screen, button, and speaker off-board to simplify a future enclosure.
 - Added a temperature limit for the power resistor to stop the test when it gets too hot.

To support different battery types, v0.2 uses a struct with battery name, minimum voltage, and maximum voltage:
```cpp
// Structure of battery type
struct BatteryType {
	char name[10];
	float maxVolt;
	float minVolt;
};

#define BATTERY_TYPE_NUMBER 4
BatteryType batteryTipes[BATTERY_TYPE_NUMBER] = {
		{ "18650", 4.3, 2.9 },
		{ "17550", 4.3, 2.9 },
		{ "14500", 4.3, 2.75 },
		{ "6v Acid", 7.2, 6.44  }
};
```
The default voltage divider uses 10k resistors. If you want to support a different voltage range, update these values:
```cpp
// Battery voltage resistance
#define BAT_RES_VALUE_GND 10.0
#define BAT_RES_VALUE_VCC 10.0
// Power resistor voltage resistance
#define RES_RES_VALUE_GND 10.0
#define RES_RES_VALUE_VCC 10.0
```
If you are not using thermistors, set these values to `false`:
```cpp
#define USING_BATTERY_TERMISTOR true
#define USING_RESISTO_TERMISTOR true
```
If you use a different I2C display, update this method:
```cpp
void draw(void);
```

The project includes Fritzing files, schematics, and build photos.

*I use a generic character display with a custom-built I2C controller and library, but you can also use a standard I2C controller board with standard libraries and keep the same overall code flow.*
*All display logic is contained in the `draw()` function, so it can be replaced without changing the rest of the project.*

Based on an original idea from [OpenGreenEnergy](http://www.instructables.com/id/DIY-Arduino-Battery-Capacity-Tester-V10-/).

## Realization ##
Voltage is measured using a voltage divider. For background, see [Wikipedia](https://en.wikipedia.org/wiki/Voltage_divider) or this [blog article](https://startingelectronics.org/articles/arduino/measuring-voltage-with-arduino/).

In simple terms, this code
```cpp
batVolt = (sample1 / (1023.0 - ((batResValueGnd / (batResValueVolt + batResValueGnd)) * 1023.0))) * vcc;
```
measures battery voltage.

`batResValueGnd / (batResValueVolt + batResValueGnd)` is the multiplier factor for the divided voltage reading.

`sample1` is the average analog reading.

`vcc` is the Arduino reference voltage.

`1023.0` is the maximum ADC value for a 10-bit Arduino analog read.
	
The circuit measures the voltage before and after the power resistor so the current drawn from the battery can be calculated.

The MOSFET starts and stops the battery discharge path through the power resistor.

Two thermistors monitor battery and power resistor temperature for additional safety.

### Versione v0.2 ###
---

**On prototype board:**
I create a prototype board very extendible, but for now I use only minimal set of pin (in future led and other button).
If you want support voltage greater than 10v you must change the resistor value of Battery and Resistance in according to the formula
`(BAT_RES_VALUE_GND / (BAT_RES_VALUE_VCC + BAT_RES_VALUE_GND)` in the schema `Resistor power voltage GND 1/2 / (Resistor power voltage 2/2 + Resistor power voltage GND 1/2)`.

![Prototype](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PTesterV02_bb.png)

![Prototype schema](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PTesterV02_schem.png)

![Prototype real](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PB_07_all.jpg)

**Shopping List**

| Amount | Part Type | Properties |
| --- | --- | --- |
| 2 |  5mm Screw TermInal [eBay](https://www.ebay.com/itm/20pcs-2-Pole-5mm-Pitch-PCB-Mount-Screw-TermInal-Block-8A-250V-LW-SZUS/181932294528?epid=1553081429&hash=item2a5c028180:g:SUIAAOSwZjhXIvSX) | pin spacing 0.2in (5.08mm);  2 Pole 5mm Pitch PCB Mount Screw TermInal Block 8A 250V LW SZUS |
| 1 | Arduino Pro Mini clone (compatible Nano) [eBay](https://www.ebay.com/itm/New-design-Pro-Mini-atmega328-5V-16M-Replace-ATmega128-Arduino-Compatible-Nano/171558370113?hash=item27f1acfb41:g:CjwAAOSwQJhUdgXb) | tipo Arduino Pro Mini (Clone comp Nano); variant variant 1 |
| 1 | Basic FET P-Channel **IRF744N or IRLZ44N** [eBay](https://www.ebay.com/itm/5PCS-IRLZ44N-TO-220-IRLZ44-TO220-IRLZ44NPBF-Effect-Transistor/192345139104?hash=item2cc8a9e7a0:g:Ca4AAOSwqbxZ8cgT) | package TO220 [THT]; tipo p-channel |
| 11 | 10kΩ Resistor | bands 4; pin spacing 400 mil; tolerance ±5%; package THT; resistenza 10kΩ |
| 1 | Temperature Sensor (Thermistor) [eBay](https://www.ebay.com/itm/10Pcs-Temperature-Sensor-Ntc-Mf58-3950-B-10K-Ohm-5-Thermistor-New-Ic-T/232317930330?epid=1048450678&hash=item36173a4b5a:g:W5wAAOSwcLxYHcpf) | package THT; thermistor type NTC; resistance at 25° **10kΩ**; tipo thermistor |
| * | Generic male header | form ♂ (male); pin spacing 0.1in (2.54mm); package THT; row single; hole size 1.0mm,0.508mm; pins 2 |
| * | Generic female header | form ♀ (female); pin spacing 0.1in (2.54mm); package THT; row single; hole size 1.0mm,0.508mm; pins 2 |
| PerfBoard board | Prototype board [eBay](https://www.ebay.com/itm/Novelty-Universal-Bakelite-Circuit-Board-DIY-Prototype-PCB-Rectangle-Device-5pcs/142528992310?epid=1263113777&hash=item212f63d436:g:2kUAAOSwT6pVg3~e) | 24x18  |

[Complete list of all part](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PTesterV02_bom.html)

Soldering prototype:

[//]: # (![Pin up]&#40;https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PB_02_pin_up.jpg&#41;)

[//]: # (![Pin bottom]&#40;https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PB_01_pin_bottom.jpg&#41;)

[//]: # (![Pull up]&#40;https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PB_03_pulldown_up.jpg&#41;)

[//]: # (![pull down]&#40;https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PB_05_pulldown_connect_bottom.jpg&#41;)
![All up](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PB_06_allother_up.jpg)
![All bottom](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PB_07_allother_bottom.jpg)
![PB_TestMAHA](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/PB_TestMAHA.jpg)

**Display**

![Sel battery](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/Disp_sel_battery.jpg)
![Discharging](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/Disp_discharging.jpg)
![Battery removed](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/Disp_battery_removed.jpg)
![Resistance hot](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/Disp_Resistance_hot.jpg)


**On breadboard:**

[//]: # (![Breadboard]&#40;https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/BTesterV02_bb.png&#41;)

![Battery selection](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v02/BB_Select_battery.jpg)

**Serial output for debug:**

```
Volt: 2.94V - B Volt: 2.94V - Res Volt: 0.00V - Curr: 292mA - mAh: 219.4 - DISCHARGING: 1 - Battery temp: 19.47 - Resistance temp: 32.28
Volt: 2.94V - B Volt: 2.94V - Res Volt: 0.00V - Curr: 292mA - mAh: 219.4 - DISCHARGING: 1 - Battery temp: 19.47 - Resistance temp: 32.24
Volt: 2.94V - B Volt: 2.94V - Res Volt: 0.00V - Curr: 292mA - mAh: 219.4 - DISCHARGING: 1 - Battery temp: 19.44 - Resistance temp: 32.28
Volt: 2.92V - B Volt: 2.92V - Res Volt: 0.00V - Curr: 292mA - mAh: 219.7 - DISCHARGING: 1 - Battery temp: 19.49 - Resistance temp: 32.28
Volt: 2.94V - B Volt: 2.94V - Res Volt: 0.00V - Curr: 292mA - mAh: 219.7 - DISCHARGING: 1 - Battery temp: 19.46 - Resistance temp: 32.26
Volt: 2.94V - B Volt: 2.94V - Res Volt: 0.00V - Curr: 292mA - mAh: 219.7 - DISCHARGING: 1 - Battery temp: 19.42 - Resistance temp: 32.28
Volt: 2.92V - B Volt: 2.92V - Res Volt: 0.00V - Curr: 292mA - mAh: 219.7 - DISCHARGING: 1 - Battery temp: 19.46 - Resistance temp: 32.22
Volt: 2.94V - B Volt: 2.94V - Res Volt: 0.00V - Curr: 294mA - mAh: 220.0 - DISCHARGING: 1 - Battery temp: 19.46 - Resistance temp: 32.26
```


[//]: # (### Version v0.1 ###)

[//]: # (---)

[//]: # ()
[//]: # (## Wire schema v0.1 ##)

[//]: # (![Breadboard schema]&#40;https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v01/BTester_bb.png&#41;)

[//]: # (![Schema]&#40;https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v01/BTester_schem.png&#41;)

## Assembly list ##
| Label | Part Type | Properties |
| --- | --- | --- |
| **(LCD bb)** Display* | LCD screen | pins 16; tipo Character |
| **(LCD bb)** i2c controller* | PCF8574 | tipo PCF8574; package DIP16 [THT] |
| **(LCD bb)** 10K Trimmer potentiometer* | TRIMPOT | variant variant 2; package trimpot_pth_s3_lock_3386P |
| **(LCD bb)** 2N2222* | NPN-Transistor | tipo NPN (EBC); package TO92 [THT]; part number 2N2222 |
| 18650 Battery | LIPO-1000mAh | variant 1000mAh; package lipo-1000 |
| Beeper | Piezo Speaker |
| For battery | Screw terminal - 2 pins | pin spacing 0.1in (2.54mm); pins 2; hole size 1.0mm,0.508mm; package THT |
| For power resistor | Screw terminal - 2 pins | pin spacing 0.1in (2.54mm); pins 2; hole size 1.0mm,0.508mm; package THT |
| IRF744N | Basic FET N-Channel | tipo n-channel; package TO220 [THT] |
| MF58 | Temperature Sensor (Thermistor) | resistance at 25° 10kΩ; tipo thermistor; package THT; thermistor type NTC |
| MF58 | Temperature Sensor (Thermistor) | resistance at 25° 10kΩ; tipo thermistor; package THT; thermistor type NTC |
| Microcontroller | Arduino Pro Mini clone (compatible Nano) [eBay](https://www.ebay.com/itm/New-design-Pro-Mini-atmega328-5V-16M-Replace-ATmega128-Arduino-Compatible-Nano/171558370113?hash=item27f1acfb41:g:CjwAAOSwQJhUdgXb) | variant variant 1; tipo Arduino Pro Mini (Clone comp Nano) |
| Power resistor | 10kΩ Resistor | pin spacing 400 mil; bands 5; resistenza 10kΩ; tolerance ±5%; package THT; part number Power resistor |
| Resistance of battery volt. 1/2 | 10kΩ Resistor | pin spacing 400 mil; bands 4; resistenza 10kΩ; tolerance ±5%; package THT |
| Resistance of battery volt. 2/2 | 10kΩ Resistor | pin spacing 400 mil; bands 4; resistenza 10kΩ; tolerance ±5%; package THT |
| Resistance of power res. volt. 1/2 | 10kΩ Resistor | pin spacing 400 mil; bands 4; resistenza 10kΩ; tolerance ±5%; package THT |
| Resistance of power res. volt. 2/2 | 10kΩ Resistor | pin spacing 400 mil; bands 4; resistenza 10kΩ; tolerance ±5%; package THT |
| Thermistor battery resistance | 10kΩ Resistor | pin spacing 400 mil; bands 4; resistenza 10kΩ; tolerance ±5%; package THT |
| Thermistor power res. resistance | 10kΩ Resistor | pin spacing 400 mil; bands 4; resistenza 10kΩ; tolerance ±5%; package THT |

## Shopping List

| Amount | Part Type | Properties |
| --- | --- | --- |
| 1* | LCD screen | pins 16; tipo Character |
| 1* | PCF8574 | tipo PCF8574; package DIP16 [THT] |
| 1* | TRIMPOT | variant variant 2; package trimpot_pth_s3_lock_3386P |
| 1* | NPN-Transistor | tipo NPN (EBC); package TO92 [THT]; part number 2N2222 |
| 1 | LIPO-1000mAh | variant 1000mAh; package lipo-1000 |
| 1 | Piezo Speaker |
| 2 | Screw terminal - 2 pins | pin spacing 0.1in (2.54mm); pins 2; hole size 1.0mm,0.508mm; package THT |
| 1 | Basic FET N-Channel | tipo n-channel; package TO220 [THT] |
| 2 | Temperature Sensor (Thermistor) | resistance at 25° 10kΩ; tipo thermistor; package THT; thermistor type NTC |
| 1 | Arduino Pro Mini clone (compatible Nano) | variant variant 1; tipo Arduino Pro Mini (Clone comp Nano) |
| 1 | 10kΩ Resistor | pin spacing 400 mil; bands 5; resistenza 10kΩ; tolerance ±5%; package THT; part number Power resistor |
| 6 | 10kΩ Resistor | pin spacing 400 mil; bands 4; resistenza 10kΩ; tolerance ±5%; package THT |

*\* Only if you want assembly LCD i2c character controller*

**Breadboard**

![Breadboard](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v01/breadboard01.jpg)
![lcd on discharging](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v01/lcdDischarging02.jpg)

Thermistor on power resistor (heat shrink sleeve to hold).

![Thermistor on power resistance](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v01/thermistorPowerResistance.jpg)

Thermistor on battery.

![Thermistor on battery](https://github.com/richard-luc/Arduino-18650-Battery-Capacity-Tester/blob/main/Resources/v01/thermistorBattery.jpg)
