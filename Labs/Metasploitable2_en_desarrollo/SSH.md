# 🔐 Evaluación de Seguridad del Servicio SSH en Metasploitable2

---

## 1️⃣ Introducción

Esta sección documenta la evaluación del servicio **SSH** que se ejecuta en **Metasploitable2 (192.168.56.101)**.  
El objetivo fue:

- Enumerar el servicio SSH  
- Identificar la versión y los riesgos potenciales  
- Intentar autenticación (credenciales por defecto / fuerza bruta)  
- Obtener acceso si era posible  
- Evaluar la postura de seguridad y concluir riesgos realistas  

---

## 2️⃣ Enumeración del Servicio

### ✅ Escaneo de Puertos

Se comenzó escaneando el servicio SSH utilizando **Nmap**:

```bash
nmap -sV -p22 192.168.56.101
```

**Resultados:**

```text
22/tcp open ssh OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
```

### ✅ Hallazgos Clave

* SSH está **activo en el puerto 22**
* El servidor ejecuta **OpenSSH 4.7p1**
* Esta versión es **extremadamente antigua** (año 2008)
* Depende de **algoritmos criptográficos obsoletos**
* Es conocida por ser vulnerable e insegura

Esto ya sugiere una **alta superficie de ataque**.

---

## 3️⃣ Intento Inicial de Autenticación

Se intentó un inicio de sesión directo utilizando las credenciales por defecto conocidas de Metasploitable:

```bash
ssh msfadmin@192.168.56.101
```

Inicialmente, la conexión SSH falló debido a algoritmos de clave no soportados.
Para evitar este problema, se forzó al cliente SSH a utilizar tipos de clave RSA antiguos:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa msfadmin@192.168.56.101
```

### ✅ Inicio de Sesión Exitoso

La contraseña `msfadmin` funcionó correctamente, otorgando acceso completo por SSH:

```text
Linux metasploitable 2.6.24-16-server
Last login: Tue Nov 25
msfadmin@metasploitable:~$
```

### 🚨 Problema Crítico de Seguridad

El sistema era accesible utilizando **credenciales por defecto**, lo que implica:

* Sin políticas de contraseñas
* Sin endurecimiento de credenciales
* Acceso remoto directo sin restricciones

Esto por sí solo constituye un **fallo grave de seguridad**.

---

## 4️⃣ Pruebas de Fuerza Bruta con Hydra

### ✅ Objetivo

Evaluar si un ataque de fuerza bruta sería viable contra el servicio SSH.

### ✅ Diccionarios Utilizados

**users.txt:**

```text
msfadmin
root
user
postgres
```

**passwords.txt:**

```text
msfadmin
password
123456
root
```

### ✅ Ejecución de Hydra

```bash
hydra -L users.txt -P passwords.txt -t 4 ssh://192.168.56.101
```

### ❌ Fallo de Hydra

Hydra no pudo ni siquiera establecer una conexión:

```text
kex error : no match for method mac algo client->server
```

### ✅ Motivo del Fallo

* Hydra utiliza **algoritmos criptográficos modernos**
* OpenSSH 4.7p1 solo soporta **algoritmos muy antiguos y obsoletos**
* Ambos extremos son **criptográficamente incompatibles**
* Por tanto, la fuerza bruta **no es posible con Hydra estándar**, ya que no se puede negociar un canal seguro

### ✅ Conclusión Profesional

El fallo de Hydra **no es una mitigación**, sino una prueba adicional de lo inseguro que es el servicio SSH.
Las herramientas modernas ni siquiera pueden comunicarse con él debido al uso de cifrados obsoletos.

En un entorno real, esto provocaría:

* Problemas de compatibilidad
* Dificultades en monitorización moderna
* Mayor exposición a exploits heredados

---

## 5️⃣ Evaluación Final

### ✅ Riesgos Confirmados

✔ Credenciales por defecto permiten acceso SSH completo
✔ Versión de OpenSSH obsoleta (2008)
✔ Algoritmos criptográficos débiles y obsoletos
✔ Sin protección contra fuerza bruta (login por contraseña habilitado)
✔ Compromiso del sistema posible sin conocimientos avanzados

### ✅ Impacto en un Entorno Real

Un atacante podría:

* Iniciar sesión remotamente
* Ejecutar comandos
* Escalar privilegios localmente
* Pivotar dentro de la red interna

Esto representa un **compromiso remoto completo del sistema**.

---

## 6️⃣ Recomendaciones de Mitigación

✅ Deshabilitar SSH o restringirlo a hosts de confianza
✅ Actualizar OpenSSH a una versión soportada
✅ Forzar políticas de contraseñas fuertes
✅ Deshabilitar autenticación por contraseña y usar claves
✅ Eliminar credenciales por defecto
✅ Habilitar detección de intrusiones y logging

---

## ✅ Conclusión

El servicio SSH en **Metasploitable2** es **críticamente vulnerable**.
El acceso remoto se logró simplemente utilizando credenciales por defecto, y la criptografía obsoleta refuerza la evidencia de una higiene de seguridad deficiente.

Incluso sin capacidad de fuerza bruta, el sistema fue comprometido de forma trivial, demostrando lo peligroso que es el uso de credenciales débiles y servicios heredados.

Esta evaluación validó con éxito:

* Enumeración
* Identificación de versión
* Pruebas de autenticación
* Acceso completo al sistema

**Resultado:** ✅ SSH completamente comprometido.

