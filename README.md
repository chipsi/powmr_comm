# PowMr Inverter serial communication protocol 

Would you like to monitor and control your PowMr solar hybrid inverter via MQTT?

Here is the reverse engineering of the communication protocol. The infomation might be incomplete, feel free to contribute. All communications has been sniffed out of the original Wifi dongle.

# Compatibility

Tested with:

- [**POW-HVM1.5H-12V**](https://powmr.com/products/all-in-one-inverter-charger-1500watt-220vac-12vdc)
- **POW-HVM2.0H-12V**
- [**POW-HVM2.4H-24V**](https://web.archive.org/web/20230329235125/https://powmr.com/inverters/all-in-one-inverter-chargers/powmr-2400watt-dc-24v-ac-220v-solar-inverter-charger)
- [**POW-HVM3.2H-24V**](https://powmr.com/products/all-in-one-inverter-charger-3000w-220vac-24vdc)
- [**POW-HVM10.2M**](https://powmr.com/products/hybrid-inverter-charger-10200w-200vac-48vdc)

However, also other models that are supported by the [**WIFI-VM**](https://powmr.com/products/powmr-wifi-module-with-rs232-remote-monitoring-solution-wifi-vm) device should work: 

- [**POW-HVM3.6M-24V**](https://powmr.com/products/hybrid-inverter-charger-3600w-220vac-24vdc)
- [**POW-HVM4.2M-24V**](https://powmr.com/products/hybrid-inverter-charger-4200w-220vac-24vdc)
- [**POW-HVM5.5K-48V**](https://powmr.com/products/all-in-one-inverter-charger-5500w-220vac-48vdc)
- [**POW-HVM6.2M-48V**](https://powmr.com/products/hybrid-inverter-charger-6200w-220vac-48vdc)
- [**POW-HVM8.2M**](https://powmr.com/products/hybrid-inverter-charger-8000w-220vac-48vdc)

# Communication

The device communicates with RS232 modbus protocol at 2400 baud (1 stop bit, no parity control).
PowMr device slave id is 5.

RJ45 connector is located on the left side of the inverter and it has the following pinout:

- pin 8 (Brown)          Ground 
- pin 4 (Blue)           +12V
- pin 2 (Orange)         RX (data to inverter)
- pin 1 (White Orange)   TX (data from inverter)

<img src="rs232_pinout.svg" width="512px" />

Modbus Holding registers to control inverter:
- 5017 charger source priority (range 0-3, settings menu 16)
- 5018 output source priority (range 0-2, settings menu 1)
- 5024 utility charge current (one of 2, 10, 20, 30, 40, 50, 60, settings menu 11)
- 5002 buzzer alarm (range 0-1, settings menu 18)
- 5007 beeps when primary source interrupted (range 0-1, settings menu 22)
- 5009 transfer to bypass on overload (0-1, settings menu 23)
- 5022 max total charge current (range 10-80, settings menu 2)
- 5025 comeback utility mode voltage (SBU) (0.5 volts step, settings menu 12) 
- 5026 comeback battery mode voltage (SBU) (0.5 volts step, settings menu 13)

  Thanks to Andrii Ganzevych we have more complete registers list here:

  https://github.com/odya/esphome-powmr-hybrid-inverter/blob/main/docs/registers-map.md


Original WiFi dongle sends two requests every few seconds to monitor inverter state. 
- read 45 registers (func 3) from slave 5 starting from address 4501 (Decimal) (raw: 05031195002d9143)
- read 16 registers (func 3) from slave 5 starting from address 4546 (Decimal) (raw: 050311c20010e142)

Please refer wifi bridge code for state registers description.

# ESP8266 MQTT Modbus RS232 bridge
The code in this repo providing a wifi MQTT-modbus bridge. It compiles in Platformio. The hardware part is simple esp8266 module with RS232 level converter attached to the 
>#define SS_TX_PIN   D8    // GPIO15
>
>#define SS_RX_PIN   D7    // GPIO13


It's possible to read and write modbus registers. Also it constatly pools data from inverter and publishes varios state variables like battery voltage, pv power, etc. You can store that data later in influxdb/postgres and visualize in Grafana.
The controller part of the code responsible for keeping battery voltage at the predefined value. (to prolong li-ion battery life)

**Important note:** during operation inverter produces a lot of RF noise. To make communnication with inverter more stable you might want to attach small capactor to RX pin of the ESP8266 (GPIO13). It helps to filter out RF noise.

# Additional references:
https://www.dessmonitor.com/ main website for data logging (official PowMr dongle)

https://powmr.com/ PowMr website with manuals

https://github.com/syssi/esphome-smg-ii ESPHome project to monitor and control a ISolar/EASUN SMG II inverter via RS232

https://github.com/odya/esphome-powmr-hybrid-inverter ESPHome config for various PowMr Hybrid Inverter models.
