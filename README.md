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
