
# Instalación y Configuración de Fail2Ban en Debian

**Fail2Ban** es una herramienta de seguridad diseñada para proteger tu servidor contra ataques de fuerza bruta y accesos no autorizados bloqueando automáticamente las direcciones IP sospechosas.

## Pasos para Instalar y Configurar Fail2Ban

### 1. Actualizar el Índice de Paquetes
```bash
sudo aptitude update
```
Este comando actualiza la lista de paquetes disponibles en el sistema. Es importante para asegurarse de instalar la última versión de Fail2Ban.

---

### 2. Instalar Fail2Ban
```bash
sudo aptitude install fail2ban
```
Instala el servicio Fail2Ban, que monitoriza los archivos de registro del sistema para detectar actividad sospechosa.

---

### 3. Cambiar al Directorio de Configuración
```bash
cd /etc/fail2ban
```
Este directorio contiene los archivos de configuración necesarios para personalizar Fail2Ban.

---

### 4. Crear una Copia del Archivo de Configuración Base
```bash
sudo cp jail.conf jail.local
```
Crea una copia del archivo de configuración predeterminado `jail.conf` como `jail.local`. Este paso asegura que los cambios personalizados no se sobrescriban con futuras actualizaciones del paquete.

---

### 5. Editar la Configuración Personalizada
```bash
sudo nano jail.local
```
Abre el archivo `jail.local` para realizar configuraciones específicas, como:
- Habilitar servicios protegidos, por ejemplo, SSH:
  ```
  [ssh]
  enabled = true
  port = ssh
  logpath = /var/log/auth.log
  maxretry = 5
  ```
- Configurar el tiempo de bloqueo (`bantime`) y el número de intentos permitidos (`maxretry`):
  ```
  bantime = 600  # Tiempo de bloqueo en segundos (10 minutos)
  maxretry = 3   # Intentos permitidos antes de bloquear
  ```

---

### 6. Reiniciar el Servicio Fail2Ban
```bash
sudo systemctl restart fail2ban
```
Reinicia el servicio para aplicar los cambios realizados en el archivo de configuración.

---

### 7. Verificar el Estado del Servicio
```bash
sudo systemctl status fail2ban
```
Este comando muestra el estado actual de Fail2Ban. Asegúrate de que esté **activo** y funcionando correctamente.

---

## Notas Adicionales
- Puedes consultar las IP bloqueadas ejecutando:
  ```bash
  sudo fail2ban-client status
  ```
- Para desbloquear una IP específica:
  ```bash
  sudo fail2ban-client set <jail> unbanip <IP>
  ```

Con estos pasos, Fail2Ban estará configurado y operativo, proporcionando una capa adicional de seguridad para tu servidor.
