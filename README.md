# HomeLab: Servidor Casero en Debian + CasaOS sobre Hardware Reciclado

Documentación técnica de la instalación, configuración base, gestión de permisos y despliegue de servicios self-hosted sobre una notebook reutilizada de bajos recursos, orientado a prácticas de administración de sistemas Linux, redes y soporte técnico.

---

## 1. Especificaciones del Hardware y Sistema Base

* **Equipo base:** Notebook Lenovo Thinkpad X230
* **Procesador:** Intel Core i5-3320M 2.60GHz 
* **Memoria RAM:** 4 GB DDR3
* **Almacenamiento:** HDD 500 GB
* **Sistema Operativo:** Debian (instalación minimalista sin entorno gráfico / *headless* + paquete `openssh-server`)
* **Plataforma de gestión:** CasaOS (administración de contenedores Docker vía interfaz web)
* **Servicio principal desplegado:** Immich (nube privada de fotos y videos con módulo de Machine Learning desactivado para optimizar consumo de RAM/CPU)

---

## 2. Proceso de Instalación y Configuración Inicial

1. **Instalación minimalista de Debian:** Se realizó una instalación limpia desmarcando el entorno de escritorio para minimizar el consumo base de memoria RAM y seleccionando únicamente las utilidades estándar del sistema y el servidor **SSH** (`openssh-server`).
2. **Identificación de red:** Tras iniciar sesión en la terminal local, se obtuvo la dirección IP asignada en la red local mediante el comando:
   ```bash
   hostname -I
   ```

3. **Configuración de energía (operación con tapa cerrada):** Al tratarse de una notebook, se modificó /etc/systemd/logind.conf (`HandleLidSwitch=ignore` `HandleLidSwitchExternalPower=ignore` `HandleLidSwitchDocked=ignore` y `LidSwitchIgnoreInhibited=ignore`) para evitar que el sistema entre en suspensión al cerrar la tapa física y mantener la disponibilidad por red.

4. **Administración de privilegios (sudo):** Se instaló y configuró sudo para operar con el usuario estándar (david) siguiendo buenas prácticas de seguridad, evitando el uso continuo de la cuenta root.

5. **Instalación de dependencias y CasaOS:** Se instaló curl para descargar e implementar CasaOS, logrando administrar el servidor de forma remota desde el navegador de mi PC principal.

6. **Despliegue de Immich optimizado:** Desde CasaOS se instaló Immich como solución de almacenamiento multimedia en red local, deshabilitando el contenedor/servicio de Machine Learning para mantener el consumo de memoria estable dentro del límite de 4 GB de RAM.

## 3. Bitácora de Incidentes y Resoluciones (Troubleshooting Log)
### Incidente 1: Comando sudo no encontrado en instalación limpia de Debian

**Síntoma:** Al intentar editar la configuración de energía con sudo nano /etc/systemd/logind.conf, la terminal devolvió el error sudo: command not found.
**Causa raíz:** En instalaciones minimalistas de Debian donde se define una contraseña para root durante el instalador, el paquete sudo no se instala ni se configura por defecto.
**Solución aplicada:**

- Se escaló privilegios temporalmente mediante `su -` e ingreso de credenciales de root.
- Se realizó la edición inicial de /etc/systemd/logind.conf.
- Se actualizaron repositorios e instaló el paquete correspondiente:   
```bash
apt update && apt install sudo -y
```
- Se incorporó el usuario principal al grupo de administradores:
```bash
usermod -aG sudo david
```
**Verificación:** Se reinició la sesión del usuario david y se validó la ejecución de comandos administrativos anteponiendo sudo.

### Incidente 2: Dependencia faltante (curl) al instalar CasaOS

**Síntoma:** El script de instalación automatizada de CasaOS requería curl, herramienta no incluida en el perfil base instalado.
**Causa raíz:** Instalación minimalista de paquetes en Debian.
**Solución aplicada:** Instalación directa utilizando los privilegios recién configurados:
```bash
sudo apt install curl -y
```
**Verificación:** Ejecución exitosa del instalador de CasaOS y acceso al panel web desde otro equipo de la red local.

### Incidente 3: Prevención de saturación de RAM/CPU en Immich

**Riesgo identificado:** El módulo de Machine Learning de Immich (indexación por IA y reconocimiento facial) demanda altos recursos de memoria y procesador, lo que puede degradar el rendimiento general en un equipo con 4 GB de RAM y procesador Intel Core i3.
**Solución aplicada:** Se desactivó el componente de Machine Learning en la configuración de Immich dentro de CasaOS, priorizando la estabilidad del servicio de carga, respaldo y visualización de archivos.
**Verificación:** Monitoreo del consumo de memoria desde el panel de CasaOS y respuesta fluida de la interfaz web y conexiones SSH.

## 4. Próximas Mejoras Planificadas

    [ ] Configurar una IP estática (o reserva DHCP en el router) para evitar cambios de dirección tras reinicios.

    [ ] Implementar autenticación por llaves SSH y deshabilitar el acceso directo por contraseña de root.

    [ ] Configurar un firewall básico con UFW permitiendo únicamente los puertos necesarios (SSH, HTTP/HTTPS de CasaOS e Immich).

    [ ] Documentar métricas de consumo de RAM y temperatura en reposo y bajo carga.
