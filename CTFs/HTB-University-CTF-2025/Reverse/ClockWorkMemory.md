
#  Clock Work Memory

##  Información del reto

- **Nombre:** Clock Work Memory  
- **Dificultad:** Easy  
- **Categoría:** Reversing  
- **Plataforma:** Hack The Box  
- **Objetivo:** Recuperar la flag oculta dentro de un binario WebAssembly  

---

##  Enunciado

**Clock Work Memory**

Twillie's *"Clockwork Memory"* pocketwatch is broken. The memory it holds, a precious story about the Starshard, has been distorted. By reverse-engineering the intricate *"clockwork"* mechanism of the `pocketwatch.wasm` file, you can discover the source of the distortion and apply the correct *"peppermint"* key to remember the truth.

---

##  Descripción

La historia narra un reloj con memoria distorsionada que debe restaurarse con una clave correcta. Esto sugiere que **la flag está cifrada dentro del binario**, y que debemos analizar su lógica interna para recuperarla.

Nuestro objetivo fue **entender cómo reconstruir dicha flag y extraerla sin explotación externa**, basándonos únicamente en **análisis estático y dinámico** del binario.

Se nos da un archivo `pocketwatch.wasm`, un binario **WebAssembly**.

<img width="932" height="300" alt="image" src="https://github.com/user-attachments/assets/ca2beb13-5de5-49f5-979c-37b58eb57fdb" />

Al analizar el archivo con `file`, se confirma que se trata de un binario WebAssembly versión 1 (MVP), compatible con las herramientas estándar de análisis como `wasm2wat`.

---

##  Análisis del binario

Convertimos el binario WebAssembly a formato texto (WAT) con:

```bash
wasm2wat pocketwatch.wasm
```


Esto me permitió inspeccionar la lógica interna del programa.  
Para facilitar el análisis, se han omitido partes del código WAT que no aportan información relevante al proceso de construcción y verificación de la flag.

Funciones:
<img width="973" height="315" alt="image" src="https://github.com/user-attachments/assets/cfab8141-42ac-4beb-aa9a-0ae39d1556e3" />


Encontramos que la función exportada principal es:

```wat
(export "check_flag" (func 1))
```

>  **Importante:**  
> Esta función **comprueba la flag**, pero **no la imprime**.

---

### Fragmentos relevantes de `check_flag`

#### Reserva de memoria en el stack
```wat
global.get 0
i32.const 32
i32.sub
local.tee 2
global.set 0
```

#### Descifrado de la flag mediante XOR

```wat
loop
  ...
  i32.load8_u offset=1024
  i32.xor
  i32.store8
  ...
  i32.const 23
  i32.ne
  br_if 0
end
```

#### Terminación de la cadena

```wat
i32.store8 offset=23
```

#### Comparación con la entrada del usuario

```wat
block
  ...
  i32.eq
  br_if
end
```

#### Global utilizado como puntero de stack

```wat
(global (;0;) (mut i32) (i32.const 66592))
```

El módulo define un global inicializado en `66592`, que actúa como puntero de stack.
Al reservar memoria para la flag, se restan `32` bytes a este valor, por lo que la flag se construye en la dirección `66592 - 32`.
De esta forma, es posible leer directamente los `23` bytes generados para recuperar la flag.

### Valores constantes relevantes

* `32`: tamaño del buffer reservado en el stack.
* `23`: longitud exacta de la flag.
* `1024`: offset en memoria donde se encuentran los datos cifrados.


###  Lógica identificada

La lógica detectada fue la siguiente:

- Se reserva memoria en el stack para construir la flag.
- Se rellena el buffer con datos descifrados mediante **XOR**.
- Se compara la cadena generada con la entrada del usuario.

 Esto indica que **la flag se construye internamente antes de la comparación**.

##  Enfoque correcto: lectura directa de memoria

El enfoque correcto es **leer el buffer justo después de que se genere la cadena**.

El valor inicial del stack global es:

```wat
(global (;0;) (mut i32) (i32.const 66592))
```

Como se reservan 32 bytes, la flag se encuentra en:

```
66592 - 32
```

Leyendo **23 bytes** desde esa dirección se obtiene la flag completa.

---

##  Flag

```text
HTB{cl0ck_w0rk_m3m0ry}
```

---

##  Conclusiones

 **Lecciones clave:**

- No siempre es necesario usar brute force.
- Entender el flujo de ejecución permite acceder a datos intermedios.
- WebAssembly puede analizarse eficazmente con herramientas como `wasm2wat`.

Este reto es un excelente ejemplo de **reversing orientado a análisis de memoria y lógica interna**, más que a explotación directa.
