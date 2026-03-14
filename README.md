# cheap-diy-camera

# Cámara DIY — XIAO ESP32-S3 + GC9A01

Una cámara digital minimalista construida desde cero, inspirada en la estética de cámaras de juguete como la Kodak Charmer y la Pixless. Basada en el módulo XIAO ESP32-S3 Sense con display circular GC9A01 de 1.28" y batería 18650 recargable.

---

## Concepto

El proyecto nace de la idea de construir una cámara que sea a la vez funcional y un objeto de diseño — pequeña, con display redondo, alimentada por batería 18650 recargable con carga via USB-C, y fabricable como PCB propia. El firmware corre directamente sobre el ESP32-S3 con su módulo de cámara integrado (OV3660), transmitiendo el feed en tiempo real al display circular.

---

## Hardware

### Componentes principales

| Componente | Descripción | Notas |
|------------|-------------|-------|
| Seeed XIAO ESP32-S3 Sense | Microcontrolador + cámara OV3660 | Con PSRAM habilitado (OPI PSRAM) |
| GC9A01 1.28" | Display circular TFT 240×240 | Comunicación SPI |
| Batería 18650 Li-Ion | 3.7V, 3000mAh | Portapilas externo con cables a J3 |
| TP4056-42-ESOP8 | Cargador LiIon | 500mA, carga via USB-C, SOP-8 |
| Diodo Schottky SS14 | Protección polaridad inversa | SMA, entre BAT_18650 y pin 5V XIAO |
| Botón shutter | 6mm tactile switch THT | GPIO1 (D0) |
| Botón modo | 6mm tactile switch THT | GPIO2 (D1) |
| LED rojo 0402 | Indicador cargando | Activo bajo — pin CHRG TP4056 |
| LED verde 0402 | Indicador carga completa | Activo bajo — pin STDBY TP4056 |
| Capacitores 100nF ×4 | Desacople alimentación XIAO | SMD 0402 |
| Capacitor 100nF | Desacople VCC TP4056 (C5) | SMD 0402 |
| Capacitor 10µF ×2 | Bulk cap VCC y BAT TP4056 (C6, C7) | SMD 0805 |
| R1, R2 — 1kΩ ×2 | Serie con LEDs indicadores | SMD 0402 |
| R3 — 2kΩ | PROG TP4056 — fija corriente 500mA | SMD 0402 |
| Conector 1x07 2.54mm (J1) | Display GC9A01 | Pin header vertical |
| Conector 1x02 2mm (J3) | Portapilas 18650 | JST-PH o cables directos |

---

## Conexiones

### Display GC9A01 → XIAO ESP32-S3

| Pin display | Señal | Pin XIAO | GPIO |
|-------------|-------|----------|------|
| 1 | VCC | 3V3 | — |
| 2 | GND | GND | — |
| 3 | SCL (CLK) | D8 | GPIO7 |
| 4 | SDA (MOSI) | D10 | GPIO9 |
| 5 | RES (Reset) | D7 | GPIO44 |
| 6 | DC | D6 (TX) | GPIO43 |
| 7 | CS | D5 | GPIO6 |

### Botones

| Componente | Pin 1 | Pin 2 |
|------------|-------|-------|
| SW1 (shutter) | GPIO1 (D0) | GND |
| SW2 (modo) | GPIO2 (D1) | GND |

### Circuito de carga TP4056 + 18650

```
USB-C (5V)
    │
    ▼
┌─────────────────────────────────────────────────────┐
│                   TP4056 (U2)                       │
│                                                     │
│  VCC (pin4) ← +5V     BAT (pin5) ──── BAT_18650     │
│  CE  (pin8) ← +5V     TEMP(pin1) ──── GND           │
│  GND (pin3) ← GND     PROG(pin2) ──── R3(2kΩ)─GND   │
│  CHRG(pin7) ──R1(1kΩ)──LED rojo──GND  (cargando)    │
│  STDBY(pin6)──R2(1kΩ)──LED verde─GND  (carga OK)    │
└─────────────────────────────────────────────────────┘
         │
    BAT_18650
         │
    [Portapilas 18650 — J3]
         │
    BAT_18650 ──► D3 (Schottky SS14) ──► +5V ──► XIAO pin 14
```

| Pin TP4056 | Conexión |
|------------|----------|
| 1 TEMP | GND (sin sensor NTC) |
| 2 PROG | R3 (2kΩ) → GND |
| 3 GND | GND |
| 4 VCC | +5V + C5(100nF) + C6(10µF) a GND |
| 5 BAT | BAT_18650 + C7(10µF) a GND |
| 6 STDBY | R2(1kΩ) → LED verde → GND |
| 7 CHRG | R1(1kΩ) → LED rojo → GND |
| 8 CE | +5V (siempre habilitado) |

**Nota:** La batería 18650 alimenta el XIAO por el pin 5V (no BAT) a través del diodo Schottky D3. El XIAO regula internamente 5V→3.3V. Corriente de carga: 500mA (R3=2kΩ).

### GPIOs disponibles

| GPIO | Pin XIAO | Uso |
|------|----------|-----|
| GPIO1 | D0 | BTN_SHUTTER |
| GPIO2 | D1 | BTN_MODO |
| GPIO3 | D2 | libre |
| GPIO4 | D3 | libre |
| GPIO5 | D4/SDA | libre |
| GPIO6 | D5/SCL | DISP_CS |
| GPIO7 | D8/SCK | SPI_CLK |
| GPIO8 | D9/MISO | libre (NC) |
| GPIO9 | D10/MOSI | SPI_MOSI |
| GPIO43 | D6/TX | DISP_DC |
| GPIO44 | D7/RX | DISP_RES |

---

## PCB

Diseñada en KiCad 9. Placa de 2 capas, ~50×55mm.

- **F.Cu** — alimentación (+5V, +3V3) y señales de botones (0.5–0.8mm)
- **B.Cu** — señales SPI del display + plano GND (copper pour)
- Capacitores de desacople orientados 90°, apilados junto al XIAO
- Cluster TP4056 agrupado en esquina inferior derecha
- Portapilas 18650 externo conectado via J3 con cables

### Anchos de pista

| Red | Ancho | Capa |
|-----|-------|------|
| GND | copper pour | B.Cu |
| +5V / BAT_18650 | 0.8mm | F.Cu |
| +3V3 | 0.5mm | F.Cu |
| SPI signals | 0.25mm | B.Cu |
| Botones | 0.25mm | F.Cu |

### Archivos del proyecto

```
camara-diy/
├── camara-diy.kicad_pro      # Proyecto KiCad
├── camara-diy.kicad_sch      # Esquemático
├── camara-diy.kicad_pcb      # Layout PCB
├── libs/
│   └── Seeed_Studio_XIAO_Series/   # Librería footprint XIAO
│   └── MODULE_113991054.kicad_mod  # Footprint XIAO SMD
└── gerbers/                  # Archivos para fabricación (JLCPCB / PCBWay)
```

---

## Firmware

### Dependencias (Arduino IDE / PlatformIO)

- `TFT_eSPI` — driver para GC9A01
- `esp32-camera` — módulo de cámara ESP32
- PSRAM habilitado en la configuración de placa

### Configuración TFT_eSPI (`User_Setup.h`)

```cpp
#define GC9A01_DRIVER
#define TFT_MOSI  9   // GPIO9
#define TFT_SCLK  7   // GPIO7
#define TFT_CS    6   // GPIO6
#define TFT_DC    43  // GPIO43
#define TFT_RST   44  // GPIO44
#define TFT_WIDTH  240
#define TFT_HEIGHT 240
```

### Resolución del problema PSRAM (Arduino IDE)

En Arduino IDE, seleccionar la placa `XIAO_ESP32S3` y en `Tools → PSRAM` elegir **OPI PSRAM**. Sin esto la cámara no inicializa correctamente.

---

## Ensamblaje

1. Soldar componentes SMD: TP4056, diodo D3, LEDs D1/D2, resistencias R1-R3, capacitores C1-C7
2. Soldar el pin header J1 del display (7 pines, 2.54mm)
3. Soldar el conector J3 para la batería (o cables directos al portapilas)
4. Soldar los botones SW1 y SW2
5. Colocar el XIAO ESP32-S3 Sense sobre los pads (soldar los 14 pines visibles)
6. Conectar el display GC9A01 al conector J1
7. Conectar el portapilas 18650 a J3
8. Programar via USB-C — el LED rojo indica que la batería está cargando

---

## BOM (Bill of Materials)

| Ref | Componente | Valor | Footprint | Cantidad |
|-----|-----------|-------|-----------|----------|
| U1 | XIAO ESP32-S3 Sense | — | MODULE_113991054 | 1 |
| U2 | TP4056-42-ESOP8 | — | SOIC-8-1EP | 1 |
| J1 | Pin header 1×07 | 2.54mm | PinHeader_1x07_P2.54mm | 1 |
| J3 | Conector batería | 1×02 | JST-PH 2mm o cables | 1 |
| D1 | LED rojo | — | 0402 | 1 |
| D2 | LED verde | — | 0402 | 1 |
| D3 | Diodo Schottky SS14 | — | SMA | 1 |
| SW1 | Botón shutter | SW_Push | SW_PUSH_6mm | 1 |
| SW2 | Botón modo | SW_Push | SW_PUSH_6mm | 1 |
| R1, R2 | Resistencia | 1kΩ | 0402 | 2 |
| R3 | Resistencia | 2kΩ | 0402 | 1 |
| C1–C4 | Capacitor desacople XIAO | 100nF | 0402 | 4 |
| C5 | Capacitor desacople VCC | 100nF | 0402 | 1 |
| C6, C7 | Capacitor bulk TP4056 | 10µF | 0805 | 2 |

---

## Inspiración y referencias

- [Kodak Charmer](https://www.kodak.com) — estética de cámara de juguete
- [Pixless Camera](https://pixless.com) — cámara minimalista DIY
- [Seeed XIAO ESP32-S3 Sense Wiki](https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/) — documentación oficial
- [GC9A01 datasheet](https://www.buydisplay.com/download/ic/GC9A01A.pdf)
- [TP4056 datasheet](https://dlnmh9ip6v2uc.cloudfront.net/datasheets/Prototyping/TP4056.pdf)

---

## Licencia

MIT — libre para usar, modificar y fabricar.

---

*Proyecto desarrollado en Santiago, Chile.*