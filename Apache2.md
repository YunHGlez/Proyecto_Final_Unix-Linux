# INSTALL

Primero que nada se deberá instalar el apache con algunos paquetes extra de php que nos permitan su correcto funcionamiento
'sudo apt install apache2 libapache2-mod-php php php-mysql php-ldap -y'

Posteriormente hacemos habilitamos a apache, lo iniciamos y revisaremos su Status 

''' 
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl status apache2

'''

Dado que usaremos WordPress para nuestra base, en particular usaremos mySQL como nuestro sistema manejador de base de datos, por lo que necesitamos tenerlo instalado y habilitado
'''
wget https://dev.mysql.com/get/mysql-apt-config_0.8.30-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.30-1_all.deb
sudo apt update
sudo apt install mysql-server -y
sudo systemctl enable mysql
sudo systemctl start mysql
'''
Creamos la base de datos en MySQL, este proceso lo realizaremos en la maquina que contiene nuestra base de datos

'''
sudo mysql -u root -p
CREATE DATABASE wordpress;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.\* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
''' 

Instalamos las ultimas extensiones de PHP necesarias

''' 
sudo apt install php-curl php-gd php-mbstring php-xml php-zip -y
'''


Comenzamos la instalación de Wordpress
'''
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/html/
'''

Configuramos los permisos de las carpetas

'''
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
'''

Configuramos el virtual Host agregando lo sigiente texto en el siguiente archivo
'''
sudo nano /etc/apache2/sites-available/wordpress.conf
'''

Agregamos lo siguiente:

'''
<VirtualHost *:80>
    ServerName wordpress.dominio.local
    DocumentRoot /var/www/html/wordpress

    <Directory /var/www/html/wordpress>
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>
'''

Habilitamos el sitio y módulos necesarios

'''
sudo a2ensite wordpress.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
'''

Entramos a la pagina y terminamos la configuración o copiamos el archivo de /var/www/html/wordpress/wp-config-sample.php en /var/www/html/wordpress/wp-config.php 

y llenamos los siguientes datos

'''
define('DB_NAME', 'wordpress');        // Nombre de la base de datos
define('DB_USER', 'wp_user');          // Usuario de PostgreSQL
define('DB_PASSWORD', 'password');    // Contraseña de PostgreSQL
define('DB_HOST', '[IP_VM_POSTGRESQL]'); // Dirección IP o nombre de host del servidor PostgreSQL
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
'''


Procedemos con la instalación de squirremail a través de git 

'''
wget https://sourceforge.net/projects/squirrelmail/files/stable/1.4.22/squirrelmail-webmail-1.4.22.zip
unzip squirrelmail-webmail-1.4.22.zip
sudo mv squirrelmail-webmail-1.4.22 /var/www/html/
sudo chown -R www-data:www-data /var/www/html/squirrelmail-webmail-1.4.22/
sudo chmod 755 -R /var/www/html/squirrelmail-webmail-1.4.22/
sudo mv /var/www/html/squirrelmail-webmail-1.4.22/ /var/www/html/squirrelmail
'''

Después configuramos SquirreMail con el siguiente comando:

'''
sudo perl /var/www/html/squirrelmail/config/conf.pl
'''

Selecciona 2. Server Setting 
Selecciona 1 Domain e introduce el dominio que hayas usado en la instalación de postfix 
Después Seleccionas R para guardar el cambio y 4. General Options.
Modificamos la opción 1,2 y 11 y hacemos que las rutas sean de la siguiente manera:

![Img 1](https://dungeonofbits.com/images/postfix7.jpg)

y Terminamos
