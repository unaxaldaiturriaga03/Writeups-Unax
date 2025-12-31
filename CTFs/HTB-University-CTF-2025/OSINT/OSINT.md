
# OSINT Writeup – SnowSignal: SleighComms & FrostFleet: RiverWatch

##  SnowSignal – SleighComms

###  Enunciado (Original)

> *"A Signal Operator at Tinselwick Signal Intercept Station has intercepted an emergency transmission from a civilian sleigh that went off-course during a snow squall on Christmas Eve. The pilot managed to send a distress beacon using encoded bell tones before their communication equipment failed. The operator must analyze the bell tone sequence, decode it using the station's reference manual, cross-reference the decoded patterns with the landmark database, and identify the correct location to dispatch the rescue team before time runs out."*  
>  
> **Flag Format:** `HTB{LOCATION_PREFIX}`

---

###  Análisis del reto

Al acceder a la plataforma del reto, se disponía de varias secciones:

- **Signal Player**: permitía escuchar el audio interceptado.
- **Decoder Manual**: contenía la tabla de equivalencias entre pitidos y códigos.
- **Landmark Database**: servía para contextualizar los códigos decodificados.
- **Submit Location**: campo para introducir la flag final.

El elemento clave era un **audio con un patrón de pitidos**, que debía ser interpretado correctamente.

---

###  Decodificación del audio

Tras reproducir el audio, se identificó claramente la siguiente secuencia numérica:
```
12132

```
Cada número representaba una cantidad de pitidos consecutivos.  
Consultando el **Decoder Manual**, se obtuvo la siguiente equivalencia:

| Pitidos | Código |
|-------|--------|
| 1 | ST |
| 2 | FR |
| 3 | EV |

Aplicando esta tabla a la secuencia `1 2 1 3 2`:

- **1 → ST**
- **2 → FR**
- **1 → ST**
- **3 → EV**
- **2 → FR**

---

###  Construcción de la flag

Uniendo los códigos obtenidos en el orden correcto:
```
STFRSTEVFR
```
Aplicando el formato solicitado por el reto:
```
HTB{STFRSTEVFR}
```
---

### ✅ Flag final – SnowSignal
```
HTB{STFRSTEVFR}
```
---

##  FrostFleet – RiverWatch


###  Enunciado (Original)

> *"A Maritime Investigator at Tinselwick River Authority has received an automated distress alert from patrol vessel FROSTSTAR, which went dark during a Christmas Eve patrol along the Frostwick River. The vessel's AIS system transmitted one final ""ghost ping"" before all communications ceased. The investigator must analyze the AIS coordinates, calculate distances to all registered river docks using the Haversine formula, cross-reference Captain Wintergale's log entries for observational clues, and identify the exact dock where the vessel ran aground to dispatch the rescue team before the incoming storm makes the river impassable."*  
>  
> **Flag Format:** `HTB{DOCK_NAME_DOCK_ID} Example: HTB{HARBOR_POINT_D042}`


---

###  Última señal AIS

La última transmisión registrada fue:

Latitud: 42.345600
Longitud: -71.089200

A partir de esta ubicación, se sabía que el barco debía encontrarse cerca de uno de los **cuatro muelles registrados** en la base de datos.

---

###  Muelles candidatos

| Dock ID | Nombre | Distancia |
|------|----------------------|-----------|
| D-001 | Starlight Pier | 143 m |
| D-002 | Peppermint Wharf | 80 m |
| D-003 | Coodlewick Marina | 552 m |
| D-004 | (Dock más cercano) | **28 m** |

Las distancias se obtuvieron aplicando el **cálculo de distancia geográfica (Haversine)** entre la última posición AIS y las coordenadas de cada muelle.

---

###  Análisis del diario del capitán

Además de los datos geográficos, el reto incluía un **diario de navegación** del capitán, donde se describían:

- Maniobras finales del barco
- Referencias visuales cercanas
- Dificultades para maniobrar en zonas estrechas

Las descripciones encajaban claramente con un **muelle muy cercano y de acceso reducido**, lo que descartaba opciones más lejanas como D-003.

La combinación de:
- **Menor distancia**
- **Coincidencia narrativa con el diario**
- **Lógica de navegación en situación de emergencia**

apuntaba de forma clara al muelle **D-004**.

---

###  Construcción de la flag

Siguiendo el formato especificado:
```
HTB{DOCK_NAME_DOCK_ID}
```
Y sabiendo que el muelle correcto es **D-004**, la flag queda:
```
HTB{NombreDelPuerto_D-004}
```

---

### ✅ Flag final – FrostFleet
```
HTB{NombreDelPuerto_D-004}
```
---

##  Conclusiones

Estos dos retos destacan por:

- Uso de **narrativa inmersiva**
- Enfoque en **análisis lógico y correlación de datos**
- Ausencia de explotación técnica avanzada
- Énfasis en **interpretar correctamente la información disponible**

Son ejemplos claros de **retos OSINT narrativos**, donde el éxito depende más del razonamiento que de herramientas complejas.

---

##  Resumen de flags

| Reto | Flag |
|----|------|
| SnowSignal – SleighComms | `HTB{STFRSTEVFR}` |
| FrostFleet – RiverWatch | `HTB{D-004}` |

---


