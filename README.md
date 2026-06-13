# Configuración de MySQL Server para Acceso desde Otras Computadoras

## Objetivo

Permitir que varias computadoras de la red local se conecten a una misma base de datos MySQL instalada en una computadora servidor.

```bash
CREATE DATABASE sistema_empresa
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```
---

# Información de la Red

## Servidor MySQL

```text
IP: 192.168.1.38
Puerto: 3306
```

## Cliente

```text
IP: 192.168.1.41
```

---

# Paso 0 - Obtener la IP del Servidor MySQL

En la computadora donde está instalado MySQL:

1. Presionar:

```text
Windows + R
```

2. Escribir:

```text
cmd
```

3. Presionar Enter.

4. Ejecutar:

```cmd
ipconfig
```

5. Buscar la línea:

```text
Dirección IPv4
```

Ejemplo:

```text
Dirección IPv4 . . . . . . . . . : 192.168.1.38
```

6. Anotar esa dirección IP, ya que será utilizada por las demás computadoras para conectarse al servidor MySQL.

Ejemplo de conexión:

```text
Hostname: 192.168.1.38
Puerto: 3306
```

---

# Paso 1 - Comprobar Conectividad de Red

Desde la computadora cliente:

```cmd
ping 192.168.1.38
```

Si aparece:

```text
Respuesta desde 192.168.1.38
```

la comunicación funciona correctamente.

---

# Paso 2 - Habilitar Ping en Windows Firewall

En la computadora servidor:

```text
Windows + R
```

Escribir:

```text
wf.msc
```

Abrir:

```text
Reglas de entrada
```

Habilitar las reglas:

```text
Compartir archivos e impresoras (Solicitud de eco - ICMPv4 de entrada)
```

o

```text
File and Printer Sharing (Echo Request - ICMPv4-In)
```

Después de habilitar las reglas, volver a probar:

```cmd
ping 192.168.1.38
```

desde la computadora cliente.

---

# Paso 3 - Verificar que MySQL Escuche en la Red

En la computadora servidor ejecutar:

```bash
netstat -an | find "3306"
```

Resultado esperado:

```text
TCP    0.0.0.0:3306    0.0.0.0:0    LISTENING
```

Esto indica que MySQL acepta conexiones desde otras computadoras.

---

# Paso 4 - Abrir Puerto 3306 en Firewall

Abrir:

```text
wf.msc
```

Crear una nueva regla:

```text
Reglas de entrada
→ Nueva regla
→ Puerto
→ TCP
→ Puertos locales especificos
→ Puerto 3306
→ Permitir conexión
```

Aplicar a:

```text
Dominio
Privado
Público
```

Nombre:

```text
MySQL 3306
```

---

# Paso 5 - Crear Usuario para Conexiones Remotas

En MySQL Workbench:

```sql
CREATE USER 'appuser'@'%' IDENTIFIED BY 'adminsp';
```

Si el usuario ya existe:

```sql
ALTER USER 'appuser'@'%' IDENTIFIED BY 'adminsp';
```

Otorgar permisos:

```sql
GRANT ALL PRIVILEGES ON xumal_db.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
```

---

# Paso 6 - Verificar Usuarios

Consultar usuarios:

```sql
SELECT User, Host
FROM mysql.user;
```

Debe existir un usuario con:

```text
User: appuser
Host: %
```

El símbolo `%` permite conexiones desde cualquier equipo de la red.

---

# Paso 7 - Conectarse desde Otra Computadora

Instalar MySQL Workbench.
 * https://dev.mysql.com/downloads/installer/

Crear una nueva conexión:

```text
Connection Name: ServidorMySQL

Hostname: 192.168.1.38
Port: 3306

Username: appuser
Password: adminsp
```

Presionar:

```text
Test Connection
```

Resultado esperado:

```text
Successfully made the MySQL connection
```

---

# Paso 8 - Configuración para Aplicaciones C#

Cadena de conexión:

```csharp
Server=192.168.1.38;
Database=xumal_db;
User=appuser;
Password=adminsp;
```

Ejemplo:

```csharp
string connectionString =
"Server=192.168.1.38;Database=xumal_db;User=appuser;Password=adminsp;";
```

---

# Resultado Final

Arquitectura funcionando:

```text
Servidor MySQL (192.168.1.38)
        │
        ├── PC 1
        ├── PC 2
        ├── PC 3
        ├── PC 4
        ├── PC 5
        ├── PC 6
        ├── PC 7
        └── PC 8
```

Todas las aplicaciones comparten la misma base de datos y los cambios se reflejan en tiempo real.
