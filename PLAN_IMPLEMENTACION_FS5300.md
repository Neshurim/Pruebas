# Plan de Implementación - Almacenamiento FS5300 50TB

**Documento:** Plan de Implementación  
**Fecha Creación:** 2026-05-08  
**Duración Proyecto:** 8 Semanas (2 meses)  
**Estado:** En Preparación  
**Criticidad:** CRÍTICA - Sistema soporta operación de la compañía  

---

## 1. RESUMEN EJECUTIVO

Este documento detalla la implementación de un sistema de almacenamiento FS5300 de 50TB que servirá como repositorio centralizado para bases de datos, reemplazando/complementando la actual infraestructura SAN HP. El proyecto tiene una duración máxima de 8 semanas y debe garantizar cero interrupciones en la operación de la compañía.

**Objetivos Clave:**
- Implementar FS5300 con 50TB de capacidad
- Migrar datos de SAN HP sin downtime
- Soportar 50 usuarios/aplicaciones simultáneas
- Garantizar disponibilidad 99.9%
- Completar proyecto en máximo 8 semanas

---

## 2. ANÁLISIS DE RIESGOS

### 2.1 Matriz de Riesgos

| ID | Riesgo | Probabilidad | Impacto | Criticidad | Plan Mitigación |
|----|--------|--------------|---------|-----------|-----------------|
| R1 | Pérdida de datos durante migración | Media (60%) | Crítico (10) | **CRÍTICO** | Backup completo pre-migración, verificación checksum |
| R2 | Incompatibilidad de aplicaciones con FS5300 | Media (50%) | Alto (8) | **ALTO** | Testing en ambiente de staging 2 semanas antes |
| R3 | Problemas de rendimiento post-implementación | Media (55%) | Alto (8) | **ALTO** | Tuning previo, benchmarking, monitoreo 24/7 |
| R4 | Falla de hardware FS5300 | Baja (15%) | Crítico (10) | **ALTO** | Garantía 24/7, hardware redundante disponible |
| R5 | Cortes de energía durante implementación | Baja (20%) | Crítico (10) | **ALTO** | UPS redundante, horario mantenimiento nocturno |
| R6 | Problemas de conectividad de red | Baja (25%) | Alto (8) | **MEDIO-ALTO** | Testing de ancho de banda, switch redundante |
| R7 | Falta de capacitación del personal | Media (50%) | Medio (6) | **MEDIO** | Sesiones de training, documentación técnica |
| R8 | Retrasos en procurement de hardware | Media (45%) | Medio (6) | **MEDIO** | Orden de componentes inmediata, buffer de 1 semana |
| R9 | Incompatibilidad de protocolos (NFS/SMB/iSCSI) | Media (40%) | Alto (8) | **MEDIO-ALTO** | Testing previo de protocolos soportados |
| R10 | Licencias/certificaciones incompletas | Baja (20%) | Medio (6) | **MEDIO** | Validación contractual en Semana 1 |

---

## 3. CONSIDERACIONES DE RIESGOS DETALLADAS

### 3.1 Riesgos de DATOS

**R1: Pérdida de Datos Durante Migración**
- **Descripción:** Corrupción o pérdida de datos al transferir 50TB desde SAN HP a FS5300
- **Probabilidad:** 60% (si no se ejecuta protocolo riguroso)
- **Impacto:** Crítico - Paralización completa de operaciones
- **Mitigación:**
  - Backup full + incremental del SAN HP antes de cualquier operación
  - Verificación de integridad con checksums MD5/SHA256
  - Migración en fases: test con 10% primero
  - Mantener SAN HP durante 30 días post-migración como rollback
  - Validación de datos post-transferencia (byte-level comparison)

**R2: Inconsistencia de Datos en Aplicaciones**
- **Descripción:** Aplicaciones dependientes de ciertas características de SAN que FS5300 no soporta igual
- **Probabilidad:** 50%
- **Impacto:** Alto - Errores operacionales, corrupción de índices
- **Mitigación:**
  - Auditoría de compatibilidad en Semana 2
  - Testing de todas 50 aplicaciones en staging
  - Validación de IOPS/latencia requerida por app

---

### 3.2 Riesgos de INFRAESTRUCTURA

**R3: Fallas de Rendimiento**
- **Descripción:** FS5300 no alcanza performance esperada bajo carga de 50 usuarios/apps
- **Probabilidad:** 55%
- **Impacto:** Alto - Degradación de velocidad, timeouts de aplicaciones
- **Mitigación:**
  - Benchmark pre-implementación contra SAN HP
  - Configuración optimizada de cache/buffers
  - Load testing en staging (simulación de 50 usuarios)
  - Monitoreo de latencia p95/p99
  - Plan de escalado horizontal si es necesario

**R4: Falla de Hardware FS5300**
- **Descripción:** Fallo catastrófico del storage (disco, controlador, PSU)
- **Probabilidad:** 15% en primeros 6 meses
- **Impacto:** Crítico - Downtime total
- **Mitigación:**
  - Garantía 24/7 con reemplazo de piezas en 4 horas
  - RAID redundante (RAID-6 mínimo)
  - Snapshot horarios automáticos
  - Replicación a SAN HP como DR (si presupuesto permite)
  - Hardware spare en sitio para componentes críticos

**R5: Corte de Energía**
- **Descripción:** Fallo de suministro eléctrico durante migración/operación
- **Probabilidad:** 20% (si no hay UPS o falla)
- **Impacto:** Crítico
- **Mitigación:**
  - UPS con capacidad para 30 min mínimo (FS5300 + networking)
  - PDU redundante
  - Generador diésel en sitio o contratado
  - Implementación en horario nocturno cuando hay menor consumo

**R6: Problemas de Conectividad de Red**
- **Descripción:** Latencia, pérdida de paquetes, ancho de banda insuficiente
- **Probabilidad:** 25%
- **Impacto:** Alto - Timeouts, degradación de performance
- **Mitigación:**
  - Testing de ancho de banda requerido (50TB en N semanas = X Mbps)
  - Bonding de NICs (link aggregation)
  - Switch redundante en la arquitectura
  - QoS configurado para storage traffic
  - Monitoreo de jitter/latencia

---

### 3.3 Riesgos de OPERACIÓN

**R7: Falta de Capacitación**
- **Descripción:** Personal no preparado para administrar FS5300
- **Probabilidad:** 50%
- **Impacto:** Medio - Errores de administración, downtime innecesario
- **Mitigación:**
  - Capacitación obligatoria en Semana 7
  - Documentación técnica completa (runbooks)
  - Designar 2 admin como "expertos FS5300"
  - Período de acompañamiento de 2 semanas post go-live
  - Soporte del vendor disponible 24/7 inicialmente

**R8: Retrasos en Procurement**
- **Descripción:** Hardware no llega a tiempo
- **Probabilidad:** 45%
- **Impacto:** Medio - Retraso de proyecto
- **Mitigación:**
  - Órdenes en Semana 1 inmediata
  - Contrato con penalización por demora
  - Buffer de 1 semana en cronograma
  - Proveedor backup identificado

---

### 3.4 Riesgos de COMPATIBILIDAD

**R9: Incompatibilidad de Protocolos**
- **Descripción:** Aplicaciones requieren protocolos no soportados (NFS v3/v4, SMB v2/v3, iSCSI, etc.)
- **Probabilidad:** 40%
- **Impacto:** Alto - Aplicaciones no pueden conectar
- **Mitigación:**
  - Auditoría de protocolos en Semana 1
  - Testing de cada protocolo en staging
  - Potencial necesidad de gateways/adaptadores
  - Plan B: mantener SAN HP para apps incompatibles

**R10: Licencias/Certificaciones**
- **Descripción:** Falta de licencias de software, certificaciones no vigentes
- **Probabilidad:** 20%
- **Impacto:** Medio - Cumplimiento legal, soporte no disponible
- **Mitigación:**
  - Validación contractual en Semana 1
  - Asegurar renovación de licencias
  - Documentar todos los SLAs con vendor

---

## 4. POSIBLES IMPREVISTOS Y PLANES DE CONTINGENCIA

### 4.1 Imprevistos Técnicos

| Imprevisto | Probabilidad | Respuesta | Responsable | Tiempo |
|-----------|--------------|----------|------------|--------|
| **Migración más lenta de lo previsto** | Alta | Paralelizar migración, aumentar ancho de banda | Ing. Infraestructura | 4-8h |
| **Aplicación rechaza conectar a FS5300** | Media | Depuración, ajuste de configuración, revert temporal | Ing. Soporte | 2-4h |
| **Degradación de performance post go-live** | Media | Análisis de logs, retuning, rollback parcial si necesario | Ing. Performance | 2-6h |
| **Disco del FS5300 falla** | Baja | Activar RAID rebuild, reemplazar disco | Vendor + Interno | 1-4h |
| **Pérdida de conectividad de red** | Baja | Failover a NIC secundaria, diagnóstico de switch | Ing. Redes | 1-2h |

### 4.2 Imprevistos Operacionales

| Imprevisto | Probabilidad | Respuesta | Responsable |
|-----------|--------------|----------|------------|
| **Personal clave no disponible** | Media | Activar plan de backup, soporte del vendor 24/7 | Gerencia |
| **Problema con licencias/contrato** | Baja | Escalación inmediata con vendor | Legal + Gerencia |
| **Retrasos en shipping** | Media | Activar proveedor backup, usar SAN HP temporalmente | Procurement |
| **Cambios de requisitos** | Alta | Documentar cambio, impacto analysis, reprogramar | PM + Stakeholders |

### 4.3 Plan de Rollback Inmediato

**Si algo falla crítico:**

1. **Dentro de 4 horas:**
   - Mantener SAN HP funcional durante todo el proyecto
   - Revertir aplicaciones a apuntar a SAN HP
   - Generar incidente post-mortem

2. **Después de 30 días:**
   - Eliminar FS5300 solo después de confirmar estabilidad
   - Recuperar datos del backup si hubo corrupción

3. **Comunicación:**
   - Notificar a usuarios 15 min después de identificar problema
   - Actualizaciones cada 15 minutos durante incidente

---

## 5. CRONOGRAMA DE IMPLEMENTACIÓN (8 SEMANAS)

### SEMANA 1: PREPARACIÓN Y PLANIFICACIÓN
**Duración:** 5 días de trabajo

| Día | Actividad | Responsable | Deliverable |
|-----|-----------|------------|-------------|
| L-M | Kick-off meeting, validación de requisitos | PM | Acta de reunión |
| M | Auditoría de aplicaciones y protocolos | Ing. Infraestructura | Reporte de compatibilidad |
| J | Validación de licencias y contratos | Legal + Vendor | Checklist de SLAs |
| V | Planning detallado, asignación de recursos | PM | Cronograma refinado |
| V | Órdenes de hardware y componentes | Procurement | PO generadas |

**Hito:** Equipo listo, hardware ordenado, plan validado ✅

---

### SEMANA 2: INFRAESTRUCTURA FÍSICA Y STAGING
**Duración:** 7 días

| Día | Actividad | Responsable | Deliverable |
|-----|-----------|------------|-------------|
| L-M | Instalación física del FS5300 | Ing. Infraestructura | Fotos/documentación |
| M-J | Configuración inicial (networking, RAID, firmware) | Ing. Infraestructura | Documento de configuración |
| J-V | Preparación del ambiente de staging | Ing. QA | Staging listo |
| V | Testing inicial de conectividad | Ing. Redes | Test report |

**Hito:** FS5300 físicamente instalado y funcional en staging ✅

---

### SEMANA 3: TESTING Y VALIDACIÓN
**Duración:** 7 días

| Día | Actividad | Responsable | Deliverable |
|-----|-----------|------------|-------------|
| L-M | Testing de protocolos (NFS, SMB, iSCSI) | Ing. QA | Protocol compatibility matrix |
| M-J | Benchmark de performance vs SAN HP | Ing. Performance | Performance report |
| J-V | Testing de failover y recuperación | Ing. QA | DR test results |
| V | Load testing con 50 usuarios simulados | Ing. QA | Load test report |

**Hito:** Todos los protocolos validados, performance cumple requisitos ✅

---

### SEMANA 4: TESTING DE APLICACIONES
**Duración:** 7 días

| Día | Actividad | Responsable | Deliverable |
|-----|-----------|------------|-------------|
| L-M-J | Testing de primeras 20 aplicaciones | Ing. Soporte + Devs | Reporte de compatibilidad (20/50) |
| J-V | Testing de 15-30 aplicaciones | Ing. Soporte + Devs | Reporte de compatibilidad (35/50) |
| V | Identificación de incompatibilidades | Ing. Soporte | Lista de issues + soluciones |

**Hito:** 35/50 aplicaciones validadas, problemas documentados ✅

---

### SEMANA 5: PREPARACIÓN DE MIGRACIÓN
**Duración:** 7 días

| Día | Actividad | Responsable | Deliverable |
|-----|-----------|------------|-------------|
| L-M | Backup completo del SAN HP (50TB) | Ing. Backup | Backup verification report |
| M-J | Planificación detallada de migración por fases | PM + Ing. Infraestructura | Migration plan document |
| J-V | Preparación de scripts de migración | Ing. Infraestructura | Migration scripts + testing |
| V | Definición de ventanas de mantenimiento | PM | Maintenance window approved |

**Hito:** Backup validado, scripts de migración listos ✅

---

### SEMANA 6: MIGRACIÓN FASE 1 (10TB - Test)
**Duración:** 7 días (principalmente nocturno)

| Día | Actividad | Responsable | Deliverable |
|-----|-----------|------------|-------------|
| L | Comunicación a usuarios (migration window L-M noche) | PM | Email de notificación |
| L Noche | Migración de 10TB (test) | Ing. Infraestructura | Migration log |
| M | Validación de datos migrados | Ing. QA | Validation report |
| M | Testing de aplicaciones con 10TB datos | Ing. Soporte | Compatibility report |
| M-J | Resolución de issues encontrados | Ing. Soporte | Issue log + fixes |
| J-V | Stabilidad monitoring | Ing. Operaciones | 48h monitoring report |

**Hito:** Primer 10TB migrado exitosamente, aplicaciones funcionando ✅

---

### SEMANA 7: MIGRACIÓN FASE 2 (40TB - Producción)
**Duración:** 7 días

| Día | Actividad | Responsable | Deliverable |
|-----|-----------|------------|-------------|
| L | Última comunicación a usuarios | PM | Email notification |
| L-M Noche | Migración de 40TB restantes (batch parallel) | Ing. Infraestructura | Migration completion report |
| M | Validación completa (50TB checksums) | Ing. QA | Full validation report |
| M-J | Cutover: apuntar 50 apps a FS5300 | Ing. Soporte | Cutover log |
| J-V | Monitoreo intensivo 24/7 | Ing. Operaciones | 72h monitoring report |
| V | Capacitación del equipo de operaciones | Ing. Infraestructura | Training attendance list |

**Hito:** 100% datos migrados, todas las aplicaciones apuntando a FS5300 ✅

---

### SEMANA 8: ESTABILIZACIÓN Y CIERRE
**Duración:** 7 días

| Día | Actividad | Responsable | Deliverable |
|-----|-----------|------------|-------------|
| L-M-J | Monitoreo 24/7 de performance y disponibilidad | Ing. Operaciones | Daily status reports |
| M | Creación de runbooks de operación | Ing. Infraestructura | Operations documentation |
| J | Review post-implementación (lecciones aprendidas) | PM + Team | Lessons learned document |
| V | Aceptación formal de FS5300 en producción | Gerencia | Sign-off document |

**Hito:** FS5300 estable en producción, proyecto cerrado ✅

---

## 6. DETALLES DEL CRONOGRAMA

### 6.1 Hitos Críticos

```
Semana 1 → Hardware Ordenado ✅
Semana 2 → FS5300 Instalado ✅
Semana 3 → Testing Completo ✅
Semana 4 → Apps Validadas ✅
Semana 5 → Backup y Scripts Listos ✅
Semana 6 → Fase 1 Migración (10TB) ✅
Semana 7 → Fase 2 Migración (40TB) + Cutover ✅ ⭐ CRÍTICO
Semana 8 → Estabilización y Cierre ✅
```

### 6.2 Duración por Tipo de Actividad

| Tipo | Horas | % del Proyecto |
|------|-------|----------------|
| Preparación/Planificación | 40h | 10% |
| Infraestructura Física | 30h | 7% |
| Testing y Validación | 80h | 20% |
| Capacitación | 20h | 5% |
| Migración de Datos | 100h | 25% |
| Monitoreo y Estabilización | 100h | 25% |
| Documentación | 30h | 8% |
| **TOTAL** | **400h** | **100%** |

### 6.3 Equipo Requerido

| Rol | Cantidad | Horas/Semana | Semanas |
|-----|----------|--------------|---------|
| Project Manager | 1 | 40h | 8 |
| Ing. Infraestructura (Lead) | 1 | 40h | 8 |
| Ing. Infraestructura (Support) | 1 | 30h | 6-8 |
| Ing. QA/Testing | 1 | 30h | 3-4 |
| Ing. Redes | 1 | 20h | 2-3 |
| Ing. Soporte (Apps) | 2 | 40h | 4-7 |
| Ing. Operaciones | 1 | 50h | 6-8 |
| Vendor Support | On-call | 24/7 | 8 |

**Total recursos:** ~7-8 personas full-time equivalente + vendor

---

## 7. RECURSOS NECESARIOS

### 7.1 Hardware

| Item | Especificación | Cantidad | Costo Est. |
|------|----------------|----------|-----------|
| FS5300 | 50TB, RAID-6 | 1 | $$$$$$ |
| NICs 10Gb | Para link aggregation | 2 | $ |
| Switch manageable | 10Gb ports, redundancia | 1 | $$ |
| UPS | 30 min mínimo, FS5300 + networking | 1 | $$ |
| Cables de red Cat6A | Meter apropriado | 100m | $ |
| Componentes spare | Discos, PSU, controlador | Set | $$ |

### 7.2 Software

| Item | Descripción | Licencias |
|------|-------------|-----------|
| FS5300 OS | Sistema operativo | Incluido |
| Monitoreo (Nagios/Prometheus) | Para alertas 24/7 | Licencia existente o free |
| Backup (Veeam/Veritas) | Incrementales automáticas | Licencia existente + extensión |
| Scripts de migración | rsync/robocopy/native tools | Herramientas free |

### 7.3 Servicios

| Servicio | Descripción | Duración |
|---------|------------|----------|
| Soporte Vendor 24/7 | Technical support nivel 3 | 8 semanas |
| Capacitación on-site | Entrenamiento del equipo | 2 días |
| Consultoría (opcional) | Si hay incompatibilidades | As-needed |

---

## 8. VALIDACIÓN Y PRUEBAS

### 8.1 Testing Pre-Producción (Semanas 3-4)

**Protocolos a validar:**
- ✅ NFS v3 / v4 (si aplica para Unix/Linux)
- ✅ SMB v2 / v3 (Windows)
- ✅ iSCSI (si hay conexiones directas)
- ✅ HTTP/HTTPS (si aplica)
- ✅ API nativas del FS5300

**Benchmark de Performance:**
```
Métrica         | SAN HP Actual | FS5300 Target | Pass/Fail
IOPS Lectura    | X IOPS        | ≥ X IOPS      |
IOPS Escritura  | Y IOPS        | ≥ 0.9Y IOPS   |
Latencia p95    | A ms          | ≤ A ms        |
Latencia p99    | B ms          | ≤ B ms        |
Throughput      | Z MB/s        | ≥ Z MB/s      |
```

### 8.2 Testing de Migración (Semana 6)

**Fase 1: Test con 10TB**
- Migración de datos de test
- Validación de checksums (MD5/SHA256)
- Testing de 5 aplicaciones críticas
- Monitoreo de 48 horas
- Criterio de aceptación: 100% datos íntegros, 0 errores de aplicación

### 8.3 Testing Post Go-Live (Semana 7-8)

**Monitoreo 24/7:**
- IOPS / Throughput en tiempo real
- Latencia de lectura/escritura
- Errores de red / timeouts
- Uso de CPU/Memoria del FS5300
- Errores en logs de aplicaciones
- Disponibilidad (target: 99.9%)

---

## 9. PLAN DE SEGURIDAD Y COMPLIANCE

### 9.1 Seguridad de Datos

- ✅ Autenticación (LDAP/AD si aplica)
- ✅ Encriptación en reposo (si soportado)
- ✅ Encriptación en tránsito (SSL/TLS)
- ✅ Acceso basado en roles (RBAC)
- ✅ Auditoría de acceso a datos

### 9.2 Backup y Disaster Recovery

- ✅ Snapshot horario del FS5300
- ✅ Backup incremental diario
- ✅ Replicación a ubicación alterna (si presupuesto)
- ✅ RTO: < 4 horas
- ✅ RPO: < 1 hora

### 9.3 Cumplimiento

- ✅ Validar compliance con políticas internas
- ✅ Documentar todas las decisiones
- ✅ Mantener logs de auditoría
- ✅ Acuerdos SLA con vendor

---

## 10. PLAN DE COMUNICACIÓN

### 10.1 Stakeholders

| Grupo | Frecuencia | Canales |
|-------|-----------|---------|
| Ejecutivos | Semanal | Email ejecutivo |
| Usuarios | Diaria (si hay issues) | Email + Chat |
| Equipo técnico | Diaria | Standup 15 min |
| Vendor | As-needed | Llamadas + tickets |

### 10.2 Ventanas de Comunicación

```
Semana 6 (L 18:00): "Migración de test planificada L-M 22:00-06:00"
Semana 7 (L 18:00): "Migración final planificada L-M 22:00-08:00"
Cada día: Status actualizado en Slack/Teams
```

---

## 11. DOCUMENTACIÓN REQUERIDA

### 11.1 Documentos a Generar

1. ✅ **Configuración del FS5300** (networking, RAID, snapshots)
2. ✅ **Matriz de Compatibilidad de Aplicaciones** (50 apps validadas)
3. ✅ **Runbooks Operacionales** (start/stop, recovery, troubleshooting)
4. ✅ **Procedimiento de Backup** (frecuencia, validación, restore)
5. ✅ **Plan de Escalabilidad** (si se necesita más almacenamiento)
6. ✅ **Logs de Migración** (auditoría completa)
7. ✅ **Lecciones Aprendidas** (post-implementación)

### 11.2 Documentación Existente a Actualizar

- Diagrama de infraestructura
- Runbooks de operaciones
- Matriz de disponibilidad/SLA
- Matriz de recuperación ante desastres

---

## 12. MÉTRICAS DE ÉXITO

### 12.1 Criterios de Aceptación

| Métrica | Objetivo | Cómo Medir |
|---------|----------|-----------|
| Integridad de Datos | 100% datos migrados sin pérdida | Checksums validados |
| Disponibilidad | 99.9% (uptime) | Monitoring 24/7 |
| Performance | ≥ 90% comparado con SAN HP | Benchmarks |
| Compatibilidad Apps | 50/50 apps funcionando | Testing validado |
| Tiempo de Migración | ≤ 8 semanas | Cronograma cumplido |
| Capacitación | 100% del equipo entrenado | Attendance validado |
| Documentación | 7/7 docs completadas | Review aprobado |

### 12.2 Métricas Operacionales (Post Go-Live)

Medir durante las primeras 4 semanas:
- MTTR (Mean Time To Recovery): < 1 hora
- MTBF (Mean Time Between Failures): > 720 horas
- Satisfacción de usuarios: ≥ 90%
- Errores de aplicación relacionados a storage: ≤ 1 por semana

---

## 13. PLAN B (CONTINGENCIA TOTAL)

Si el proyecto enfrenta problemas críticos irrecuperables:

### 13.1 Rollback Completo
- Mantener SAN HP operativo
- Revertir todas las aplicaciones a SAN HP dentro de 4 horas
- Evaluación de alternativas post-crisis

### 13.2 Alternativas

**Si FS5300 no es viable:**
1. Continuar con SAN HP + upgrades
2. Evaluar otros proveedores (NetApp, Dell EMC, Huawei)
3. Considerar solución híbrida (SAN HP + NAS)

---

## 14. PRESUPUESTO ESTIMADO

| Categoría | Estimado | Notas |
|-----------|----------|-------|
| Hardware FS5300 | $$ | Incluye discos 50TB, controlador, PSU redundante |
| Infraestructura (NICs, switch, UPS) | $ | |
| Licencias/Soporte Vendor | $ | 24/7 support por 8 semanas |
| Recursos Internos (400h @ $75/h) | $30,000 | 7-8 personas x 8 semanas |
| Capacitación | $ | On-site training del vendor |
| Contingencia (10%) | Variable | Para imprevistos |
| **TOTAL ESTIMADO** | **$$$$$** | Requiere quote específico del vendor |

---

## 15. AUTORIZACIÓN Y FIRMA

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| Project Manager | _____________ | _______ | _______ |
| Director Infraestructura | _____________ | _______ | _______ |
| CIO/Gerente General | _____________ | _______ | _______ |
| Vendor Representante | _____________ | _______ | _______ |

---

## ANEXO A: COMANDOS DE MIGRACIÓN (BORRADOR)

```bash
# Validación de origen
$ rsync -vvn --checksum --recursive \
  /mnt/san_hp/databases/ \
  /mnt/fs5300_staging/databases/

# Migración de prueba (10TB)
$ rsync -av --checksum --recursive \
  --bwlimit=100000 \
  /mnt/san_hp/databases/ \
  /mnt/fs5300_staging/databases/

# Validación de integridad
$ find /mnt/san_hp/databases -type f -exec md5sum {} \; > /tmp/md5sum_sanHP.txt
$ find /mnt/fs5300_staging/databases -type f -exec md5sum {} \; > /tmp/md5sum_fs5300.txt
$ diff /tmp/md5sum_sanHP.txt /tmp/md5sum_fs5300.txt

# Migración producción (paralela, 40TB)
$ rsync -av --checksum --recursive --parallel=4 \
  /mnt/san_hp/ \
  /mnt/fs5300/

# Monitoreo de migración
$ watch -n 5 'df -h /mnt/fs5300'
$ iostat -x 1 /dev/storagedevice
```

---

## ANEXO B: MATRIZ DE RESPONSABILIDADES (RACI)

```
                                R = Responsible (Ejecuta)
                                A = Accountable (Autoriza)
                                C = Consulted (Consulta)
                                I = Informed (Informado)

Actividad                       | PM | Ing.Infra | Ing.QA | Vendor | Usuarios
--------------------------------|----| ---------|--------|--------|----------
Planificación proyecto          | R  | C        | C      | C      | I
Instalación hardware            | C  | R        | -      | C      | -
Testing protocolos              | I  | C        | R      | C      | -
Testing aplicaciones            | C  | C        | R      | -      | C
Migración datos                 | A  | R        | C      | C      | I
Capacitación equipo             | C  | R        | -      | R      | I
Cutover a producción            | A  | R        | C      | C      | I
Monitoreo post go-live          | C  | R        | C      | C      | I
Documentación final             | R  | C        | C      | -      | -
```

---

**Versión:** 1.0  
**Última Actualización:** 2026-05-08  
**Siguiente Revisión:** Semana 2 del proyecto

---

*Este documento es CONFIDENCIAL y debe ser controlado según políticas de seguridad de la organización.*
