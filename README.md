PROYECTO: SERVIDOR DE MONITORIZACIÓN CON GRAFANA, PROMETHEUS Y NODE EXPORTER
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
RESUMEN:

Este proyecto consiste en implementar un sistema de monitorización en tiempo real para servidores Linux utilizando herramientas de código abierto. El objetivo es obtener métricas detalladas del sistema (uso de CPU, memoria, disco, etc.) y visualizarlas en dashboards mediante Grafana.

Lo he instalado sobre Ubuntu Zorin OS, y es fácilmente adaptable a otras distribuciones como Debian. Todo el sistema funciona como servicios systemd para garantizar su ejecución automática al iniciar el servidor.

OBJETIVOS:

Recolectar métricas del sistema de forma continua.

Exponer y almacenar esas métricas usando Prometheus.

Visualizar datos en tiempo real mediante dashboards de Grafana.

Automatizar la ejecución de todos los componentes con systemd.

Configurar alertas visuales para detectar problemas en el servidor.

TECNOLOGÍAS UTILIZADAS:

Sistema Operativo: Ubuntu Zorin OS
Herramientas: Prometheus, Node Exporter, Grafana
Servicios: systemd
Visualización: Dashboards y alertas en Grafana

PREPARACIÓN DEL SISTEMA

1.1 ACTUALIZACIÓN DE PAQUETES:

sudo apt update && sudo apt upgrade -y

INSTALACIÓN DE PROMETHEUS

2.1 CREAR USUARIO Y DIRECTORIOS:

sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus

2.2 DESCARGAR Y DESCOMPRIMIR PROMETHEUS:

cd /tmp
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep "browser_download_url.linux-amd64.tar.gz" | cut -d '"' -f 4 | wget -i -
tar xvf prometheus--linux-amd64.tar.gz
cd prometheus-*-linux-amd64

2.3 MOVER BINARIOS Y ASIGNAR PERMISOS:

sudo cp prometheus promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
sudo cp -r consoles/ console_libraries/ /etc/prometheus/
sudo cp prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus

2.4 CREAR SERVICIO SYSTEMD:

Archivo: /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus \
--config.file=/etc/prometheus/prometheus.yml \
--storage.tsdb.path=/var/lib/prometheus \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

2.5 INICIAR Y HABILITAR EL SERVICIO:

sudo systemctl daemon-reexec
sudo systemctl start prometheus
sudo systemctl enable prometheus

Verificamos acceso en navegador: http://localhost:9090

INSTALACIÓN DE NODE EXPORTER

3.1 CREAR USUARIO Y DESCARGAR BINARIO:

sudo useradd --no-create-home --shell /bin/false node_exporter
cd /tmp
curl -s https://api.github.com/repos/prometheus/node_exporter/releases/latest | grep "browser_download_url.linux-amd64.tar.gz" | cut -d '"' -f 4 | wget -i -
tar xvf node_exporter--linux-amd64.tar.gz
cd node_exporter-*-linux-amd64
sudo cp node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

3.2 CREAR SERVICIO SYSTEMD:

Archivo: /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target

3.3 INICIAR Y HABILITAR EL SERVICIO:

sudo systemctl daemon-reexec
sudo systemctl start node_exporter
sudo systemctl enable node_exporter

Verificamos acceso en navegador: http://localhost:9100/metrics

CONFIGURACIÓN DE PROMETHEUS PARA RECOLECTAR MÉTRICAS

Editar el archivo /etc/prometheus/prometheus.yml y añadir:

job_name: 'node_exporter'
static_configs:

targets: ['localhost:9100']

Reiniciar Prometheus:
sudo systemctl restart prometheus

INSTALACIÓN DE GRAFANA

sudo apt install -y apt-transport-https software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y

5.1 INICIAR Y HABILITAR GRAFANA:

sudo systemctl start grafana-server
sudo systemctl enable grafana-server

Acceder a: http://localhost:3000

CONFIGURACIÓN DE DASHBOARD Y ALERTAS EN GRAFANA

En el menú lateral izquierdo de Grafana, ir a "Connections" y agregar Prometheus como fuente de datos.

Ir al botón "+" en la parte superior derecha y seleccionar "Import dashboard".

Escribir 1860 en el campo de búsqueda y cargar el dashboard de Node Exporter Full.

Seleccionar Prometheus como fuente de datos para el dashboard.

Crear un "Contact point" con tu correo como destino para las alertas.

Crear una alerta, por ejemplo: avisar cuando la RAM usada supere el 30%.

(Opcional) Configurar el archivo /etc/grafana/grafana.ini para que Grafana pueda enviar correos. Para Gmail, se debe configurar el servidor SMTP, el puerto, el usuario y una contraseña de aplicación.

LECCIONES APRENDIDAS:

MONITORIZACIÓN AVANZADA: Aprendí a instalar y combinar herramientas de observabilidad para tener métricas precisas del sistema.

VISUALIZACIÓN EN TIEMPO REAL: Integré Grafana para ofrecer interfaces visuales que permiten diagnosticar problemas fácilmente.

SERVICIOS AUTOMATIZADOS: Reforcé habilidades usando systemd para asegurar que todos los servicios inicien automáticamente.

ANÁLISIS DE RENDIMIENTO: Obtuve conocimientos prácticos para interpretar métricas clave y detectar cuellos de botella en el servidor.
