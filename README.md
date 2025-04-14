# Ariton Adrian TSC - proiect

# Implementation Steps

1. **Schematic Creation**
2. **Push to 2D PCB**
3. **Routing**
   - GND plane  
   - Net Class for Power Trails  
   - Custom rules & constraints upload  
   - Routing
4. **3D Models Design from Datasheets**
   - Battery  
   - Screen

---

# Hardware Modules and Components

## ESP32-C6-WROOM-1-N8 – Main Microcontroller
- WiFi 6, Bluetooth 5 LE, RISC-V core  
- Power Supply: 3.3V  
- Communications: SPI, I2C, UART, GPIO  

## SD Card Module
- Interface: SPI  
- Pins Used:
  - `SS_SD` → IO4  
  - `MOSI` → IO7  
  - `MISO` → IO2  
  - `SCK` → IO6  

## E-Paper Display
- Interface: SPI  
- Pins Used:
  - `EPD_CS` → IO10  
  - `EPD_DC` → IO5  
  - `EPD_RST` → IO23  
  - `EPD_BUSY` → IO3 (shared with MISO)  
  - MOSI, SCK (shared with SD)

## RTC Module - DS3231SN
- Interface: I2C  
- Pins Used:
  - `SCL` → IO22  
  - `SDA` → IO21  
  - `INT_RTC` → IO8  
  - `32KHZ` → IO1  
  - `RTC_RST` → IO18  

## Environmental Sensor - BME688
- Interface: I2C (shared with RTC)  
- Power Supply: 3.3V  

## External Flash - NORFlash64MB
- Interface: SPI  
- Pins Used:
  - `FLASH_CS` → IO12  
  - Rest (MOSI, MISO, SCK) shared  

## SD/USB Interface
- `USB_D+` → IO13  
- `USB_D-` → IO12  

## Reset and Boot Buttons
- `IO/BOOT` → IO9  
- `RESET` → EN  

## Li-Po Battery Charging Controller
- TP4056 or similar  
- Built-in charging control  
- Battery level monitoring with:
  - Battery ChargeLevel IC → I2C (IO19, IO20)

## Qwiic / Stemma QT Connector
- Communicates entirely via I2C  

## SPI ESD Protection Lines
- Protects the SPI lines for the SD card, e-paper, external flash  

## LDO Voltage Regulator
- Steps down 5V to 3.3V for ESP32-C6 and modules  

## Diodes
- For reverse polarity protection  

## USB-C Connector + ESD Protection
- Main power and data input  
- TVS diodes for ESD protection  

---

# 📊 Block Diagram: Schematic Design

```
+----------------+                                  +--------------------+
| USB C connector|                                  |      Buttons       |
|                |                                  |(Reset, Change, Boot|
+-------+--------+                                  |        IOs)        |
        |                                           +----------+---------+
        |                                                      |
+-------+--------+                                             |
|Battery Charging|                                             |
|  Controller    |-----------+                                   | USB
|   MCP73831     |           |                      +----------+---------+
+-------+--------+           |                      |                    |
        |                    |                      |                    |
        |          +----------------+    3v3 power  |                    |      +----------+
        |          |Voltage Regulator+------------->|                    |      |  SD Card  |
        |          |XC6220A331MR-G  |               |                    +----->|           |
        |          +--------+-------+               |                    |  SPI |    3v3    |
        |                   |                       |      ESP 32        |      +----------+
        |                   |                       |                    |
+-------+--------+          |                       |                    |      +----------+
|                |          |                       |        3v3         |      |NOR Flash  |
|     Battery    +----------+                       |                    +----->|           |
|                |                                  |                    |      |    3v3    |
+-------+--------+                                  |                    |      +----------+
        |                                           |                    |
        |                                           |                    |      +----------+
        |                    I2C                    |                    |      | Display   |
        |         +-------------+------------+------+                    +----->|           |
        |         |             |            |      +----------+---------+      |    3v3    |
        |   +----------+        |        +----------+          |                +----------+
        |   |          |        |        |          |          |         
        +---+ Battery  |  +-----+-----+  | BME688   |  +--------------+   
            | Level    |  |Real Time  |  | Sensor   |  | Stemma QT    |   
            |          |  |  Clock    |  |          |  |              |   
            |          |  |           |  |          |  |     3v3      |   
            +----------+  +-----------+  +----------+  +--------------+   

```

<img width="1477" alt="Screenshot 2025-04-14 at 20 53 05" src="https://github.com/user-attachments/assets/1cb439fe-4f91-40cd-89a8-e61312a6adb4" />





BOM:
Piece Name |	Piece Type (Optional) |	Link
--|--|--|
ESP32-CAP C0402 	|C |	https://componentsearchengine.com/part-view/CC0402MRX5R5BB106/YAGEO
ADAFRUIT_CHIP-LED0603 |	LED |	https://www.snapeda.com/parts/KP-1608SURCK/Kingbright/view-part/?ref=search&t=LED%200603
112ATAARR03 |	microSD |	https://www.snapeda.com/parts/112A-TAAR-R03/Attend/view-part/
744043680 |	L |	https://eu.mouser.com/ProductDetail/Wurth-Elektronik/744043680?qs=PGXP4M47uW6VkZq%252BkzjrHA%3D%3D
BD5229G-TR |	Voltage Detector |	https://www.snapeda.com/parts/BD5229G-TR/Rohm/view-part/?ref=search&t=BD5229G-TR
BUTTON_CUSYOMV1 |	Button |	https://www.snapeda.com/search/?q=EVQP7L01P&search-type=parts
CPH3225A |	C |	https://www.snapeda.com/parts/CPH3225A/Seiko/view-part/
DS3231SN# |	I²C-Integrated RTC/TCXO/Crystal |	https://www.snapeda.com/parts/DS3231SN%23/Analog%20Devices/view-part/?ref=search&t=DS3231SN%23
ESP32-C6-WROOM-1-N8 |	ESP32 |	https://www.snapeda.com/parts/ESP32-C6-WROOM-1-N8/Espressif%20Systems/view-part/?ref=search&t=ESP32-C6-WROOM-1-N8
ESP VARSISTOR |	Varsistor (B72520T0350K062) |	https://ro.mouser.com/ProductDetail/EPCOS-TDK/B72520T0350K062?qs=dEfas%2FXlABIszF52uu7vrg%3D%3D
ESP32_WROVER_AVX---SD0805S020S1R0 |	DIODE SCHOTTKY |	https://componentsearchengine.com/part-view/SD0805S020S1R0/Kyocera%20AVX
ESP32_WROVER_BME680_BME680 |	Env Senzor |	https://www.snapeda.com/parts/BME680/Bosch%20Sensortec/view-part/?ref=search&t=bme680
ESP32_WROVER_EAGLE-LTSPICE_R |	R |	https://componentsearchengine.com/part-view/R0402%201%25%20100%20K%20(RC0402FR-07100KL)/YAGEO
ESP32_WROVER_SPARKFUN-DISCRETESEMI_MOSFET_PCH |	T DMG2305UX-7 |	https://componentsearchengine.com/part-view/DMG2305UX-7/Diodes%20Incorporated
ESP32_WROVER_SPARKFUN-IC-POWER_MCP73831 |	TINY INTEGRATED LI-ION/LI-POLY CHARGE MGNT CONTROLLER |	https://componentsearchengine.com/part-view/MCP73831T-2ACI%2FOT/Microchip
FH34SRJ-24S-0.5SH_99_ |	FH34SRJ-24S-0.5SH(99) |	https://componentsearchengine.com/part-view/FH34SRJ-24S-0.5SH(99)/Hirose
MAX17048G+T10 |	Cell Fuel Gauge with ModelGauge |	https://www.snapeda.com/parts/MAX17048G+T10/Analog%20Devices/view-part/
MBR0530 |	Diode Schottky |	https://www.snapeda.com/parts/MBR0530/Onsemi/view-part/
PGB1010603MR |	Ipp Tvs Diode Surface Mount 0603 |	https://www.snapeda.com/parts/PGB1010603MR/Littelfuse/view-part/
QWIIC_CONNECTOR |	PRT-14417 QWIIC_CONNECTOR |	https://www.snapeda.com/parts/PRT-14417/SparkFun/view-part/
RCL_CPOL-EU |	C pol |	https://grabcad.com/library/tantalum-smd-capacitor-type-b-3528-1
SAMACSYS_PARTS_USB4110-GF-A |	USB4110-GF-A |	https://www.snapeda.com/parts/USB4110-GF-A./Global%20Connector%20Technology/view-part/
SJ |	Jumper-SolderPasteJumper3way |	https://grabcad.com/library/solder-jumpers-1
TP |	Test-Pad |	self-made
SI1308EDL-T1-GE3 |	MOSFET Transistor |	https://www.snapeda.com/parts/SI1308EDL-T1-GE3/Vishay/view-part/
USBLC6-2SC6Y |	Ipp Tvs Diode Surface Mount |	https://www.snapeda.com/parts/USBLC6-2SC6Y/STMicroelectronics/view-part/?ref=dk&t=USBLC6-2SC6Y&con_ref=None
W25Q512JVEIQ |	FLASH - NOR Memory |	https://www.snapeda.com/parts/W25Q512JVEIQ/Winbond%20Electronics/view-part/?ref=search&t=W25Q512JVEIQ
XC6220A331MR-G |	Voltage Regulator |	https://ro.mouser.com/ProductDetail/Torex-Semiconductor/XC6220A331MR-G?qs=AsjdqWjXhJ8ZSWznL1J0gg%3D%3D&utm_source=octopart&utm_medium=aggregator&utm_campaign=865-XC6220A331MR-G&utm_content=Torex%20Semiconductor
