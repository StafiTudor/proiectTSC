# proiectTSC
# InkTime Smart Watch

Acest proiect reprezintă designul hardware complet pentru un smartwatch bazat pe ecosistemul nRF52840, incluzând management avansat al bateriei, afișaj E-paper și feedback haptic.

## 1. Diagrama Bloc a Sistemului

Arhitectura proiectului este împărțită în 3 etaje principale: Input & Management Energie (Power), Unitatea Centrală de Procesare (Core MCU) și Interfețe Externe (Periferice și RF).

```mermaid
flowchart LR
    %% Setare grupuri vizuale (Subgrafe)
    
    subgraph Sistem_Alimentare [1. Management Energie]
        direction TB
        USB([Port USB-C KH-16P]) -->|5V| BQ[LiPo Charger BQ25180]
        BQ -->|Încărcare| BAT[(Baterie LiPo 3.7V)]
        BAT -->|Tensiune| MAX[Fuel Gauge MAX17048]
        BAT -->|Tensiune| RT[Convertor DC/DC RT6160]
    end

    subgraph Procesare_Centrala [2. Core Processing]
        MCU{{SoC nRF52840 + BLE}}
        SWD([Interfață Debug TC2030]) <-->|SWD| MCU
    end

    subgraph Periferice [3. Senzori & Interfețe]
        direction TB
        BTN((Butoane Tactile x3))
        IMU[/Accelerometru BMA423/]
        EPD[/Ecran E-Paper/]
        HAPTIC[Driver Haptic DRV2605] -->|Semnal Analog| MOT((Motor Vibrații LRA))
        ANT>Antenă Ceramică Johanson]
    end

    %% Legături de Alimentare (Power)
    RT -->|Alimentare 3.3V| MCU

    %% Legături Magistrala I2C (Catre Alimentare)
    MCU <-->|Magistrală I2C| BQ
    MCU <-->|Magistrală I2C| MAX
    MCU <-->|Magistrală I2C| RT

    %% Legături Magistrala I2C (Catre Periferice)
    MCU <-->|Magistrală I2C| IMU
    MCU <-->|Magistrală I2C| HAPTIC

    %% Legături Specifice
    MCU -->|Interfață SPI| EPD
    MCU ---|Semnal RF 2.4GHz| ANT
    BTN -->|Pini GPIO| MCU
```

## 2. Componente principale și linkuri utile (BOM)
| Componenta | Link JLCPCB | DATASHEET |
| :--- | :--- | :--- |
| nRF52840 | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=nRF52840) | [Nordic Product Spec](https://docs.nordicsemi.com/bundle/ps_nrf52840/page/keyfeatures_html5.html) |
| BQ25180YBGR | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=BQ25180YBGR) | [TI BQ25180 datasheet](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| RT6160AWSC | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=RT6160A) | [Richtek RT6160A datasheet](https://www.richtek.com/assets/product_datasheet/RT6160/DS6160-01.pdf) |
| MAX17048G+T10 | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=MAX17048) | [Analog/Maxim datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX17048-MAX17049.pdf) |
| BMA423 | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=BMA423) | [BMA423 datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma423-ds004.pdf) |
| DRV2605YZFR | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=DRV2605) | [TI DRV2605 datasheet](https://www.ti.com/lit/ds/symlink/drv2605.pdf) |
| 2450AT18B100E | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=2450AT18B100E) | [Johanson antenna datasheet](https://www.johansontechnology.com/datasheets/2450AT18B100.pdf) |
| USBLC6-2SC6Y | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=USBLC6-2SC6Y) | [ST USBLC6-2 datasheet](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) |
| KH-TYPE-C-16P | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=KH-TYPE-C-16P) | [KH-TYPE-C-16P datasheet](https://datasheet.lcsc.com/lcsc/2112101730_Kinghelm-KH-TYPE-C-16P_C2954157.pdf) |
| 5034802400 | [JLCPCB Search](https://jlcpcb.com/parts/componentSearch?searchTxt=5034802400) | [Molex 503480 datasheet](https://www.molex.com/p/molex/en-us/products/product-data-sheet?partNumber=5034802400) |

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

