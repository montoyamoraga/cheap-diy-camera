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
| GC9A01 1.28" | Display circular TFT 240×240 | Comunicación SPI, conector 8 pines |
| Batería 18650 Li-Ion | 3.7V, 3000mAh | Portapilas externo con cables a J3 |
| TP4056-42-ESOP8 | Cargador LiIon | 500mA, carga via USB-C, SOP-8 |
| GCT USB4085 (J2) | Conector USB-C SMD | 14 pines, montado en PCB |
| Diodo Schottky SS14 | Protección polaridad inversa | SMA, entre BAT_18650 y pin 5V XIAO |
| Botón shutter | 6mm tactile switch THT | GPIO1 (D0) |
| Botón modo | 6mm tactile switch THT | GPIO2 (D1) |
| LED rojo 0402 | Indicador cargando | Activo bajo — pin CHRG TP4056 |
| LED verde 0402 | Indicador carga completa | Activo bajo — pin STDBY TP4056 |
| Capacitores 100nF ×5 | Desacople alimentación (C1–C5) | SMD 0402 |
| Capacitor 10µF ×2 | Bulk cap VCC y BAT TP4056 (C6, C7) | SMD 0805 |
| R1, R2 — 1kΩ ×2 | Serie con LEDs indicadores | SMD 0402 |
| R3 — 2kΩ | PROG TP4056 — fija corriente 500mA | SMD 0402 |
| R4, R5 — 5.1kΩ ×2 | Pull-down CC1/CC2 USB-C | SMD 0402 |
| Conector 1x08 2.54mm (J1) | Display GC9A01 | Pin header vertical, 8 pines |
| Conector JST-PH 1x02 2mm (J3) | Portapilas 18650 | JST-PH o cables directos |

---

## Conexiones

### Display GC9A01 → XIAO ESP32-S3

| Pin display | Señal | Pin XIAO | GPIO | Notas |
|-------------|-------|----------|------|-------|
| 1 | VCC | 3V3 | — | Alimentación |
| 2 | GND | GND | — | Masa |
| 3 | SCL (CLK) | D8 | GPIO7 | SPI clock |
| 4 | SDA (MOSI) | D10 | GPIO9 | SPI data |
| 5 | RES (Reset) | D7 | GPIO44 | Reset activo bajo |
| 6 | DC | D6 (TX) | GPIO43 | Data/Command |
| 7 | CS | D5 | GPIO6 | Chip select activo bajo |
| 8 | BLK | +3V3 | — | Backlight siempre encendido |

### Botones

| Componente | Pin 1 | Pin 2 |
|------------|-------|-------|
| SW1 (shutter) | GPIO1 (D0) | GND |
| SW2 (modo) | GPIO2 (D1) | GND |

### Conector USB-C (J2) — GCT USB4085

El conector USB-C está montado directamente en la PCB. Solo se usa para alimentación (5V), no para datos.

| Pin J2 | Señal | Conexión |
|--------|-------|----------|
| A4, A9, B4, B9 | VBUS | VBUS_CHG → VCC TP4056 + CE TP4056 |
| A1, A12, B1, B12 | GND | GND |
| A5 | CC1 | R4 (5.1kΩ) → GND |
| B5 | CC2 | R5 (5.1kΩ) → GND |
| A6, A7, B6, B7 | D+, D− | NC (no conectado) |

**Nota:** Las resistencias R4/R5 de 5.1kΩ en CC1/CC2 identifican el dispositivo como sink USB-C de 5V/500mA (modo "default power"), permitiendo el uso con cualquier fuente USB-C estándar.

### Circuito de carga TP4056 + 18650

```
USB-C (J2 — GCT USB4085)
    │ VBUS_CHG (5V)
    ▼
┌─────────────────────────────────────────────────────┐
│                   TP4056 (U2)                       │
│                                                     │
│  VCC (pin4) ← VBUS_CHG   BAT (pin5) ── BAT_18650   │
│  CE  (pin8) ← VBUS_CHG   TEMP(pin1) ── GND         │
│  GND (pin3) ← GND        PROG(pin2) ── R3(2kΩ)─GND │
│  CHRG(pin7) ──R1(1kΩ)──LED rojo──GND  (cargando)   │
│  STDBY(pin6)──R2(1kΩ)──LED verde─GND  (carga OK)   │
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
| 4 VCC | VBUS_CHG + C5(100nF) + C6(10µF) a GND |
| 5 BAT | BAT_18650 + C7(10µF) a GND |
| 6 STDBY | R2(1kΩ) → LED verde → GND |
| 7 CHRG | R1(1kΩ) → LED rojo → GND |
| 8 CE | VBUS_CHG (siempre habilitado con USB conectado) |

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
- Conector USB-C (J2) montado en PCB — GCT USB4085 SMD
- Capacitores de desacople orientados 90°, apilados junto al XIAO
- Cluster TP4056 agrupado en esquina inferior derecha
- Portapilas 18650 externo conectado via J3 con cables

### Anchos de pista

| Red | Ancho | Capa |
|-----|-------|------|
| GND | copper pour | B.Cu |
| +5V / BAT_18650 / VBUS_CHG | 0.8mm | F.Cu |
| +3V3 | 0.5mm | F.Cu |
| SPI signals | 0.25mm | B.Cu |
| Botones | 0.25mm | F.Cu |

### Archivos del proyecto

```
kicad/V2/
├── camara-diy.kicad_pro      # Proyecto KiCad
├── camara-diy.kicad_sch      # Esquemático
├── camara-diy.kicad_pcb      # Layout PCB
├── camara-diy.csv            # BOM exportado
└── gerbers/                  # Archivos para fabricación (JLCPCB / PCBWay)
```

Librerías externas requeridas:
- `Seeed Studio XIAO Series Library/` — footprint XIAO ESP32-S3
- `GC9A01/` — footprint y símbolo display GC9A01 8P

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

1. Soldar componentes SMD del conector USB-C: J2 (GCT USB4085), R4 y R5 (5.1kΩ CC resistors)
2. Soldar componentes SMD del cargador: TP4056 (U2), diodo D3, LEDs D1/D2, resistencias R1–R3, capacitores C1–C7
3. Soldar el pin header J1 del display (8 pines, 2.54mm) — marcar pin 1 (cuadrado en silkscreen)
4. Soldar el conector J3 para la batería (o cables directos al portapilas)
5. Soldar los botones SW1 y SW2
6. Colocar el XIAO ESP32-S3 Sense sobre los pads (soldar los 14 pines visibles)
7. Conectar el display GC9A01 al conector J1 respetando orientación (pin 1 = VCC)
8. Conectar el portapilas 18650 a J3
9. Programar via USB-C — el LED rojo indica que la batería está cargando

---

## BOM (Bill of Materials)

| Ref | Componente | Valor | Footprint | Cantidad |
|-----|-----------|-------|-----------|----------|
| U1 | XIAO ESP32-S3 Sense | — | XIAO-ESP32-S3-SMD | 1 |
| U2 | TP4056-42-ESOP8 | — | SOIC-8-1EP | 1 |
| J1 | Pin header 1×08 | 2.54mm | PinHeader_1x08_P2.54mm_Vertical | 1 |
| J2 | USB-C receptacle | GCT USB4085 | USB_C_Receptacle_GCT_USB4085 | 1 |
| J3 | Conector batería | 1×02 JST-PH | JST_PH_B2B-PH-K_1x02_P2.00mm | 1 |
| D1 | LED rojo | — | 0402 | 1 |
| D2 | LED verde | — | 0402 | 1 |
| D3 | Diodo Schottky SS14 | — | SMA | 1 |
| SW1 | Botón shutter | SW_Push | SW_PUSH_6mm_H5mm | 1 |
| SW2 | Botón modo | SW_Push | SW_PUSH_6mm_H5mm | 1 |
| R1, R2 | Resistencia | 1kΩ | 0402 | 2 |
| R3 | Resistencia PROG | 2kΩ | 0402 | 1 |
| R4, R5 | Resistencia CC USB-C | 5.1kΩ | 0402 | 2 |
| C1–C5 | Capacitor desacople | 100nF | 0402 | 5 |
| C6, C7 | Capacitor bulk TP4056 | 10µF | 0805 | 2 |

---

## Inspiración y referencias

- [Kodak Charmer](https://www.kodak.com) — estética de cámara de juguete
- [Pixless Camera](https://pixless.com) — cámara minimalista DIY
- [Seeed XIAO ESP32-S3 Sense Wiki](https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/) — documentación oficial
- [GC9A01 datasheet](https://www.buydisplay.com/download/ic/GC9A01A.pdf)
- [TP4056 datasheet](https://dlnmh9ip6v2uc.cloudfront.net/datasheets/Prototyping/TP4056.pdf)
- [GCT USB4085 datasheet](https://gct.co/connector/usb4085) — conector USB-C SMD

---

## Licencia

MIT — libre para usar, modificar y fabricar.

---

*Proyecto desarrollado en Santiago, Chile.*
