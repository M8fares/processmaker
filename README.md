# Stack Processmaker 3.1.3 con Nginx y SQL Server

Este es un stack que permite la instalación de `Processmaker 3.1.3` o anteriores, en `CentOS 7.x/RHEL` junto con `Nginx` y conexión a `SQL Server`.


Para la instalación de este stack, se debe fusionar la documentación de dos stacks de ProcessMaker.

* ([ProcessMaker Stack 205](https://wiki.processmaker.com/3.1/Stack_205)): Distribuye documentación para la instalación de `MySQL 5.5`.
* ([ProcessMaker Stack N220](https://wiki.processmaker.com/3.2/Stack_N220)): Distribuye documentación para la instalación del resto del entorno.

> El motivo por el cuál se debe realizar una fusión de estos dos stacks, es que Microsoft SQL Server no provee un soporte funcional para conexiones a SQL Server en versiones anteriores a `PHP 7.0` y ProcessMaker aún no soporta `PHP 7.0`.<br/><br/>
> La única alternativa es usar [FreeTDS](https://blog.remirepo.net/post/2016/09/20/Microsoft-SQL-Server-from-PHP). <br/><br/>
> Más información en: [Microsoft Drivers for PHP for SQL Server](https://docs.microsoft.com/en-us/sql/connect/php/system-requirements-for-the-php-sql-driver?view=sql-server-2017)

# Descripción general

Para instalar ProcessMaker, verifique si su servidor cumple con los requisitos de hardware y software necesarios, que se enumeran a continuación.

# Requisitos de Instalación

## Requisitos de Hardware

Para uso personal, ProcessMaker es compatible con cualquier computadora con una CPU moderna, una conexión a Internet y 2 GB de RAM.

Para uso de producción, los requisitos de hardware pueden variar según la cantidad de usuarios concurrentes, el tamaño del repositorio y la configuración del sistema. Las implementaciones más grandes pueden requerir algunos ajustes de configuración para funcionar de manera óptima. Para obtener más información, consulte el ([tamaño del servidor ProcessMaker](https://wiki.processmaker.com/3.2/ProcessMaker_Server_Sizing)).

Para uso de producción, instale ProcessMaker en un servidor dedicado o máquina virtual con una conexión de Internet dedicada.

## Requisitos de Software

El stack actual está adaptado para instalar [ProcessMaker 3.1.3](https://sourceforge.net/projects/processmaker/files/ProcessMaker/3.1.3/) o anteriores y posee las siguientes especificaciones:

<table>
  <tr>
    <td><b>Componente</b></td>
    <td><b>Versiones compatibles</b></td>
    <td><b>Comentarios</b></td>
  </tr>
  <tr>
    <td colspan="3">Plataforma</td>
  </tr>
  <tr>
    <td>Linux / Unix</td>
    <td>CentOS 7.x/RHEL</td>
    <td>Aunque ProcessMaker se puede instalar en cualquier plataforma que admita PHP, ProcessMaker recomienda el uso de distribuciones basadas en Red Hat, ya que los equipos de control de calidad y soporte SOLO realizan pruebas y brindan soporte para los <a href="https://wiki.processmaker.com/3.1/Supported_Stacks">stacks admitidos</a> por ellos.</td>
  </tr>
  <tr>
    <td>PHP</td>
    <td>PHP 5.5</td>
    <td>Usar PHP 5.5 para utilizar una conexión a SQL Server. <br/><b>Advertencia:</b> No se puede usar PHP 7.0 porque no es compatible con ProcessMaker.</td>
  </tr>
  <tr>
    <td colspan="3">Base de datos</td>
  </tr>
  <tr>
    <td>MySQL</td>
    <td>MySQL 5.5</td>
    <td>Usar MySQL 5.5 para utilizar una versión igual o anterior a ProcessMaker 3.2.1 . <br/><b>Advertencia:</b> El <a href="https://wiki.processmaker.com/3.0/Windows_Manual_Installation#Disabling_strict_mode">modo estricto</a> de MySQL no es compatible.<br/><b>Advertencia:</b> El <a href="https://wiki.processmaker.com/3.0/Additional_Configuration#disablingFullGroupBy">modo ONLY_FULL_GROUP_BY</a> no es compatible.</td>
  </tr>
  <tr>
    <td colspan="3">Servidor Web</td>
  </tr>
  <tr>
    <td>Nginx</td>
    <td>1.x.x (Última versión)</td>
    <td></td>
  </tr>
</table>

Se recomienda encarecidamente instalar ProcessMaker con las configuraciones compatibles de este stack donde ProcessMaker se ha probado por completo. Sin embargo, también es posible instalar `ProcessMaker 3.2.1` pero no se han realizado pruebas.

# Configuraciones del entorno del servidor

Suponiendo que `CentOS 7.x/RHEL` ya está instalado, lea las siguientes instrucciones para configurar el stack antes de instalar ProcessMaker.

## 1. Actualizar CentOS 7.x/RHEL

La recomendación estándar antes de cualquier cosa es actualizar `CentOS 7.x/RHEL` para asegurarse de que todos los paquetes estén en las últimas versiones con los parches de seguridad instalados.

Abra una terminal e ingrese los siguientes comandos:

```
yum check-update
yum update
```

> **Advertencia**: Dependiendo de su versión de `CentOS/RHEL`, la actualización del servidor tomará el tiempo necesario para completar la actualización.

## 2. Deshabilitar MariaDB

MariaDB es un reemplazo directo para MySQL instalado en `CentOS 7.x/RHEL` de forma predeterminada. Es necesario desinstalar MariaDB para evitar problemas con MySQL, que es el sistema de base de datos predeterminado para ProcessMaker.

Para desinstalar MariaDB:

```
yum -y remove mariadb*
```

![alt text](https://wiki.processmaker.com/sites/default/files/remove_mariadb.png)

## 3. Instalar MySQL 5.5

> **Advertencia**: ProcessMaker no es compatible con el ([modo MySQL STRICT](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_sql_mode)) , que está habilitado de forma predeterminada a partir de `MySQL 5.6.6`. Lea la sección ([Desactivar el modo STRICT de MySQL](https://wiki.processmaker.com/3.0/Changing_the_ProcessMaker_Configuration#Turning_Off_Strict_Mode)) para saber cómo desactivarlo.

Para instalar `MySQL 5.5`, siga los siguientes pasos:

1. Ejecute las líneas de comando para descargar los repositorios:

```
yum install -y yum-utils
yum localinstall -y https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm
```

2. Use estas líneas de comando para configurar el repositorio e instalar `MySQL 5.5`:

```
yum-config-manager --disable mysql57-community
yum-config-manager --enable mysql55-community
yum install -y mysql-community-server
```

3. Inicie el servicio MySQL y configúrelo para que se inicie automáticamente en el arranque.

```
systemctl start mysqld
systemctl enable mysqld
```

4. Asegúrese de que el servicio mysql se esté ejecutando al verificar su estado con el siguiente comando:

```
systemctl status mysql
```

El estado del servicio mysql debe ser "activo (en ejecución)":

![alt text](https://wiki.processmaker.com/sites/default/files/PM3.0CentosInstallMySQLStatus.png)

### Configuración de MySQL

Antes de usar MySQL, use el comando [mysql_secure_installation](https://dev.mysql.com/doc/refman/8.0/en/mysql-secure-installation.html) para configurar un entorno de base de datos seguro. 

Inicie sesión como usuario `root` y ejecute el siguiente comando:

```
mysql_secure_installation
```

> **Nota**: En algunos casos, la contraseña ya está definida, por lo que debe verificar cuál es antes de ejecutarla mysql_secure_installation y cambiarla; Para ello utilizar el siguiente comando:
> ```
> cat /root/.mysql_secret
> ```

Luego siga las instrucciones del asistente para asegurar MySQL como sigue:

![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68MYSQLSecure0.png)

1. Introduzca la contraseña de `root`:
![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68MYSQLSecure01.png)

2. Cambie la contraseña de `root`:
> **Advertencia**: ProcessMaker NO admite caracteres especiales (como: `@ # $ % ^ & ( /`) en la contraseña de `root`. Para más información, lea esta [sección](https://wiki.processmaker.com/3.1/Stack_205#MySQL_Password_with_Special_Characters).

![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68MYSQLSecure1.png)

3. Confirmar para eliminar usuarios anónimos.

![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68MYSQLSecure02.png)

4. Confirme para deshabilitar el inicio de sesión de `root`.

![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68MYSQLSecure03.png)

En el caso de que MySQL esté en otro servidor, debe crear un nuevo usuario y otorgarle a este usuario los permisos de acceso.

5. Confirmar para eliminar la base de datos de prueba.

![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68MYSQLSecure04.png)

6. Volver a cargar las tablas de privilegios.

![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68MYSQLSecure05.png)

La instalación de MySQL ahora es segura.

![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68MYSQLSecure06.png)

7. Reinicie el servicio mysql.

```
systemctl restart mysql
```

## 4. Instalar Nginx

Para instalar `Nginx` sigue estos pasos:

1. Agregue el archivo de repositorio `Nginx`.

```
nano /etc/yum.repos.d/nginx.repo
```

2. Agregue las siguientes líneas en el archivo del repositorio.

```
[nginx]

name=nginx repo

#####rhel/6 should be changed to rhel/7 for RHEL/CentOS 7

baseurl=http://nginx.org/packages/rhel/7/$basearch/

gpgcheck=0

enabled=1
```

3. Instalar `Nginx` e iniciar el servicio.

```
yum clean all && yum -y install nginx
systemctl start nginx
systemctl enable nginx
```

## 5. Instalar PHP 5.5

1. Para instalar `PHP 5.5`, debe instalar y habilitar el repositorio `EPEL` y `Remi` en su sistema `CentOS 7.x/RHEL` utilizando los siguientes comandos:

```
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

2. Luego, verifique la instalación de `yum-utils`.
``` 
yum install yum-utils 
```

3. Uno de los programas más importantes proporcionados por `yum-utils` es `yum-config-manager`, que puede usar para activar el repositorio `Remi` como el repositorio predeterminado para instalar varias versiones de PHP. Para instalar `PHP 5.5` en `CentOS 7.x/RHEL`, simplemente habilítelo e instálelo como se muestra.

```
yum-config-manager --enable remi-php55
```

4. Instalar PHP y sus módulos.

```
yum -y install php
yum -y install php-mysqlnd php-gd php-soap php-ldap php-xml php-mbstring php-cli php-curl php-mcrypt
yum -y install php-common php-devel php-pecl-apcu php-opcache php-fpm 
```

### Iniciar php-fpm

Ejecute los siguientes pasos para iniciar `php-fpm`:

1. Iniciar el servicio

```
systemctl start php-fpm
systemctl enable php-fpm
```

2. Establezca las configuraciones estándar de ProcessMaker.

<pre>
sed -i '/short_open_tag = Off/c\short_open_tag = On' /etc/php.ini

sed -i '/post_max_size = 8M/c\post_max_size = 24M' /etc/php.ini

sed -i '/upload_max_filesize = 2M/c\upload_max_filesize = 24M' /etc/php.ini
</pre>

3. Establezca esta configuración estándar de ProcessMaker y cambie el atributo `date.timezone` con su zona horaria, como ejemplo:

<pre>
sed -i '/;date.timezone =/c\date.timezone = <b>America/Bogota</b>' /etc/php.ini
</pre>


### Instalar y configurar OpCache

Ejecuta los siguientes pasos:

1. Establecer las preconfiguraciones de `OpCache`.

```
sed -i '/expose_php = On/c\expose_php = Off' /etc/php.ini
```

2. Asegurarse de instalar el módulo `OpCache`

```
yum -y install php-opcache
```

3. Establecer configuraciones de `OpCache`

```
sed -i '/;opcache.enable_cli=0/c\opcache.enable_cli=1' /etc/php.d/opcache.ini


sed -i '/opcache.max_accelerated_files=4000/c\opcache.max_accelerated_files=10000' /etc/php.d/opcache.ini


sed -i '/;opcache.max_wasted_percentage=5/c\opcache.max_wasted_percentage=5' /etc/php.d/opcache.ini


sed -i '/;opcache.use_cwd=1/c\opcache.use_cwd=1' /etc/php.d/opcache.ini


sed -i '/;opcache.validate_timestamps=1/c\opcache.validate_timestamps=1' /etc/php.d/opcache.ini


sed -i '/;opcache.fast_shutdown=0/c\opcache.fast_shutdown=1' /etc/php.d/opcache.ini
```

### Configurar el archivo php-fpm

Para configurar `php-fpm` sigue estos pasos:

1. Abra el archivo de configuración `php-fpm`.

```
nano /etc/php-fpm.d/processmaker.conf
```

2. Incluya lo siguiente en el archivo de configuración:

```
[processmaker]

user = nginx

group = nginx

listen = /var/run/php-fpm/processmaker.sock

listen.mode = 0664

listen.owner = nginx

listen.group = nginx

pm = dynamic

pm.max_children = 100 

pm.start_servers = 20

pm.min_spare_servers = 20

pm.max_spare_servers = 50

pm.max_requests = 500

php_admin_value[error_log] = /var/log/php-fpm/processmaker-error.log

php_admin_flag[log_errors] = on
```

3. Haz una copia de seguridad de la configuración por defecto de `Nginx` para trabajar con ProcessMaker.

```
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bk
```

4. Crea un nuevo archivo

```
nano /etc/nginx/nginx.conf
```

5. El archivo de configuración del servidor `Nginx` debe tener:

```
user nginx;

worker_processes auto;

error_log /var/log/nginx/error.log warn;

pid /var/run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
  worker_connections 1024;
}

http {
  include  /etc/nginx/mime.types;
  
  default_type  application/octet-stream;

  log_format     main '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

  access_log  /var/log/nginx/access.log  main;

  log_format combined_ssl '$remote_addr - $remote_user [$time_local] '
                          '$ssl_protocol/$ssl_cipher '
                          '"$request" $status $body_bytes_sent '
                          '"$http_referer" "$http_user_agent"';

  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 120;
  keepalive_requests 100;
  types_hash_max_size 2048;


  # Enable Compression
  gzip on;
  gzip_disable "msie6";
  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 6;
  gzip_buffers 16 8k;
  gzip_http_version 1.1;
  gzip_types text/css text/plain text/xml text/x-component text/javascript application/x-javascript application/javascript application/json application/xml application/xhtml+xml application/x-font-ttf application/x-font-opentype application/x-font-truetype image/svg+xml image/x-icon image/vnd.microsoft.icon font/ttf font/eot font/otf font/opentype;

  include /etc/nginx/conf.d/*.conf;

  # Comment out ServerTokens OS
  server_tokens off;


  # Prevent ClickJacking Attacks
  add_header X-Frame-Options SAMEORIGIN;

  # Load Balancer/Reverse Proxy Header
  real_ip_header X-Forwarded-For;
  set_real_ip_from 0.0.0.0/0;
}
```

6. Reiniciar `Nginx`

```
systemctl restart nginx
```

## 6. Instalar FreeTDS y UnixODBC

La recomendación estándar antes de cualquier cosa es actualizar `CentOS 7.x/RHEL` para asegurarse de que todos los paquetes estén en las últimas versiones con los parches de seguridad instalados.

Abra una terminal e ingrese los siguientes comandos:

```
yum check-update
yum update
```

> **Advertencia**: Dependiendo de su versión de `CentOS/RHEL`, la actualización del servidor tomará el tiempo necesario para completar la actualización.

Ahora es el turno de instalar los módulos de `PHP` y `UnixODBC`.

```
yum -y install php-odbc php-pdo php-mssql unixODBC unixODBC-devel
```

### Instalar y configurar FreeTDS

[FreeTDS](http://www.freetds.org/) es una implementación de código abierto del protocolo TDS (Tabular Data Stream) que utiliza SQL Server y Sybase, y permite a los hosts Unix / Linux capaz de conectarse a estas bases de datos.

`CentOS 7.x/RHEL` no trae un paquete listo para FreeTDS de instalación, pero el [repositorio EPEL (paquetes adicionales para Enterprise Linux)](https://fedoraproject.org/wiki/EPEL) tiene una lista de paquetes para su uso. 

1. Ahora, debe configurar `CentOS 7.x/RHEL` para usar el repositorio `EPEL` e instalar `FreeTDS`:

```
yum install epel-release
yum check-update
yum install freetds freetds-devel
```

2. El archivo de configuración de FreeTDS es `/etc/freetds.conf`. Edite este archivo para que sea similar al siguiente:

<pre>
[global]
 # Versión global del protocolo TDS
 #tds version = 4.2
 tds version = 5.0

 # Almacenar dump (TDSDUMP file) para debug?
 dump file = /tmp/freetds.log
 debug flags = 0xffff

 # Timeouts
 timeout = 10
 connect timeout = 10

 # Tamaño de buffer para campo TEXT para evitar
 # errores de tipo out-of-memory errors
 text size = 64512

[<b>AQUI SU NOMBRE DE SERVIDOR DE BASE DE DATOS</b>]
 host = <b>AQUI LA IP DE SU SERVIDOR</b>
 port = <b>AQUI EL PUERTO PARA RECIBIR CONEXIÓN A SQL SERVER</b>
 tds version = 7.0
</pre>

3. Ahora tenemos que probar si la conexión a SQL Server, vía `FreeTDS`, para probar si está funcionando correctamente. Para ello utilizamos la utilidad `tsql` de `FreeTDS`.

<pre>
tsql -S <b>AQUI SU NOMBRE DE SERVIDOR DE BASE DE DATOS</b> -U <b>usuario</b> -P <b>contraseña</b>
locale is "en_US.UTF-8"
locale charset is "UTF-8"
using default charset "UTF-8"
1> quit
</pre>

> **Nota**: Si ha llegado al indicador `1>`, como se muestra arriba, puede ejecutar `quit` y estar lleno de felicidad porque la conexión `FreeTDS` con `SQL Server` está funcionando correctamente.

### Configurar el UnixODBC

Hasta el momento ya tenemos `FreeTDS` instalado, configurado y accediendo a la bases de datos `SQL Server`.

Ahora es el momento de configurar el `UnixODBC`.

1. Editar el `/etc/odbcinst.ini` y añadir el siguiente contenido:

```
[ODBC]
Trace = No
TraceFile = /tmp/sql.log
ForceTrace = No

[FreeTDS]
Driver = /usr/lib64/libtdsodbc.so.0
FileUsage = 1
```

En el archivo anterior estamos diciendo que el `UnixODBC` debe utilizar el controlador FreeTDS (`/usr/lib64/libtdsodbc.so.0`) para las conexiones. Si queremos, para depurar las conexiones problemáticas, podemos habilitar `TraceFile` cambiando de `No` a `Yes` (en sistemas de producción, dejar como `No`).

2. Ahora se debe crear o editar el archivo `/etc/odbc.ini` y añadir el siguiente contenido:

<pre>
# Nombre de origen de datos (DSN) para MSSQL Server:
[<b>DSNAlias</b>]
Description = Conexión a SQL Server 2008 R2
Driver = FreeTDS
Trace = No
Server = <b>AQUI LA IP DE SU SERVIDOR</b>
# Database = <b>AQUI EL NOMBRE DE SU BASE DE DATOS</b> (opcional)
Port = <b>AQUI EL PUERTO PARA RECIBIR CONEXIÓN A SQL SERVER</b>
</pre>

Explicando el archivo anterior para SQL Server:

- **DSNAlias** = Es un alias, el nombre del DSN. Puede ser cualquier cosa y se utilizará en las llamadas de conexión a SQL Server.
- **Server** = IP del servidor de SQL Server.
- **Port** = Puerto de SQL Server para recibir conexiones.
- > (**Opcional**): Database = Nombre de la base de datos a la que desea conectarse.

3. Ahora vamos a probar las conexiones a las bases de datos vía `UnixODBC`, a través de la utilidad `isql` de `UnixODBC`.

- Prueba de conexión a SQL Server mediante el DSN `DSNAlias`:

<pre>
isql <b>DSNAlias usuario contraseña </b> -v
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> select count(*) from <b>tabla</b>;
+------------+
|            |
+------------+
| 2094       |
+------------+
SQLRowCount returns 1
1 rows fetched
SQL> quit
</pre>

Ahora todo está funcionando. Nos conectamos con el DSN denominado `DSNAlias` (que el archivo `/etc/odbc.ini` apunta a un SQL Server) con un nombre de usuario y contraseña. La conexión tuvo éxito y hicimos un select simple de una tabla cualquiera que tenía 2094 líneas. Como todo funciona, dimos el `quit` para salir.

4. Luego se necesita habilitar la extensión `mssql.so` en `/etc/php.ini`, para ello ejecutaremos el siguiente comando:

```
echo "extension=mssql.so" > /etc/php.ini
```

## 7. Instale un Firewall y abra el puerto para ProcessMaker

Por defecto, `CentOS 7.x/RHEL` no puede funcionar sin firewall, por lo tanto, se recomienda instalar `Firewalld` para que pueda configurarse fácilmente. `Firewalld` es un servicio dinámico que administra un firewall con soporte para zonas de red. Para instalarlo ejecuta los siguientes pasos:

1. Instala `Firewalld`

```
yum -y install firewalld
```

2. Configura el servicio para que se inicie automáticamente.

```
systemctl start firewalld
systemctl enable firewalld
```

3. Abra el puerto donde se ejecutará ProcessMaker, que es el puerto 80 de manera predeterminada o el puerto 443. Para usar un puerto que no sea el puerto 80 o 443, es necesario cambiar el número de puerto con el siguiente comando.

<pre>
firewall-cmd --zone=public --add-port=<b>80</b>/tcp --permanent
firewall-cmd --zone=public --add-port=<b>443</b>/tcp --permanent
firewall-cmd --reload
</pre>

# Configuración e instalación de ProcessMaker

## 1. Descargar ProcessMaker

* [Community Edition](#down-com-edition)
* [Standard, Corporate or Enterprise Editions](#down-s-c-e-editions)

### <a name="down-com-edition">Community Edition</a>

Vaya a [la página de SourceForge de ProcessMaker](http://sourceforge.net/projects/processmaker/files/ProcessMaker/) y descargue el archivo más reciente de ProcessMaker, que debe llamarse `processmaker-X.X.X-community.tar.gz`.

Alternativamente, descargue el `tar.gz` archivo con `wget`.

```
wget 'https://sourceforge.net/projects/processmaker/files/ProcessMaker/3.x.x/processmaker-3.x.x-community.tar.gz'
```

Reemplace la "`x`" con la versión que desea utilizar.

### <a name="down-s-c-e-editions">Standard, Corporate or Enterprise Editions</a>

No es posible obtener la edición Standard, Corporate o Enterprise hasta que compre uno de los [Planes de suscripción Enterprise](https://www.processmaker.com/products/pricing).

Después de completar el proceso de compra de su Plan de Suscripción Empresarial, recibirá un correo electrónico que incluye enlaces donde puede descargar la versión empresarial y la licencia correspondiente.

## 2. Extraer ProcessMaker

Una vez finalizada la descarga, descomprima el archivo comprimido en el directorio donde se instalará ProcessMaker. ProcessMaker se puede instalar en cualquier directorio que no sea de acceso público desde Internet (así que NO lo instale en `/var/www`). ProcessMaker generalmente se instala en `/opt`, ya que es un programa opcional que no proviene de repositorios estándar. Realice una de las siguientes acciones según la edición de ProcessMaker que tenga:

### Community Edition

<pre>
tar -C <b>/opt</b> -xzvf processmaker-X.X.X-community.tar.gz 
</pre>

### Standard, Corporate or Enterprise editions

<pre>
tar -C <b>/opt</b> -xzvf processmaker-X.X.X.tar.gz 
</pre>

Verifique que ProcessMaker se haya descomprimido correctamente:

<pre>
ls <b>/opt/processmaker</b>
</pre>

El directorio de `/opt/processmaker` debe ser similar al siguiente contenido:

![alt text](https://wiki.processmaker.com/sites/default/files/3.1Centos68PMContained.png)

### Establecer permisos de archivo

Emita los siguientes comandos como usuario `root` para que ProcessMaker pueda acceder a los archivos necesarios en `CentOS 7.x/RHEL`.

El servicio `Nginx` se ejecuta como el usuario `nginx` de forma predeterminada y de forma independiente.

Por lo tanto, el servicio web debe ser propietario del directorio ProcessMaker para que el servicio web pueda leer y escribir los datos. El `-R` hace que los cambios de propiedad sean recursivos (se aplican a todos los archivos y directorios dentro de `/opt/processmaker`).

Para hacer recursivos los cambios de propiedad en `Nginx`, use el siguiente comando:

<pre>
chown -R nginx:nginx <b>/opt/processmaker</b>
</pre>

Después de estos cambios, verifique los permisos y el propietario del directorio de processmaker con el comando `ls -l`. A continuación se muestra un ejemplo del servidor web `Nginx`.

![alt text](https://i.imgur.com/lFKQIrA.png)

## 3. Configuración de Nginx

Cree un archivo de configuración dentro de `/etc/nginx/conf.d/` con el comando:

```
nano /etc/nginx/conf.d/processmaker.conf
```

Para configurar este archivo realice uno de los siguientes pasos:

* [VirtualHost en Nginx con SSL](#virtualhost-nginx-with-ssl)
* [VirtualHost en Nginx sin SSL](#virtualhost-nginx-without-ssl)

### <a name="virtualhost-nginx-with-ssl">VirtualHost en Nginx con SSL</a>

Si tiene una certificación SSL, siga estos pasos para realizar la configuración de NGINX:

1. En el archivo creado `processmaker.conf`, el archivo de configuración debe ser el siguiente:

<pre>
# Force HTTPS redirection
server {
  listen 80;
  listen [::]:80;
  # User server DNS name
  server_name <b>nginx.processmaker.com</b>;
  return 301 <b>https://nginx.processmaker.com</b>$request_uri;
}

# ProcessMaker HTTPS Virtual Host
server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;

  server_name <b>nginx.processmaker.com</b>;
         
  # Should point to where you SSL certificates are located
  ssl_certificate /etc/ssl/certs/processmaker/cert-n.pem;
  ssl_certificate_key /etc/ssl/certs/processmaker/key.pem;

  # SSL Session resumption
  ssl_session_cache shared:SSL:50m;
  ssl_session_timeout 1d;
  ssl_session_tickets off;
  
  ssl_prefer_server_ciphers on;
  
  # Only allow TLS v1.2, if you plan to support older browsers or mobile devices, this setting should be changed.
  ssl_protocols TLSv1.2;
  # Recommended SSL ciphers allowed
  ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:!DSS';
   
  # Enable HSTS
  add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
 
  # Prevent ClickJacking, XSS attacks
  add_header X-Frame-Options SAMEORIGIN;
  add_header X-Content-Type-Options nosniff;
  add_header X-XSS-Protection "1; mode=block";

  # Include phpMyAdmin configuration file
  include /opt/phpMyAdmin/phpMyAdmin.conf;

  root <b>/opt/processmaker</b>/workflow/public_html; #where ProcessMaker is installed
  index index.html index.htm app.php index.php;
  try_files $uri $uri/ /index.php?$args;
  charset utf-8;

  location / {
    try_files $uri $uri/ /index.php?$query_string;
  }

  location = /favicon.ico { access_log off; log_not_found off; }
  location = /robots.txt  { access_log off; log_not_found off; }
 
  access_log /var/log/nginx/pm-ssl-access.log combined_ssl; #enables access logs
  error_log  /var/log/nginx/pm-ssl-error.log error; #enables error logs

  sendfile off;
  client_max_body_size 100m;
        
  # Every PHP script must be routed to PHP-FPM
  location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass unix:/var/run/php-fpm/processmaker.sock;
    fastcgi_index    app.php;
    include fastcgi_params;
    fastcgi_param    SCRIPT_FILENAME  <b>/opt/processmaker</b>/workflow/public_html/app.php;
    fastcgi_intercept_errors off;

    fastcgi_buffers 8 16k;
    fastcgi_buffer_size 32k;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;     
  }
         
  # Leverage Browser Caching
  location ~* \.(ico|css|html|js|gif|jpeg|jpg|png|woff|ttf|otf|svg|woff2|eot)$ {
    expires 24h;
    add_header Cache-Control public;
    access_log off;
    log_not_found off;

    fastcgi_pass unix:/var/run/php-fpm/processmaker.sock;
    fastcgi_index    app.php;
    include fastcgi_params;
    fastcgi_param    SCRIPT_FILENAME  <b>/opt/processmaker</b>/workflow/public_html/app.php;
    fastcgi_intercept_errors off;

    fastcgi_buffers 8 16k;
    fastcgi_buffer_size 32k;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
  }
         
  location ~ /\.ht {
    deny all;
  }
 
  error_page 404 /404.html;
  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;     
  }                    
}
</pre>

2. En el mismo archivo, cambie el atributo `server_name` con la IP de su servidor como ejemplo:

<pre>
server_name <b>192.168.1.100</b>;
</pre>

3. En dando caso que haya hecho la instalación de ProcessMaker en otro directorio, asegurese de cambiar los directorios marcados en **negrita**.

### <a name="virtualhost-nginx-without-ssl">VirtualHost en Nginx sin SSL</a>

Si el servidor no tiene una certificación SSL, se debe utilizar la siguiente configuración:

1. En el archivo creado `processmaker.conf`, el archivo de configuración debe ser el siguiente:

<pre>
# ProcessMaker HTTP Virtual Host
server {
  listen 80;
  listen [::]:80;
  # Change for server DNS name
  server_name <b>nginx.processmaker.com</b>;
  # The following line must be added Only if phpMyAdmin is configured
  # include /opt/phpMyAdmin/phpMyAdmin.conf;
 
  root <b>/opt/processmaker</b>/workflow/public_html; #where ProcessMaker is installed
 
  index index.html index.htm app.php index.php;
 
  try_files $uri $uri/ /index.php?$args;
 
  charset utf-8;
 
  location / {
    try_files $uri $uri/ /index.php?$query_string;
  }
 
  location = /favicon.ico { access_log off; log_not_found off; }
 
  location = /robots.txt  { access_log off; log_not_found off; }
 
  access_log /var/log/nginx/pm-access.log combined; #enables access logs
 
  error_log  /var/log/nginx/pm-error.log error; #enables error logs
 
  sendfile off;
 
  client_max_body_size 100m;
 
  # Every PHP script must be routed to PHP-FPM
  location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass unix:/var/run/php-fpm/processmaker.sock;
    fastcgi_index    app.php;
 
    include fastcgi_params;
    fastcgi_param    SCRIPT_FILENAME  <b>/opt/processmaker</b>/workflow/public_html/app.php;
    fastcgi_intercept_errors off;
    fastcgi_buffers 8 16k;
    fastcgi_buffer_size 32k;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
  }
 
  # Browser Caching
  location ~* \.(ico|css|js|gif|jpeg|jpg|png|woff|ttf|otf|svg|woff2|eot)$ {
    expires 24h;
    add_header Cache-Control public;
    access_log off;
    log_not_found off;
 
    fastcgi_pass unix:/var/run/php-fpm/processmaker.sock;
    fastcgi_index    app.php;
 
    include fastcgi_params;
    fastcgi_param    SCRIPT_FILENAME  <b>/opt/processmaker</b>/workflow/public_html/app.php;
    fastcgi_intercept_errors off;
    fastcgi_buffers 8 16k;
    fastcgi_buffer_size 32k;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
  }
  
  location ~ /\.ht {
    deny all;
  }
 
  error_page 404 /404.html;
  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }
}
</pre>

2. En el mismo archivo, cambie el atributo `server_name` con la IP de su servidor como ejemplo:

<pre>
server_name <b>192.168.1.100</b>;
</pre>

3. En dando caso que haya hecho la instalación de ProcessMaker en otro directorio, asegurese de cambiar los directorios marcados en **negrita**.

## 4. Configuraciones SELinux

Independientemente de la aplicación de servidor web que use su stack ProcessMaker, si ProcessMaker está instalado en el directorio `/opt`, es necesario configurar `SELinux` para permitir que el servidor web pueda leer y escribir el directorio donde está instalado ProcessMaker. 

Alternativamente, puede desactivar `SELinux` temporalmente o permanentemente. Consulte las secciones a continuación que describen cómo realizar cada una de estas opciones:

- [Configuración de SELinux](#selinux-conf)
- [Deshabilitar SELinux temporalmente](#selinux-disable-temporarily)
- [Deshabilitar SELinux permanentemente](#selinux-disable-permanently)

### <a name="selinux-conf">Configuración de SELinux</a>

Para configurar `SELinux` para que el servidor web pueda leer y escribir en el directorio `/opt/processmaker`, siga estos pasos:

1. Verificar la instalación de `policycoreutils-python`

```
yum install policycoreutils-python
```

2. Inicie sesión como usuario `root` y emite los siguientes comandos desde el terminal:

> En la documentación de ProcessMaker, exponen el contexto `httpd_sys_content_rw_t` el cuál no existe en `CentOS 7.x/RHEL`. El contexto adaptado para la plataforma es el siguiente `httpd_sys_rw_content_t`.

<pre>
semanage fcontext -a -t <b>httpd_sys_rw_content_t</b> '/opt/processmaker(/.*)?'
restorecon -R -v <b>/opt/processmaker</b>
</pre>

En caso de que haya problemas al ejecutar el comando `semanage` se deben revisar los `locales` del sistema. Ejemplo:

```
export LC_ALL="es_CO.utf8"
```

Luego de ejecutar este comando, volver a repetir el **paso 2**.

3. Configure el servidor web para enviar correos electrónicos:

```
setsebool -P httpd_can_sendmail 1
```

4. Para ejecutar ProcessMaker en cualquier puerto distinto de los puertos predeterminados de 80, 443, 488, 8008, 8009 y 8443, se debe configurar SELinux para permitir que se use otro puerto. Por ejemplo, para usar el puerto 8080:

```
semanage port -a -t http_port_t -p tcp 8080
```

### <a name="selinux-disable-temporarily">Deshabilitar SELinux temporalmente</a>

Para deshabilitar SELinux temporalmente y depurar un problema, inicie sesión como usuario `root` y ejecute el comando:

```
setenforce 0
```

Si necesita volver, solo tiene que ejecutar el siguiente comando o reiniciar el servidor:

```
setenforce 1
```

> Los cambios se llevarán a cabo de inmediato.

### <a name="selinux-disable-permanently">Deshabilitar SELinux permanentemente</a>

Deshabilitar SELinux causa tantos problemas que a menudo es más fácil deshabilitarlo. Siga estos pasos para deshabilitar SELinux:

1. Ejecute los siguientes comandos para deshabilitar SELinux:

```
echo "SELINUX=disabled" > /etc/selinux/config 
echo "SELINUXTYPE=targeted" >> /etc/selinux/config
```

2. NO se olvide de reiniciar el servidor para desactivar SELinux de forma permanente.

## 5. Reinicia el servidor

Después de todas estas instalaciones, el servidor debe ser reiniciado.

## 6. Instalar ProcessMaker

Una vez completadas todas las configuraciones del stack, abra un navegador web e ingrese la dirección IP (y el número de puerto si no usa el puerto 80 predeterminado) donde se instalará ProcessMaker. Por ejemplo, si ProcessMaker debe instalarse en la dirección `192.168.10.100`, vaya a:

```
http://192.168.10.100
```

Si está instalado localmente en el puerto 8080, vaya a:


```
http://127.0.0.1:8080
```

Luego, en el navegador web, use el asistente de instalación para completar la instalación de ProcessMaker.

# Recursos

- **Gist for Stack Installation**: [Gist Stack Installation](**install-processmaker.sh**: ([Bash Gist]())
- **Stack 205**: [ProcessMaker Stack 205](https://wiki.processmaker.com/3.1/Stack_205)
- **Stack N220**: [ProcessMaker Stack N220](https://wiki.processmaker.com/3.2/Stack_N220)
- **PHP 5.5**: [PHP 5.5 Instalación](https://www.tecmint.com/install-php-5-6-on-centos-7/)
- **SQL Server y Centos 7.x**: [Microsoft SQL Server, PHP & CentOS 7.x](https://blog.remirepo.net/post/2016/09/20/Microsoft-SQL-Server-from-PHP)
- **Freetds**: [Página oficial](http://www.freetds.org/) - [Instalación en CentOS 7.x](http://fdta.com.br/2017/03/19/como-usar-o-freetds-e-o-unixodbc-em-um-centos-7-com-php-5-4-para-acessar-bancos-de-dados-sql-server-e-sybase-asa/)
- **SEManage**: [SEManage command not found](https://www.ostechnix.com/linux-troubleshooting-semanage-command-not-found-in-centos-7rhel-7/)
- **Politicas SELinux**: [Configurando politicas SELinux](https://www.serverlab.ca/tutorials/linux/web-servers-linux/configuring-selinux-policies-for-apache-web-servers/)

Mucha más documentación puedes encontrarla en [ProcessMaker Wiki](https://wiki.processmaker.com/)

