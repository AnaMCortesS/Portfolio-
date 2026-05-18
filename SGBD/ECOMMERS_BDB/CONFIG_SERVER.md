# 📊 Proyecto Olist eCommerce - Configuración del Entorno de Datos

Este repositorio contiene el esquema, las consultas de análisis y la documentación para el dataset de comercio electrónico de Olist. 

Para poder trabajar con los datos de forma eficiente, se ha desplegado una arquitectura **Cliente-Servidor** que separa el motor de almacenamiento de la interfaz de desarrollo.

## 📌 1. Requisitos del Sistema y Arquitectura

* **Servidor de Base de Datos (Back-End):** Servidor Linux (Ubuntu) con **MariaDB** (Puerto `3306`).
* **Cliente de Desarrollo (Front-End):** Sistema Operativo Windows con **DBeaver Community Edition**.
* **Red:** Ambas máquinas se encuentran en la misma red local (LAN).


## 🐧 2. Configuración del Servidor (Linux Ubuntu)

### Paso 2.1: Habilitar Escucha Remota e Idioma Global
Para permitir conexiones externas de forma segura y garantizar el soporte correcto de caracteres especiales (tildes, eñes y caracteres portugueses del dataset), se editó el archivo de configuración principal de MariaDB:



sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
[mysqld]
# Permitir que el motor escuche en todas las interfaces de red locales
bind-address            = 0.0.0.0

# Configuración del estándar UTF8MB4 para soporte de caracteres y emojis
character-set-server  = utf8mb4
collation-server      = utf8mb4_unicode_ci



## 🔒 3. Seguridad, Gestión de Usuarios y Cortafuegos

### Paso 3.1: Configuración del Firewall del Sistema (UFW)
Por defecto, el guardaespaldas interno de Ubuntu bloquea las conexiones entrantes al puerto de la base de datos. Para permitir el acceso exclusivo a la máquina de desarrollo (Windows) sin comprometer la seguridad del servidor, se añade una regla estricta en el cortafuegos:


sudo ufw allow from [IP_DE_TU_WINDOWS] to any port 3306

### Paso 3.2: Creación del Usuario en MariaDB
-- Creación del usuario especificando procedencia y plugin de contraseña nativo
CREATE USER 'root'@'[IP_DE_TU_WINDOWS]' IDENTIFIED VIA mysql_native_password BY '[TU_CONTRASEÑA]';

-- Asignación de privilegios administrativos sobre la base de datos del proyecto
GRANT ALL PRIVILEGES ON olist_ecommerce.* TO 'root'@'[IP_DE_TU_WINDOWS]' WITH GRANT OPTION;

-- Recarga de las tablas de privilegios en la memoria RAM
FLUSH PRIVILEGES;


## 🛠️ 4. Configuración del Cliente Gráfico (DBeaver en Windows)

Para enlazar la interfaz visual con el servidor Linux de forma exitosa, se optó por utilizar el conector de MySQL debido a su compatibilidad nativa y mayor flexibilidad con los parámetros de seguridad remotos de MariaDB.

### Paso 4.1: Creación de la Conexión
1. En DBeaver, hacer clic en **Nueva conexión** y seleccionar el driver de **MySQL** (icono del delfín).
2. En la pestaña **Main**, rellenar los datos de acceso:

   * **Server Host:** `[IP_DE_TU_LINUX]`
   * **Port:** `3306`
   * **Database:** `olist_ecommerce`
   * **Username:** `root` (o el usuario asignado)
   * **Password:** `[TU_CONTRASEÑA]`

### Paso 4.2: Ajuste de Propiedades Avanzadas del Driver
Para evitar bloqueos por el intercambio de llaves cifradas o la falta de certificados SSL en el entorno de desarrollo local, se modificaron los parámetros de comunicación. 

En la pestaña **Driver Properties**, se agregaron/modificaron las siguientes variables de entorno fijando su valor a través del botón `+` (Añadir propiedad) y confirmando con `Enter` y `Apply`:

* **`allowPublicKeyRetrieval`** = `true` *(Permite al cliente solicitar la clave pública del servidor para transmitir la contraseña de forma segura).*
* **`useSSL`** = `false` *(Desactiva el cifrado SSL estricto, ideal para conexiones directas dentro de la red LAN de desarrollo).*

### Paso 4.3: Verificación del Enlace
Hacer clic en el botón **Test Connection...**. DBeaver descargará los binarios del driver necesarios automáticamente. Si la configuración es correcta, el sistema devolverá un estado de éxito (`Connected`), mostrando la versión del motor MariaDB remoto. El entorno queda listo para la ejecución de scripts SQL.

```bash
