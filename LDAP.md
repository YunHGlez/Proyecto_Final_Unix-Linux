
# Instalación de un Servidor LDAP en Debian

## 1. Configuración del Nombre del Host
```bash
sudo hostnamectl set-hostname ldapserver.debian.local
```

Editar el archivo `/etc/hosts` para sustituir el nombre del host por el introducido anteriormente y agregar la IP local junto con el hostname.

```bash
sudo nano /etc/hosts
```

## 2. Actualización del Sistema
Actualizar el sistema operativo:

```bash
sudo apt update -y && sudo apt dist-upgrade -y
```

## 3. Instalación de OpenLDAP
Instalar los paquetes necesarios:

```bash
sudo apt install slapd ldap-utils -y
```

Se pedirá una contraseña para el administrador de LDAP. Utilizar: **4321**.

Reconfigurar OpenLDAP:

```bash
sudo dpkg-reconfigure slapd
```

Durante la configuración:
- Dominio: `dns.com`
- Organización: `debian`
- Contraseña del administrador: **4321**

Verificar la configuración:

```bash
sudo slapcat
```

## 4. Creación de Grupos en LDAP
Crear un archivo llamado `grp.ldif` para definir los grupos:

```bash
sudo nano grp.ldif
```

Ejemplo de contenido:

```ldif
dn: cn=sftp,dc=dns,dc=com
objectClass: top
objectClass: posixGroup
gidNumber: 10000
cn: sftp
```

Agregar el grupo al servidor LDAP:

```bash
sudo ldapadd -x -D cn=admin,dc=dns,dc=com -W -f grp.ldif
```

Repetir el proceso para agregar más grupos, modificando los campos `cn` y `gidNumber`.

## 5. Creación de Usuarios en LDAP
Crear un archivo llamado `usr.ldif` para definir los usuarios:

```bash
sudo nano usr.ldif
```

Ejemplo de contenido:

```ldif
dn: uid=ratateam,dc=dns,dc=com
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: person
cn: ratateam
uid: ratateam
uidNumber: 10003
gidNumber: 10000
homeDirectory: /home/ratateam
loginShell: /bin/bash
userPassword: 4321
sn: team
mail: ratateam@mail.com
givenName: ratateam
```

Agregar el usuario al servidor LDAP:

```bash
sudo ldapadd -x -D cn=admin,dc=dns,dc=com -W -f usr.ldif
```

---

# Conexión de un Cliente al Servidor LDAP

1. **Instalar el cliente LDAP en la máquina cliente:**

   ```bash
   sudo apt install ldap-utils -y
   ```

2. **Probar la conexión al servidor LDAP:**

   ```bash
   ldapsearch -x -H ldap://ldapserver.debian.local -D "cn=admin,dc=dns,dc=com" -W
   ```

3. **Configurar la autenticación para que el cliente use LDAP.**

   Editar `/etc/ldap/ldap.conf` en el cliente:

   ```bash
   BASE dc=dns,dc=com
   URI ldap://ldapserver.debian.local
   ```

---

# Configuración de la Interfaz Gráfica: LDAP Account Manager (LAM)

1. **Instalar LDAP Account Manager:**

   ```bash
   sudo apt install ldap-account-manager -y
   ```

2. **Configurar LAM:**
   - Abrir el archivo de configuración:

     ```bash
     sudo nano /etc/ldap-account-manager/config.cfg
     ```

   - Ajustar los parámetros según tu dominio LDAP.

3. **Acceder a LAM:**
   - Abrir un navegador y visitar `http://<ip-servidor>/lam`.
   - Iniciar sesión con las credenciales de administrador LDAP (`cn=admin,dc=dns,dc=com` y contraseña **4321**).

4. **Agregar usuarios o grupos:**
   - Navegar en la interfaz gráfica para gestionar usuarios, grupos y otros objetos LDAP.

---

¡Con esto, tu servidor LDAP estará completamente configurado, junto con un cliente y una interfaz gráfica para facilitar la administración!
