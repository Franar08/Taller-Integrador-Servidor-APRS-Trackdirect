# Servidor APRS — Manual de Despliegue Paso a Paso

> Manual técnico para recrear el servidor APRS (aprsc + Trackdirect) desde un Ubuntu Server limpio.  
> Curso: Taller Integrador | Grupo 8

---

## Tabla de contenidos

1. [Requisitos previos](#1-requisitos-previos)
2. [Instalar VirtualBox y crear la máquina virtual](#2-instalar-virtualbox-y-crear-la-máquina-virtual)
3. [Configurar la máquina virtual](#3-configurar-la-máquina-virtual)
4. [Instalar Webmin](#4-instalar-webmin)
5. [Instalar Docker y Docker Compose](#5-instalar-docker-y-docker-compose)
6. [Instalar y configurar aprsc](#6-instalar-y-configurar-aprsc)
7. [Desplegar Trackdirect con Docker Compose](#7-desplegar-trackdirect-con-docker-compose)
8. [Configurar redirección de puertos en VirtualBox](#8-configurar-redirección-de-puertos-en-virtualbox)
9. [Verificar el sistema completo](#9-verificar-el-sistema-completo)
10. [Monitoreo con Webmin](#10-monitoreo-con-webmin)
11. [Conectar con iGates reales (Grupos 1 y 2)](#11-conectar-con-igates-reales-grupos-1-y-2)
12. [Solución de problemas frecuentes](#12-solución-de-problemas-frecuentes)

---

## 1. Requisitos previos

| Componente | Versión recomendada |
|---|---|
| Sistema operativo host | Windows 10/11 o Ubuntu 22.04+ |
| VirtualBox | 7.x |
| Ubuntu Server ISO | 24.04 LTS |
| RAM disponible para la VM | 2 GB mínimo |
| Disco disponible para la VM | 20 GB mínimo |
| Conexión a internet | Requerida durante instalación |

Descarga el ISO de Ubuntu Server desde: https://ubuntu.com/download/server

---

## 2. Instalar VirtualBox y crear la máquina virtual

### 2.1 Instalar dependencias en el host (si el host es Ubuntu)

```bash
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
```

### 2.2 Instalar VirtualBox

Descarga el instalador desde https://www.virtualbox.org y ejecuta:

```bash
cd ~/Downloads
sudo apt install ./virtualbox-*.deb
```

### 2.3 Crear la máquina virtual

1. Abre VirtualBox y haz clic en **New**
2. Completa los campos:
   - **Name:** `Ubuntu Server`
   - **ISO Image:** selecciona el ISO de Ubuntu Server descargado
   - **Type:** Linux
   - **Version:** Ubuntu (64-bit)
3. Desmarca la opción **Proceed with Unattended Installation**
4. Asigna los recursos:
   - **RAM:** 2048 MB
   - **CPU:** 2 núcleos
   - **Disco:** 20 GB
5. Deja EFI desmarcado

---

## 3. Configurar la máquina virtual

### 3.1 Instalar Ubuntu Server

Inicia la VM y sigue el asistente de instalación:

1. Selecciona idioma y distribución de teclado
2. Elige **Install Ubuntu Server**
3. Deja el proxy vacío
4. En la configuración de disco selecciona **Use an entire disk**
5. Crea el usuario del sistema:
   - **Nombre del servidor:** `ubuntuserver`
   - **Nombre de usuario:** `pooh` (o el nombre de tu equipo)
   - **Contraseña:** define una segura

6. Cuando finalice, desmonta el disco virtual:  
   VirtualBox → **Devices → Optical Drives → Remove disk from virtual drive**

### 3.2 Actualizar el sistema

Una vez que inicies sesión:

```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

---

## 4. Instalar Webmin

> Webmin permite administrar el servidor gráficamente desde el navegador.

### 4.1 Descargar e instalar el paquete .deb

```bash
wget https://prdownloads.sourceforge.net/webadmin/webmin_2.105_all.deb
sudo dpkg -i webmin_2.105_all.deb
```

### 4.2 Resolver dependencias faltantes

```bash
sudo apt -f install -y
```

### 4.3 Verificar que el servicio esté activo

```bash
sudo systemctl status webmin
```

Si el servicio no está corriendo:

```bash
sudo systemctl start webmin
sudo systemctl enable webmin
```

### 4.4 Acceder a Webmin

Desde el navegador del host:

```
https://127.0.0.1:10000
```

> El navegador mostrará una advertencia de certificado autofirmado. Haz clic en **Avanzado → Continuar** para acceder.

Inicia sesión con el usuario y contraseña del sistema operativo.

---

## 5. Instalar Docker y Docker Compose

> El sistema usa Docker para levantar aprsc, Trackdirect, PostgreSQL y Nginx en contenedores.

### 5.1 Instalar dependencias

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

### 5.2 Agregar el repositorio oficial de Docker

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 5.3 Instalar Docker

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### 5.4 Permitir uso de Docker sin sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 5.5 Verificar instalación

```bash
docker --version
docker compose version
```

---

## 6. Instalar y configurar aprsc

### 6.1 Clonar el repositorio del proyecto

```bash
git clone https://github.com/Franar08/Taller-Integrador-Servidor-APRS-Trackdirect.git
cd Taller-Integrador-Servidor-APRS-Trackdirect
```

### 6.2 Revisar el archivo de configuración de aprsc

El archivo `aprsc.conf` define los parámetros del servidor APRS. Los valores clave son:

```
ServerId   TI3AFK
MyAdmin    "EquipoA, TI3AFK"
MyEmail    kendallmacampos2941@estudiantec.cr

Listen "Client-Defined Filters" igate tcp :: 14580
HTTPStatus 0.0.0.0 14501
```

> **Puertos importantes:**
> - `14580` — Puerto principal donde aprsc recibe conexiones de iGates y clientes APRS
> - `14501` — Puerto HTTP de monitoreo de estado de aprsc
> - `8081`  — Puerto de la interfaz web de Trackdirect

---

## 7. Desplegar Trackdirect con Docker Compose

### 7.1 Levantar todos los servicios

Desde el directorio del repositorio:

```bash
sudo docker compose up -d
```

Este comando levanta los siguientes contenedores:

| Contenedor | Función |
|---|---|
| `aprsc` | Servidor APRS — recibe paquetes de los iGates |
| `collector` | Procesa paquetes y los almacena en la base de datos |
| `db` | Base de datos PostgreSQL |
| `web` | Interfaz web Trackdirect |
| `reverseproxy` | Servidor Nginx |
| `websocket` | Comunicación en tiempo real con el mapa |

### 7.2 Verificar que todos los contenedores estén corriendo

```bash
sudo docker compose ps
```

Todos los servicios deben mostrar estado `running`.

### 7.3 Verificar que Trackdirect responde

```bash
curl -I http://127.0.0.1:8081
```

Resultado esperado:

```
HTTP/1.1 200 OK
```

---

## 8. Configurar redirección de puertos en VirtualBox

Para acceder a los servicios desde el host, configura **Port Forwarding** en la VM:

1. VirtualBox → selecciona la VM → **Settings → Network → Adapter 1 → Advanced → Port Forwarding**
2. Agrega estas reglas:

| Nombre | Protocolo | IP Host | Puerto Host | IP Guest | Puerto Guest |
|---|---|---|---|---|---|
| Trackdirect | TCP | 127.0.0.1 | 8081 | — | 8081 |
| Webmin | TCP | 127.0.0.1 | 10000 | — | 10000 |
| APRS | TCP | 127.0.0.1 | 14580 | — | 14580 |

Desde el host podrás acceder a:

- **Trackdirect:** http://127.0.0.1:8081
- **Webmin:** https://127.0.0.1:10000

---

## 9. Verificar el sistema completo

### Flujo de datos del sistema

```
iGate (Grupos 1-2)
       ↓
   aprsc :14580
       ↓
   collector
       ↓
  PostgreSQL
       ↓
  Trackdirect :8081
```

### Comandos de verificación

```bash
# Estado de todos los contenedores
sudo docker compose ps

# Logs de aprsc en vivo
sudo docker compose logs -f aprsc

# Conexiones activas en el puerto APRS
sudo ss -tnp | grep 14580

# Uso de disco
df -h /

# CPU y RAM en tiempo real
htop
```

---

## 10. Monitoreo con Webmin

### 10.1 Acceder al monitoreo

Ir a: **Tools → System and Server Status**

### 10.2 Monitores configurados

| Monitor | Tipo | Umbral de alerta |
|---|---|---|
| Monitor APRSC | Check Process | Si el proceso se detiene |
| Monitor Disco APRS | Disk Space | Si el espacio libre cae bajo 15% |
| Monitor CPU APRS | Load Average | Si el load average supera 1.5 |
| Monitor RAM APRS | Free Memory | Si la RAM libre cae bajo 300 MB |
| Monitor Puerto APRS 14580 | TCP Connection | Si el puerto no responde |
| Monitor Trackdirect Web 8081 | TCP Connection | Si la interfaz web no responde |

### 10.3 Activar alertas por email

1. Ir a **Tools → System and Server Status → Scheduled Monitoring**
2. Configurar:
   - **Scheduled checking enabled:** Yes
   - **Check every:** 5 minutes
   - **Send email when:** When a service goes down
   - **Email:** correo del equipo en estudiantec.cr

### 10.4 Métricas de referencia

| Métrica | Valor normal | Alerta si... |
|---|---|---|
| CPU load avg | < 1.0 | > 1.5 |
| RAM libre | > 500 MB | < 300 MB |
| Disco usado | < 80% | > 85% |
| Puerto 14580 | Responde | No responde |
| Puerto 8081 | HTTP 200 | No responde |

---

## 11. Conectar con iGates reales (Grupos 1 y 2)

Para recibir tráfico de los iGates reales, los grupos 1 y 2 deben configurar sus iGates apuntando a la IP del servidor en el puerto `14580`.

### 11.1 Verificar que el servidor recibe conexiones externas

```bash
sudo ss -tnp | grep 14580
```

Cada línea representa una conexión activa de un iGate.

### 11.2 Verificar paquetes en los logs

```bash
sudo docker compose logs -f aprsc | grep "connected"
```

### 11.3 Verificar en el mapa

Abrir http://127.0.0.1:8081 y confirmar que aparecen estaciones en el mapa en tiempo real.

---

## 12. Solución de problemas frecuentes

### El contenedor aprsc no inicia

```bash
sudo docker compose logs aprsc
```

Verificar que el archivo `aprsc.conf` tenga el formato correcto y que el puerto 14580 no esté ocupado:

```bash
sudo ss -tnlp | grep 14580
```

### Trackdirect no muestra paquetes en el mapa

Verificar que el collector esté conectado a aprsc:

```bash
sudo docker compose logs collector
```

### El disco está lleno

```bash
# Limpiar paquetes innecesarios
sudo apt autoremove -y && sudo apt clean

# Limpiar logs del sistema
sudo journalctl --vacuum-size=200M

# Limpiar imágenes Docker no usadas
sudo docker system prune -f

# Verificar espacio liberado
df -h /
```

### Webmin no abre en el navegador

```bash
sudo systemctl restart webmin
sudo systemctl status webmin
```

### Reiniciar todos los servicios Docker

```bash
sudo docker compose down
sudo docker compose up -d
```

---

## Arquitectura del sistema

```
┌─────────────────────────────────────────────┐
│              VirtualBox (NAT)               │
│  ┌──────────────────────────────────────┐   │
│  │         Ubuntu Server 24.04          │   │
│  │                                      │   │
│  │  ┌────────┐    ┌──────────────────┐  │   │
│  │  │ Webmin │    │  Docker Compose  │  │   │
│  │  │ :10000 │    │                  │  │   │
│  │  └────────┘    │ aprsc     :14580 │  │   │
│  │                │ collector        │  │   │
│  │                │ PostgreSQL       │  │   │
│  │                │ Trackdirect      │  │   │
│  │                │ Nginx     :8081  │  │   │
│  │                └──────────────────┘  │   │
│  └──────────────────────────────────────┘   │
│         Port Forwarding: 8081, 10000,       │
│                          14580              │
└─────────────────────────────────────────────┘
         ↑
   iGates reales
   (Grupos 1 y 2)
```

---

*Repositorio del proyecto: https://github.com/Franar08/Taller-Integrador-Servidor-APRS-Trackdirect*  
*Curso: Taller Integrador — Instituto Tecnológico de Costa Rica*
