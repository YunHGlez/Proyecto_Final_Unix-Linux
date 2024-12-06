# Install y config postfix 

Actualizamos y upgradeamos los paquetes de la maquina

''' 
sudo apt update && sudo apt upgrade -y
'''

Instalamos lo siguiente: 

'''
apt install postfix mailutils libsal2-modules libsasl2-modules-ldap logwatch fail2ban -y
'''

Realizamos la configuración de postfix con internet site.

Después entraremos a /etc/postfix/main.cf y mantendremos la siguiente configuración, donde podremos notar que remarcamos que el mailbox deberá tener un tamaño correspondiente a 1GB

''' 
\# Información del servidor
myhostname = mail.example.com             
mydomain = example.com                    
myorigin = $mydomain
inet_interfaces = all                    
inet_protocols = ipv4     
maillog_file = /var/log/mail.log

\# Configuración del correo
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
home_mailbox = Maildir/                  
mailbox_size_limit = 1073741824                   
recipient_delimiter = +               

\# Configuración de relay
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_local_domain = $myhostname
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_use_tls = yes
smtp_tls_security_level = may
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt                              
mynetworks = 127.0.0.0/8 [::1]/128 192.168.220.146/25    

\# Configuración de SASL y LDAP
smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot           
smtpd_sasl_path = private/auth
smtpd_sasl_security_options = noanonymous
smtpd_recipient_restrictions = permit_sasl_authenticated, reject_unauth_destination

\# Integración con LDAP para usuarios virtuales
\#virtual_mailbox_domains = $mydomain
\#virtual_mailbox_maps = ldap:/etc/postfix/ldap-users.cf
\#virtual_uid_maps = static:1000            # UID de usuarios virtuales (ajusta según tu configuración)
\#virtual_gid_maps = static:1000            # GID de usuarios virtuales (ajusta según tu configuración)

\# Seguridad
smtpd_tls_cert_file = /etc/ssl/certs/mail.example.com.pem 
smtpd_tls_key_file = /etc/ssl/private/mail.example.com.key 
smtpd_tls_security_level = may
smtp_tls_security_level = may
smtpd_tls_auth_only = yes          
smtpd_use_tls = yes

smtpd_recipient_restrictions = 
    permit_sasl_authenticated,
    permit_mynetworks,
    reject_unauth_destination
'''

Reinicamos nuestro postfix

''' 
sudo systemctl restart postfix
'''

Entramos a /etc/postfix/sasl_passwd y asignamos un correo junto con el hash de una contraseña correspondiente de la siguiente forma 

'''
[smtp.gmail.com]:587 correo@example.com:contraseñaHasheada 
'''

Modificamos los permisos del archivo y se lo pasamos a postmap como parametro, de tal forma que:

'''
sudo chmod 600 /etc/postfix/sasl_passwd 
sudo postmap /etc/postfix/sasl_passwd
sudo systemctl restart postfix
'''

Entramos en el archivo de configuración del fail2ban y realizamos lo siguiente: 

'''
nano /etc/fail2ban/jail.d/postfix.local 

[postfix]
enabled = true
port = 587
filter = postfix 
logpath = /var/log/mail.log 
maxretry = 3
bantime = 600 
findtime = 600
action = iptables[name=postfix, port="587", protocol=tcp]
'''
Tambien modificamos el siguiente archivo 
''' 
nano /etc/fail2ban/filter.d/postfix.conf 

y realizamos cambios en las siguientes lineal

mode = more

failregex = ^%(__prefix_line)s.* ? warning: .*? \[<HOST>\]: SASL (PLAIN|LOGIN) authentication failed(: . *|$)
ignoreregex =
*
'''
De ahi seguimos con la configuracion de los aliases

''' 
nano /etc/aliases 

sudo newaliases
sudo cp /usr/share/logwatch/default.conf/logwatch.conf /etc/logwatch/conf/
'''
Configuramos el siguiente parametro
Output = mail 
allowipv6 = true

Seguimos copiando unas configuraciones para mantener los cambios en el local y no se vean afectados al momento de actualizar

'''
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local 
'''

Cambiaremos los permisos de los archivos de tal forma que 

'''
sudo chmod 644 /etc/fail2ban/\*. conf
sudo chmod 644 /etc/fail2ban/\*.local
'''

Para el uso junto con Telegram, configuraremos un archivo que será ejecutado por el Cron, de tal forma que el archivo deberá contener lo siguiente, enlazandose a un bot de telegram

''' 
#!/bin/bash

# Variables
TOKEN="7650903640:AAGo9qJ86hPfiA5qCBp-HW4WgOeLZoLCrrg"
CHAT_ID="1311454099"
LOGWATCH_OUTPUT="/var/cache/logwatch/logwatch.log"

# Generar informe de Logwatch
/usr/sbin/logwatch --output file --format text --detail high --file "$LOGWATCH_OUTPUT" > /dev/null 2>&1
# Leer el informe
LOG=$(cat "$LOGWATCH_OUTPUT")

# Enviar informe a Telegram
curl -s -X POST "https://api.telegram.org/bot$TOKEN/sendMessage" \
    -d chat_id="$CHAT_ID" \
    -d text="$LOG" \
    -d parse_mode="Markdown" > /dev/null 2>&1

'''
Posteriormente Instalamos el paquete maildrop 

'''
sudo apt install maildrop -y
'''

Para después realizar el siguiente proceso para asegurar que el buzon se encuentre en el home correspondiente del usuario. 

'''
sudo maildirmake /home/usuario/Maildir
sudo chown -R usuario:usuario /home/usuario/Maildir
'''

donde usuario corresponde al usuario actual de nuestro sistema, para después asegurarnos que los permisos sean correctos.

'''
sudo chmod -R 700 /home/usuario/Maildir
'''

Haremos la siguiente modificación para que cada que agregemos un nuevo usuario, este le cree su directorio en el home

'''
sudo maildirmake /etc/skel/Maildir
sudo chmod -R 700 /etc/skel/Maildir
'''

Es de esta forma que asignamos buzones en el home de los usuarios. 
