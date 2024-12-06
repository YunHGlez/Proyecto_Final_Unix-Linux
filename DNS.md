# Servidor:



1.   Actualiza los repositorios de los paquetes:
  ```
  sudo apt update
  ```
2. Instala bind9 y bind-utils. 
  ```
  sudo apt install bind9 bind-utils
  ```
3.Verifica el estado del servicio BIND9.
  ```
  sudo systemctl status bind9
  ```
4. Abre el archivo de configuración de opciones globales de BIND9.
  ```
  sudo nano /etc/bind/named.conf.options
  ```
  Modificamos el archivo como sigue:
  ```
  options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        listen-on { any; };
        allow-query { localhost; 192.168.13.0/24; };
        forwarders {
                8.8.8.8;
         };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation no;

        # listen-on-v6 { any; };
  };
  ```
  Donde:
  - ```listen-on { any; };```

    Configura las interfaces de red en las que el servidor DNS escuchará las consultas. El argumento ```any```, hace que escucha en todas las interfaces disponibles.
  - ```allow-query {192.168.13.0/24; };```

    Define qué hosts pueden realizar consultas al servidor DNS. El argumento
    ```192.168.13.0/24``` permite consultas desde toda la subred 192.168.13.0/24 (rango de direcciones desde 192.168.13.1 hasta 192.168.13.254). En este caso, se asigno una IP dentro de este rango para todos nuestros servidores y clientes (estática para los servidores).

  - ```forwarders { 8.8.8.8; };```

    Configura servidores DNS externos (reenviadores) para consultas que el servidor no pueda resolver localmente. El argumento ```8.8.8.8``` es el servidor DNS público de Google.

  - ```# listen-on-v6 { any; };```

    Configura el servidor para escuchar en direcciones IPv6, en este caso se comenta para evitarlo.

5. Abre el archivo de configuración por defecto del servicio named (el proceso de BIND9).

  ```
  sudo nano /etc/default/named
  ```

  Modificamos el archivo como sigue:
  ```
  # run resolvconf?
  RESOLVCONF=no

  # startup options for the server
  OPTIONS="-u bind -4"
  ```

  Donde:
  - ```RESOLVCONF=no```

    BIND9 no actualizará automáticamente el archivo /etc/resolv.conf.

  - ```OPTIONS="-u bind -4"```

    Fuerza al servidor a operar solo en IPv4, ignorando solicitudes o configuraciones relacionadas con IPv6.



6. Verifica la sintaxis de los archivos de configuración de BIND9.
  ```
  sudo named-checkconf
  ```


7. Reinicia el servicio BIND9 para aplicar cambios realizados en su configuración.
  ```
  sudo systemctl restart bind9
  ```

8. Comprueba nuevamente el estado de BIND9 tras el reinicio para asegurarse de que el servidor se reinició correctamente y está funcionando.
  ```
  sudo systemctl status bind9
  ```

9. Es necesario modificar el archivo ```sudo nano /etc/bind/named.conf.local``` para agregar la zona y la zona inversa de cada dominio, además de crear los archivos que definan sus configuraciones. Para ello creamos el siguiente script:

  ```
  #!/bin/bash

  # Declaración de variables
  NAME="nagios"                          # Nombre del dominio
  FINAL_IP_NUMBER="246"                 # Último octeto de la dirección IP estática
  STATIC_IP="192.168.13.$FINAL_IP_NUMBER"  # Dirección IP completa
  REVERSE="13.168.192"                  # Dirección para la zona inversa
  DOMAIN="$NAME.com"                   # Dominio completo
  DIRECTORY="/etc/bind/zones"           # Directorio donde se almacenarán las zonas
  OUT_FILE="$DIRECTORY/$DOMAIN.zone"    # Archivo para la zona directa
  OUT_FILE_REVERSE="$DIRECTORY/$DOMAIN.$REVERSE.zone" # Archivo para la zona inversa

  # Agregar configuración al archivo named.conf.local
  sudo tee -a /etc/bind/named.conf.local <<EOF

  zone "$DOMAIN" IN {
    type master;
    file "$OUT_FILE";
  };

  zone "$DOMAIN.in-addr.arpa" IN {
    type master;
    file "$OUT_FILE_REVERSE";
  };
  EOF

  # Verifica si el directorio existe
  if [ ! -d "$DIRECTORY" ]; then
  sudo mkdir "$DIRECTORY"
  fi

  # Crear el archivo de zona directa (db.DOMAIN)
  sudo tee $OUT_FILE <<EOF
  $TTL    604800
  @       IN      SOA     $DOMAIN. root.$DOMAIN. (
                    1                 ; Serial
                    604800            ; Refresh
                    86400             ; Retry
                    2419200           ; Expire
                    604800 )          ; Negative Cache TTL
  @       IN      NS      $DOMAIN.
  @       IN      A       $STATIC_IP
  www     IN      CNAME   $DOMAIN.
  EOF

  # Crear el archivo de zona inversa
  sudo tee $OUT_FILE_REVERSE <<EOF
  $TTL    604800
  @       IN      SOA     $DOMAIN. root.$DOMAIN. (
                    1                 ; Serial
                    604800            ; Refresh
                    86400             ; Retry
                    2419200           ; Expire
                    604800 )          ; Negative Cache TTL
  @                    IN      NS      $DOMAIN.
  $FINAL_IP_NUMBER     IN      PTR     $DOMAIN.
  EOF

  ```
  Donde: 
  - ```NAME```: Nombre base del dominio, en este caso nagios (dns,postfix,etc..).
  - ```FINAL_IP_NUMBER```: Último octeto de la dirección IP, aquí 246 (40 - 46).
  - ```STATIC_IP```: Dirección IP completa estática, formada por 192.168.13 y ```FINAL_IP_NUMBER```.
  - ```REVERSE```: Parte de la dirección IP en formato inverso para la zona inversa, ```13.168.192```.
  - ```DOMAIN```: Nombre completo del dominio, creado como ```NAME``` y .com.
  - ```DIRECTORY```: Ruta del directorio donde se almacenarán los archivos de configuración de las zonas.
  - ```OUT_FILE```: Ruta del archivo que contendrá la configuración de la zona directa (DNS directo).
  - ```OUT_FILE_REVERSE```: Ruta del archivo que contendrá la configuración de la zona inversa (DNS inverso).

10. Reinicia el servicio BIND9 para aplicar cambios realizados en su configuración.
  ```
  sudo systemctl restart bind9
  ```

11. Comprueba nuevamente el estado de BIND9 tras el reinicio para asegurarse de que el servidor se reinició correctamente y está funcionando.
  ```
  sudo systemctl status bind9
  ```

# Cliente

1. Configura el cliente utilizando `systemd-resolved`.

2. Edita el archivo `/etc/systemd/resolved.conf`:
   ```bash
   sudo nano /etc/systemd/resolved.conf
   ```
   Agrega o actualiza las siguientes líneas:
   ```plaintext
   [Resolve]
   DNS=192.168.13.240
   ```
   Donde:
   - ```DNS=192.168.13.240```: Dirección IP del servidor DNS.

3. Reinicia el servicio `systemd-resolved`:
   ```bash
   sudo systemctl restart systemd-resolved
   ```

4. Prueba la resolución de nombres:
   ```bash
   ping nagios.com
   ```


