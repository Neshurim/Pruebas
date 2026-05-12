# Instructivo: Despliegue de Ansible Automation Platform en WSL2 e Integración con Zabbix

**Versión:** 1.0  
**Fecha:** 2026-05-12  
**Autor:** Equipo de Automatización  
**Estado:** Instructivo Técnico Completo

---

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Requisitos Previos](#requisitos-previos)
3. [Parte 1: Preparación del Entorno WSL2](#parte-1-preparación-del-entorno-wsl2)
4. [Parte 2: Instalación de Ansible Automation Platform](#parte-2-instalación-de-ansible-automation-platform)
5. [Parte 3: Configuración de Zabbix](#parte-3-configuración-de-zabbix)
6. [Parte 4: Integración AAP con Zabbix](#parte-4-integración-aap-con-zabbix)
7. [Parte 5: Automatización Basada en Eventos](#parte-5-automatización-basada-en-eventos)
8. [Parte 6: Validación y Pruebas](#parte-6-validación-y-pruebas)
9. [Troubleshooting](#troubleshooting)
10. [Referencia Rápida](#referencia-rápida)

---

## Introducción

Este instructivo proporciona un procedimiento paso a paso para:

- Instalar **Ansible Automation Platform (AAP)** en **Windows Subsystem for Linux 2 (WSL2)**
- Integrar **AAP** con **Zabbix** para monitoreo
- Crear **automatizaciones basadas en eventos** disparadas por alertas de Zabbix
- Configurar **webhooks** y **APIs** para comunicación bidireccional

### Casos de Uso

- Automatizar respuestas a alertas de Zabbix
- Ejecutar playbooks al detectar problemas
- Escalar automáticamente recursos
- Generar reportes de incidentes
- Orquestar tareas de remediación

### Arquitectura General

```
┌──────────────────────────────────────────────────────────────┐
│                    Windows 10/11/Server                      │
│  ┌────────────────────────────────────────────────────────┐  │
│  │        Windows Subsystem for Linux 2 (WSL2)          │  │
│  │  ┌──────────────────────────────────────────────────┐ │  │
│  │  │  Ansible Automation Platform (Controller)       │ │  │
│  │  │  - API REST (puerto 443)                        │ │  │
│  │  │  - WebSocket (puerto 8002)                      │ │  │
│  │  │  - PostgreSQL (puerto 5432)                     │ │  │
│  │  └──────────────────────────────────────────────────┘ │  │
│  │  ┌──────────────────────────────────────────────────┐ │  │
│  │  │  Execution Environment (Contenedor)             │ │  │
│  │  │  - Ansible Runtime                              │ │  │
│  │  │  - Colecciones y roles                          │ │  │
│  │  └──────────────────────────────────────────────────┘ │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
                            │
                    (Webhooks / API)
                            │
┌──────────────────────────────────────────────────────────────┐
│                    Zabbix Server                             │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  - Monitoreo (Agentes)                               │  │
│  │  - Alertas automáticas                               │  │
│  │  - Webhooks para AAP                                 │  │
│  │  - Integración bidireccional                         │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
                            │
                    (Eventos / Métricas)
                            │
                  ┌─────────┴─────────┐
                  │                   │
        ┌─────────────────┐  ┌──────────────────┐
        │  Hosts Linux    │  │  Hosts Windows   │
        │  - Agentes      │  │  - Agentes       │
        │  - Monitores    │  │  - Monitores     │
        └─────────────────┘  └──────────────────┘
```

---

## Requisitos Previos

### Máquina Windows

| Requisito | Versión | Notas |
|-----------|---------|-------|
| **Windows** | 10 Pro/Enterprise o 11 | Build 19041+ |
| **RAM** | 8 GB mínimo (16 GB recomendado) | Para AAP + Zabbix |
| **Almacenamiento** | 50 GB libre | Para WSL2 y contenedores |
| **Procesador** | Soporta virtualización | Habilitar Hyper-V |

### Requisitos Técnicos

- WSL2 habilitado
- Docker Desktop o Podman
- Acceso administrativo a Windows
- Conexión de red (saliente puertos 443, 10050, 10051)

### Software a Instalar

```
Sistema:
├── Windows Subsystem for Linux 2 (WSL2)
├── Ubuntu 22.04 LTS (en WSL2)
├── Docker Desktop (Windows)
└── PowerShell 7+ (opcional)

En WSL2:
├── Python 3.9+
├── PostgreSQL 13+
├── Ansible Automation Platform
├── Zabbix Agent
└── curl, git, jq
```

---

## Parte 1: Preparación del Entorno WSL2

### 1.1 Habilitar WSL2 en Windows

**Paso 1: Abrir PowerShell como Administrador**

```powershell
# Presionar Win+X y seleccionar "Windows PowerShell (Admin)"
# O ejecutar:
Start-Process powershell -Verb RunAs
```

**Paso 2: Ejecutar comandos de instalación**

```powershell
# Habilitar WSL2
wsl --install

# Si WSL ya está instalado, actualizar a WSL2
wsl --update

# Verificar versión
wsl --version
# Debe mostrar: WSL version: 2.x.x
```

**Paso 3: Reiniciar Windows**

```powershell
Restart-Computer
```

### 1.2 Instalar Distribución Linux

**Paso 1: Descargar Ubuntu 22.04 LTS**

```powershell
# Listar distribuciones disponibles
wsl --list --online

# Instalar Ubuntu 22.04 LTS
wsl --install -d Ubuntu-22.04

# Verificar instalación
wsl --list --verbose
```

**Paso 2: Iniciar WSL2 e Configurar Usuario**

```powershell
# Iniciar WSL2
wsl

# Dentro de WSL2:
# Se solicitará crear un usuario (ejemplo: ansible)
# Contraseña: (crear una segura)
```

### 1.3 Configurar WSL2

**Crear archivo de configuración:** `C:\Users\<TuUsuario>\.wslconfig`

```ini
[wsl2]
# Memoria máxima para WSL2
memory=8GB

# Procesadores disponibles
processors=4

# Almacenamiento
swap=4GB

# Interop con Windows
interopEnabled=true
interopPrependedPath=true

# Montaje de unidades Windows
appendWindowsPath=true

# Red
networkingMode=mirrored
```

**Reiniciar WSL2:**

```powershell
wsl --shutdown
# Esperar 30 segundos
wsl
```

### 1.4 Preparar Ubuntu en WSL2

**Abrir terminal de WSL2:**

```bash
# Actualizar lista de paquetes
sudo apt update && sudo apt upgrade -y

# Instalar dependencias básicas
sudo apt install -y \
    build-essential \
    curl \
    wget \
    git \
    python3.10 \
    python3.10-dev \
    python3.10-venv \
    python3-pip \
    postgresql \
    postgresql-contrib \
    libpq-dev \
    ssl-cert \
    openssl \
    net-tools \
    jq \
    vim \
    nano \
    htop \
    systemd

# Verificar versiones instaladas
python3 --version
pip3 --version
postgres --version
```

### 1.5 Instalar Docker en WSL2

**Opción A: Docker Desktop para Windows**

```bash
# En Windows (PowerShell Admin):
# 1. Descargar Docker Desktop: https://www.docker.com/products/docker-desktop
# 2. Instalar normalmente
# 3. En Docker Desktop Settings:
#    - Resources -> WSL Integration
#    - Habilitar "Ubuntu-22.04"
```

**Opción B: Docker dentro de WSL2**

```bash
# En WSL2:
sudo apt install -y docker.io docker-compose

# Agregar usuario actual al grupo docker
sudo usermod -aG docker $USER

# Iniciar servicio
sudo systemctl start docker
sudo systemctl enable docker

# Verificar instalación
docker --version
```

### 1.6 Configurar PostgreSQL en WSL2

```bash
# Iniciar servicio PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Verificar estado
sudo systemctl status postgresql

# Crear usuario y base de datos para AAP
sudo sudo -u postgres psql << EOF
CREATE USER awx WITH ENCRYPTED PASSWORD 'awx_secure_password';
CREATE DATABASE awx OWNER awx;
ALTER ROLE awx WITH CREATEDB;
GRANT ALL PRIVILEGES ON DATABASE awx TO awx;
\q
EOF

# Verificar conexión
psql -h localhost -U awx -d awx -c "SELECT version();"
```

**Archivo: `/etc/postgresql/14/main/postgresql.conf`**

```ini
# Configurar para acceso desde Windows
listen_addresses = '*'
port = 5432
```

**Archivo: `/etc/postgresql/14/main/pg_hba.conf`**

Agregar al final:

```ini
# Para conexiones desde Windows
host    awx    awx    0.0.0.0/0    md5
```

**Reiniciar PostgreSQL:**

```bash
sudo systemctl restart postgresql
```

---

## Parte 2: Instalación de Ansible Automation Platform

### 2.1 Descargar Instalador de AAP

```bash
# Crear directorio de trabajo
mkdir -p ~/aap-installation
cd ~/aap-installation

# Descargar AAP (requiere cuenta Red Hat)
# Opción 1: Desde Red Hat Portal (descargar manualmente)
# https://access.redhat.com/downloads/content/ansible-automation-platform/

# Opción 2: Usando wget (si tienes credenciales)
wget --user=your_rh_username --password=your_rh_password \
  https://access.redhat.com/downloads/content/ansible-automation-platform/2.4/rhel-8/x86_64/product-software/ansible-automation-platform-setup-latest.tar.gz

# O usar versión de prueba comunitaria
wget https://releases.ansible.com/ansible-tower/setup-latest.tar.gz
tar -xzf ansible-tower-setup-latest.tar.gz
cd ansible-tower-setup-*
```

### 2.2 Preparar Archivo de Inventario

**Crear archivo:** `~/aap-installation/inventory.ini`

```ini
[all:vars]
# Red Hat Subscription (opcional)
rh_username=your_rh_username
rh_password=your_rh_password

# PostgreSQL
automationcontroller_pg_host=localhost
automationcontroller_pg_port=5432
automationcontroller_pg_database=awx
automationcontroller_pg_username=awx
automationcontroller_pg_password=awx_secure_password
automationcontroller_pg_sslmode=prefer

# AAP Admin
automationcontroller_admin_username=admin
automationcontroller_admin_password=admin_secure_password_here
automationcontroller_admin_email=admin@localhost

# SSL Certificates (generar antes)
automationcontroller_ssl_cert=/etc/pki/tls/certs/controller.crt
automationcontroller_ssl_key=/etc/pki/tls/private/controller.key

# Host configuration
automationcontroller_hostname=localhost
automationcontroller_http_port=80
automationcontroller_https_port=443

# Networking
automationcontroller_create_preload_data=true

[automationcontroller]
localhost ansible_connection=local

[automationhub]
# Comentar si no se usará

[database]
# Comentar para usar PostgreSQL externo
```

### 2.3 Generar Certificados SSL Auto-Firmados

```bash
# Crear directorio para certificados
sudo mkdir -p /etc/pki/tls/certs /etc/pki/tls/private

# Generar clave privada
sudo openssl genrsa -out /etc/pki/tls/private/controller.key 2048

# Generar certificado autofirmado (válido 365 días)
sudo openssl req -new -x509 -key /etc/pki/tls/private/controller.key \
  -out /etc/pki/tls/certs/controller.crt -days 365 \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=localhost"

# Configurar permisos
sudo chmod 600 /etc/pki/tls/private/controller.key
sudo chmod 644 /etc/pki/tls/certs/controller.crt

# Verificar certificado
sudo openssl x509 -in /etc/pki/tls/certs/controller.crt -text -noout
```

### 2.4 Instalación de AAP

```bash
cd ~/aap-installation/ansible-tower-setup-*

# Ejecutar instalador (puede tomar 30-60 minutos)
sudo bash setup.sh -i inventory.ini

# El instalador realizará:
# - Instalación de dependencias
# - Configuración de base de datos
# - Inicialización de AAP
# - Configuración de servicios systemd
```

**Monitorear instalación:**

```bash
# En otra terminal, ver logs
sudo tail -f /var/log/tower/setup.log

# Verificar servicios
sudo systemctl status automation-controller
sudo systemctl status automation-hub
```

### 2.5 Validar Instalación

```bash
# Verificar que los servicios estén corriendo
sudo systemctl status automation-controller
sudo systemctl status automation-hub

# Verificar conectividad a la API
curl -k https://localhost/api/v2/ \
  -u admin:admin_secure_password_here | jq .

# Obtener token de API
curl -k https://localhost/api/v2/auth/token/ \
  -d '{"username":"admin","password":"admin_secure_password_here"}' \
  -H 'Content-Type: application/json' | jq .

# Acceder a la interfaz web
# https://localhost (con navegador)
# Usuario: admin
# Contraseña: admin_secure_password_here
```

### 2.6 Configuración Post-Instalación

**Permitir acceso desde Windows:**

```bash
# Editar configuración de AAP
sudo nano /etc/tower/settings.py

# Agregar/modificar:
ALLOWED_HOSTS = ['*']
CSRF_TRUSTED_ORIGINS = ['https://127.0.0.1', 'https://localhost']
```

**Configurar puerto en WSL2:**

```bash
# En PowerShell (Windows), agregar regla de firewall
# Permitir que Windows acceda a WSL2
netsh advfirewall firewall add rule name="WSL2 AAP" \
  dir=in action=allow protocol=tcp \
  localport=443 remoteip=127.0.0.1,172.16.0.0/12

# Acceso desde Windows: https://127.0.0.1
```

---

## Parte 3: Configuración de Zabbix

### 3.1 Opciones de Instalación de Zabbix

**Opción A: Zabbix Server en Host Windows (Recomendado)**

```powershell
# Descargar Zabbix para Windows
# https://www.zabbix.com/download?os_group=windows

# Instalar normalmente como aplicación Windows
# El servidor Zabbix se ejecutará como servicio Windows
```

**Opción B: Zabbix Server en Contenedor Docker**

```bash
# En WSL2, crear docker-compose.yml

mkdir -p ~/zabbix-docker
cat > ~/zabbix-docker/docker-compose.yml << 'EOF'
version: '3.8'

services:
  zabbix-mysql:
    image: mysql:8.0
    container_name: zabbix-mysql
    environment:
      MYSQL_ROOT_PASSWORD: zabbix_root_pass
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - zabbix-network

  zabbix-server:
    image: zabbix/zabbix-server-mysql:latest
    container_name: zabbix-server
    environment:
      DB_SERVER_HOST: zabbix-mysql
      MYSQL_ROOT_PASSWORD: zabbix_root_pass
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
      ZBX_LISTENIP: 0.0.0.0
    ports:
      - "10051:10051"
    depends_on:
      - zabbix-mysql
    networks:
      - zabbix-network

  zabbix-frontend:
    image: zabbix/zabbix-web-nginx-mysql:latest
    container_name: zabbix-frontend
    environment:
      DB_SERVER_HOST: zabbix-mysql
      MYSQL_ROOT_PASSWORD: zabbix_root_pass
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
      ZBX_SERVER_HOST: zabbix-server
      ZBX_SERVER_PORT: 10051
    ports:
      - "8080:8080"
    depends_on:
      - zabbix-server
    networks:
      - zabbix-network

volumes:
  mysql_data:

networks:
  zabbix-network:
    driver: bridge
EOF

# Iniciar servicios
cd ~/zabbix-docker
docker-compose up -d

# Verificar servicios
docker-compose ps

# Acceso a Zabbix Frontend: http://localhost:8080
# Usuario: Admin
# Contraseña: zabbix
```

### 3.2 Instalar Agente Zabbix en WSL2

```bash
# Descargar repositorio Zabbix
cd /tmp
wget https://repo.zabbix.com/zabbix-official-repo.key
apt-key add zabbix-official-repo.key

# Instalar agente Zabbix
apt install -y zabbix-agent

# Configurar agente
sudo nano /etc/zabbix/zabbix_agentd.conf

# Modificar:
Server=127.0.0.1
ServerActive=127.0.0.1
Hostname=WSL2-AAP-Host

# Iniciar servicio
sudo systemctl start zabbix-agent
sudo systemctl enable zabbix-agent

# Verificar estado
sudo systemctl status zabbix-agent
```

### 3.3 Registrar Host en Zabbix

**En la interfaz web de Zabbix:**

1. Ir a: **Configuration** → **Hosts**
2. Hacer clic en **Create host**
3. Llenar datos:
   - **Host name:** WSL2-AAP-Host
   - **Visible name:** WSL2 - Ansible Automation Platform
   - **Groups:** Linux servers
   - **Interfaces:** Agregar
     - **Type:** Agent
     - **IP address:** 127.0.0.1 (o IP de WSL2)
     - **Port:** 10050

4. En la pestaña **Templates:**
   - Agregar: `Template OS Linux`
   - Agregar: `Template App Zabbix Agent`

5. Hacer clic en **Add**

### 3.4 Crear Items Personalizados en Zabbix

**Para monitorear AAP, crear items en Zabbix:**

```bash
# 1. Acceder a Zabbix Web
# 2. Configuration → Hosts → WSL2-AAP-Host → Items

# Item 1: Verificar estado de AAP API
# - Name: AAP API Status
# - Type: HTTP agent
# - URL: https://127.0.0.1/api/v2/
# - Username: admin
# - Password: {$AAP_PASSWORD}
# - Post data: {"username":"admin","password":"{$AAP_PASSWORD}"}

# Item 2: Verificar CPU usage
# - Name: CPU usage
# - Type: Agent
# - Key: system.cpu.load[all,avg5]
# - Update interval: 30s

# Item 3: Verificar memoria
# - Name: Memory used
# - Type: Agent
# - Key: vm.memory.size[used]
# - Update interval: 30s
```

---

## Parte 4: Integración AAP con Zabbix

### 4.1 Crear Token de API en AAP

**En la interfaz de AAP:**

1. Ir a: **Administration** → **Users**
2. Seleccionar usuario **admin**
3. Ir a pestaña: **Tokens**
4. Hacer clic en **Add**
   - **Description:** Zabbix Integration Token
   - **Scope:** Write (para ejecutar playbooks)

5. Copiar y guardar el token (no se mostrará de nuevo)

**Desde CLI en WSL2:**

```bash
# Obtener token de API
curl -k https://127.0.0.1/api/v2/auth/token/ \
  -d '{"username":"admin","password":"your_admin_password"}' \
  -H 'Content-Type: application/json' -s | jq -r '.token'

# Guardar token en variable
export AAP_TOKEN="your_token_here"
export AAP_HOST="https://127.0.0.1"
```

### 4.2 Crear User y API Token para Zabbix

**Script:** `~/aap-setup/create-zabbix-user.yml`

```yaml
---
- name: Crear usuario y token para Zabbix en AAP
  hosts: localhost
  gather_facts: no
  vars:
    controller_host: "https://127.0.0.1"
    controller_username: "admin"
    controller_password: "your_admin_password"
    controller_verify_ssl: false

  tasks:
    - name: Crear usuario para Zabbix
      awx.awx.user:
        username: "zabbix_automation"
        password: "zabbix_secure_password"
        email: "zabbix@domain.local"
        first_name: "Zabbix"
        last_name: "Automation"
        is_superuser: false
        is_staff: true
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"
        controller_verify_ssl: "{{ controller_verify_ssl }}"

    - name: Crear token de API para Zabbix
      awx.awx.token:
        description: "Token para integración Zabbix"
        user: "zabbix_automation"
        scope: "write"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"
        controller_verify_ssl: "{{ controller_verify_ssl }}"
      register: zabbix_token

    - name: Mostrar token generado
      debug:
        var: zabbix_token.token
```

**Ejecutar playbook:**

```bash
cd ~/aap-setup
ansible-playbook create-zabbix-user.yml -e "ansible_python_interpreter=/usr/bin/python3"
```

### 4.3 Configurar Webhook en Zabbix

**En Zabbix Web, ir a:** **Administration** → **Media types**

**Crear nuevo Media Type:**

1. Hacer clic en **Create media type**
2. **Name:** AAP Webhook
3. **Type:** Webhook
4. **Script:** Copiar código siguiente

**Script de Webhook:**

```javascript
var params = JSON.parse(value);
var request = new HttpRequest();
var url = params.aap_host + '/api/v2/job_templates/';
var headers = {};

// Autenticación con token
headers['Authorization'] = 'Bearer ' + params.aap_token;
headers['Content-Type'] = 'application/json';

// Datos para disparar el playbook
var data = {
  'name': params.job_template_name,
  'extra_vars': {
    'zabbix_alert_subject': params.alert_subject,
    'zabbix_alert_message': params.alert_message,
    'zabbix_host': params.host,
    'zabbix_item': params.item,
    'zabbix_value': params.value,
    'zabbix_severity': params.severity
  }
};

// Realizar request POST
request.addHeader('Authorization', headers['Authorization']);
request.addHeader('Content-Type', headers['Content-Type']);

var response = request.post(url, JSON.stringify(data));

if (request.getStatus() != 201 && request.getStatus() != 200) {
  return 'Error: ' + request.getStatus() + ' - ' + response;
}

return 'Job submitted to AAP successfully';
```

**Parámetros del Webhook:**

```
aap_host: https://127.0.0.1
aap_token: your_zabbix_token_here
job_template_name: {EVENT.TAGS.__aap_template}
alert_subject: {ALERT.SUBJECT}
alert_message: {ALERT.MESSAGE}
host: {HOST.NAME}
item: {ITEM.NAME}
value: {ITEM.VALUE}
severity: {EVENT.SEVERITY}
```

### 4.4 Crear Acciones en Zabbix

**En Zabbix Web, ir a:** **Configuration** → **Actions** → **Create action**

**Acción 1: Auto-remediación de CPU alta**

```
Name: Auto-remediate High CPU
Condition: Host name is WSL2-AAP-Host AND Problem is CPU usage > 80%
Operations:
  Send to Media type: AAP Webhook
  Default message subject: High CPU detected on {HOST.NAME}
  Message: CPU usage is {ITEM.VALUE}% on host {HOST.NAME}. Running remediation playbook.
  
Custom event tags (para enlazar con job template):
  __aap_template: remediate-high-cpu
```

**Crear el playbook en AAP:**

Archivo: `~/playbooks/remediate-high-cpu.yml`

```yaml
---
- name: Remediación automática - CPU alta
  hosts: localhost
  gather_facts: no
  vars:
    zabbix_alert_subject: "{{ zabbix_alert_subject | default('CPU alert') }}"
    zabbix_alert_message: "{{ zabbix_alert_message | default('') }}"
    affected_host: "{{ zabbix_host | default('unknown') }}"

  tasks:
    - name: Registrar alerta en logs
      debug:
        msg: |
          Alert recibida desde Zabbix:
          Host: {{ affected_host }}
          Asunto: {{ zabbix_alert_subject }}
          Mensaje: {{ zabbix_alert_message }}

    - name: Obtener procesos con alto CPU
      shell: ps aux --sort=-%cpu | head -6
      register: high_cpu_processes

    - name: Mostrar procesos con alto CPU
      debug:
        msg: "{{ high_cpu_processes.stdout_lines }}"

    - name: Intentar liberar memoria
      shell: |
        sync
        echo 3 > /proc/sys/vm/drop_caches
      become: yes
      ignore_errors: yes

    - name: Reiniciar servicios de aplicación (ejemplo)
      systemd:
        name: "{{ item }}"
        state: restarted
      loop:
        - automation-controller
      become: yes
      ignore_errors: yes

    - name: Enviar notificación de remediación
      debug:
        msg: "Remediación completada para CPU alta en {{ affected_host }}"
```

---

## Parte 5: Automatización Basada en Eventos

### 5.1 Estructura de Playbooks para Eventos

**Directorio de proyectos:**

```
~/playbooks/
├── remediation/
│   ├── remediate-high-cpu.yml
│   ├── remediate-high-memory.yml
│   ├── remediate-disk-full.yml
│   └── remediate-service-down.yml
├── monitoring/
│   ├── collect-system-metrics.yml
│   ├── generate-incident-report.yml
│   └── escalate-critical-alert.yml
├── incident-response/
│   ├── incident-triage.yml
│   ├── isolate-compromised-host.yml
│   └── restore-from-backup.yml
└── group_vars/
    └── all.yml
```

### 5.2 Playbook: Respuesta Automática a Alertas

**Archivo:** `~/playbooks/event-handler.yml`

```yaml
---
- name: Manejador Central de Eventos Zabbix
  hosts: localhost
  gather_facts: no
  vars:
    # Variables de la alerta (enviadas por Zabbix)
    alert_severity: "{{ event_severity | default('unknown') }}"
    alert_host: "{{ zabbix_host | default('unknown') }}"
    alert_type: "{{ event_type | default('unknown') }}"
    alert_value: "{{ event_value | default('') }}"
    
    # Configuración
    log_dir: "/var/log/aap-events"
    max_auto_remediation_severity: "average"

  pre_tasks:
    - name: Crear directorio de logs
      file:
        path: "{{ log_dir }}"
        state: directory
        mode: '0755'

    - name: Registrar evento
      lineinfile:
        path: "{{ log_dir }}/events.log"
        line: "[{{ ansible_date_time.iso8601 }}] Severity: {{ alert_severity }}, Host: {{ alert_host }}, Type: {{ alert_type }}, Value: {{ alert_value }}"
        create: yes

    - name: Mostrar detalles de alerta
      debug:
        msg: |
          ===== EVENTO ZABBIX RECIBIDO =====
          Timestamp: {{ ansible_date_time.iso8601 }}
          Host: {{ alert_host }}
          Tipo de Alerta: {{ alert_type }}
          Severidad: {{ alert_severity }}
          Valor: {{ alert_value }}
          ================================

  tasks:
    # Routing de eventos basado en tipo
    
    - name: Procesar eventos de Performance
      include_tasks: handlers/performance-handler.yml
      when: 'alert_type in ["cpu_high", "memory_high", "disk_high"]'

    - name: Procesar eventos de Disponibilidad
      include_tasks: handlers/availability-handler.yml
      when: 'alert_type in ["service_down", "host_unreachable"]'

    - name: Procesar eventos de Seguridad
      include_tasks: handlers/security-handler.yml
      when: 'alert_type in ["unauthorized_access", "port_scan", "malware_detected"]'

    - name: Procesar eventos de Aplicación
      include_tasks: handlers/application-handler.yml
      when: 'alert_type in ["app_error", "database_error", "backup_failed"]'

  post_tasks:
    - name: Generar resumen de acciones
      template:
        src: event-summary.j2
        dest: "{{ log_dir }}/event-{{ ansible_date_time.iso8601_basic }}.html"

    - name: Enviar notificación de evento procesado
      mail:
        host: "{{ smtp_host | default('localhost') }}"
        port: "{{ smtp_port | default('25') }}"
        from: "aap-automation@domain.local"
        to: "{{ event_notification_email }}"
        subject: "[{{ alert_severity }}] Evento procesado en {{ alert_host }}"
        body: |
          Evento de Zabbix procesado automáticamente.
          
          Host: {{ alert_host }}
          Tipo: {{ alert_type }}
          Severidad: {{ alert_severity }}
          Acción: {{ action_taken | default('Ninguna acción requerida') }}
          
          Más detalles en: {{ log_dir }}/event-{{ ansible_date_time.iso8601_basic }}.html
      when: event_notification_enabled | default(true)
      delegate_to: localhost
      ignore_errors: yes
```

### 5.3 Handlers de Eventos

**Archivo:** `~/playbooks/handlers/performance-handler.yml`

```yaml
---
- name: Handler de eventos de Performance
  block:
    - name: CPU Alta - Análisis y remediación
      block:
        - name: Obtener estado actual de CPU
          shell: cat /proc/loadavg | awk '{print $1, $2, $3}'
          register: cpu_load
          changed_when: false

        - name: Registrar CPU load
          debug:
            msg: "CPU Load: {{ cpu_load.stdout }}"

        - name: Ejecutar remediación si está habilitada
          include_tasks: remediation/remediate-high-cpu.yml
          when:
            - auto_remediation_enabled | default(true)
            - alert_severity in ["average", "high", "disaster"]

      when: alert_type == "cpu_high"

    - name: Memoria Alta - Análisis y remediación
      block:
        - name: Obtener estado de memoria
          shell: free -m | awk 'NR==2{printf("%.2f", $3/$2 * 100.0)}'
          register: memory_usage
          changed_when: false

        - name: Registrar uso de memoria
          debug:
            msg: "Memory Usage: {{ memory_usage.stdout }}%"

        - name: Ejecutar remediación
          include_tasks: remediation/remediate-high-memory.yml
          when:
            - auto_remediation_enabled | default(true)
            - memory_usage.stdout | float > 80

      when: alert_type == "memory_high"

    - name: Disco Lleno - Análisis y remediación
      block:
        - name: Obtener uso de disco
          shell: df -h / | awk 'NR==2{print $5}'
          register: disk_usage
          changed_when: false

        - name: Registrar uso de disco
          debug:
            msg: "Disk Usage: {{ disk_usage.stdout }}"

        - name: Ejecutar limpieza
          include_tasks: remediation/remediate-disk-full.yml
          when:
            - auto_remediation_enabled | default(true)
            - disk_usage.stdout | regex_replace('%', '') | int > 85

      when: alert_type == "disk_high"

  rescue:
    - name: Registrar error en handler
      debug:
        msg: "Error en performance handler: {{ ansible_failed_result.msg }}"
      failed_when: false
```

**Archivo:** `~/playbooks/handlers/availability-handler.yml`

```yaml
---
- name: Handler de eventos de Disponibilidad
  block:
    - name: Servicio No Disponible
      block:
        - name: Obtener estado del servicio
          shell: systemctl status "{{ service_name | default('automation-controller') }}" | grep Active
          register: service_status
          changed_when: false
          failed_when: false

        - name: Intentar reiniciar servicio
          systemd:
            name: "{{ service_name | default('automation-controller') }}"
            state: restarted
          become: yes
          when:
            - auto_remediation_enabled | default(true)
            - alert_severity in ["high", "disaster"]

        - name: Esperar a que el servicio inicie
          wait_for:
            delay: 5
            timeout: 60
            port: "{{ service_port | default(443) }}"
            host: localhost

        - name: Verificar servicio nuevamente
          uri:
            url: "https://localhost/api/v2/"
            validate_certs: no
            ignore_errors: yes
          register: service_check

      when: alert_type == "service_down"

    - name: Host No Accesible
      block:
        - name: Intentar ping al host
          shell: ping -c 1 "{{ alert_host }}" 2>/dev/null
          register: ping_result
          failed_when: false
          changed_when: false

        - name: Crear ticket de soporte si no responde
          uri:
            url: "{{ ticketing_system_url }}/api/tickets"
            method: POST
            body_format: json
            body:
              title: "Host no accesible: {{ alert_host }}"
              description: "El host {{ alert_host }} no responde a conectividad"
              priority: "high"
              category: "infrastructure"
          when: ping_result.rc != 0

      when: alert_type == "host_unreachable"

  rescue:
    - name: Registrar error en availability handler
      debug:
        msg: "Error en availability handler: {{ ansible_failed_result.msg }}"
      failed_when: false
```

### 5.4 Crear Job Template en AAP

**En AAP Web Interface:**

1. Ir a: **Resources** → **Templates** → **Create template** → **Job template**

2. **Configuración:**
   - **Name:** Zabbix Event Handler
   - **Description:** Maneja eventos automáticos de Zabbix
   - **Job type:** Run
   - **Inventory:** Localhost
   - **Project:** (seleccionar proyecto que contiene playbooks)
   - **Playbook:** event-handler.yml
   - **Credentials:** (seleccionar Machine credential para localhost)
   - **Verbosity:** 1 (Basic)

3. **Variables:**
   
   En la pestaña **Variables**, agregar:
   
   ```yaml
   event_severity: "{{ zabbix_alert_severity | default('unknown') }}"
   event_type: "{{ zabbix_alert_type | default('unknown') }}"
   event_value: "{{ zabbix_alert_value | default('') }}"
   zabbix_host: "{{ zabbix_host | default('unknown') }}"
   auto_remediation_enabled: true
   event_notification_enabled: true
   smtp_host: "smtp.domain.local"
   event_notification_email: "ops-team@domain.local"
   ```

4. Hacer clic en **Save**

### 5.5 Habilitar Webhook Payload en AAP

**Crear script de webhook:** `/opt/aap/webhook-receiver.py`

```python
#!/usr/bin/env python3
"""
Script para recibir webhooks de Zabbix y disparar job templates en AAP
"""

from flask import Flask, request, jsonify
import requests
import json
import os
from datetime import datetime

app = Flask(__name__)

# Configuración
AAP_HOST = os.environ.get('AAP_HOST', 'https://127.0.0.1')
AAP_TOKEN = os.environ.get('AAP_TOKEN', '')
AAP_VERIFY_SSL = os.environ.get('AAP_VERIFY_SSL', 'false').lower() == 'true'

# Mapeo de tipos de evento a job templates
EVENT_MAPPING = {
    'cpu_high': 1,
    'memory_high': 2,
    'disk_high': 3,
    'service_down': 4,
    'host_unreachable': 5,
    'unauthorized_access': 6,
    'app_error': 7,
    'database_error': 8,
}

@app.route('/webhook/zabbix', methods=['POST'])
def zabbix_webhook():
    """
    Recibe webhook de Zabbix y dispara job template en AAP
    """
    try:
        data = request.get_json()
        
        # Validar datos
        if not data:
            return jsonify({'error': 'No data received'}), 400
        
        # Extraer información del evento
        alert_severity = data.get('severity', 'unknown')
        alert_type = data.get('event_type', 'unknown')
        alert_host = data.get('host', 'unknown')
        alert_value = data.get('value', '')
        alert_subject = data.get('subject', 'Zabbix Alert')
        alert_message = data.get('message', '')
        
        # Obtener template ID
        job_template_id = EVENT_MAPPING.get(alert_type)
        
        if not job_template_id:
            return jsonify({
                'warning': f'No template mapping for event type: {alert_type}'
            }), 202
        
        # Preparar datos para AAP
        extra_vars = {
            'zabbix_alert_severity': alert_severity,
            'zabbix_alert_type': alert_type,
            'zabbix_host': alert_host,
            'zabbix_alert_value': alert_value,
            'zabbix_alert_subject': alert_subject,
            'zabbix_alert_message': alert_message,
        }
        
        # Disparar job template
        headers = {
            'Authorization': f'Bearer {AAP_TOKEN}',
            'Content-Type': 'application/json',
        }
        
        url = f'{AAP_HOST}/api/v2/job_templates/{job_template_id}/launch/'
        
        payload = {
            'extra_vars': json.dumps(extra_vars),
        }
        
        response = requests.post(
            url,
            json=payload,
            headers=headers,
            verify=AAP_VERIFY_SSL
        )
        
        if response.status_code in [200, 201]:
            job_data = response.json()
            return jsonify({
                'status': 'success',
                'job_id': job_data.get('id'),
                'message': f'Job launched for event: {alert_type}'
            }), 201
        else:
            return jsonify({
                'error': 'Failed to launch job',
                'status_code': response.status_code,
                'details': response.text
            }), response.status_code
            
    except Exception as e:
        return jsonify({
            'error': str(e)
        }), 500

@app.route('/health', methods=['GET'])
def health_check():
    """Health check endpoint"""
    return jsonify({'status': 'healthy'}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False, ssl_context='adhoc')
```

**Instalar y ejecutar servidor de webhook:**

```bash
# Instalar dependencias
pip3 install flask requests pyopenssl

# Crear servicio systemd
sudo tee /etc/systemd/system/aap-webhook-receiver.service << EOF
[Unit]
Description=AAP Webhook Receiver for Zabbix
After=network.target

[Service]
Type=simple
User=ansible
WorkingDirectory=/opt/aap
Environment="AAP_HOST=https://127.0.0.1"
Environment="AAP_TOKEN=your_api_token_here"
Environment="AAP_VERIFY_SSL=false"
ExecStart=/usr/bin/python3 /opt/aap/webhook-receiver.py
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Crear directorio
sudo mkdir -p /opt/aap
sudo cp webhook-receiver.py /opt/aap/
sudo chown -R ansible:ansible /opt/aap

# Habilitar y iniciar servicio
sudo systemctl daemon-reload
sudo systemctl enable aap-webhook-receiver
sudo systemctl start aap-webhook-receiver

# Verificar estado
sudo systemctl status aap-webhook-receiver
```

---

## Parte 6: Validación y Pruebas

### 6.1 Pruebas de Conectividad

```bash
# Test 1: Verificar AAP API
curl -k https://127.0.0.1/api/v2/ -u admin:password | jq .

# Test 2: Verificar Zabbix Server
zabbix_get -s 127.0.0.1 -k system.cpu.load[all,avg5]

# Test 3: Verificar PostgreSQL
psql -h localhost -U awx -d awx -c "SELECT version();"

# Test 4: Verificar Docker
docker ps

# Test 5: Verificar Webhook Receiver
curl -k https://127.0.0.1:5000/health | jq .
```

### 6.2 Prueba de Trigger Manual

**Script de prueba:** `~/test-integration.sh`

```bash
#!/bin/bash

echo "=== Test de Integración AAP + Zabbix ==="

# Variables
AAP_HOST="https://127.0.0.1"
AAP_TOKEN="your_token_here"
WEBHOOK_URL="https://127.0.0.1:5000/webhook/zabbix"

# Test 1: Crear evento de prueba
echo -e "\n[1] Enviando evento de prueba a webhook..."

EVENT_PAYLOAD='{
  "severity": "average",
  "event_type": "cpu_high",
  "host": "test-host",
  "value": "85%",
  "subject": "CPU usage is high",
  "message": "CPU usage on test-host is 85%"
}'

RESPONSE=$(curl -k -X POST \
  -H "Content-Type: application/json" \
  -d "$EVENT_PAYLOAD" \
  $WEBHOOK_URL)

echo "Response: $RESPONSE"

# Test 2: Listar jobs en ejecución
echo -e "\n[2] Jobs en ejecución en AAP..."

curl -k -H "Authorization: Bearer $AAP_TOKEN" \
  "$AAP_HOST/api/v2/jobs/?status=running" | jq '.results[] | {id, name, status}'

# Test 3: Obtener historial de jobs
echo -e "\n[3] Últimos 5 jobs ejecutados..."

curl -k -H "Authorization: Bearer $AAP_TOKEN" \
  "$AAP_HOST/api/v2/jobs/?limit=5&order_by=-id" | \
  jq '.results[] | {id, name, status, created}'

# Test 4: Verificar logs
echo -e "\n[4] Últimas líneas de logs..."

tail -20 /var/log/aap-events/events.log

echo -e "\n=== Test completado ==="
```

**Ejecutar prueba:**

```bash
chmod +x ~/test-integration.sh
~/test-integration.sh
```

### 6.3 Monitorear Ejecuciones

```bash
# Ver jobs activos
watch -n 1 'curl -k -s -H "Authorization: Bearer $AAP_TOKEN" \
  "https://127.0.0.1/api/v2/jobs/?status=running" | jq ".count"'

# Ver logs del webhook receiver
sudo journalctl -u aap-webhook-receiver -f

# Ver logs de AAP
sudo tail -f /var/log/tower/callback_receiver.log

# Ver eventos procesados
tail -f /var/log/aap-events/events.log
```

---

## Parte 7: Configuración de Ejemplos Avanzados

### 7.1 Escalado Automático Basado en Eventos

**Archivo:** `~/playbooks/auto-scaling.yml`

```yaml
---
- name: Auto-scaling basado en alertas Zabbix
  hosts: localhost
  gather_facts: no
  vars:
    vm_platform: "vmware"  # vmware, kvm, hyperv
    scale_threshold: 80
    scale_up_count: 1
    scale_down_count: 1

  tasks:
    - name: Analizar evento de escalado
      debug:
        msg: |
          Host: {{ zabbix_host }}
          Métrica: {{ zabbix_metric }}
          Valor Actual: {{ zabbix_value }}
          Acción: {% if scale_action == 'up' %}Escalar arriba{% else %}Escalar abajo{% endif %}

    - name: Escalar arriba - Crear nuevas VMs
      block:
        - name: Clonar template de VM
          shell: |
            virt-clone --name prod-app-{{ ansible_date_time.iso8601_basic }} \
              --original app-template \
              --auto-clone
          when: vm_platform == "kvm"

        - name: Configurar VM nueva
          include_tasks: configure-new-vm.yml

        - name: Agregar a balanceador de carga
          include_tasks: add-to-load-balancer.yml

      when: scale_action == "up"

    - name: Escalar abajo - Eliminar VMs
      block:
        - name: Drenar conexiones
          shell: |
            # Lógica para drenar conexiones
            echo "Drenando conexiones de {{ scale_target_vm }}"

        - name: Eliminar VM
          shell: |
            virsh shutdown {{ scale_target_vm }}
            virsh undefine {{ scale_target_vm }}
          when: vm_platform == "kvm"

        - name: Remover del balanceador de carga
          include_tasks: remove-from-load-balancer.yml

      when: scale_action == "down"

    - name: Notificar resultado del escalado
      uri:
        url: "{{ zabbix_webhook_url }}"
        method: POST
        body_format: json
        body:
          status: "scaling_completed"
          action: "{{ scale_action }}"
          vms_affected: "{{ vms_created | default(vms_removed) }}"
          timestamp: "{{ ansible_date_time.iso8601 }}"
```

### 7.2 Respuesta a Incidentes de Seguridad

**Archivo:** `~/playbooks/security-incident-response.yml`

```yaml
---
- name: Respuesta automática a incidentes de seguridad
  hosts: localhost
  gather_facts: no
  vars:
    incident_severity: "{{ zabbix_severity | default('unknown') }}"
    affected_host: "{{ zabbix_host }}"
    incident_type: "{{ security_incident_type }}"

  tasks:
    - name: Crear ticket de incidente
      uri:
        url: "{{ incident_management_url }}/api/incidents"
        method: POST
        body_format: json
        body:
          title: "[SECURITY] {{ incident_type }} on {{ affected_host }}"
          description: "Automatic incident created from Zabbix alert"
          severity: "{{ incident_severity }}"
          assigned_to: "security-team"
          tags:
            - "auto-created"
            - "security"
            - "{{ incident_type }}"

    - name: Aislar host comprometido
      include_tasks: isolate-host.yml
      when: incident_type == "malware_detected"

    - name: Ejecutar análisis forense
      include_tasks: forensic-analysis.yml
      when: incident_severity in ["high", "critical"]

    - name: Restaurar desde backup
      include_tasks: restore-from-backup.yml
      when: incident_type == "ransomware_detected"

    - name: Notificar a CISO
      mail:
        host: "{{ smtp_host }}"
        port: "{{ smtp_port }}"
        from: "aap-security@domain.local"
        to: "ciso@domain.local"
        subject: "[SECURITY ALERT] {{ incident_type }}"
        body: |
          Incident automático reportado:
          
          Host: {{ affected_host }}
          Tipo: {{ incident_type }}
          Severidad: {{ incident_severity }}
          Timestamp: {{ ansible_date_time.iso8601 }}
          
          Acciones automáticas tomadas:
          - Ticket de incidente creado
          - Host aislado de la red
          - Análisis forense iniciado
```

---

## Troubleshooting

### Problema 1: AAP no es accesible desde Windows

**Síntoma:** `curl: (7) Failed to connect to 127.0.0.1:443`

**Soluciones:**

```powershell
# 1. Verificar que WSL2 está corriendo
wsl --list --verbose

# 2. Obtener IP de WSL2
wsl hostname -I

# 3. Agregar regla de firewall
netsh advfirewall firewall add rule name="AAP WSL2" \
  dir=in action=allow protocol=tcp localport=443

# 4. Probar con IP de WSL2
$WSL_IP = wsl hostname -I | Select-Object -First 1
$WSL_IP = $WSL_IP.Trim()
Write-Host "WSL2 IP: $WSL_IP"
Invoke-WebRequest -Uri "https://$WSL_IP" -SkipCertificateCheck
```

### Problema 2: PostgreSQL no inicia en WSL2

**Síntoma:** `systemctl start postgresql` falla

**Soluciones:**

```bash
# 1. Verificar logs
sudo journalctl -u postgresql -n 50

# 2. Verificar permisos de directorio
sudo ls -la /var/lib/postgresql/14/main/

# 3. Reparar permisos
sudo chown -R postgres:postgres /var/lib/postgresql
sudo chmod 700 /var/lib/postgresql/14/main

# 4. Verificar puerto
sudo netstat -tlnp | grep 5432

# 5. Limpiar archivos de bloqueo
sudo rm -f /var/lib/postgresql/14/main/postmaster.pid
sudo systemctl start postgresql
```

### Problema 3: Zabbix agent no se conecta

**Síntoma:** `ZBX_NOTSUPPORTED` en items de Zabbix

**Soluciones:**

```bash
# 1. Verificar que el agente está corriendo
sudo systemctl status zabbix-agent

# 2. Verificar puerto 10050
sudo netstat -tlnp | grep 10050

# 3. Verificar configuración del agente
sudo grep -E "^Server=|^ServerActive=" /etc/zabbix/zabbix_agentd.conf

# 4. Reiniciar agente
sudo systemctl restart zabbix-agent

# 5. Ver logs del agente
sudo tail -f /var/log/zabbix/zabbix_agentd.log
```

### Problema 4: Webhook no recibe eventos

**Síntoma:** Webhook receiver no procesa eventos

**Soluciones:**

```bash
# 1. Verificar que el servicio está corriendo
sudo systemctl status aap-webhook-receiver

# 2. Ver logs
sudo journalctl -u aap-webhook-receiver -n 50

# 3. Verificar puerto 5000
sudo netstat -tlnp | grep 5000

# 4. Probar manualmente
curl -k -X POST https://127.0.0.1:5000/webhook/zabbix \
  -H "Content-Type: application/json" \
  -d '{"severity":"average","event_type":"cpu_high","host":"test"}'

# 5. Validar token de API
export AAP_TOKEN=$(curl -k -s https://127.0.0.1/api/v2/auth/token/ \
  -d '{"username":"admin","password":"password"}' \
  -H 'Content-Type: application/json' | jq -r '.token')

echo "Token: $AAP_TOKEN"
```

### Problema 5: Job template no ejecuta

**Síntoma:** Job queda en "pending" o falla

**Soluciones:**

```bash
# 1. Verificar logs del job
curl -k -H "Authorization: Bearer $AAP_TOKEN" \
  "https://127.0.0.1/api/v2/jobs/ID/stdout/?format=json" | jq .

# 2. Verificar credenciales
curl -k -H "Authorization: Bearer $AAP_TOKEN" \
  "https://127.0.0.1/api/v2/job_templates/ID/credentials/" | jq .

# 3. Verificar inventario
curl -k -H "Authorization: Bearer $AAP_TOKEN" \
  "https://127.0.0.1/api/v2/inventories/" | jq .

# 4. Re-sincronizar proyecto
curl -k -X POST \
  -H "Authorization: Bearer $AAP_TOKEN" \
  "https://127.0.0.1/api/v2/projects/ID/update/" | jq .
```

---

## Referencia Rápida

### Comandos Útiles en WSL2

```bash
# Iniciar/parar WSL2
wsl --shutdown
wsl

# Ver todas las distribuciones
wsl --list --verbose

# Acceder a WSL2 desde PowerShell
wsl -d Ubuntu-22.04

# Ver IP de WSL2
wsl hostname -I

# Exportar/Importar WSL2
wsl --export Ubuntu-22.04 backup.tar.gz
wsl --import Ubuntu-22.04-Restored D:\wsl\restored backup.tar.gz
```

### Comandos Útiles de AAP

```bash
# Obtener token
curl -k -X POST https://127.0.0.1/api/v2/auth/token/ \
  -d '{"username":"admin","password":"password"}' \
  -H 'Content-Type: application/json'

# Listar job templates
curl -k -H "Authorization: Bearer $TOKEN" \
  "https://127.0.0.1/api/v2/job_templates/"

# Ejecutar job template
curl -k -X POST -H "Authorization: Bearer $TOKEN" \
  "https://127.0.0.1/api/v2/job_templates/ID/launch/"

# Ver estado de job
curl -k -H "Authorization: Bearer $TOKEN" \
  "https://127.0.0.1/api/v2/jobs/JOB_ID/"

# Ver output de job
curl -k -H "Authorization: Bearer $TOKEN" \
  "https://127.0.0.1/api/v2/jobs/JOB_ID/stdout/?format=json"
```

### Variables de Entorno Útiles

```bash
# En ~/.bashrc o ~/.zshrc

export AAP_HOST="https://127.0.0.1"
export AAP_USERNAME="admin"
export AAP_PASSWORD="password"
export ZABBIX_SERVER="127.0.0.1"
export ZABBIX_AGENT_PORT="10050"
export LOG_DIR="/var/log/aap-events"
```

### Firewall Rules en Windows

```powershell
# AAP
netsh advfirewall firewall add rule name="AAP HTTPS" \
  dir=in action=allow protocol=tcp localport=443

# Zabbix Server
netsh advfirewall firewall add rule name="Zabbix Server" \
  dir=in action=allow protocol=tcp localport=10051

# Zabbix Agent
netsh advfirewall firewall add rule name="Zabbix Agent" \
  dir=in action=allow protocol=tcp localport=10050

# PostgreSQL (solo WSL2)
netsh advfirewall firewall add rule name="PostgreSQL" \
  dir=in action=allow protocol=tcp localport=5432 remoteip=127.0.0.1

# Webhook Receiver
netsh advfirewall firewall add rule name="AAP Webhook" \
  dir=in action=allow protocol=tcp localport=5000
```

---

## Checklist de Validación Final

### Pre-Implementación

- [ ] WSL2 habilitado en Windows
- [ ] Ubuntu 22.04 instalado en WSL2
- [ ] Docker Desktop instalado y configurado
- [ ] Certificados SSL generados
- [ ] Puertos requeridos disponibles

### Instalación AAP

- [ ] PostgreSQL funcionando
- [ ] AAP Controller instalado
- [ ] AAP Hub instalado (opcional)
- [ ] Acceso a UI web confirma
- [ ] Token de API generado

### Configuración Zabbix

- [ ] Zabbix Server instalado/ejecutando
- [ ] Zabbix Agent registrado
- [ ] Items monitoreando correctamente
- [ ] Triggers configurados
- [ ] Media type de webhook creado

### Integración

- [ ] Credenciales de AAP en Zabbix configuradas
- [ ] Webhook receiver ejecutando
- [ ] Job templates en AAP creados
- [ ] Acciones de Zabbix disparando jobs

### Pruebas

- [ ] Webhook recibe eventos desde Zabbix
- [ ] Job templates se ejecutan correctamente
- [ ] Logs se registran apropiadamente
- [ ] Notificaciones funcionan
- [ ] Remediation playbooks responden

---

## Conclusión

Este instructivo proporciona una guía completa para desplegar Ansible Automation Platform en WSL2 e integrarlo con Zabbix para automatización basada en eventos. La arquitectura permite:

✅ Monitoreo centralizado con Zabbix  
✅ Respuesta automática a eventos  
✅ Ejecución de playbooks bajo demanda  
✅ Escalado automático de recursos  
✅ Gestión de incidentes  
✅ Auditoría completa de acciones  

### Próximos Pasos

1. Implementar casos de uso específicos
2. Crear playbooks adicionales para su entorno
3. Integrar con sistemas de tickets/ITSM
4. Configurar dashboard de monitoreo
5. Establecer políticas de retención de logs

**Versión:** 1.0  
**Última Actualización:** 2026-05-12  
**Próxima Revisión:** 2026-08-12