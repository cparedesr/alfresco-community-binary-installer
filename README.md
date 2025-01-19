# Alfresco Community Binary Installer
El repositorio tiene como objetivo proporcionar un instalador práctico y sencillo basado en la documentación oficial de Alfresco. Su propósito es automatizar una instalación basada en la distribución ZIP, empaquetándola en un instalador binario.

Este instalador es compatible exclusivamente con distribuciones de Linux que utilicen la arquitectura AMD64 (x86_64).

A continuación, se detallan las instrucciones, requisitos y observaciones del instalador.

## Requisitos previos
Para el correcto funcionamiento del instalador, es necesario que las siguientes aplicaciones estén previamente instaladas en el sistema:

    unzip
    zip
    tar
    openssl

Nota sobre la base de datos:
El instalador no despliega una base de datos (BBDD), pero está configurado para utilizar PostgreSQL. Por lo tanto, es indispensable contar con una instancia completamente operativa de PostgreSQL.

Si se desea utilizar una base de datos diferente compatible con la versión Community, será necesario reemplazar el controlador de la base de datos y modificar el fichero alfresco-global.properties. Para ello, se debe eliminar el controlador actual y colocar el correspondiente en la siguiente ruta:

    ${INSTALL_DIR}/tomcat/shared/lib

## Instalación
### Librerías de terceros:
- Imagemacik: Versión 7.1.0-16.
- Libreoffice: Versión 7.2.5.
- Para la descarga de estos componentes se puede buscar en la siguiente URL https://nexus.alfresco.com/nexus/, buscar la versión y descargar, acorde al sistema operativo, las librerías de SO que se necesiten  .
- URL de la documentación de Alfresco Transform Services: https://docs.alfresco.com/transform-service/latest/install/.

### Instalación de Alfresco Content Services Community Edition
1. Selección de la ruta de instalación
Durante el proceso, se solicitará especificar la ruta de instalación. Por defecto, el sistema realizará la instalación en /opt/alfresco.

2. ¿Instalar Java?
    - Sí: Java será desplegado en la ruta de instalación seleccionada, y las demás aplicaciones lo configurarán automáticamente.
    - No: No se desplegará Java, y las aplicaciones no lo configurarán de manera automática. Sin embargo, se utilizará la variable JAVA_HOME del sistema operativo si está definida.
3. ¿Instalar ActiveMQ?
    - Sí: ActiveMQ será desplegado en el sistema.
    - No: ActiveMQ no será desplegado.
4. ¿Instalar Alfresco Search Services (ASS)?
    - Sí: Alfresco Search Services será desplegado. Es importante tener en cuenta que se deberá modificar el archivo alfresco-global.properties, ya sea porque no se utilicen los servicios de búsqueda o porque estos estén externalizados en otra máquina.
    - No: Alfresco Search Services no será desplegado.
5. ¿Instalar Alfresco Content Services Community Edition (ACS)?
    - Sí: Se desplegará Alfresco Content Services Community Edition.
    - No: No se desplegará esta edición.
6. ¿Instalar Alfresco Transform Core AIO?
    - Sí: Se desplegará Alfresco Transform Core AIO.
    - No: No se desplegará Alfresco Transform Core AIO.
7. ¿Configurar una ruta personalizada para los logs de Alfresco?
De forma predeterminada, los archivos de logs de Alfresco (`alfresco.log` y `share.log`) se generan en la ubicación en la que se ejecuta el script de arranque de Tomcat o donde se encuentre el terminal (prompt) desde el que se lanza el proceso. Este comportamiento puede variar en función de cómo se inicie el servicio y puede ser confuso en implementaciones complejas.
    - Sí: Se modificará dentro de los archivos WAR (alfresco y share) la configuración para apuntar los logs a una ruta personalizada.
    - No: Se mantendrá la configuración estándar, donde los logs se generarán en las rutas predeterminadas de Tomcat, modificando tanto el war de `alfresco` y `share`.

Nota:
Se ha intentado configurar una ruta personalizada a través de un archivo custom-log4j2.properties sin éxito. En contenedores Docker, se aborda este problema configurando una ruta personalizada directamente en los WARS. También se ha revisado la instalación mediante Ansible, donde el script de arranque de Tomcat primero cambia a la carpeta de logs y desde ahí inicia el servidor. Sin embargo, esta práctica no es recomendable.

Si algún miembro de la comunidad desea contribuir con una solución más adecuada, sería bienvenido. 😊

### Instalación de alfresco-share-services
El proceso de despliegue en la ruta de instalación es muy similar al de Alfresco 5.2.X o versiones anteriores, por lo que no es necesario modificar el script apply_amps.sh.
```
# Se asignan permisos de ejecución al script
chmod +x $INSTALL_DIR/bin/apply_amps.sh
# Se ejecuta el script
$INSTALL_DIR/bin/apply_amps.sh
# Se presiona Enter hasta que se regrese al prompt.
```

### Administración
El instalador despliega un script denominado alfresco.sh, a través del cual se deben administrar las aplicaciones que se hayan instalado.
La ubicación del script es la siguiente:

    $INSTALL_DIR/alfresco.sh

Desde este script es posible gestionar los diferentes servicios desplegados mediante los siguientes comandos:

    alfresco.sh start: Inicia todos los servicios.
    alfresco.sh stop: Detiene todos los servicios.
    alfresco.sh start activemq: Inicia únicamente el servicio de ActiveMQ.
    alfresco.sh start solr: Inicia únicamente el servicio de Solr.
    alfresco.sh start tomcat: Inicia únicamente los servicios de Alfresco y Share.
    alfresco.sh start ats: Inicia únicamente el servicio de Transform Core AIO.
    alfresco.sh stop activemq: Detiene únicamente el servicio de ActiveMQ.
    alfresco.sh stop solr: Detiene únicamente el servicio de Solr.
    alfresco.sh stop tomcat: Detiene únicamente los servicios de Alfresco y Share.
    alfresco.sh stop ats: Detiene únicamente el servicio de Transform Core AIO

El instalador también incluye un servicio que puede configurarse a nivel del sistema utilizando systemctl. Este servicio los podemos encontrar en la carpeta service de este proyecto.

### Información Adicional
En todas las aplicaciones se realiza una instalación estándar. Esto significa que se utilizan los scripts y archivos predeterminados de cada aplicación, de manera que cualquier administrador de sistemas pueda comprender y gestionar la configuración sin dificultad.

A continuación, se detallan los archivos más importantes de cada aplicación:

ActiveMQ:

    $INSTALL_DIR/activemq/conf/jetty.xml
    $INSTALL_DIR/activemq/conf/log4j2.properties
    $INSTALL_DIR/activemq/bin/env

Solr:

    $INSTALL_DIR/solr/solr.in.sh
    $INSTALL_DIR/solr/solrhome/conf/shared.properties
    $INSTALL_DIR/solr/solrhome/templates/rerank/conf/solrcore.properties

Alfresco Community Edition:

    $INSTALL_DIR/tomcat/bin/setenv.sh
    $INSTALL_DIR/tomcat/conf/server.xml
    $INSTALL_DIR/tomcat/shared/classes/alfresco-global.properties

Alfresco Transform Core AIO:

    $INSTALL_DIR/ats/transform-service.sh

### Bibliografía
https://nexus.alfresco.com/nexus/
https://docs.alfresco.com/content-services/community/install/zip/
https://docs.alfresco.com/search-services/latest/
https://docs.alfresco.com/transform-service/latest/install/