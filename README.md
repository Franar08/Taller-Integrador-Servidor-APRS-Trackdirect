# Taller-Integrador-Servidor-APRS-Trackdirect

Repositorio para el Proyecto correspondiente al curso de Taller Integrador con temática Servidor APRS Trackdirect

Gantt inicial del proyecto

![Gantt](gantt/Gantt.png)


# Instalar VirtualBox y crear una máquina virtual con Ubuntu Server LTS.

Se creó correctamente una máquina virtual en VirtualBox con **Ubuntu Server LTS**.
Iniciando con la descarga del Ubuntu server desde la página oficial de ubuntu para la configuración dentro del VirtualBox, también se descargó el VirtualBox
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
**Dentro del VirtualBox**
- Pulsa **New**
- En name se escribe **Ubuntu Server**
- En ISO se selecciona la imagen que descargamos anteriormente de Ubuntu Server
- Luego en **Type** Linux y **Versión** Ubuntu (64 bit)
- Desmarca la opción de instalación automática, que suele salir como:
Proceed with Unattended Installation

**Asignación de recursos de la VirtualBox**
- Ram de 2048MB, CPU 2 y un disco de 20GB
- Se deja EFI desmarcado si no te lo piden específicamente.

**Luego se inicia la máquina virtual**
- Se selecciona el idioma
- Se selecciona el idioma del teclado
- Install Ubuntu Server
****

# Instalar Webmin para la administración gráfica del servidor.


# Instalar y configurar el servicio aprsc y configurar aprsc para recibir el tráfico del iGate de prueba (TI3WTI-10).
