## Resumen
driver.py
Actúa como orquestador / punto de entrada del sistema. Importa y coordina múltiples controladores de instrumentos (Fluke, PSU, ESA620, osciloscopio Tektronix, PROSIM8, carga programable, etc.), maneja identificación de dispositivos, versionado y logging básico. Está pensado como capa superior para automatizar ensayos con varios equipos conectados (principalmente por serial).
ESA620.py
Es un driver específico del analizador de seguridad eléctrica Fluke ESA620. Maneja la comunicación por puerto serie, tiempos de timeout, parsing de respuestas y control de errores (por ejemplo, el error -102 cuando no se puede convertir la respuesta a float). Se nota evolución enfocada en robustez de comunicación y evitar cuelgues del puerto.



# ESA620 Driver – Fluke ESA 620 Electrical Safety Analyzer

Este módulo implementa el control completo del **Fluke ESA 620** mediante comunicación serie, permitiendo automatizar ensayos de seguridad eléctrica según IEC 60601 y normas relacionadas.

El ESA620 se integra a través de un **driver de alto nivel** (`driver.py`) que interpreta comandos tipo CLI y delega la ejecución al controlador específico (`ESA620.py`).

---

##  Estructura relevante

CONTROLADORES/
├── ESA620.py # Driver específico del Fluke ESA620
├── ident_devices.py # Detección automática de puertos
driver.py # Orquestador principal


---

##  Conexión y detección automática

El sistema **detecta automáticamente el puerto serie** del ESA620 utilizando un archivo JSON intermedio (`devices.json`).

- Si el dispositivo ya fue detectado, se reutiliza la conexión.
- Si no existe o falla, se vuelve a ejecutar la detección.
- El ESA620 se mantiene en un **DEVICE_POOL** para evitar reconexiones innecesarias.

---

## Uso básico

La interacción se realiza enviando un **string de comando** al driver:

--> python.exe
from driver import DRIVER

driver = DRIVER()
resultado = driver.run("ESA620 --run Voltage")
print(resultado)
Formato general del comando
ESA620 --run <ENSAYO> --parametro valor --parametro valor


## Ensayos soportados (--run)

| Ensayo                    | Alias aceptados                       | Descripción                          |
| ------------------------- | ------------------------------------- | ------------------------------------ |
| `Voltage`                 | `MainVoltage`, `mainVoltage`          | Medición de tensión de red           |
| `Resistencia`             | `EarthResistance`, `TierraProteccion` | Resistencia de tierra de protección  |
| `Insulation`              | `InsulationResistance`                | Resistencia de aislación             |
| `Current`                 | `EquipmentCurrent`                    | Corriente de consumo del equipo      |
| `LeakageEarth`            | `FugaATierra`                         | Corriente de fuga a tierra           |
| `EnclosureLeakageCurrent` | `Carcasa`                             | Corriente de fuga por carcasa        |
| `PatientLeakageCurrent`   | `Paciente`                            | Corriente de fuga a paciente         |
| `MainsAppliedParts`       | `Mains`                               | Corriente de red en partes aplicadas |
| `AuxiliaryCurrent`        | `Auxiliary`                           | Corriente auxiliar de paciente       |


# Parámetros configurables
(Estos son algunos ejemplos, ya que se han agregadi nombres en distintos idiomas con el fin de facilitar)
Cantidad de derivaciones
--leads 3 | 5 | 10
Polaridad
--pol N        # Normal
--pol R        # Reversa
--pol OFF      # Apagado
Neutro
--neutro C     # Cerrado
--neutro O     # Abierto
Tierra
--tierra C     # Cerrada
--tierra O     # Abierta
Tipo de ensayo (aislación)
--tipo LIVE_TO_NEUTRAL
--tipo LIVE_TO_EARTH
--tipo NEUTRAL_TO_EARTH
--tipo MAINS-PE
--tipo AP-PE
--tipo MAIN-A.P

# Alimentación del equipo bajo ensayo

El ESA620 puede encender o apagar el equipo bajo ensayo:

ESA620 --power On
ESA620 --power Off

# Ejemplos completos
Corriente de fuga a paciente con 5 derivaciones
driver.run("ESA620 --run PatientLeakageCurrent --leads 5 --pol N --neutro C --tierra C")
Corriente por carcasa con neutro abierto
driver.run("ESA620 --run EnclosureLeakageCurrent --pol R --neutro O")
Resistencia de tierra
driver.run("ESA620 --run Resistencia")

 
## Manejo de errores
Código	Significado
-101	Puerto serie no encontrado o acceso denegado
-102	Error al convertir la respuesta del ESA620
-103	Error al abrir o configurar el puerto serie
Los errores se devuelven como string, no como excepción, para facilitar la integración con sistemas externos (HMI, secuenciadores, etc.).

## Modo remoto / local
El driver coloca automáticamente el ESA620 en modo REMOTE antes de cada ensayo.

Puede volver a modo LOCAL usando el método LOCAL() antes de cerrar el puerto (recomendado).


## Notas de diseño
Muchas de las caracteristicas y estructuras se hizo ingenieria inversa con cables espias

Comunicación robusta con timeout y write_timeout.

Reintentos cuando la respuesta es "*".

Selección automática del valor máximo en ensayos multipunto (paciente, partes aplicadas).

Pensado para ejecución automatizada y repetible en entornos de laboratorio.