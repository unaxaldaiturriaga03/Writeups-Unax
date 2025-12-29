# ✅ Análisis de Vulnerabilidad del Servicio FTP – Metasploitable2

## 1. Visión General

Durante la evaluación de seguridad del sistema objetivo **Metasploitable2**, se identificó un servicio **FTP** activo y se analizó en busca de posibles configuraciones incorrectas y debilidades. El objetivo principal fue enumerar el servicio, validar los mecanismos de autenticación y determinar si la configuración podía exponer datos sensibles o permitir una mayor explotación del sistema.

---

## 2. Enumeración del Servicio

### 🔹 Escaneo de Puertos

Un escaneo inicial de red reveló que el **puerto 21/TCP** estaba abierto:

```bash
nmap -sV -p 21 <TARGET-IP>

**Resultado:**

```text
21/tcp open  ftp     vsFTPd 2.3.4
```

Se confirmó que el servicio estaba ejecutando **vsFTPd 2.3.4**, una versión históricamente asociada a vulnerabilidades conocidas, lo que lo convierte en un objetivo de alto valor para una inspección más profunda.

---

## 3. Pruebas de Autenticación

### 🔹 Intento de Inicio de Sesión Anónimo

Basándose en configuraciones incorrectas comunes en servicios FTP, se realizó una prueba de autenticación utilizando las credenciales anónimas por defecto:

```text
ftp <TARGET-IP>
Name: anonymous
Password: anonymous
```

**Resultado:**

```text
230 Login successful.
```

✅ **La autenticación anónima estaba habilitada**, lo que indica una política de control de acceso mal configurada.

---

## 4. Enumeración Posterior a la Autenticación

Una vez autenticado, se solicitó un listado de directorios:

```text
ftp> ls
```

**Resultado:**

El servidor permitió **acceso de solo lectura**, pero no expuso archivos ni directorios de valor inmediato. No se concedieron permisos de escritura, como se confirmó con el siguiente intento fallido de subida de archivo:

```text
ftp> put test.txt
553 Could not create file.
```

Esto verifica que el usuario anónimo está restringido a **privilegios de solo lectura**.

---

## 5. Análisis de la Vulnerabilidad

### ✅ Problema Identificado

**Acceso FTP Anónimo Habilitado**

**Descripción:**
El servidor FTP permite que usuarios no autenticados inicien sesión e interactúen con el servicio. Aunque el acceso está limitado a modo de solo lectura y no se expuso contenido sensible durante las pruebas, esta configuración sigue violando las buenas prácticas de seguridad.

### 🔥 Factores de Riesgo

* Incrementa la superficie de ataque para escáneres automatizados y herramientas de fuerza bruta.
* Podría exponer archivos sensibles si en el futuro se añade contenido al directorio raíz del FTP.
* La versión del servicio (**vsFTPd 2.3.4**) está históricamente vinculada a exploits conocidos, lo que agrava el riesgo.

### ✅ Evaluación de Impacto

| Factor           | Nivel                                          |
| ---------------- | ---------------------------------------------- |
| Confidencialidad | Baja (no se expusieron datos sensibles)        |
| Integridad       | Baja (sin permisos de escritura)               |
| Disponibilidad   | Ninguno                                        |
| Riesgo Global    | Bajo, pero la postura de seguridad se debilita |

Aunque el impacto actual es limitado, el acceso anónimo en servicios FTP antiguos se considera una **debilidad de configuración y de política**, y podría convertirse en un punto de pivote dentro de una cadena de ataque real.

---

## 6. Recomendaciones

✔️ Deshabilitar el acceso anónimo en la configuración de **vsFTPd**:

```ini
anonymous_enable=NO
```

✔️ Restringir el acceso únicamente a usuarios autenticados.

✔️ Implementar controles de acceso más estrictos, incluyendo:

* Autenticación obligatoria
* Aislamiento de usuarios (chroot)
* Registro y monitorización de actividad

✔️ Considerar la migración a protocolos más seguros (por ejemplo, **SFTP** o **FTPS**).

✔️ Dada la versión obsoleta del servicio, actualizarlo o desmantelarlo si no es necesario.

---

## 7. Conclusión

El servicio FTP en **Metasploitable2** está mal configurado, permitiendo autenticación anónima. Aunque no se identificaron datos sensibles ni privilegios de escritura, el servicio incrementa la exposición del sistema y contradice las buenas prácticas de seguridad. En un entorno real, esta debilidad podría explotarse junto con otras vulnerabilidades para escalar privilegios o extraer información sensible.

La remediación adecuada debe centrarse en eliminar el acceso anónimo, actualizar el servicio y migrar a protocolos de comunicación seguros.



