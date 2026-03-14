# cheap-diy-camera

# Cámara DIY — XIAO ESP32-S3 + GC9A01

Una cámara digital minimalista construida desde cero, inspirada en la estética de cámaras de juguete como la Kodak Charmer y la Pixless. Basada en el módulo XIAO ESP32-S3 Sense con display circular GC9A01 de 1.28".

---

## Concepto

El proyecto nace de la idea de construir una cámara que sea a la vez funcional y un objeto de diseño — pequeña, con display redondo, alimentada por batería LiPo, y fabricable como PCB propia. El firmware corre directamente sobre el ESP32-S3 con su módulo de cámara integrado (OV2640), transmitiendo el feed en tiempo real al display circular.

---

## Hardware

### Componentes principales

| Componente | Descripción | Notas |
|------------|-------------|-------|
| Seeed XIAO ESP32-S3 Sense | Microcontrolador + cámara OV2640 | Con PSRAM habilitado |
| GC9A01 1.28" | Display circular TFT 240×240 | Comunicación SPI |
| Batería LiPo | 3.7V, 500mAh recomendado | Conector JST-PH 2mm |
| Botón shutter | 6mm tactile switch | GPIO1 |
| Botón modo | 6mm tactile switch | GPIO2 |
| Capacitores 100nF (×4) | Desacople de alimentación | SMD 0402 |
| Conector JST-PH 1x02 | Batería | 2mm pitch |
| Conector 1x07 2.54mm | Display | Pin header vertical |

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

### Batería

| Pin JST | Conexión |
|---------|----------|
| 1 (+) | BAT (pin 15 XIAO) |
| 2 (−) | GND |

---

## Esquemático simplificado

```
                    ┌───────────────────────────────┐
                    │      XIAO ESP32-S3 Sense      │
                    │                               │
  SW1 ──── GPIO1    │  D0 ──────────────── 3V3 ───► │──── VCC (Display)
  SW2 ──── GPIO2    │  D1                  GND      │
                    │                               │
  Display CS ─────  │  D5 (GPIO6)                   │
  Display DC ─────  │  D6 (GPIO43)    ┌── BAT ──────│──── JST+ (LiPo)
  Display RES ────  │  D7 (GPIO44)    │   GND ──────│──── JST−
  Display MOSI ───  │  D10 (GPIO9)    │             │
  Display CLK ────  │  D8 (GPIO7)     │             │
                    │                 │             │
                    │  5V ── USB      │             │
                    └─────────────────┴─────────────┘
                                      │
                          ┌───────────┘
                          │   LiPo 3.7V
                          └──────────

  Caps desacople: 4× 100nF entre 3V3 y GND
```

---

## PCB

Diseñada en KiCad 9. Placa de 2 capas, 60×40mm.

- **F.Cu** — alimentación (GND, +3V3) y señales de botones
- **B.Cu** — señales SPI del display
- Capacitores de desacople orientados 90°, apilados junto al XIAO
- Conector de display como pin header 2.54mm (modular, permite cambiar display)
- Conector JST-PH para batería LiPo

### Archivos del proyecto

```
camara-diy/
├── camara-diy.kicad_pro      # Proyecto KiCad
├── camara-diy.kicad_sch      # Esquemático
├── camara-diy.kicad_pcb      # Layout PCB
├── libs/
│   └── Seeed_Studio_XIAO_Series/   # Librería footprint XIAO
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

1. Soldar el conector JST-PH para la batería
2. Soldar el pin header del display (7 pines, 2.54mm)
3. Colocar el XIAO ESP32-S3 Sense sobre los pads (soldar los 14 pines visibles)
4. Conectar el display GC9A01 al conector
5. Conectar la batería LiPo
6. Programar via USB-C

---

## Inspiración y referencias

- [Kodak Charmer](https://www.kodak.com) — estética de cámara de juguete
- [Pixless Camera](https://pixless.com) — cámara minimalista DIY
- [Seeed XIAO ESP32-S3 Sense Wiki](https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/) — documentación oficial
- [GC9A01 datasheet](https://www.buydisplay.com/download/ic/GC9A01A.pdf)

---

## Licencia

MIT — libre para usar, modificar y fabricar.

---

*Proyecto desarrollado en Santiago, Chile.*