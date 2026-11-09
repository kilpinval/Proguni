# Proguni
Programa que lee y graba microcontroladores, memorias prom, eeprom, i2c, fpga y demas 
README.md — Sistema de Programación Universal (UProg)

🎯 UProg — Programador Universal por Puerto Serial

https://img.shields.io/badge/estado-en%20desarrollo-yellow
https://img.shields.io/badge/interfaz-RS--232%20%2F%20UART-blue
https://img.shields.io/badge/host-PC%20%28Linux%2FWindows%29-lightgrey

Herramienta de programación desde PC vía puerto serial para microcontroladores, memorias EPROM/EEPROM/NVRAM/I²C y FPGA, con capa de abstracción de protocolos y control físico de señales.

📋 Tabla de Contenidos

· Descripción General
· Dispositivos Soportados
· Arquitectura del Sistema
· Requisitos de Hardware
· Consideraciones Físicas Críticas
· Instalación y Compilación
· Uso
· Solución de Problemas
· Licencia

---

🔍 Descripción General

UProg es un sistema de programación que permite grabar y leer dispositivos lógicos y de memoria desde una computadora personal utilizando el puerto serial (UART/RS-232) como único canal de comunicación. El software host, escrito en C, se encarga de generar las secuencias de temporización y los voltajes de programación a través de una interfaz de hardware externa (adaptador USB-Serial o UART nativa).

El proyecto nace de la necesidad de contar con una herramienta abierta y verificable para tareas de recuperación, ingeniería inversa y producción de bajo volumen, donde los programadores comerciales resultan costosos o cerrados.

A diferencia de soluciones que emplean USB de alta velocidad, el uso del puerto serial impone restricciones de ancho de banda que se compensan con un protocolo de tramas optimizado y almacenamiento en búfer en el firmware del adaptador.

---

✨ Características

· ✅ Soporte multi-dispositivo: Microcontroladores (AVR, PIC, STM32 vía bootloader UART), memorias EPROM paralelas, EEPROM seriales I²C, NVRAM y configuración de FPGA (bitstreams).
· ✅ Interfaz serial configurable: Velocidades desde 9600 bps hasta 2 Mbps, con control de flujo por hardware opcional .
· ✅ Generación de Vpp: Control de voltaje de programación para EPROM y microcontroladores que lo requieran, mediante DAC o resistencia conmutada desde el adaptador.
· ✅ Protocolo de tramas con verificación: Comandos con CRC para asegurar la integridad de los datos transmitidos por el cable serial.
· ✅ Modo batch: Programación y verificación automática de múltiples dispositivos en secuencia.
· ✅ Registro de operaciones: Archivo de log con marcas de tiempo para trazabilidad de producción.

---

🧠 Arquitectura del Sistema

```
[PC Host] ←→ [Adaptador Serial] ←→ [Zócalo / Cableado al dispositivo]
   (C)          (MCU puente)           (Señales físicas)
```

Componentes:

1. Host (C): Interfaz de línea de comandos que carga el archivo (HEX/BIN), selecciona el algoritmo de programación y gestiona la comunicación serial.
2. Adaptador serial: Microcontrolador dedicado (ej. FTDI o MCU con UART) que traduce los comandos del host a señales de temporización (I²C, SPI, JTAG, paralelo) y controla los voltajes de programación.
3. Interfaz física: Zócalo ZIF para memorias paralelas y conectores para ICSP/ISP/JTAG según el dispositivo .

---

🛠️ Requisitos de Hardware

Componente Especificación Mínima
PC Host Puerto USB 2.0 o RS-232 nativo
Adaptador Serial FT232RL, CH340 o MCU con UART (ej. STM32)
Nivel Lógico 3.3V / 5V conmutables
Alimentación Dispositivo 1.8V – 5V regulada, con protección de sobrecorriente
Vpp (EPROM/MCU) Hasta 12.5V – 21V según dispositivo 
Zócalo ZIF de 40 pines (mínimo) para memorias paralelas

---

⚡ Consideraciones Físicas Críticas

Esta sección es obligatoria antes de conectar cualquier dispositivo.

1. Niveles de Voltaje y Vpp

· Vcc del dispositivo: Verificar en el datasheet. Un EPROM antiguo puede requerir 5V, mientras que una FPGA moderna usa 1.8V. Nunca apliques 5V a un dispositivo de 3.3V sin nivelador.
· Vpp (voltaje de programación): Las EPROM y algunos microcontroladores requieren pulsos de 12V a 21V en el pin de programación. El adaptador debe generar este voltaje de forma controlada y solo durante la ventana de programación, nunca de forma permanente .
· Secuencia de encendido: Aplica Vcc antes que Vpp; retira Vpp antes que Vcc. Invertir el orden puede destruir el dispositivo.

2. Protección de Sobrecorriente y Sobretensión

· Implementa limitación de corriente en la línea de Vcc. Un dispositivo insertado al revés o defectuoso puede consumir corrientes destructivas.
· Usa diodos de sujeción (clamp) en las líneas de datos y programación para evitar que las señales del adaptador excedan el voltaje de alimentación del dispositivo.
· Coloca capacitores de desacoplo (100 nF cerámico + 10 µF electrolítico) lo más cerca posible del zócalo. Las conmutaciones de Vpp generan transitorios que pueden corromper datos o dañar el chip .

3. Integridad de Señales en el Puerto Serial

· Longitud del cable: Para velocidades superiores a 115200 bps, limita el cable USB-Serial a menos de 1 metro. Para RS-232 nativo, no excedas 3 metros sin aislamiento.
· Niveles RS-232 vs TTL: El puerto serial de un PC clásico entrega niveles ±12V. Nunca conectes directamente un dispositivo TTL (3.3V/5V) a un puerto RS-232 nativo; usa un transceptor MAX232 o un adaptador USB-TTL.
· Aislamiento: Si el dispositivo objetivo está alimentado por una fuente distinta a la del PC, utiliza aislamiento galvánico (opt acopladores o digital isolators) en las líneas TX/RX para evitar lazos de tierra que pueden dañar ambos equipos.

4. Conexión Física y Zócalos

· Zócalo ZIF: Para memorias EPROM/EEPROM paralelas, utiliza un zócalo de fuerza de inserción cero. Insertar un chip con el zócalo energizado puede generar picos de corriente.
· Orientación del chip: Marca el pin 1 en el zócalo y en el PCB. La inversión del chip es la causa más común de destrucción de dispositivos durante la programación.
· Cables de programación (ISP/JTAG): Mantén los cables cortos (< 20 cm) y trenzados con retorno de tierra. Las señales de reloj (TCK, SCK) son especialmente sensibles a ruido e reflexiones.

5. Aislamiento Térmico y Disipación

· Los reguladores lineales que generan Vpp desde 12V/24V disipan potencia considerable. Calcula la potencia disipada y añade disipador si es necesario.
· No programes dispositivos en un entorno con temperatura ambiente superior a 40°C sin ventilación forzada.

---

📦 Instalación y Compilación

```bash
# Dependencias (Debian/Ubuntu)
sudo apt install build-essential libserial-dev

# Clonar y compilar
git clone https://github.com/tu-usuario/uprog.git
cd uprog
make

# El binario se genera en ./build/uprog
```

Para Windows, se requiere MSYS2 o WSL con acceso al puerto serial (/dev/ttyS* o COMx).

---

🚀 Uso

```bash
# Listar puertos disponibles
./uprog --list-ports

# Programar una EEPROM I2C (ej. 24C256)
./uprog -p /dev/ttyUSB0 -d 24c256 -w firmware.bin -v

# Leer una EPROM 27C256
./uprog -p /dev/ttyUSB0 -d 27c256 -r dump.bin

# Programar un microcontrolador AVR vía ISP
./uprog -p /dev/ttyUSB0 -d atmega328p -i isp -w firmware.hex

# Cargar bitstream en FPGA (JTAG)
./uprog -p /dev/ttyUSB0 -d fpga -i jtag -w design.bit
```

Opciones principales:

Flag Descripción
-p Puerto serial (/dev/ttyUSB0, COM3)
-d Dispositivo objetivo (definido en devices.conf)
-w Escribir archivo
-r Leer y volcar a archivo
-v Verificar después de escribir
-i Interfaz (i2c, spi, isp, jtag, parallel)

---

🔧 Solución de Problemas

Síntoma Causa Probable Solución
No detecta el puerto Permisos insuficientes sudo usermod -a -G dialout $USER (Linux)
Error de verificación Vpp inestable o cable largo Acortar cable, revisar capacitores de desacoplo
Dispositivo no responde Orientación incorrecta o Vcc ausente Verificar pin 1 y voltaje en zócalo
Lectura corrupta Baud rate demasiado alto para el cable Reducir a 115200 bps
Adaptador se calienta Regulador de Vpp sobrecargado Añadir disipador o reducir corriente

---

📄 Licencia

MIT License. Consulta LICENSE para más detalles.

---

Advertencia: La programación de dispositivos electrónicos con voltajes superiores a 5V conlleva riesgo de daño material. Verifica siempre el datasheet del dispositivo antes de conectarlo. El autor no se hace responsable de dispositivos dañados por uso indebido.
