# Taller-Integrador-Servidor-APRS-Trackdirect

Repositorio para el Proyecto correspondiente al curso de Taller Integrador con temática Servidor APRS Trackdirect

Gantt inicial del proyecto

![Gantt](gantt/Gantt.png)


# Instalar VirtualBox y crear una máquina virtual con Ubuntu Server LTS.

Se creó correctamente una máquina virtual en VirtualBox con **Ubuntu Server LTS**.
**Validación realizada:**
- Verificación del arranque del sistema operativo
- Inicio de sesión exitoso en Ubuntu Server
- Confirmación de versión del sistema
Primero se instalaron dependencias necesarias:
```bash
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
```
Luego se instaló el paquete .deb de VirtualBox descargado previamente:
```bash
cd ~/Downloads
sudo apt install ./virtualbox-*.deb
```
Una vez instalada la máquina virtual con Ubuntu Server, se actualizó el sistema:
```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

# Instalar Webmin para la administración gráfica del servidor.


# Instalar y configurar el servicio aprsc y configurar aprsc para recibir el tráfico del iGate de prueba (TI3WTI-10).
