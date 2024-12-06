# NFS

## Instalación

### Descargamos el paquete de NFS server system.

#### sudo apt update
#### sudo apt install nfs-kernel-server

![image](https://github.com/user-attachments/assets/a98ad5c7-fca7-4bb7-82ab-73cfecf684e5)

### Generamos un directorio para ser usado como servidor

![image](https://github.com/user-attachments/assets/b956bc2a-e762-4fa7-9378-fd5b438eb537)

### Modificamos el archivo /etc/exports para establecer los permisos del directorio

![image](https://github.com/user-attachments/assets/18327171-3eaf-43c6-9ed6-4f8faa4c494d)

### Exportamos el directorio, reiniciamos el servicio, y lo activamos

![image](https://github.com/user-attachments/assets/3bcfc921-076c-4bab-9c7d-5b604185feee)

### Verificamos que la carpeta haya sido exportada

![image](https://github.com/user-attachments/assets/4bf2c63f-a7fb-43d1-bcb6-6c8a1d767f92)

## Montar el servicio desde otra máquina


