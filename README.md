# GNS3 + Hipervisores en Windows 11

![Estado](https://img.shields.io/badge/Estado-Investigación_en_Curso-yellow)
![Plataforma](https://img.shields.io/badge/Plataforma-GNS3%20%2B%20Windows11-blue)

## Objetivo

Investigar y documentar la implementación de laboratorios de red avanzados utilizando GNS3 sobre Windows 11, integrando hipervisores de tipo 1 (VMware ESXi) y tipo 2 (VirtualBox), analizando su arquitectura, configuración y resolución de errores.

## 1. Arquitectura de Virtualización en Windows 11

### Aislamiento de Núcleo y VBS

El Aislamiento de Núcleo (Core Isolation) y Virtualization-Based Security (VBS) son mecanismos de seguridad que utilizan virtualización para proteger procesos críticos del sistema operativo.

**Impacto en la virtualización:**

- Activan Hyper-V de forma implícita
- Bloquean el acceso a VT-x/AMD-V para otros hipervisores
- Generan errores como:
  - KVM support available: False
  - Bajo rendimiento en GNS3

**Solución recomendada:**

Desactivar:
- Aislamiento de núcleo
- Integridad de memoria

### Activación de VT-x / AMD-V

**Procedimiento:**

1. Reiniciar el equipo e ingresar al BIOS/UEFI
2. Activar:
   - Intel: Intel VT-x
   - AMD: SVM Mode
3. Guardar cambios

**Verificación en Windows:**

systeminfo

Buscar la línea:

Virtualization Enabled In Firmware: Yes


---

## 2. GNS3 VM: El Motor de Simulación

### KVM (Kernel-based Virtual Machine)

KVM es una tecnología de virtualización que permite ejecutar máquinas virtuales con acceso directo al hardware.

**Importancia en GNS3:**

- Permite ejecutar dispositivos de red
- Mejora significativamente el rendimiento
- Debe mostrarse como:

KVM support available: True

Si aparece en False:

No hay virtualización anidada
Hyper-V está interfiriendo
VT-x/AMD-V no está habilitado


---

### Configuración de Recursos

| Recurso | Recomendación |
|--------|-------------|
| CPU | 2 - 4 cores |
| RAM | 4GB - 8GB |
| Disco | 20GB mínimo |

**Criterios:**

- No asignar más del 50% de los recursos del host  
- Mantener estabilidad del sistema  
- Ajustar según la complejidad de la topología

## 3. Integración con VirtualBox (Local)

### Configuración de Red (Host-Only)

**Objetivo:**

Permitir comunicación entre el host (GUI) y la GNS3 VM.

**Pasos:**

1. Ir a VirtualBox → File → Host Network Manager  
2. Crear adaptador Host-Only  
3. Configurar:
   - IP: 192.168.56.1  
   - DHCP habilitado  
4. Asignar adaptador a la GNS3 VM:
   - Adapter 1: Host-Only

### Modo Promiscuo

El modo promiscuo permite que una interfaz capture todo el tráfico de red.

**Importancia en GNS3:**

- Necesario para tráfico de Capa 2  
- Permite:
  - ARP  
  - Broadcast  
  - VLANs  

**Configuración:**

Allow All


---


## 4. Integración con VMware ESXi (Remoto)

### Arquitectura Cliente-Servidor

**Componentes:**

- Cliente: GNS3 GUI (Windows 11)  
- Servidor: GNS3 VM en ESXi  

**Flujo de comunicación:**

Laptop (GUI) → Red → ESXi → GNS3 VM → Dispositivos

Ventajas:

Mayor rendimiento
Uso de hardware dedicado
Escalabilidad


---

### Seguridad en vSwitch

**Configuraciones importantes:**

- Promiscuous Mode  
- MAC Address Changes  
- Forged Transmits  

**Configuración recomendada:**

Promiscuous Mode: Accept
MAC Address Changes: Accept
Forged Transmits: Accept

Ubicación:

ESXi → Networking → Port Groups → Security

---

## 5. Matriz de Solución de Errores (Troubleshooting)

| Error Detectado | Causa Técnica | Solución Implementada |
|----------------|-------------|----------------------|
| KVM support available: False | No hay virtualización anidada o Hyper-V activo | Desactivar Hyper-V y ejecutar: `VBoxManage modifyvm "GNS3 VM" --nested-hw-virt on` |
| No hay conectividad entre Router y VM | Modo promiscuo deshabilitado | Activar "Allow All" en VirtualBox |
| Error conexión puerto 3080 | Firewall bloqueando la API de GNS3 | Abrir puertos 3080 y 5000-10000 |

## Conclusiones

- GNS3 requiere virtualización avanzada para funcionar correctamente  
- Windows 11 puede limitar el uso de hipervisores debido a VBS  
- VMware ESXi ofrece mejor rendimiento que VirtualBox  
- La configuración de red es fundamental en la simulación  
- El troubleshooting es clave en entornos virtualizados  
