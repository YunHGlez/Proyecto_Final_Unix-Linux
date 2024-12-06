# Instalación de MySQL 
'''
wget https://dev.mysql.com/get/mysql-apt-config_0.8.30-1_all.deb
'''

Instalamos la información del repositorio 

'''
sudo dpkg -i mysql-apt-config_0.8.30-1_all.deb
'''

Despues actualizamos los servidores de paquetes con 

''' 
sudo apt update
'''

Finalmente podemos realizar la instalación de MySQL 

'''
sudo apt install mysql-server -y 
'''

Ingresas la contraseña del root de la base de datos. 

y es de esta forma que tienes instalado mysql.

Para realizar el resto, entramos al mysql y creamos la base de datos

''' 
sudo mysql -u root -p
'''

Creamos la base de datos para WordPress 

'''
CREATE DATABASE wordpress;
'''

Creamos un usuario y le otorgamos privilegios de base de datos wordpress

'''
CREATE USER 'wp_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.\* TO 'wp_user'@'%';
FLUSH PRIVILEGES;
'''

Verificamos los privilegios del nuevo usuario 

''' 
SHOW GRANTS FOR 'wp_user'@'%';
'''


Finalmente salimos de MySQL 

''' 
EXIT
'''
