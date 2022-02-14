# Práctica ApacheServer WEB

#### Contenedor Apache_Server
###### Creamos el contenedor apache_server
###### Empleamos la imagen httpd:latest
###### La red creada será nt01 con ip estática 10.0.0.10
###### Mapeo de puertos en el puerto 80, tanto en la máquina como en el contenedor.
###### Ruta del volumen a emplear /usr/local/apache2
```
version: "3.8"
services:
  apache:
    container_name: apache_server
    image: httpd:latest
    networks:
      nt01:
        ipv4_address: 10.0.0.10
    ports:
      - 80:80
    volumes:
      - apache2:/usr/local/apache2
      
 ```
 #### Contenedor DNS bind
 ###### Creación de contenedor bind9_server
 ###### Imagen a emplear : internetsystemsconsortium/bind9:9.16
 ###### La red creada empleada será nt01 con ip estática 10.0.0.254
 ###### Mapeo de puertos en el puerto 53, tanto en la máquina como en el contenedor.
 ###### Ruta del volumen a emplear /etc/bind
 ```
  bind9:
    container_name: bind9_server
    image: internetsystemsconsortium/bind9:9.16
    networks:
      nt01:
        ipv4_address: 10.0.0.254
    ports:
      - 53:53
    volumes:
      - bind:/etc/bind
      
  ```
  #### Contenedor Cliente
  ###### Creación de contenedor apache_cliente
  ###### Imagen a emplear : kasmweb/desktop-deluxe:1.9.0-rolling
  ###### La red creada empleada será nt01 con ip estática 10.0.0.2
  ###### Mapeo de puertos en el puerto 6901, tanto en la máquina como en el contenedor.
  ###### Referencia al contenedor dns.
  ###### Parámetro que nos permite cambiar la contraseña por defecto por la introducida abc123.
  ```
  cliente:
    container_name: apache_cliente
    image: kasmweb/desktop-deluxe:1.9.0-rolling
    networks:
      nt01:
        ipv4_address: 10.0.0.2
    ports:
      - 6901:6901
    dns:
      - 10.0.0.254  # contenedor de dns server
    environment:
      - VNC_PW=abc123. # Parámetro creación de la contraseña
 ```
 #### Creación de volúmenes
 ###### external: true con esta opción no se nos crean los volumenes en el caso de existir
 ###### networks: nos crea la red en los contenedores
 ###### ipam: Administrador de direcciones IP
 ###### subnet: 10.0.0.0/24 red interna de los volúmenes
 ```
volumes:
  bind:
    external: true
  apache2:
    external: true
networks:
  nt01:
    ipam: 
      config:
        - subnet: 10.0.0.0/24

```

 #### Httpd.conf apache_server volume
 ###### Incluír los nuevos vhosts

```

# Virtual hosts
Include conf/extra/httpd-vhosts.conf #descomentar para que funcion VHOST

```

#### Httpd-vhosts.conf apache_server volume
###### Añadimos los nuevos VirtualHosts.

```
<VirtualHost *:80>
    
    DocumentRoot "/usr/local/apache2/htdocs/index1.html"
    ServerName paxina1.example.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>

<VirtualHost *:80>
    
    DocumentRoot "/usr/local/apache2/htdocs/index2.html"
    ServerName paxina2.example.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>

```

#### Configuramos los dns en el bind
###### Creamos la zona, forwarders y el CNAME

#### DNS
```
;
; BIND data file for example.com
;
$TTL	604800
@	IN	SOA	example.com root.example.com. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	ns.example.com.
@	IN	A	10.0.0.10
@	IN	AAAA	::1
ns  IN  A   10.0.0.254
example.com. IN  A   10.0.0.10
paxina1 IN  CNAME   example.com.
paxina2 IN  CNAME   example.com.

```
#### ZONA

```
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};

```
#### FORWARD

```
forwarders {
	8.8.8.8; //google.com
	8.8.4.4; //google.com
	};
```

#### Crear dos Virtualhosts que securizar
###### Añadimos los nuevos VirtualHosts.

```
<VirtualHost *:80>
    
    DocumentRoot "/usr/local/apache2/htdocs/index3.html"
    ServerName paxina3.example.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>

<VirtualHost *:80>
    
    DocumentRoot "/usr/local/apache2/htdocs/index4.html"
    ServerName paxina4.example.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>

```

#### Generar certificados digitales
###### Generamos certificados digitales para posteriormente aplicar securización ssl-hsts

```
apt get install openssl-server

openssl genrsa -out nombre.key 2048

openssl req -new -key nombre.key -out nombre.csr

```
###### Introducimos los datos necesarios y generamos el certificado. Una vez realizado el siguiente comando, podremos importarlo.
```
openssl x509 -req -days 365 -in nombre.csr -signkey nombre.key -out nombre.crt

```

#### Securización ssl-hsts
###### Lo que realizaremos será securizar empleando el puerto 443 y hsts que nos permite obligar a los navegadores a usar comunicaciones https

```
<VirtualHost *:443>

```
###### Ponemos a la escucha el puerto 443 y añadimos los VirtualHost creados. Una vez realizado reiniciamos el servidor.
```
nano /etc/apache2/ports.conf
```
```
Listen 443
NameVirtualHost www.paxina3.example.com
NameVirtualHost www.paxina4.example.com

```

#### Generar certificados ssl
###### Generamos certificado ssl y lo añadimos.

```
openssl req -x509 -newkey rsa:2048 -keyout certificado.key -out certificado.pem -nodes -days 365
```

###### Añadimos el certificado ssl en los sites de cada web. Posteriormente reiniciamos el servidor apache2


```
SSLEngine ON
SSLCertificateKeyFile /etc/ssl/certificado/certificado.key //ruta del certificado
SSLCertificateFile /etc/ssl/certificado/certificado.pem

```

#### Habilitar HSTS
###### Habilitamos hsts con a2enmod para decir a los navegadores que un sitio web solo se puede comunicar por https

```
a2enmod headers

service apache2 restart

nano /etc/apache2/sites-availables/paxina3.conf

<VirtualHost *:443>
//añadimos lo siguiente dentro del VirtualHost

Headers always set Strict=Trasport-Security "max-age=999999999; includessubdomain"
</VirtualHost>

```
