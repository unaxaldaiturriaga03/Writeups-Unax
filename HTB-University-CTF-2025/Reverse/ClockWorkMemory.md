
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

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bdd6a960-35a2-43f7-a5dd-9a0c62796a3f" />

La historia narra un reloj con memoria distorsionada que debe restaurarse con una clave correcta. Esto sugiere que **la flag está ofuscada dentro del binario**, y que debemos analizar su lógica interna para recuperarla.

Nuestro objetivo fue **entender cómo reconstruir dicha flag y extraerla sin explotación externa**, basándonos únicamente en **análisis estático y dinámico** del binario.

---

## 🔍 Análisis del binario

Convertimos el binario WebAssembly a formato `.wat` con:

```bash
wasm2wat pocketwatch.wasm
```

Esto me permitió inspeccionar la lógica interna del programa.

```wat
[/Descargas/rev_clock_work_memory]
$ wasm2wat pocketwatch.wasm    
(module
  (type (;0;) (func))
  (type (;1;) (func (param i32) (result i32)))
  (type (;2;) (func (param i32)))
  (type (;3;) (func (result i32)))
  (func (;0;) (type 0)
    nop)
  (func (;1;) (type 1) (param i32) (result i32)
    (local i32 i32 i32 i32)
    global.get 0
    i32.const 32
    i32.sub
    local.tee 2
    global.set 0
    local.get 2
    i32.const 1262702420
    i32.store offset=27 align=1
    loop  ;; label = @1
      local.get 1
      local.get 2
      i32.add
      local.get 2
      i32.const 27
      i32.add
      local.get 1
      i32.const 3
      i32.and
      i32.add
      i32.load8_u
      local.get 1
      i32.load8_u offset=1024
      i32.xor
      i32.store8
      local.get 1
      i32.const 1
      i32.add
      local.tee 1
      i32.const 23
      i32.ne
      br_if 0 (;@1;)
    end
    local.get 2
    i32.const 0
    i32.store8 offset=23
    block  ;; label = @1
      local.get 0
      i32.load8_u
      local.tee 3
      i32.eqz
      local.get 3
      local.get 2
      local.tee 1
      i32.load8_u
      local.tee 4
      i32.ne
      i32.or
      br_if 0 (;@1;)
      loop  ;; label = @2
        local.get 1
        i32.load8_u offset=1
        local.set 4
        local.get 0
        i32.load8_u offset=1
        local.tee 3
        i32.eqz
        br_if 1 (;@1;)
        local.get 1
        i32.const 1
        i32.add
        local.set 1
        local.get 0
        i32.const 1
        i32.add
        local.set 0
        local.get 3
        local.get 4
        i32.eq
        br_if 0 (;@2;)
      end
    end
    local.get 3
    local.get 4
    i32.sub
    local.set 0
    local.get 2
    i32.const 32
    i32.add
    global.set 0
    local.get 0
    i32.eqz)
  (func (;2;) (type 2) (param i32)
    local.get 0
    global.set 0)
  (func (;3;) (type 3) (result i32)
    global.get 0)
  (table (;0;) 2 2 funcref)
  (memory (;0;) 258 258)
  (global (;0;) (mut i32) (i32.const 66592))
  (export "memory" (memory 0))
  (export "check_flag" (func 1))
  (export "__indirect_function_table" (table 0))
  (export "_initialize" (func 0))
  (export "_emscripten_stack_restore" (func 2))
  (export "emscripten_stack_get_current" (func 3))
  (elem (;0;) (i32.const 1) func 0)
  (data (;0;) (i32.const 1024) "\1c\1b\010#{0&\0b=p=\0b~0\147\7fs'un>"))
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9fd15035-f865-4736-8026-df26155de441" />

Funciones:
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f198c43d-d46c-4c86-972d-616f76fa21d7" />

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

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bf9cc784-f263-4f56-8db1-edaa2560bf53" />

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
