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

## 4. Alocarea Pinilor (Pinout nRF52840) și Justificare

Microcontrolerul Nordic nRF52840 permite maparea flexibilă a interfețelor (I2C, SPI) pe majoritatea pinilor GPIO. Alocarea de mai jos a fost aleasă strategic pentru a optimiza rutarea pe PCB, minimizând încrucișările de trasee (vias) și asigurând integritatea semnalului.

### Tabelul alocărilor hardware:

| Componentă | Nume Pin nRF | Tip Semnal | Explicație / Rol în sistem |
| :--- | :--- | :--- | :--- |
| **Magistrala I2C (Comună)** | **P0.06**<br>**P0.07** | `SDA`<br>`SCL` | Comunicația I2C principală a plăcii. Leagă senzorii și cipurile de power (IMU, Charger, Fuel Gauge, Haptic). S-au folosit pinii 0.06 și 0.07 datorită proximității față de zona de power a plăcii. |
| **E-Paper Display** | **P1.15**<br>**P1.14**<br>**P0.05** | `SCK`<br>`MOSI`<br>`CS` | Magistrala SPI (fără MISO, deoarece ecranul doar primește date, nu transmite). Gruparea pinilor facilitează rutarea unui bus curat către conectorul FPC. `CS` (Chip Select) activează ecranul. |
| **E-Paper Control** | **P0.15**<br>**P0.16**<br>**P0.17** | `EPD_DC`<br>`EPD_RST`<br>`EPD_BUSY` | Pini dedicați controlului E-ink: `DC` (Data/Command toggle), `RST` (Hardware Reset) și `BUSY` (intrare pentru a citi când ecranul a terminat refresh-ul, economisind energie). |
| **Accelerometru (BMA423)** | **P0.08**<br>**P1.08** | `IMU_INT1`<br>`IMU_INT2` | Pini de Interrupt. Extrem de importanți pentru consumul redus: MCU-ul stă în *Deep Sleep* și este trezit hardware doar când BMA423 detectează o mișcare (ex: ridicarea mâinii). |
| **Power Mgmt (BQ / MAX)** | **P0.11**<br>**P0.09** | `PMIC_INT`<br>`ALERT` | Pini de Interrupt pentru energie. Trezesc MCU-ul când se conectează cablul USB, se termină încărcarea sau bateria scade sub un prag critic. |
| **Haptic Driver (DRV2605)**| **P0.12** | `HAPTIC_EN` | Pin de Enable. Pornește sau oprește alimentarea driverului haptic hardware, prevenind scurgerile de curent când motorul nu vibrează. |
| **Port USB-C** | **D-** / **D+** | `USB_D-`<br>`USB_D+` | nRF52840 are suport hardware USB nativ, așadar acești pini duc direct datele către conectorul USB-C. |
| **Interfață Debug (TC2030)**| **SWDIO**<br>**SWDCLK**<br>**P1.00** | `SWDIO`<br>`SWDCLK`<br>`SWO` | Interfața standard Serial Wire Debug. Necesară pentru flash-uirea bootloader-ului, a firmware-ului și pentru live-debugging. |
| **Oscilatoare (Cristale)** | **P0.00 / 0.01**<br>**XC1 / XC2** | `XL1 / XL2`<br>`XC1 / XC2` | Conectate la cristalele de 32.768kHz (pentru RTC / timp real) și 32MHz (pentru radio Bluetooth). |
