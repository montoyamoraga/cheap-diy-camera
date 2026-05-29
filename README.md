# DIY Pixel Camera ESP32

Cámara digital experimental hecha con una **Seeed Studio XIAO ESP32S3 Sense**, una pantalla redonda **GC9A01 de 240x240 px**, dos botones físicos y un script en Python para guardar las fotos en un computador Mac.

El proyecto muestra en tiempo real la imagen de la cámara en una pantalla circular, aplica un efecto pixelado con paletas cromáticas intercambiables y permite capturar una imagen para enviarla por USB/Serial al computador, donde se guarda como archivo de imagen.

## Características

* Vista en vivo desde cámara ESP32.
* Pantalla redonda GC9A01 de 240x240 px.
* Efecto pixelado en tiempo real.
* Paletas cromáticas intercambiables.
* Captura de foto mediante botón físico.
* Envío de imagen al Mac por USB/Serial.
* Guardado automático de imagen con Python.
* Rotación de pantalla mediante presión larga.
* Diseño simple para prototipado en breadboard.

## Hardware utilizado

| Componente                      | Cantidad | Descripción                                   |
| ------------------------------- | -------: | --------------------------------------------- |
| Seeed Studio XIAO ESP32S3 Sense |        1 | Microcontrolador ESP32S3 con módulo de cámara |
| Pantalla GC9A01 240x240 SPI     |        1 | Pantalla circular de 8 pines                  |
| Botón pulsador                  |        2 | Para captura, rotación y cambio de paleta     |
| Breadboard                      |        1 | Para prototipado                              |
| Cables jumper                   |   varios | Conexiones macho-macho o macho-hembra         |
| Cable USB-C                     |        1 | Programación y comunicación Serial con el Mac |
| Computador Mac                  |        1 | Recibe y guarda las fotos                     |

## Conexiones

### Pantalla GC9A01

| Pin GC9A01       | Pin XIAO ESP32S3 | Función                     |
| ---------------- | ---------------- | --------------------------- |
| VCC              | 3V3              | Alimentación                |
| GND              | GND              | Tierra                      |
| SCL / SCK / CLK  | D8 / GPIO7       | Reloj SPI                   |
| SDA / MOSI / DIN | D10 / GPIO9      | Datos SPI                   |
| RES / RST        | D2 / GPIO3       | Reset pantalla              |
| DC               | D3 / GPIO4       | Data / Command              |
| CS               | D7 / GPIO44      | Chip Select                 |
| BL / LED         | 3V3              | Backlight siempre encendido |

> Nota: se recomienda alimentar la pantalla con **3.3V** para mantener compatibilidad lógica con la ESP32.

### Botones

Ambos botones se conectan entre un pin digital y GND. En el código se usa `INPUT_PULLUP`, por lo que no es necesario agregar resistencias externas.

| Botón   | Pin XIAO ESP32S3 | Función                                                |
| ------- | ---------------- | ------------------------------------------------------ |
| Botón 1 | D0 / GPIO1       | Click: sacar foto. Mantener 3 segundos: rotar pantalla |
| Botón 2 | D1 / GPIO2       | Click: cambiar paleta cromática                        |

Conexión de cada botón:

```txt
GPIO ---- botón ---- GND
```

Con `INPUT_PULLUP`:

```txt
Suelto     = HIGH
Presionado = LOW
```

## Funciones de los botones

| Acción                          | Resultado                          |
| ------------------------------- | ---------------------------------- |
| Click corto en botón 1          | Captura una foto y la envía al Mac |
| Mantener botón 1 por 3 segundos | Rota la pantalla                   |
| Click corto en botón 2          | Cambia la paleta cromática         |

## Paletas incluidas

El proyecto incluye varias paletas cromáticas aplicadas sobre la luminancia de la imagen:

1. Cyber / rosado azul
2. Verde vintage
3. Cálida naranja
4. Azul / crema
5. Blanco y negro
6. Riso rosado / amarillo / azul
7. Vaporwave
8. Fuego
9. Bosque
10. Pastel extraño

## Software necesario

### Arduino IDE

Instalar el soporte para placas ESP32 en Arduino IDE.

En:

```txt
Arduino IDE > Settings / Preferences > Additional Boards Manager URLs
```

Agregar:

```txt
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

Luego ir a:

```txt
Tools > Board > Boards Manager
```

Buscar e instalar:

```txt
esp32 by Espressif Systems
```

Seleccionar la placa:

```txt
XIAO_ESP32S3
```

o similar:

```txt
Seeed XIAO ESP32S3
```

### Librería Arduino

Instalar desde:

```txt
Tools > Manage Libraries
```

Buscar e instalar:

```txt
Arduino_GFX Library
```

La cámara usa `esp_camera.h`, incluida en el soporte ESP32.

### Python en Mac

Instalar dependencias:

```bash
pip3 install pyserial pillow
```

## Código Arduino

El archivo principal del proyecto debe estar en una carpeta de Arduino, por ejemplo:

```txt
diy-camera/diy-camera.ino
```

El código controla:

* Cámara ESP32S3.
* Pantalla GC9A01.
* Efecto pixelado.
* Paletas cromáticas.
* Botón de captura.
* Botón de paleta.
* Envío Serial de imagen RGB al Mac.

Parámetros principales modificables:

```cpp
#define PIXEL_SIZE 4
```

Mientras menor sea el valor, más detalle tendrá la imagen, pero también será más lenta.

Ejemplos:

```cpp
#define PIXEL_SIZE 4   // más detalle
#define PIXEL_SIZE 8   // más pixelado y rápido
#define PIXEL_SIZE 12  // muy pixelado y más liviano
```

La comunicación Serial usa:

```cpp
Serial.begin(921600);
```

## Script Python para recibir fotos

Crear un archivo en el escritorio llamado:

```txt
recibir_foto_esp32.py
```

Este script escucha el puerto Serial, recibe la imagen enviada por la ESP32 y la guarda en una carpeta del escritorio.

La carpeta de salida es:

```txt
~/Desktop/fotos_esp32/
```

Para encontrar el puerto de la ESP32 en Mac:

```bash
ls /dev/cu.*
```

Ejemplo de puerto:

```txt
/dev/cu.usbmodem1101
```

En el script, ajustar la línea:

```python
PORT = "/dev/cu.usbmodem1101"
```

Ejecutar el receptor:

```bash
python3 ~/Desktop/recibir_foto_esp32.py
```

> Importante: cerrar el Serial Monitor de Arduino IDE antes de ejecutar el script, porque el puerto Serial solo puede ser usado por una aplicación a la vez.

## Flujo de uso

1. Conectar la XIAO ESP32S3 Sense al Mac por USB-C.
2. Subir el código desde Arduino IDE.
3. Cerrar el Serial Monitor.
4. Ejecutar el script Python:

```bash
python3 ~/Desktop/recibir_foto_esp32.py
```

5. Usar la cámara:

   * Botón 1: sacar foto.
   * Botón 2: cambiar paleta.
   * Mantener botón 1: rotar pantalla.

6. Las fotos se guardan automáticamente en:

```txt
~/Desktop/fotos_esp32/
```

## Guardado de imágenes HD

La ESP32 envía una imagen de 240x240 px. Para obtener una imagen de mayor tamaño, el escalado se realiza en Python en el computador.

Se recomienda usar escalado tipo `NEAREST`, ya que mantiene la estética pixelada sin suavizar los bordes.

Ejemplo:

```python
UPSCALE_SIZE = 1920

img_hd = img.resize(
    (UPSCALE_SIZE, UPSCALE_SIZE),
    Image.Resampling.NEAREST
)
```

Esto transforma la imagen de:

```txt
240x240 px
```

a:

```txt
1920x1920 px
```

## Notas técnicas

* La pantalla GC9A01 usa comunicación SPI.
* La imagen de cámara se captura en formato `RGB565`.
* El efecto visual se calcula usando luminancia y una tabla de paleta.
* Para mejorar rendimiento, no se procesa cada pixel como imagen compleja, sino que se renderiza por bloques definidos por `PIXEL_SIZE`.
* Las fotos se envían como datos RGB888 por Serial.
* La imagen final se reconstruye en Python usando Pillow.

## Problemas comunes

### La pantalla queda negra

Revisar:

* `VCC` conectado a `3V3`.
* `GND` común entre pantalla y ESP32.
* `BL / LED` conectado a `3V3`.
* Pines `SCK`, `MOSI`, `DC`, `CS` y `RST` bien conectados.
* Librería `Arduino_GFX Library` instalada.

### El script Python no encuentra el puerto

Verificar puertos disponibles:

```bash
ls /dev/cu.*
```

Actualizar el valor de `PORT` en el script Python.

Ejemplo:

```python
PORT = "/dev/cu.usbmodem1101"
```

### El puerto está ocupado

Cerrar:

* Serial Monitor de Arduino IDE.
* Serial Plotter.
* Otros scripts de Python usando el puerto.

Luego volver a ejecutar:

```bash
python3 ~/Desktop/recibir_foto_esp32.py
```

### La imagen se ve lenta

Aumentar el tamaño de pixel:

```cpp
#define PIXEL_SIZE 8
```

o:

```cpp
#define PIXEL_SIZE 12
```

### La foto se guarda en baja resolución

La captura original es de 240x240 px, pero se puede escalar en Python a 1080x1080, 1920x1920 o más usando `Image.Resampling.NEAREST`.

## Posibles mejoras

* Agregar carcasa impresa en 3D.
* Integrar batería LiPo.
* Agregar interruptor de encendido.
* Guardar imágenes en microSD.
* Enviar fotos por WiFi.
* Agregar modo galería.
* Agregar más paletas cromáticas.
* Agregar efectos como threshold, dithering Bayer, posterizado o glitch.
* Crear una interfaz visual con íconos para indicar paleta, rotación y estado de captura.

## Créditos

Proyecto desarrollado como una cámara experimental DIY basada en ESP32, explorando imagen digital, hardware abierto, estética pixelada y fotografía computacional de bajo costo.
