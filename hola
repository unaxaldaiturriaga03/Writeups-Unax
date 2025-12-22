# 🕰️ Clock Work Memory

## 📌 Información del reto

- **Nombre:** Clock Work Memory  
- **Dificultad:** Easy  
- **Categoría:** Reversing  
- **Plataforma:** Hack The Box  
- **Objetivo:** Recuperar la flag oculta dentro de un binario WebAssembly  

---

## 🧾 Enunciado

**Clock Work Memory**

Twillie's *"Clockwork Memory"* pocketwatch is broken. The memory it holds, a precious story about the Starshard, has been distorted. By reverse-engineering the intricate *"clockwork"* mechanism of the `pocketwatch.wasm` file, you can discover the source of the distortion and apply the correct *"peppermint"* key to remember the truth.

---

## 🧠 Descripción

Se nos da un archivo `pocketwatch.wasm`, un binario **WebAssembly**.

La historia narra un reloj con memoria distorsionada que debe restaurarse con una clave correcta. Esto sugiere que **la flag está ofuscada dentro del binario**, y que debemos analizar su lógica interna para recuperarla.

Nuestro objetivo fue **entender cómo reconstruir dicha flag y extraerla sin explotación externa**, basándonos únicamente en **análisis estático y dinámico** del binario.

---

## 🔍 Análisis del binario

Convertimos el binario WebAssembly a formato `.wat` con:

```bash
wasm2wat pocketwatch.wasm
```

Esto nos permitió inspeccionar la lógica interna del programa.

Encontramos que la función exportada principal es:

```wat
(export "check_flag" (func 1))
```

> ⚠️ **Importante:**  
> Esta función **comprueba la flag**, pero **no la imprime**.

---

### 🧠 Lógica identificada

La lógica detectada fue la siguiente:

- Se reserva memoria en el stack para construir la flag.
- Se rellena el buffer con datos descifrados mediante **XOR**.
- Se compara la cadena generada con la entrada del usuario.

📌 Esto indica que **la flag se construye internamente antes de la comparación**.

---

## 🧱 Funcionamiento interno

El flujo más relevante es:

### 📦 Reserva de espacio en el stack

```wat
global.get 0
i32.const 32
i32.sub
local.tee 2
global.set 0
```

Se reservan **32 bytes de stack** para construir la cadena.

---

### 🔐 Descifrado / XOR

El programa recorre **23 bytes** de datos ofuscados, aplica una operación **XOR** y escribe el resultado en el buffer reservado.

---

### 🧵 Terminador nulo

```wat
i32.store8 offset=23
```

Esto añade un terminador `\0`, indicando el final de la cadena.

---

### 🔎 Comparación contra la entrada

El binario compara byte a byte la cadena generada con la entrada del usuario:

- Si coinciden completamente → retorna `1`
- Si no coinciden → retorna `0`

Este diseño **no permite brute force incremental**, ya que el valor `0` solo indica *"no es correcto"*, sin revelar información parcial.

---

## 🚫 Métodos descartados

Tras el análisis, se descartaron los siguientes enfoques:

- Fuerza bruta carácter a carácter  
- Oracle parcial basado en el valor de retorno  
- Brute force usando terminadores manuales  

Esto se debe a que la función **solo devuelve éxito si la cadena completa coincide**, sin diferenciar prefijos correctos o incorrectos.

---

## 🧪 Enfoque correcto: lectura directa de memoria

Sabemos que el binario **construye completamente la flag en memoria antes de compararla**.  
Por tanto, el enfoque correcto es **leer el buffer justo después de que se genere la cadena**.

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

## 🏁 Flag

```text
HTB{cl0ck_w0rk_m3m0ry}
```

---

## 📌 Conclusiones

🎯 **Lecciones clave:**

- No siempre es necesario usar brute force.
- Entender el flujo de ejecución permite acceder a datos intermedios.
- WebAssembly puede analizarse eficazmente con herramientas como `wasm2wat`.

Este reto es un excelente ejemplo de **reversing orientado a análisis de memoria y lógica interna**, más que a explotación directa.
