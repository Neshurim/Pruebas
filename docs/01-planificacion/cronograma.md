# Cronograma del Proyecto

## 📅 Línea de Tiempo General

```
FASE 1: PLANIFICACIÓN        FASE 2: PREPARACIÓN      FASE 3: IMPLEMENTACIÓN    FASE 4: VALIDACIÓN
     (2 sem)                      (3 sem)                   (6 sem)                   (2 sem)
   └─────────────┘              └──────────────┘          └─────────────────┘      └───────────────┘
   Sem 1-2       Sem 3-5        Sem 6-8        Sem 9-14       Sem 15-16
   ─────────────────────────────────────────────────────────────────────────────────────────────────
```

---

## 📋 Cronograma Detallado por Fase

### FASE 1: PLANIFICACIÓN (Semanas 1-2)

| Semana | Actividad | Duración | Responsable | Estado |
|--------|-----------|----------|-------------|--------|
| 1 | Kickoff Meeting | 1 día | PM + Stakeholders | ☐ |
| 1 | Validación de Arquitectura | 2 días | Arch Team | ☐ |
| 1 | Revisión de Hardware | 2 días | Procurement | ☐ |
| 1 | Confirmación de Terceros (OLVM) | 1 día | PM | ☐ |
| 2 | Documentación Completa | 3 días | Tech Writer | ☐ |
| 2 | Aprobaciones de Stakeholders | 2 días | PM | ☐ |

**Entregables:**
- ✅ Documento de Arquitectura aprobado
- ✅ Especificaciones de hardware confirmadas
- ✅ Plan de comunicación establecido
- ✅ Recursos asignados

---

### FASE 2: PREPARACIÓN (Semanas 3-5)

#### Semana 3: Adquisición y Recepción de Hardware

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Validación de proveedores | 1 día | Procurement | ☐ |
| Recepción de hardware | 2 días | Warehouse | ☐ |
| Inspección física | 2 días | QA + Tech | ☐ |
| Documentación serial numbers | 1 día | Asset Mgmt | ☐ |

#### Semana 4: Pre-instalación y Cableado

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Preparación de racks | 2 días | DC Team | ☐ |
| Instalación física de servidores | 2 días | Hardware Tech | ☐ |
| Cableado SAN (Fiber) | 2 días | SAN Team | ☐ |
| Cableado Ethernet | 1 día | Network Team | ☐ |

#### Semana 5: Configuración Inicial

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Setup de red (VLAN, routing) | 2 días | Network Team | ☐ |
| Configuración IPMI/iLO | 1 día | Admin Team | ☐ |
| Firmware updates | 1 día | Hardware Tech | ☐ |
| BIOS tuning | 1 día | Performance Team | ☐ |

**Entregables:**
- ✅ Hardware instalado y cableado
- ✅ Red funcional entre datacenters
- ✅ Acceso IPMI validado
- ✅ Backups de configuración listos

---

### FASE 3: IMPLEMENTACIÓN (Semanas 6-11)

#### Semana 6: Instalación de Hypervisores (DC Principal)

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Instalación SO - HV-DC1-01 | 1 día | OS Team | ☐ |
| Instalación SO - HV-DC1-02 | 1 día | OS Team | ☐ |
| Instalación SO - HV-DC1-03 | 1 día | OS Team | ☐ |
| Instalación SO - HV-DC1-04 | 1 día | OS Team | ☐ |
| Validación inicial | 1 día | QA | ☐ |

#### Semana 7: Instalación de Hypervisores (DC Alterno)

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Instalación SO - HV-DC2-01,02,03 | 2 días | OS Team | ☐ |
| Configuración de red | 1 día | Network Team | ☐ |
| Validación y testing | 1 día | QA | ☐ |
| Sincronización de configuración | 1 día | Admin | ☐ |

#### Semana 8: Configuración OLVM (DC Principal)

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Instalación RHEL - OLVM nodes | 1.5 días | OS Team + Tercero | ☐ |
| Instalación del Engine OLVM | 1.5 días | Tercero especializado | ☐ |
| Configuración de Storage | 1 día | Storage Team | ☐ |
| Validación cluster | 0.5 días | QA | ☐ |

#### Semana 9: Configuración OLVM (DC Alterno) + Replicación

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Instalación RHEL - OLVM nodes | 1.5 días | OS Team + Tercero | ☐ |
| Instalación del Engine OLVM | 1.5 días | Tercero especializado | ☐ |
| Configuración de replicación SAN | 1 día | Storage Team | ☐ |
| Sincronización de clusters | 1 día | Admin + Tercero | ☐ |

#### Semana 10: Validación de Infraestructura Base

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Test de failover | 1 día | QA + Admin | ☐ |
| Test de replicación sincrónica | 1 día | Storage Team | ☐ |
| Validación de performance | 1 día | Performance Team | ☐ |
| Pruebas de backup/restore | 1 día | Backup Team | ☐ |

#### Semana 11: Preparación para Migración

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Auditoría de VMs existentes | 1.5 días | Migration Team | ☐ |
| Planificación de batch migration | 1.5 días | Migration Lead | ☐ |
| Creación de playbooks | 1 día | Automation Team | ☐ |
| Training del equipo | 1 día | Trainer | ☐ |

**Entregables:**
- ✅ 7 Hypervisores funcionales
- ✅ 8 nodos OLVM configurados
- ✅ Replicación sincrónica validada
- ✅ Plano de migración aprobado

---

### FASE 4: MIGRACIÓN DE VMs (Semanas 12-14)

#### Semana 12: Batch 1 - VMs Críticas (35 VMs)

| Batch | VMs | Horario | Responsable | Estado |
|-------|-----|---------|-------------|--------|
| Batch 1A | Bases de Datos (15 VMs) | Lunes 22:00 - Martes 04:00 | Migration Team | ☐ |
| Batch 1B | Aplicaciones críticas (20 VMs) | Miércoles 22:00 - Jueves 04:00 | Migration Team | ☐ |

**Actividades por Batch:**
- Snapshot previo
- Validación de replicación
- Migración de VM
- Testing post-migración (4 horas)
- Rollback si es necesario

#### Semana 13: Batch 2 - VMs Estándar (50 VMs)

| Batch | VMs | Horario | Responsable | Estado |
|-------|-----|---------|-------------|--------|
| Batch 2A | 25 VMs | Lunes-Martes nocturno | Migration Team | ☐ |
| Batch 2B | 25 VMs | Miércoles-Jueves nocturno | Migration Team | ☐ |

#### Semana 14: Batch 3 - VMs No-Críticas (20 VMs)

| Batch | VMs | Horario | Responsable | Estado |
|-------|-----|---------|-------------|--------|
| Batch 3A | 20 VMs | Viernes-Sábado | Migration Team | ☐ |

**Entregables:**
- ✅ 105 VMs migradas exitosamente
- ✅ Validación completa de todas las VMs
- ✅ Documentación de changes
- ✅ Plan de reversión si es necesario

---

### FASE 5: VALIDACIÓN Y OPTIMIZACIÓN (Semanas 15-16)

#### Semana 15: Testing Completo

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| UAT de aplicaciones | 2 días | App Team | ☐ |
| Validación de performance | 1 día | Performance Team | ☐ |
| Pruebas de seguridad | 1 día | Security Team | ☐ |
| Validación de backups | 0.5 días | Backup Team | ☐ |

#### Semana 16: Finalización y Go-Live

| Actividad | Duración | Responsable | Estado |
|-----------|----------|-------------|--------|
| Signoff final | 0.5 días | Stakeholders | ☐ |
| Turnover a Ops | 0.5 días | Admin Team | ☐ |
| Documentación final | 1 día | Tech Writer | ☐ |
| Post-implementation review | 1 día | PM | ☐ |

**Entregables:**
- ✅ Sistema en producción
- ✅ Documentación operacional completa
- ✅ Plan de soporte 24/7
- ✅ Lecciones aprendidas documentadas

---

## 🎯 Hitos Clave

| Hito | Fecha | Entregable | Responsable |
|------|-------|-----------|-------------|
| Kickoff | Sem 1 | Proyecto iniciado | PM |
| Hardware Recibido | Sem 3 | Todos los servidores en DC | Procurement |
| Infraestructura Base | Sem 11 | Hypervisores + OLVM + Storage | Tech Lead |
| Migración Iniciada | Sem 12 | Batch 1 completado | Migration Lead |
| Migración Completada | Sem 14 | 105 VMs en DC principal | Migration Lead |
| Go-Live | Sem 16 | Sistema operativo | PM |

---

## 👥 Asignación de Recursos

### Equipo Permanente
```
Project Manager: 1 FTE
├─ Supervisión general
├─ Coordinación entre equipos
└─ Reporte a stakeholders

Technical Lead: 1 FTE
├─ Arquitectura
├─ Decisiones técnicas
└─ Escalaciones

DC/Infrastructure Team: 4 FTE
├─ Instalación física
├─ Cableado
├─ Rack & stack
└─ Mantenimiento

Network Team: 2 FTE
├─ Configuración de red
├─ SAN fabric setup
└─ Monitoreo

Storage Team: 2 FTE
├─ SAN arrays setup
├─ Replicación
└─ Performance tuning
```

### Equipo Especializado (Tercero OLVM)
```
Tercero OLVM: 2-3 especialistas
├─ Instalación de RHEL
├─ Configuración OLVM
├─ Cluster setup
├─ Training
└─ Support inicial (4 semanas post-implementación)
```

### Equipo de Migración
```
Migration Lead: 1 FTE
├─ Planificación
├─ Automatización
└─ Coordinación

Admins de Sistemas: 2-3 FTE
├─ Migración de VMs
├─ Testing post-migración
├─ Validación
└─ Rollback si es necesario

QA/Testing: 1-2 FTE
├─ Validación funcional
├─ Performance testing
└─ Signoff
```

---

## 📊 Matriz de Riesgos por Fase

### Semanas 1-2 (Planificación)
| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|--------|-----------|
| Scope creep | Medio | Alto | Gate formal de aprobación |
| Falta de aprobación | Bajo | Alto | Comunicación temprana |

### Semanas 3-5 (Preparación)
| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|--------|-----------|
| Retraso en delivery hardware | Medio | Medio | Plan B con proveedores alternativos |
| Problemas de cableado | Medio | Medio | Validación previa + checklist |
| Falta de espacio en rack | Bajo | Alto | Pre-auditoría del DC |

### Semanas 6-11 (Implementación)
| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|--------|-----------|
| Issues con replicación SAN | Medio | Alto | Testing exhaustivo en lab |
| Tercero no disponible | Bajo | Alto | Backup técnico identificado |
| Problemas de performance | Medio | Medio | Tuning proactivo, benchmarking |

### Semanas 12-14 (Migración)
| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|--------|-----------|
| Migración fallida | Bajo | Alto | Rollback plan detallado |
| Data corruption | Muy Bajo | Crítico | Snapshots pre-migración |
| Downtime no planeado | Bajo | Alto | Ventana de tiempo reservada |

---

## 📞 Contactos Clave

| Rol | Nombre | Email | Teléfono |
|-----|--------|-------|----------|
| Project Manager | [NOMBRE] | [EMAIL] | [TELÉFONO] |
| Technical Lead | [NOMBRE] | [EMAIL] | [TELÉFONO] |
| Storage Lead | [NOMBRE] | [EMAIL] | [TELÉFONO] |
| Network Lead | [NOMBRE] | [EMAIL] | [TELÉFONO] |
| Migration Lead | [NOMBRE] | [EMAIL] | [TELÉFONO] |
| Tercero OLVM Lead | [NOMBRE] | [EMAIL] | [TELÉFONO] |

---

## 📝 Notas Importantes

- **Comunicación:** Reunión diaria (15 min) durante implementación
- **Escalaciones:** Directamente al PM si hay bloqueadores
- **Cambios:** Requerir aprobación formal vía Change Advisory Board (CAB)
- **Downtime permitido:** 4 horas máximo por migración
- **Rollback:** Posible hasta 24 horas post-migración

---

**Documento versión:** 1.0  
**Última actualización:** 2026-06-10  
**Próxima revisión:** Semanal
