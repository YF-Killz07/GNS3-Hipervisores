# Laboratorio Avanzado de GNS3 sobre Windows 11 con Hipervisores

---

## Arquitectura de Virtualización en Windows 11

### Aislamiento de Núcleo y Virtualization-based Security (VBS)

Windows 11 implementa **Aislamiento de núcleo** y **Virtualization-based Security (VBS)** para proteger el sistema mediante un hipervisor integrado.  
Estas funciones de seguridad pueden bloquear tecnologías esenciales de virtualización como VT-x/AMD-V, provocando que GNS3 o VirtualBox no funcionen correctamente y que KVM aparezca como “False”.

**Recomendación:**  
Para laboratorios avanzados en GNS3, se recomienda desactivar temporalmente aislamiento de núcleo y VBS.

# Verificar estado de virtualización y seguridad
systeminfo

Para desactivar VBS:
Seguridad de Windows → Seguridad del dispositivo → Aislamiento de núcleo → Desactivar integridad de memoria

Activación de VT-x/AMD-V en BIOS/UEFI

Habilita la virtualización asistida desde BIOS/UEFI para que GNS3 VM funcione correctamente y soporte hipervisores anidados.

# Confirmar activación de virtualización asistida
systeminfo

Debe mostrar:
Virtualization Enabled In Firmware: Yes

Si no, habilitar en BIOS/UEFI.

GNS3 VM: Motor de Simulación con KVM

KVM (Kernel-based Virtual Machine) es un módulo Linux que utiliza VT-x/AMD-V para virtualización eficiente, indispensable para laboratorios complejos en GNS3.

# Verificar soporte de virtualización en VM
grep -E 'vmx|svm' /proc/cpuinfo

Si no devuelve resultados, la virtualización asistida no está activa.

Configuración recomendada de recursos
Recurso	Valor recomendado
CPU	2 a 4 núcleos físicos
RAM	4 a 8 GB
Disco	≥ 20 GB
Virtualización anidada	Activada para hipervisores internos
Integración con VirtualBox (Local)
Adaptador Host-Only

Configura un adaptador Host-Only para que la GUI de GNS3 se comunique con la VM de VirtualBox.

Ejemplo IP: 192.168.56.1

Modo Promiscuo

Permite capturar todo el tráfico de capa 2, esencial para simular redes reales.

# Habilitar virtualización anidada en VirtualBox
VBoxManage modifyvm "GNS3 VM" --nested-hw-virt on
Integración con VMware ESXi (Remoto)
Arquitectura Cliente-Servidor

La GUI de GNS3 en la laptop controla nodos virtuales en ESXi remoto mediante la API en puerto TCP 3080.

Seguridad en vSwitch
Política	Configuración
Promiscuous Mode	Permitir
MAC Address Changes	Permitir
Forged Transmits	Permitir

Esto garantiza el paso adecuado de tráfico de capa 2.

Solución de Problemas Comunes
Error Detectado	Causa Técnica	Solución Implementada
KVM support available: False	Hyper-V o VBS bloquea VT-x/AMD-V	Desactivar aislamiento de núcleo y Hyper-V
Sin conectividad en VirtualBox	Modo promiscuo deshabilitado o IP errónea	Activar modo promiscuo (“Permitir todo”), corregir IP
Error conexión puerto 3080	Firewall bloquea API GNS3	Permitir tráfico TCP en puertos 3080 y 5000-10000
VM GNS3 lenta	Recursos insuficientes	Aumentar CPU, RAM y activar virtualización anidada
Nodo no arranca en ESXi	Virtualización asistida no expuesta	Habilitar “Expose hardware assisted virtualization”
Buenas Prácticas
Realizar snapshots antes de cambios importantes.
Documentar asignaciones y configuraciones de recursos.
Evitar conflictos de IP en redes Host-Only y Bridge.
Revisar logs de GNS3 para detección temprana de errores.
Mantener actualizado GNS3 y los hipervisores.
Recursos y Referencias
Documentación Oficial GNS3
Manual VirtualBox
Guía Redes VMware ESXi
Visión General VBS Microsoft
