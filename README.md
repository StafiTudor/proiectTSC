# proiectTSC
# InkTime Smart Watch

Acest proiect reprezintă designul hardware complet pentru un smartwatch bazat pe ecosistemul nRF52840, incluzând management avansat al bateriei, afișaj E-paper și feedback haptic.

## 1. Diagrama Bloc a Sistemului

Arhitectura proiectului este împărțită în 3 etaje principale: Input & Management Energie (Power), Unitatea Centrală de Procesare (Core MCU) și Interfețe Externe (Periferice și RF).

```mermaid
graph LR
    %% Alimentare
    USB[USB-C KH-TYPE-C-16P] -->|5V| BQ[BQ25180 LiPo Charger]
    BQ -->|Încărcare| BAT[LiPo Battery 3.7V]
    BAT -->|VBAT| MAX[MAX17048 Fuel Gauge]
    BAT -->|VBAT| RT[RT6160 DC/DC Converter]
    
    %% Conexiuni Power catre MCU
    RT -->|3.3V| MCU[nRF52840 MCU + BLE]

    %% I2C Bus
    BQ <-->|I2C| MCU
    MAX <-->|I2C| MCU
    RT <-->|I2C| MCU
    IMU[BMA423 Accelerometru] <-->|I2C| MCU
    HAPTIC[DRV2605 Haptic Driver] <-->|I2C| MCU

    %% Ieșiri Haptice
    HAPTIC -->|Drive| MOT[Motor Vibrații LRA]

    %% Alte interfețe
    MCU -->|SPI| EPD[E-Paper Display]
    MCU ---|RF| ANT[Antenă Ceramică Johanson]
    BTN[Butoane Tactile x3] -->|GPIO| MCU
    SWD[TC2030 SWD Debug] <-->|SWD| MCU


## 2. Componente principale și linkuri utile (BOM)

| Componenta | Link JLCPCB | DATASHEET |
| :--- | :--- | :--- |
| nRF52840 | [JLCPCB](https://jlcpcb.com/partdetail/NordicSemiconductor-nRF52840_QIAA_R/C190764) | [Nordic Product Specification](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.1.pdf) |
| BQ25180YBGR | [JLCPCB](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C5202685) | [TI BQ25180 datasheet](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| RT6160AWSC | [JLCPCB](https://jlcpcb.com/partdetail/Richtek-RT6160AWSC/C470211) | [Richtek RT6160A datasheet](https://www.richtek.com/assets/product_datasheet/RT6160/DS6160-01.pdf) |
| MAX17048G+T10 | [JLCPCB](https://jlcpcb.com/partdetail/AnalogDevices-MAX17048G_T/C297892) | [MAX17048/MAX17049 datasheet](https://datasheets.maximintegrated.com/en/ds/MAX17048-MAX17049.pdf) |
| BMA423 | [JLCPCB](https://jlcpcb.com/partdetail/BoschSensortec-BMA423/C236056) | [BMA423 datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma423-ds004.pdf) |
| DRV2605YZFR | [JLCPCB](https://jlcpcb.com/partdetail/TexasInstruments-DRV2605YZFR/C135540) | [TI DRV2605 datasheet](https://www.ti.com/lit/ds/symlink/drv2605.pdf) |
| 2450AT18B100E | [JLCPCB](https://jlcpcb.com/partdetail/JohansonTechnology-2450AT18B100E/C104593) | [Johanson antenna datasheet](https://www.johansontechnology.com/datasheets/2450AT18B100.pdf) |
| USBLC6-2SC6Y | [JLCPCB](https://jlcpcb.com/partdetail/STMicroelectronics-USBLC6_2SC6Y/C165609) | [ST USBLC6-2 datasheet](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) |
| KH-TYPE-C-16P | [JLCPCB](https://jlcpcb.com/partdetail/Kinghelm-KH_TYPE_C_16P/C2954157) | [KH-TYPE-C-16P datasheet](https://datasheet.lcsc.com/lcsc/2112101730_Kinghelm-KH-TYPE-C-16P_C2954157.pdf) |
| 5034802400 | [JLCPCB](https://jlcpcb.com/partdetail/Molex-5034802400/C2817812) | [Molex 503480 series chart](https://www.molex.com/p/molex/en-us/products/product-data-sheet?partNumber=5034802400) |

## 3. Descrierea funcționalității hardware

Proiectul a fost gândit să fie un dispozitiv portabil eficient energetic. 
* **Alimentarea** se face printr-un port USB-C (cu protecție ESD USBLC6), care intră în circuitul BQ25180 pentru a încărca bateria LiPo de 3.7V.
* **Monitorizarea** bateriei se realizează precis prin cipul MAX17048 (Fuel Gauge).
* **Tensiunea logică** a sistemului este coborâtă/urcată la 3.3V folosind regulatorul Buck-Boost RT6160, asigurând stabilitate indiferent de nivelul bateriei.
* **Procesarea** centrală este realizată de nRF52840, un SoC foarte capabil pe parte de BLE.
* **Perifericele** includ un modul IMU (BMA423) pentru gesturi și monitorizarea mișcării, un ecran E-Paper conectat prin SPI, 3 butoane tactile pentru navigare și un motor LRA controlat de driverul DRV2605 pentru feedback haptic. Toate cipurile inteligente comunică pe o magistrală comună I2C cu microcontrolerul.

*Nota de consum:* Cu bateria estimată la ~250-300mAh, folosind funcțiile de sleep profund ale nRF52840 (consum în repaus de aprox. 1.5µA) și ecranul E-Paper care consumă energie doar la refresh, autonomia teoretică depășește o săptămână în utilizare moderată.

## 4. Alocarea Pinilor (Pinout nRF52840)

Toate componentele principale au fost rutate către MCU respectând capabilitățile perifericelor hardware:

* **Magistrala I2C (SCL / SDA):** Conectată la BQ25180 (Charger), MAX17048 (Fuel Gauge), RT6160 (Regulator), BMA423 (IMU) și DRV2605 (Haptic).
* **Magistrala SPI:** Conectată la display-ul E-Paper.
* **Semnale RF:** Ieșirea de 2.4GHz conectată via circuit de adaptare la antena ceramică Johanson.
* **Interfața de Debug (SWD):** Pinii SWDIO și SWDCLK sunt scoși pe portul de programare (TC2030) pentru flash-uire și debug facil.
* **Butoane (GPIO):** 3 pini de intrare cu rezistențe de pull-up interne activate.

