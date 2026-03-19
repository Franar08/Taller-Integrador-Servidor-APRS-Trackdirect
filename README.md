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

Para la administración remota del servidor se utilizó Webmin, una herramienta basada en web que permite gestionar servicios, usuarios y configuraciones del sistema de forma gráfica. El proceso de instalación requirió un enfoque alternativo debido a incompatibilidades con el método tradicional en versiones recientes de Ubuntu.

Inicialmente, se intentó realizar la instalación utilizando el repositorio oficial mediante `apt`. Sin embargo, este método falló debido a errores de verificación de firma (GPG), ya que el repositorio de Webmin utiliza algoritmos de firma que han quedado obsoletos y son rechazados por Ubuntu 24. Esto impedía que el sistema confiara en el origen del paquete, bloqueando su instalación.

Como solución, se optó por instalar Webmin directamente desde su paquete oficial en formato `.deb`. Para ello, primero se descargó el archivo desde el repositorio oficial en SourceForge:

```bash
wget https://prdownloads.sourceforge.net/webadmin/webmin_2.105_all.deb
```

Una vez descargado el paquete, se procedió con su instalación utilizando la herramienta `dpkg`:

```bash
sudo dpkg -i webmin_2.105_all.deb
```

Durante este proceso, el sistema reportó la falta de algunas dependencias necesarias para completar la instalación. Este comportamiento es esperado al instalar paquetes manualmente. En lugar de instalar cada dependencia de forma individual, se utilizó el siguiente comando para que el propio sistema resolviera automáticamente todos los paquetes faltantes:

```bash
sudo apt -f install -y
```

Este paso permitió completar correctamente la instalación de Webmin junto con todas sus librerías requeridas.

Posteriormente, se verificó el estado del servicio para confirmar que Webmin estuviera activo y funcionando correctamente:

```bash
sudo systemctl status webmin
```

El servicio se encontró en estado `active (running)`, lo que confirmó que la instalación fue exitosa. En caso de que no se inicie automáticamente, se puede iniciar manualmente con:

```bash
sudo systemctl start webmin
```

Una vez en ejecución, Webmin queda disponible a través de su interfaz web. Para acceder, se utilizó un navegador web desde la máquina host, ingresando a la siguiente dirección:

```
https://<IP-del-servidor>:10000
```

Es importante tener en cuenta que Webmin utiliza un certificado autofirmado por defecto, por lo que el navegador mostrará una advertencia de seguridad. Esta advertencia es esperada en entornos de prueba o desarrollo, y se puede omitir accediendo a las opciones avanzadas del navegador y continuando al sitio.

Finalmente, se inició sesión utilizando las credenciales del sistema operativo (usuario y contraseña creados durante la instalación de Ubuntu). Con esto, se obtuvo acceso completo al panel de administración de Webmin.

En conclusión, debido a las restricciones de seguridad en versiones modernas de Ubuntu, la instalación de Webmin se realizó mediante el uso del paquete `.deb`, lo que permitió evitar problemas de firma y completar la configuración de forma exitosa. Esta herramienta quedó operativa para la gestión remota del servidor, cumpliendo con los requerimientos del proyecto.


# Instalar y configurar el servicio aprsc y configurar aprsc para recibir el tráfico del iGate de prueba (TI3WTI-10).
