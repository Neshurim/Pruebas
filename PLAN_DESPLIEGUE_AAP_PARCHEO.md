# Plan de Despliegue: Ansible Automation Platform para Automatización de Parcheo

**Versión:** 1.0  
**Fecha:** 2026-05-12  
**Autor:** Equipo de Automatización  
**Estado:** Documento de Planificación

---

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Arquitectura General](#arquitectura-general)
3. [Requisitos Previos](#requisitos-previos)
4. [Fase 1: Instalación y Configuración de AAP](#fase-1-instalación-y-configuración-de-aap)
5. [Fase 2: Integración con VMware vSphere](#fase-2-integración-con-vmware-vsphere)
6. [Fase 3: Gestión de Inventario Dinámico](#fase-3-gestión-de-inventario-dinámico)
7. [Fase 4: Configuración de Credenciales](#fase-4-configuración-de-credenciales)
8. [Fase 5: Desarrollo de Playbooks](#fase-5-desarrollo-de-playbooks)
9. [Fase 6: Monitoreo y Reportes](#fase-6-monitoreo-y-reportes)
10. [Fase 7: Políticas de Seguridad y Cumplimiento](#fase-7-políticas-de-seguridad-y-cumplimiento)
11. [Cronograma de Implementación](#cronograma-de-implementación)
12. [Consideraciones de Riesgos](#consideraciones-de-riesgos)

---

## Introducción

Este plan de despliegue describe la implementación completa de **Red Hat Ansible Automation Platform (AAP)** para automatizar procesos de parcheo en máquinas virtuales alojadas en **VMware vSphere**, incluyendo sistemas operativos **Windows Server 2016** y **RHEL 8**.

### Objetivos Principales

- Automatizar completamente el proceso de parcheo de SO
- Reducir el tiempo de ejecución de parches (ventana de 4 horas)
- Garantizar consistencia y trazabilidad en las actualizaciones
- Implementar políticas de seguridad y cumplimiento
- Proporcionar visibilidad mediante monitoreo y reportes

---

## Arquitectura General

```
┌─────────────────────────────────────────────────────────────┐
│         Ansible Automation Platform                         │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Controller (HA)                                     │  │
│  │  - Gestión de playbooks y workflows                 │  │
│  │  - API REST y UI Web                                │  │
│  │  - Auditoría y logging                              │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Execution Environment (EE)                         │  │
│  │  - Contenedores ejecutables                         │  │
│  │  - Dependencias necesarias                          │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Hub (Repositorio Central)                          │  │
│  │  - Colecciones Ansible                              │  │
│  │  - Roles y playbooks                                │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
           │
           │ (API/SSH)
           │
┌──────────────────────────────────────────────────────────────┐
│         VMware vSphere                                       │
│  ┌────────────────┐  ┌────────────────┐                    │
│  │  VM Windows    │  │  VM RHEL 8     │                    │
│  │  Server 2016   │  │  (múltiples)   │                    │
│  │  (múltiples)   │  │                │                    │
│  └────────────────┘  └────────────────┘                    │
└──────────────────────────────────────────────────────────────┘
```

---

## Requisitos Previos

### Hardware

| Componente | Especificación | Cantidad |
|-----------|----------------|----------|
| **CPU** | 8 vCPU (mínimo) | 1-3 (HA) |
| **RAM** | 16 GB (mínimo) | 1-3 (HA) |
| **Almacenamiento** | 100 GB SSD | 1-3 (HA) |
| **Red** | Conectividad a VMware vSphere | 1 Gbps |

### Software

| Componente | Versión | Propósito |
|-----------|---------|----------|
| **RHEL** | 8.x | Sistema operativo del Controller |
| **Python** | 3.9+ | Runtime de Ansible |
| **PostgreSQL** | 13+ | Base de datos de AAP |
| **Docker/Podman** | Último | Contenedores de EE |

### Acceso y Permisos

- Acceso administrativo a VMware vSphere
- Credenciales de administrador en máquinas objetivo (Windows y Linux)
- Acceso a Red Hat Subscription Management
- Puertos de red abiertos:
  - **TCP 443** (HTTPS - AAP UI/API)
  - **TCP 22** (SSH - comunicación Ansible)
  - **TCP 5432** (PostgreSQL - si es remoto)

### Licencias

- Red Hat Ansible Automation Platform (versión compatible)
- Red Hat Enterprise Linux 8 (para Controller)
- Cuentas de servicio con permisos de vSphere

---

## Fase 1: Instalación y Configuración de AAP

### 1.1 Preparación del Servidor

```bash
# Actualizar sistema
sudo dnf update -y

# Instalar dependencias básicas
sudo dnf install -y \
    git \
    python3.9-devel \
    gcc \
    openssl-devel \
    postgresql-server \
    postgresql-contrib \
    podman \
    podman-compose

# Crear usuario de servicio para Ansible
sudo useradd -m -s /bin/bash ansible
sudo usermod -aG wheel ansible
```

### 1.2 Instalación de Ansible Automation Platform

```bash
# Descargar instalador de AAP
# (Obtener de Red Hat Portal)
cd /tmp
tar -xzf ansible-automation-platform-setup-latest.tar.gz
cd ansible-automation-platform-setup-*

# Configurar inventario de instalación
# Ver sección 1.3

# Ejecutar instalación
sudo ./setup.sh
```

### 1.3 Archivo de Inventario (inventory.yml)

```yaml
---
all:
  hosts:
    localhost:
      ansible_connection: local
      ansible_user: root

  vars:
    # Red Hat subscription
    rh_username: "{{ vault_rh_username }}"
    rh_password: "{{ vault_rh_password }}"
    
    # Base de datos
    automationcontroller_pg_database: 'awx'
    automationcontroller_pg_username: 'awx'
    automationcontroller_pg_password: "{{ vault_pg_password }}"
    automationcontroller_pg_port: 5432
    automationcontroller_pg_host: 'localhost'
    
    # Configuración del Controller
    automationcontroller_host: 'aap-controller.domain.local'
    automationcontroller_username: 'admin'
    automationcontroller_password: "{{ vault_controller_password }}"
    
    # Hub (opcional, para HA)
    automationhub_host: 'aap-hub.domain.local'
    automationhub_username: 'admin'
    automationhub_password: "{{ vault_hub_password }}"
    
    # Certificados SSL
    automationcontroller_ssl_cert: '/etc/pki/tls/certs/controller.crt'
    automationcontroller_ssl_key: '/etc/pki/tls/private/controller.key'
    
    # Configuración de seguridad
    automationcontroller_admin_username: 'admin'
    automationcontroller_admin_password: "{{ vault_admin_password }}"
```

### 1.4 Validación de Instalación

```bash
# Verificar servicios activos
sudo systemctl status automation-controller
sudo systemctl status automation-hub

# Verificar conectividad
curl -k https://localhost/api/v2/ -u admin:password

# Revisar logs
sudo tail -f /var/log/controller/controller.log
```

---

## Fase 2: Integración con VMware vSphere

### 2.1 Instalación de Dependencias VMware

```bash
# En el Controller o EE
pip install pyvmomi
pip install requests
pip install urllib3

# Instalar colección VMware en Ansible
ansible-galaxy collection install community.vmware
```

### 2.2 Configuración de Credenciales de vSphere

En la UI de AAP Controller:

1. **Navegar a:** Administración → Credenciales
2. **Crear nueva credencial:**
   - **Nombre:** vmware-vsphere-prod
   - **Tipo:** VMware vCenter
   - **Detalles:**
     - Host: `vcenter.domain.local`
     - Usuario: `automation@domain.local`
     - Contraseña: (secura)
     - Validar certificado SSL: Sí/No (según configuración)

### 2.3 Playbook de Prueba de Conectividad

```yaml
---
- name: Verificar conectividad con vSphere
  hosts: localhost
  gather_facts: no
  vars:
    vcenter_hostname: "vcenter.domain.local"
    vcenter_username: "automation@domain.local"
    vcenter_validate_certs: false

  tasks:
    - name: Conectar a vSphere
      community.vmware.vmware_about_info:
        hostname: "{{ vcenter_hostname }}"
        username: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
        validate_certs: "{{ vcenter_validate_certs }}"
      register: vmware_info

    - name: Mostrar información de vSphere
      debug:
        var: vmware_info

    - name: Obtener lista de máquinas virtuales
      community.vmware.vmware_vm_info:
        hostname: "{{ vcenter_hostname }}"
        username: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
        validate_certs: "{{ vcenter_validate_certs }}"
      register: vm_list

    - name: Mostrar VMs disponibles
      debug:
        msg: "{{ item.guest_name }}"
      loop: "{{ vm_list.virtual_machines }}"
```

---

## Fase 3: Gestión de Inventario Dinámico

### 3.1 Creación de Inventario Dinámico desde vSphere

Crear archivo: `inventory_plugins/vmware_dynamic.py`

```python
#!/usr/bin/env python3
"""
Plugin de inventario dinámico para VMware vSphere
Descubre VMs automáticamente basado en tags
"""

from __future__ import absolute_import, division, print_function
__metaclass__ = type

DOCUMENTATION = r'''
    name: vmware_dynamic
    plugin_type: inventory
    short_description: Inventario dinámico desde VMware vSphere
    description:
        - Descubre máquinas virtuales en VMware vSphere
        - Agrupa VMs por tags y características
    options:
      hostname:
        description: Nombre del host vCenter
        type: string
        required: true
      username:
        description: Usuario para vCenter
        type: string
        required: true
      password:
        description: Contraseña para vCenter
        type: string
        required: true
      validate_certs:
        description: Validar certificados SSL
        type: boolean
        default: false
'''

from ansible.plugins.inventory import BaseInventoryPlugin
from ansible_collections.community.vmware.plugins.module_utils.vmware import connect_to_api
from pyVmomi import vim, vmodl

class InventoryModule(BaseInventoryPlugin):
    NAME = 'vmware_dynamic'

    def parse(self, inventory, loader, path, cache=True):
        super(InventoryModule, self).parse(inventory, loader, path, cache)
        
        # Cargar configuración del plugin
        self._read_config_data(path)
        
        # Obtener parámetros
        hostname = self.get_option('hostname')
        username = self.get_option('username')
        password = self.get_option('password')
        validate_certs = self.get_option('validate_certs')
        
        # Conectar a vSphere
        si = connect_to_api(hostname, username, password, validate_certs)
        
        # Obtener contenedor raíz
        content = si.RetrieveContent()
        container = content.rootFolder
        
        # Buscar todas las VMs
        view_type = [vim.VirtualMachine]
        recursive = True
        container_view = content.viewManager.CreateContainerView(container, view_type, recursive)
        
        for vm in container_view.view:
            # Filtrar solo VMs activas
            if vm.runtime.powerState == 'poweredOn':
                hostname = vm.guest.hostName or vm.name
                
                # Agregar host al inventario
                self.inventory.add_host(hostname)
                
                # Establecer variables del host
                self.inventory.set_variable(hostname, 'vm_name', vm.name)
                self.inventory.set_variable(hostname, 'vm_uuid', vm.config.uuid)
                self.inventory.set_variable(hostname, 'vm_os', vm.guest.guestFullName)
                self.inventory.set_variable(hostname, 'vcenter_host', hostname)
                
                # Procesar etiquetas para agrupar
                for tag in vm.config.customValue:
                    if tag.value:
                        group_name = f"vm_tag_{tag.key}"
                        self.inventory.add_group(group_name)
                        self.inventory.add_host(hostname, group=group_name)
                
                # Agrupar por SO
                if 'Windows' in vm.guest.guestFullName:
                    self.inventory.add_group('windows_servers')
                    self.inventory.add_host(hostname, group='windows_servers')
                elif 'Red Hat' in vm.guest.guestFullName or 'RHEL' in vm.guest.guestFullName:
                    self.inventory.add_group('rhel_servers')
                    self.inventory.add_host(hostname, group='rhel_servers')
        
        container_view.Destroy()
```

### 3.2 Configuración en AAP

En la UI del Controller:

1. **Inventarios → Crear**
   - **Nombre:** vmware-inventory-prod
   - **Organizació:** Default
   - **Fuente de inventario:** Seleccionar "VMware vCenter"

2. **Configurar fuente:**
   - **Credencial:** vmware-vsphere-prod
   - **Host:** vcenter.domain.local
   - **Refrescar automáticamente:** Sí
   - **Intervalo de actualización:** 3600 segundos (1 hora)

### 3.3 Verificación del Inventario

```bash
# Listar inventario desde CLI
ansible-inventory -i inventory_vmware.yml --list

# Filtrar por grupo
ansible-inventory -i inventory_vmware.yml --graph
```

---

## Fase 4: Configuración de Credenciales

### 4.1 Estructura de Almacenamiento de Credenciales

```
Credenciales Requeridas:
├── VMware vSphere
│   ├── vcenter_admin
│   └── vcenter_service_account
├── Windows Server 2016
│   ├── local_admin
│   └── domain_admin
├── RHEL 8
│   ├── ssh_key_prod
│   └── ssh_key_backup
└── Notificaciones
    ├── smtp_credentials
    └── webhook_urls
```

### 4.2 Creación de Credenciales en AAP

**Script de automatización:**

```yaml
---
- name: Configurar credenciales en AAP
  hosts: localhost
  vars:
    controller_host: "https://aap-controller.domain.local"
    controller_username: "admin"
    controller_password: "{{ vault_controller_password }}"
    controller_verify_ssl: false

  tasks:
    # Credencial: vSphere
    - name: Crear credencial vSphere
      awx.awx.credential:
        name: "vmware-prod"
        description: "Acceso a vSphere producción"
        credential_type: "VMware vCenter"
        inputs:
          host: "vcenter.domain.local"
          username: "automation@domain.local"
          password: "{{ vault_vsphere_password }}"
        organization: "Default"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"
        controller_verify_ssl: "{{ controller_verify_ssl }}"

    # Credencial: Windows (usuario local)
    - name: Crear credencial Windows local
      awx.awx.credential:
        name: "windows-local-admin"
        description: "Administrador local de Windows"
        credential_type: "Machine"
        inputs:
          username: "Administrator"
          password: "{{ vault_windows_admin_password }}"
        organization: "Default"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"
        controller_verify_ssl: "{{ controller_verify_ssl }}"

    # Credencial: Windows (dominio)
    - name: Crear credencial Windows dominio
      awx.awx.credential:
        name: "windows-domain-admin"
        description: "Administrador de dominio"
        credential_type: "Machine"
        inputs:
          username: "DOMAIN\\automation"
          password: "{{ vault_domain_password }}"
        organization: "Default"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"
        controller_verify_ssl: "{{ controller_verify_ssl }}"

    # Credencial: Linux SSH Key
    - name: Crear credencial RHEL SSH
      awx.awx.credential:
        name: "rhel-ssh-prod"
        description: "Acceso SSH a RHEL 8"
        credential_type: "Machine"
        inputs:
          username: "root"
          ssh_key_data: "{{ lookup('file', '/secure/ssh/automation_id_rsa') }}"
          ssh_key_unlock: "{{ vault_ssh_passphrase }}"
        organization: "Default"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"
        controller_verify_ssl: "{{ controller_verify_ssl }}"
```

### 4.3 Configuración de Vault Ansible

Crear archivo: `.ansible/vault/credentials.yml`

```yaml
# Credenciales sensibles (encriptadas con ansible-vault)
vault_rh_username: "your_redhat_username"
vault_rh_password: "your_redhat_password"
vault_controller_password: "secure_admin_password"
vault_pg_password: "postgres_secure_password"
vault_vsphere_password: "vcenter_service_account_password"
vault_windows_admin_password: "windows_administrator_password"
vault_domain_password: "domain_automation_account_password"
vault_ssh_passphrase: "ssh_key_passphrase"
```

**Encriptar archivo:**

```bash
ansible-vault encrypt .ansible/vault/credentials.yml
# Ingresar contraseña de vault
```

---

## Fase 5: Desarrollo de Playbooks

### 5.1 Estructura de Proyectos

```
/home/ansible/projects/
├── patch-automation/
│   ├── README.md
│   ├── roles/
│   │   ├── pre-patch-checks/
│   │   ├── windows-patch/
│   │   ├── rhel-patch/
│   │   ├── post-patch-validation/
│   │   └── patch-rollback/
│   ├── playbooks/
│   │   ├── windows-patch-main.yml
│   │   ├── rhel-patch-main.yml
│   │   ├── patch-orchestration.yml
│   │   └── emergency-rollback.yml
│   ├── inventory/
│   │   ├── production.yml
│   │   ├── staging.yml
│   │   └── group_vars/
│   ├── library/
│   │   └── custom_modules/
│   ├── files/
│   │   └── patch_baseline/
│   └── vars/
│       └── patch_schedules.yml
```

### 5.2 Playbook Principal - Parcheo Windows

`playbooks/windows-patch-main.yml`

```yaml
---
- name: Windows Server 2016 - Parcheo Automatizado
  hosts: windows_servers
  gather_facts: yes
  vars:
    maintenance_window_hours: 4
    max_concurrent_patches: 3
    pre_patch_snapshot: true
    send_notifications: true

  pre_tasks:
    - name: Verificar ventana de mantenimiento
      fail:
        msg: "Fuera de ventana de mantenimiento permitida"
      when:
        - ansible_date_time.hour < 22 or ansible_date_time.hour > 2

    - name: Crear snapshot pre-parcheo
      vmware_guest_snapshot:
        hostname: "{{ vcenter_host }}"
        username: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
        name: "{{ inventory_hostname }}"
        state: present
        snapshot_name: "pre-patch-{{ ansible_date_time.iso8601_basic }}"
      delegate_to: localhost
      when: pre_patch_snapshot | bool

    - name: Verificar estado actual del sistema
      win_command: 'systeminfo'
      register: system_info

    - name: Enviar notificación - Inicio de parcheo
      mail:
        host: "{{ smtp_host }}"
        port: "{{ smtp_port }}"
        to: "{{ notification_email }}"
        subject: "Inicio de parcheo: {{ inventory_hostname }}"
        body: |
          Servidor: {{ inventory_hostname }}
          Sistema Operativo: {{ ansible_os_family }}
          Versión: {{ ansible_distribution_version }}
          Hora de inicio: {{ ansible_date_time.iso8601 }}
      when: send_notifications | bool
      delegate_to: localhost

  tasks:
    - name: Obtener lista de actualizaciones disponibles
      win_updates:
        state: searched
        category_names:
          - SecurityUpdates
          - CriticalUpdates
          - UpdateRollups
      register: win_updates

    - name: Mostrar actualizaciones encontradas
      debug:
        msg: "Se encontraron {{ win_updates.updates | length }} actualizaciones"

    - name: Instalar actualizaciones
      win_updates:
        state: installed
        category_names:
          - SecurityUpdates
          - CriticalUpdates
          - UpdateRollups
        reboot: no
      register: patch_result

    - name: Validar instalación de parches
      win_command: 'Get-HotFix | Select-Object -Last 5 | Format-Table -AutoSize'
      register: hotfix_list

    - name: Mostrar hotfixes instalados
      debug:
        var: hotfix_list.stdout_lines

    - name: Registrar requerimientos de reinicio
      win_shell: |
        $regPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager"
        if (Test-Path "$regPath\PendingFileRenameOperations") {
            $true
        } else {
            $false
        }
      register: reboot_required

    - name: Programa reinicio del servidor
      win_scheduled_task:
        name: "AutomatedPatchReboot"
        description: "Reinicio programado post-parcheo"
        actions:
          - path: "C:\\Windows\\System32\\shutdown.exe"
            arguments: "/r /t 600 /c 'Reinicio programado post-parcheo'"
        triggers:
          - type: boot
        enabled: "{{ reboot_required.stdout | bool }}"
      when: reboot_required.stdout | bool

  post_tasks:
    - name: Validación post-parcheo
      block:
        - name: Esperar disponibilidad del servidor
          wait_for_connection:
            delay: 60
            timeout: 600

        - name: Obtener estado del sistema post-reboot
          win_command: 'systeminfo'
          register: post_system_info

        - name: Verificar servicios críticos
          win_service_info:
            name:
              - BITS
              - WinRM
              - EventLog
          register: service_status

        - name: Fallar si servicios críticos no están activos
          fail:
            msg: "Servicio crítico no está activo: {{ item.name }}"
          loop: "{{ service_status.services }}"
          when: item.state != "running"

        - name: Enviar notificación - Completación exitosa
          mail:
            host: "{{ smtp_host }}"
            port: "{{ smtp_port }}"
            to: "{{ notification_email }}"
            subject: "✓ Parcheo completado: {{ inventory_hostname }}"
            body: |
              Servidor: {{ inventory_hostname }}
              Parches instalados: {{ patch_result.updates_count | default(0) }}
              Reinicio requerido: {{ reboot_required.stdout | bool }}
              Hora de finalización: {{ ansible_date_time.iso8601 }}
              Estado: EXITOSO
          when: service_status is succeeded
          delegate_to: localhost

      rescue:
        - name: Enviar notificación - Error
          mail:
            host: "{{ smtp_host }}"
            port: "{{ smtp_port }}"
            to: "{{ notification_email }}"
            subject: "✗ ERROR en parcheo: {{ inventory_hostname }}"
            body: |
              Servidor: {{ inventory_hostname }}
              Error: {{ ansible_failed_result.msg | default('Error desconocido') }}
              Iniciando rollback...
          delegate_to: localhost

        - name: Iniciar rollback desde snapshot
          vmware_guest_snapshot:
            hostname: "{{ vcenter_host }}"
            username: "{{ vcenter_username }}"
            password: "{{ vcenter_password }}"
            name: "{{ inventory_hostname }}"
            state: revert
            snapshot_name: "pre-patch-{{ ansible_date_time.iso8601_basic }}"
          delegate_to: localhost
          when: pre_patch_snapshot | bool

    - name: Registrar métricas en el sistema de monitoreo
      uri:
        url: "{{ monitoring_endpoint }}/api/v1/metrics"
        method: POST
        headers:
          Content-Type: "application/json"
        body_format: json
        body:
          host: "{{ inventory_hostname }}"
          action: "patch"
          status: "{{ 'success' if patch_result is succeeded else 'failed' }}"
          updates_count: "{{ patch_result.updates_count | default(0) }}"
          timestamp: "{{ ansible_date_time.iso8601 }}"
      delegate_to: localhost
```

### 5.3 Playbook Principal - Parcheo RHEL 8

`playbooks/rhel-patch-main.yml`

```yaml
---
- name: RHEL 8 - Parcheo Automatizado
  hosts: rhel_servers
  gather_facts: yes
  become: yes
  vars:
    maintenance_window_hours: 4
    enable_yum_cron: true
    create_backup: true
    reboot_strategy: "scheduled"

  pre_tasks:
    - name: Verificar ventana de mantenimiento
      assert:
        that:
          - ansible_date_time.hour >= 22 or ansible_date_time.hour <= 2
        fail_msg: "Fuera de ventana de mantenimiento"

    - name: Crear snapshot pre-parcheo
      vmware_guest_snapshot:
        hostname: "{{ vcenter_host }}"
        username: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
        name: "{{ inventory_hostname }}"
        state: present
        snapshot_name: "pre-patch-{{ ansible_date_time.date_iso8601 }}"
      delegate_to: localhost

    - name: Realizar backup de configuración crítica
      shell: |
        tar -czf /backup/pre-patch-backup-{{ ansible_date_time.iso8601_basic }}.tar.gz \
          /etc/yum.repos.d/ \
          /etc/fstab \
          /boot/grub2/grub.cfg
      when: create_backup | bool

    - name: Notificar inicio de parcheo
      debug:
        msg: "Iniciando parcheo en {{ inventory_hostname }} - {{ ansible_date_time.iso8601 }}"

  tasks:
    - name: Obtener lista de actualizaciones disponibles
      dnf:
        list: upgradable
      register: available_updates

    - name: Mostrar actualizaciones disponibles
      debug:
        msg: "Actualizaciones disponibles: {{ available_updates.results | length }}"

    - name: Actualizar todos los paquetes (excluyendo kernel)
      dnf:
        name: "*"
        state: latest
        exclude:
          - kernel*
          - redhat-release*
      register: dnf_updates

    - name: Actualizar kernel
      dnf:
        name: kernel*
        state: latest
      register: kernel_update

    - name: Ejecutar dracut para regenerar initramfs
      shell: dracut -f
      when: kernel_update is changed

    - name: Limpiar caché de DNF
      dnf:
        autoremove: yes
        clean: all

    - name: Verificar actualizaciones de seguridad pendientes
      shell: dnf check-update --security 2>/dev/null | wc -l
      register: security_updates_pending
      changed_when: false

    - name: Validar integridad de paquetes
      shell: rpm --verify --all 2>&1 | grep -E "^S|^M|^5|^D" | wc -l
      register: rpm_verify_result
      changed_when: false

    - name: Verificar si reinicio es necesario
      stat:
        path: /run/reboot-required
      register: reboot_required

  post_tasks:
    - name: Validación post-parcheo
      block:
        - name: Programar reinicio (si es necesario)
          shell: |
            shutdown -r +10 "Reinicio programado post-parcheo Ansible"
          when: reboot_required.stat.exists

        - name: Esperar por reinicio
          wait_for_connection:
            delay: 60
            timeout: 600
          when: reboot_required.stat.exists

        - name: Verificar versión del kernel post-boot
          shell: uname -r
          register: post_kernel_version

        - name: Verificar servicios críticos
          systemd:
            name: "{{ item }}"
            enabled: yes
            state: started
          loop:
            - sshd
            - network
            - chronyd
          register: service_check

        - name: Registrar resultados en el sistema de monitoreo
          shell: |
            cat > /var/log/patch-execution-{{ ansible_date_time.iso8601_basic }}.log << EOF
            Host: {{ inventory_hostname }}
            Fecha: {{ ansible_date_time.iso8601 }}
            Paquetes actualizados: {{ dnf_updates.changed | int }}
            Kernel actualizado: {{ kernel_update.changed | int }}
            Reinicio requerido: {{ reboot_required.stat.exists | int }}
            Actualizaciones de seguridad pendientes: {{ security_updates_pending.stdout }}
            EOF

        - name: Mensaje de éxito
          debug:
            msg: "Parcheo completado exitosamente en {{ inventory_hostname }}"

      rescue:
        - name: Enviar alerta de error
          debug:
            msg: "ERROR: Falló el parcheo en {{ inventory_hostname }}"

        - name: Restaurar desde backup
          shell: |
            tar -xzf /backup/pre-patch-backup-*.tar.gz -C /
          when: create_backup | bool

        - name: Revertir snapshot
          vmware_guest_snapshot:
            hostname: "{{ vcenter_host }}"
            username: "{{ vcenter_username }}"
            password: "{{ vcenter_password }}"
            name: "{{ inventory_hostname }}"
            state: revert
            snapshot_name: "pre-patch-{{ ansible_date_time.date_iso8601 }}"
          delegate_to: localhost
```

### 5.4 Playbook de Orquestación

`playbooks/patch-orchestration.yml`

```yaml
---
- name: Orquestación de Parcheo - Todas las Plataformas
  hosts: localhost
  gather_facts: no
  vars:
    max_concurrent_hosts: 3
    batch_delay_minutes: 15
    rollback_on_failure: true

  tasks:
    - name: Obtener lista de hosts por grupo
      set_fact:
        windows_hosts: "{{ groups['windows_servers'] | default([]) }}"
        rhel_hosts: "{{ groups['rhel_servers'] | default([]) }}"

    - name: Procesamiento en lotes - Windows
      block:
        - name: Dividir hosts Windows en lotes
          set_fact:
            windows_batches: "{{ windows_hosts | batch(max_concurrent_hosts) | list }}"

        - name: Ejecutar parcheo de lotes Windows
          include_tasks: windows-patch-batch.yml
          vars:
            batch_number: "{{ item.0 + 1 }}"
            batch_hosts: "{{ item.1 }}"
          loop: "{{ windows_batches | enumerate }}"
          loop_control:
            label: "Lote Windows {{ item.0 + 1 }}"

        - name: Esperar entre lotes
          pause:
            minutes: "{{ batch_delay_minutes }}"
          when: (item.0 + 1) < (windows_batches | length)
          loop: "{{ windows_batches | enumerate }}"

    - name: Procesamiento en lotes - RHEL
      block:
        - name: Dividir hosts RHEL en lotes
          set_fact:
            rhel_batches: "{{ rhel_hosts | batch(max_concurrent_hosts) | list }}"

        - name: Ejecutar parcheo de lotes RHEL
          include_tasks: rhel-patch-batch.yml
          vars:
            batch_number: "{{ item.0 + 1 }}"
            batch_hosts: "{{ item.1 }}"
          loop: "{{ rhel_batches | enumerate }}"
          loop_control:
            label: "Lote RHEL {{ item.0 + 1 }}"

        - name: Esperar entre lotes
          pause:
            minutes: "{{ batch_delay_minutes }}"
          when: (item.0 + 1) < (rhel_batches | length)
          loop: "{{ rhel_batches | enumerate }}"

    - name: Reporte final
      debug:
        msg: |
          ========== REPORTE DE EJECUCIÓN ==========
          Hosts Windows parchados: {{ windows_hosts | length }}
          Hosts RHEL parchados: {{ rhel_hosts | length }}
          Duración total estimada: {{ (windows_batches | length + rhel_batches | length) * batch_delay_minutes }} minutos
          Fecha de inicio: {{ ansible_date_time.iso8601 }}
          ==========================================
```

---

## Fase 6: Monitoreo y Reportes

### 6.1 Configuración de Monitoreo

#### 6.1.1 Integración con Prometheus

```yaml
# Archivo: monitoring/prometheus-config.yml
---
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'ansible-controller'
    static_configs:
      - targets: ['aap-controller.domain.local:8181']
    metrics_path: '/api/v2/metrics'
    bearer_token: "{{ vault_prometheus_token }}"

  - job_name: 'windows-servers'
    static_configs:
      - targets: ['windows-01.domain.local:9182']
      - targets: ['windows-02.domain.local:9182']
    scrape_interval: 30s

  - job_name: 'rhel-servers'
    static_configs:
      - targets: ['rhel-01.domain.local:9100']
      - targets: ['rhel-02.domain.local:9100']
    scrape_interval: 30s
```

#### 6.1.2 Alertas de Monitoreo

```yaml
# Archivo: monitoring/alerts.yml
---
groups:
  - name: patch_automation
    interval: 1m
    rules:
      - alert: PatchExecutionFailed
        expr: patch_execution_status{status="failed"} == 1
        for: 5m
        annotations:
          summary: "Falló ejecución de parcheo en {{ $labels.host }}"
          description: "El playbook de parcheo falló en {{ $labels.host }}"

      - alert: HighPendingUpdates
        expr: pending_updates{type="security"} > 50
        for: 24h
        annotations:
          summary: "Muchas actualizaciones de seguridad pendientes: {{ $labels.host }}"

      - alert: RebootRequired
        expr: reboot_required == 1
        for: 30m
        annotations:
          summary: "Reinicio pendiente en {{ $labels.host }}"

      - alert: PatchWindowExceeded
        expr: patch_execution_duration_minutes > 240
        annotations:
          summary: "Ventana de mantenimiento excedida en {{ $labels.host }}"
```

### 6.2 Plantilla de Reporte

```yaml
# Archivo: reports/patch-execution-report.yml
---
- name: Generar Reporte de Parcheo
  hosts: localhost
  gather_facts: no
  vars:
    report_date: "{{ ansible_date_time.date }}"
    report_format: "html"

  tasks:
    - name: Obtener resultados de ejecución del último parcheo
      awx.awx.job:
        id: "{{ latest_job_id | default(omit) }}"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"
      register: job_results

    - name: Construir datos del reporte
      set_fact:
        report_data:
          title: "Reporte de Ejecución de Parcheo - {{ report_date }}"
          execution_status: "{{ job_results.status }}"
          hosts_processed: "{{ job_results.hosts_processed | default(0) }}"
          hosts_successful: "{{ job_results.hosts_ok | default(0) }}"
          hosts_failed: "{{ job_results.hosts_failed | default(0) }}"
          total_updates_installed: "{{ total_patches_installed | default(0) }}"
          execution_start: "{{ job_results.started | default('N/A') }}"
          execution_end: "{{ job_results.finished | default('N/A') }}"
          execution_duration: "{{ job_results.elapsed | default(0) }} segundos"

    - name: Generar reporte HTML
      template:
        src: patch-report.html.j2
        dest: "/reports/patch-execution-{{ report_date }}.html"
        mode: '0644'
      when: report_format == 'html'

    - name: Enviar reporte por correo
      mail:
        host: "{{ smtp_host }}"
        port: "{{ smtp_port }}"
        from: "ansible@domain.local"
        to: "{{ report_recipients }}"
        subject: "Reporte de Parcheo - {{ report_date }}"
        body: "Se adjunta el reporte de ejecución de parcheo del {{ report_date }}"
        attach: "/reports/patch-execution-{{ report_date }}.html"
        subtype: html
```

**Template HTML:** `templates/patch-report.html.j2`

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ report_data.title }}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; margin: 20px 0; }
        th, td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        th { background-color: #f2f2f2; font-weight: bold; }
        .success { color: green; }
        .failed { color: red; }
        .header { background-color: #2c3e50; color: white; padding: 20px; }
    </style>
</head>
<body>
    <div class="header">
        <h1>{{ report_data.title }}</h1>
    </div>

    <h2>Resumen de Ejecución</h2>
    <table>
        <tr>
            <td><strong>Estado:</strong></td>
            <td class="{% if report_data.execution_status == 'successful' %}success{% else %}failed{% endif %}">
                {{ report_data.execution_status }}
            </td>
        </tr>
        <tr>
            <td><strong>Hosts Procesados:</strong></td>
            <td>{{ report_data.hosts_processed }}</td>
        </tr>
        <tr>
            <td><strong>Exitosos:</strong></td>
            <td class="success">{{ report_data.hosts_successful }}</td>
        </tr>
        <tr>
            <td><strong>Fallidos:</strong></td>
            <td class="failed">{{ report_data.hosts_failed }}</td>
        </tr>
        <tr>
            <td><strong>Total de Actualizaciones Instaladas:</strong></td>
            <td>{{ report_data.total_updates_installed }}</td>
        </tr>
        <tr>
            <td><strong>Duración:</strong></td>
            <td>{{ report_data.execution_duration }}</td>
        </tr>
    </table>

    <h2>Detalles Adicionales</h2>
    <p>Para más detalles, visite el controlador de Ansible Automation Platform:</p>
    <p><a href="{{ controller_host }}">{{ controller_host }}</a></p>
</body>
</html>
```

### 6.3 Dashboard en Grafana

```json
{
  "dashboard": {
    "title": "Ansible Patch Automation Dashboard",
    "panels": [
      {
        "title": "Hosts Parchados (Últimos 7 días)",
        "targets": [
          {
            "expr": "increase(patch_execution_total[7d])"
          }
        ]
      },
      {
        "title": "Tasa de Éxito de Parcheo",
        "targets": [
          {
            "expr": "patch_execution_success_total / patch_execution_total * 100"
          }
        ]
      },
      {
        "title": "Tiempo Promedio de Ejecución",
        "targets": [
          {
            "expr": "avg(patch_execution_duration_seconds)"
          }
        ]
      },
      {
        "title": "Actualizaciones Pendientes por Host",
        "targets": [
          {
            "expr": "pending_updates by (host)"
          }
        ]
      }
    ]
  }
}
```

---

## Fase 7: Políticas de Seguridad y Cumplimiento

### 7.1 Control de Acceso Basado en Roles (RBAC)

```yaml
# Archivo: security/rbac-setup.yml
---
- name: Configurar RBAC en AAP
  hosts: localhost
  vars:
    controller_host: "{{ aap_host }}"
    controller_username: "admin"
    controller_password: "{{ vault_controller_password }}"

  tasks:
    - name: Crear equipo de operaciones de parcheo
      awx.awx.team:
        name: "Patch Operations"
        organization: "Default"
        description: "Equipo responsable de operaciones de parcheo"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"

    - name: Crear equipo de auditoria
      awx.awx.team:
        name: "Audit Team"
        organization: "Default"
        description: "Equipo de auditoría y cumplimiento"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"

    - name: Asignar permisos - Patch Operations
      awx.awx.role:
        team: "Patch Operations"
        role: "Admin"
        project: "patch-automation"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"

    - name: Asignar permisos - Audit Team (solo lectura)
      awx.awx.role:
        team: "Audit Team"
        role: "Read"
        project: "patch-automation"
        controller_host: "{{ controller_host }}"
        controller_username: "{{ controller_username }}"
        controller_password: "{{ controller_password }}"
```

### 7.2 Auditoría y Logging

```yaml
# Archivo: security/audit-logging.yml
---
- name: Configurar auditoría y logging
  hosts: localhost
  vars:
    audit_retention_days: 365
    log_level: "INFO"

  tasks:
    - name: Configurar logrotate para AAP
      template:
        src: logrotate-aap.j2
        dest: /etc/logrotate.d/ansible-automation-platform
        mode: '0644'
      become: yes

    - name: Habilitar auditoría del sistema
      lineinfile:
        path: /etc/audit/rules.d/ansible.rules
        line: "-w /opt/ansible -p wa -k ansible_changes"
        create: yes
      become: yes

    - name: Configurar envío de logs a syslog
      template:
        src: rsyslog-aap.j2
        dest: /etc/rsyslog.d/50-ansible.conf
      become: yes
      notify: restart rsyslog

    - name: Configurar almacenamiento centralizado de logs
      template:
        src: filebeat-aap.yml.j2
        dest: /etc/filebeat/inputs.d/ansible-logs.yml
      become: yes
      when: centralized_logging_enabled | default(false)

  handlers:
    - name: restart rsyslog
      systemd:
        name: rsyslog
        state: restarted
      become: yes
```

**Template:** `templates/logrotate-aap.j2`

```
/var/log/controller/controller.log
/var/log/controller/callback_receiver.log
/var/log/automation-hub/automation-hub.log
{
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 awx awx
    sharedscripts
    postrotate
        systemctl reload automation-controller > /dev/null 2>&1 || true
    endscript
}
```

### 7.3 Políticas de Cumplimiento

```yaml
# Archivo: security/compliance-policies.yml
---
- name: Validar cumplimiento de políticas
  hosts: all
  gather_facts: yes
  become: yes
  vars:
    compliance_checks:
      - name: "Parches de seguridad crítica"
        description: "Verificar que todos los parches de seguridad crítica estén aplicados"
        frequency: "weekly"
      - name: "Certificados SSL válidos"
        description: "Validar que los certificados no estén expirados"
        frequency: "monthly"
      - name: "Configuración de firewall"
        description: "Verificar reglas de firewall requeridas"
        frequency: "monthly"
      - name: "Auditoría habilitada"
        description: "Verificar que auditoría esté activa en todos los hosts"
        frequency: "daily"

  tasks:
    - name: Verificar parches de seguridad crítica - Windows
      block:
        - name: Obtener lista de parches de seguridad
          win_updates:
            state: searched
            category_names:
              - SecurityUpdates
          register: security_patches

        - name: Fallar si hay parches críticos pendientes
          fail:
            msg: "{{ security_patches.updates | length }} parches de seguridad crítica pendientes"
          when: security_patches.updates | length > 0

      when: ansible_os_family == "Windows"

    - name: Verificar parches de seguridad - RHEL
      block:
        - name: Verificar actualizaciones de seguridad pendientes
          shell: dnf check-update --security 2>/dev/null | tail -1
          register: security_update_count
          changed_when: false

        - name: Fallar si hay parches de seguridad pendientes
          fail:
            msg: "{{ security_update_count.stdout }} parches de seguridad pendientes"
          when: security_update_count.stdout | int > 0

      when: ansible_os_family == "RedHat"

    - name: Registrar cumplimiento
      uri:
        url: "{{ compliance_reporting_endpoint }}"
        method: POST
        body_format: json
        body:
          host: "{{ inventory_hostname }}"
          check_name: "{{ item.name }}"
          status: "compliant"
          timestamp: "{{ ansible_date_time.iso8601 }}"
      loop: "{{ compliance_checks }}"
      delegate_to: localhost
```

### 7.4 Gestión de Cambios

```yaml
# Archivo: security/change-management.yml
---
- name: Workflow de Gestión de Cambios
  hosts: localhost
  vars:
    change_id: "CHG-{{ ansible_date_time.date | replace('-', '') }}-001"
    change_manager: "change-approval@domain.local"
    change_type: "Patch Management"

  tasks:
    - name: Crear ticket de cambio
      uri:
        url: "{{ change_management_api }}/api/changes"
        method: POST
        headers:
          Authorization: "Bearer {{ vault_change_api_token }}"
        body_format: json
        body:
          change_id: "{{ change_id }}"
          type: "{{ change_type }}"
          description: "Parcheo automático de Windows Server 2016 y RHEL 8"
          impact: "Media"
          risk: "Media"
          scheduled_date: "{{ ansible_date_time.iso8601 }}"
          scheduled_window: "22:00 - 02:00 UTC"
          approver: "{{ change_manager }}"
          systems_affected:
            - "Windows Servers"
            - "RHEL Servers"
      register: change_ticket

    - name: Esperar aprobación del cambio
      uri:
        url: "{{ change_management_api }}/api/changes/{{ change_ticket.json.id }}/status"
        method: GET
        headers:
          Authorization: "Bearer {{ vault_change_api_token }}"
      register: change_status
      until: change_status.json.status == "approved"
      retries: 10
      delay: 30
      when: require_change_approval | default(true)

    - name: Ejecutar playbooks de parcheo
      command: |
        ansible-playbook playbooks/patch-orchestration.yml \
          --inventory inventory_vmware.yml \
          --extra-vars "change_id={{ change_id }}" \
          --vault-password-file /etc/ansible/vault-pass.txt
      register: patch_execution

    - name: Actualizar estado del cambio
      uri:
        url: "{{ change_management_api }}/api/changes/{{ change_ticket.json.id }}"
        method: PATCH
        headers:
          Authorization: "Bearer {{ vault_change_api_token }}"
        body_format: json
        body:
          status: "{% if patch_execution is succeeded %}completed{% else %}failed{% endif %}"
          execution_details: "{{ patch_execution.stdout }}"
      when: change_ticket is succeeded
```

---

## Cronograma de Implementación

### Timeline de 12 Semanas

| Semana | Fase | Hitos |
|--------|------|-------|
| 1-2 | Instalación de AAP | - Instalación de Controller<br>- Configuración inicial<br>- Certificados SSL |
| 2-3 | Integración VMware | - Instalación de dependencias<br>- Configuración de credenciales<br>- Pruebas de conectividad |
| 3-4 | Inventario Dinámico | - Plugin de inventario<br>- Población de hosts<br>- Validación de grupos |
| 4-5 | Credenciales | - Almacenamiento de credenciales<br>- Configuración de Vault<br>- Pruebas de acceso |
| 5-8 | Desarrollo Playbooks | - Playbooks Windows<br>- Playbooks RHEL<br>- Orquestación<br>- Pruebas en staging |
| 8-10 | Monitoreo | - Configuración Prometheus<br>- Dashboard Grafana<br>- Alertas |
| 10-11 | Seguridad | - RBAC<br>- Auditoría<br>- Cumplimiento |
| 11-12 | Pruebas Finales | - UAT completo<br>- Documentación<br>- Capacitación |

---

## Consideraciones de Riesgos

### Matriz de Riesgos

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|--------|-----------|
| Interrupción de servicios | Alta | Crítica | - Ventanas de mantenimiento definidas<br>- Snapshots pre-parcheo<br>- Rollback automatizado |
| Fallos de conectividad | Media | Alta | - Pruebas de conectividad pre-parcheo<br>- Retry logic en playbooks<br>- Failover a hosts alternos |
| Parches incompatibles | Baja | Crítica | - Pruebas en staging<br>- Validación post-parcheo<br>- Rollback disponible |
| Exposición de credenciales | Baja | Crítica | - Ansible Vault<br>- RBAC strict<br>- Auditoría completa |
| Ventana de mantenimiento excedida | Media | Alta | - Ejecución paralela<br>- Lotes optimizados<br>- Monitoreo en tiempo real |
| Pérdida de datos | Muy Baja | Crítica | - Backups pre-parcheo<br>- Snapshots VMware<br>- Verificación post-parcheo |

### Plan de Contingencia

```yaml
# Archivo: contingency-plan.yml
---
- name: Plan de Contingencia
  hosts: localhost
  vars:
    contingency_enabled: true
    
  handlers:
    - name: Activar modo de contingencia
      debug:
        msg: "Activando protocolo de contingencia"

    - name: Notificar equipo de emergencia
      mail:
        host: "{{ smtp_host }}"
        to: "{{ emergency_team }}"
        subject: "ALERTA: Se activó protocolo de contingencia"
        body: "Se detectó fallo crítico en el parcheo. Equipo de emergencia requerido."

    - name: Iniciar rollback global
      include_tasks: rollback-all-hosts.yml

    - name: Escalar a gerencia
      debug:
        msg: "Escalando a gerencia de IT"
```

---

## Conclusión

Este plan proporciona un framework completo para implementar **Ansible Automation Platform** como solución de automatización de parcheo en infraestructura VMware. La implementación debe realizarse de manera metódica, con validación en cada fase y participación de los equipos de operaciones, seguridad y cambios.

### Siguientes Pasos

1. Revisar y aprobar el plan con las partes interesadas
2. Asignar recursos y responsabilidades
3. Crear el ambiente de staging
4. Ejecutar Fase 1-2 en paralelo
5. Realizar pruebas de concepto (PoC) con un grupo piloto
6. Expandir gradualmente a producción
7. Documentar lecciones aprendidas
8. Establecer SLA y métricas de éxito

---

**Documento Versión:** 1.0  
**Última Actualización:** 2026-05-12  
**Próxima Revisión:** 2026-08-12