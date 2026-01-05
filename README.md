# DSA_Downscaling

Proyecto de reducción de escala (downscaling) de imágenes mediante interpolación bilinear, implementado en hardware FPGA (SystemVerilog) con comunicación JTAG.

## Descripción

Este proyecto implementa un acelerador hardware para el procesamiento de imágenes con interpolación bilinear. Soporta dos modos de operación:
- **Modo Secuencial**: Procesa un píxel a la vez
- **Modo SIMD**: Procesa hasta 4 píxeles en paralelo

El sistema permite la comunicación con la FPGA mediante JTAG utilizando un servidor TCL y un cliente Python.

## Estructura del Proyecto

```
DSA_Downscaling/
├── Comunicacion/          # Scripts de comunicación JTAG
│   ├── client.py          # Cliente Python para enviar comandos
│   └── jtag_server.tcl    # Servidor TCL para comunicación JTAG
├── Quartus/               # Archivos del proyecto Quartus
│   ├── DE1_SOC.sv         # Top-level del diseño
│   ├── instruction_handler.sv
│   ├── main_controller.sv
│   ├── memory_interface.sv
│   └── bilinear_interp*.sv
├── Modelo_Referencia/     # Implementación de referencia en C++
├── Imagenes/              # Directorio para imágenes de entrada/salida
└── venv/                  # Entorno virtual Python
```

## Requisitos

### Hardware
- FPGA DE1-SoC (Cyclone V)
- Cable USB Blaster para comunicación JTAG

### Software
- Intel Quartus Prime 22.1 Standard Edition
- Python 3.10+
- Make (opcional, para usar Makefile)

### Paquetes Python
```bash
pip install pillow numpy
```

## Instalación

1. **Clonar el repositorio**:
```bash
git clone https://github.com/Joel-Araya/DSA_Downscaling.git
cd DSA_Downscaling
```

2. **Crear y activar entorno virtual**:
```bash
make venv
# O manualmente:
python -m venv venv
.\venv\Scripts\Activate.ps1  # Windows PowerShell
```

3. **Instalar dependencias**:
```bash
pip install pillow numpy
```

4. **Compilar el diseño en Quartus** (si es necesario):
- Abrir el proyecto `Quartus/Test2.qpf` en Quartus Prime
- Compilar el diseño
- Programar la FPGA

## Uso

### 1. Iniciar el Servidor JTAG

Primero, asegúrate de que la FPGA está conectada y programada. Luego inicia el servidor TCL:

```bash
make server
# O manualmente:
quartus_stp -t .\Comunicacion\jtag_server.tcl
```

El servidor quedará escuchando en `localhost:2540`.

### 2. Ejecutar el Cliente Python

En otra terminal, activa el entorno virtual y ejecuta el cliente:

```bash
.\venv\Scripts\Activate.ps1
python .\Comunicacion\client.py
```

### 3. Cargar y Procesar una Imagen

El script te pedirá seleccionar una imagen. Luego configura los parámetros:

```python
# En client.py, configurar antes de ejecutar:
t.scale = 0.5        # Factor de escala (0.5 = 50% del tamaño original)
t.mode = 0           # 0: Secuencial, 1: SIMD
t.debug = 0          # 0: Desactivado, 1: Activado
t.N_simd = 0         # Número de lanes SIMD (si mode=1, típicamente 4)
```

### 4. Comandos Disponibles

Una vez iniciado el cliente, puedes usar los siguientes comandos:

- **`START`**: Inicia el procesamiento de la imagen
- **`STEP`**: Procesa un paso (útil en modo debug)
- **`IMAGE_CONFIG <width> <height> <scale> <mode> <debug> <N_SIMD>`**: Configura parámetros
- **`WRITE_PIXELS`**: Envía los datos de la imagen
- **`READ_REG <reg_name>`**: Lee un registro específico
  - Registros disponibles: `REG_STATUS`, `REG_IMG_WIDTH`, `REG_IMG_HEIGHT`, `REG_SCALE`, `PERF_CYCLES`, etc.
- **`READ_IMAGE`**: Lee la imagen procesada y la guarda como `Imagenes/imagen_generada.png`
- **`HELP`**: Muestra ayuda
- **`EXIT`**: Salir

### Ejemplo de Flujo Completo

```bash
# Terminal 1: Servidor JTAG
quartus_stp -t .\Comunicacion\jtag_server.tcl

# Terminal 2: Cliente Python
.\venv\Scripts\Activate.ps1
python .\Comunicacion\client.py
# Seleccionar imagen cuando se solicite

# En el prompt del cliente:
Comando TCL: START
Comando TCL: READ_REG REG_STATUS
Comando TCL: READ_IMAGE
Comando TCL: EXIT
```

## Makefile

El proyecto incluye un Makefile con comandos útiles:

```bash
make venv          # Crear entorno virtual
make server        # Iniciar servidor JTAG
make client        # Ejecutar cliente Python
make clean         # Limpiar archivos temporales
```

## Modelo de Referencia

El directorio `Modelo_Referencia/` contiene una implementación en C++ de la interpolación bilinear para verificar resultados:

```bash
cd Modelo_Referencia
mkdir build && cd build
cmake ..
make
./ref_bilinear
```

## Simulación

Para simular los módulos con iverilog:

```bash
iverilog -g2012 -o tb_simd.vvp \
  .\Interpolacion\tb_simd_integration.sv \
  .\Interpolacion\bilinear_interp_simd.sv \
  .\Interpolacion\simd_registers.sv \
  .\Interpolacion\stage1_interp_x.sv \
  .\Interpolacion\stage2_interp_y.sv \
  .\Interpolacion\stage3_convert.sv \
  .\Interpolacion\bilinear_interp.sv

vvp tb_simd.vvp
```

## Troubleshooting

### Error: "No se puede conectar al servidor"
- Verifica que el servidor JTAG esté ejecutándose
- Confirma que la FPGA está conectada y programada
- Revisa que el puerto 2540 esté disponible

### Error: "Imagen generada incorrecta"
- Verifica que la escala sea válida (0.5 a 1.0)
- Confirma que la imagen de entrada sea escala de grises
- Revisa los registros de debug con `READ_REG`

### La compilación en Quartus falla
- Asegúrate de usar Quartus Prime 22.1 Standard
- Verifica que todos los archivos .sv estén incluidos en el proyecto
- Revisa los archivos de asignación de pines (.pin)

## Autores

- Joel Araya
- Darío [Apellido]

## Licencia

[Especificar licencia si aplica]