REPORTE DE AUDITORÍA DE SEGURIDAD: Maquina SimpleCTF en TryHackMe
Auditor: Henry Leyton 
Objetivo: Identificar vectores de ataque y vulnerabilidades en el servidor objetivo para evaluar su estado de seguridad.
Herramientas: Nmap, conexion ftp y ssh, ataque de fuerza bruta con Hydra.

1. RESUMEN
Durante la auditoría se logró comprometer totalmente el sistema (Root). El vector de entrada inicial fue una combinación de divulgación de información en un servicio FTP mal configurado y credenciales débiles en el servicio SSH. Finalmente, se logró la escalada de privilegios a nivel administrativo debido a una mala configuración en los permisos de sudoers.

2. HALLAZGOS TÉCNICOS
Hallazgo 1: Escalada de Privilegios vía Sudo (Vim)
Severidad (CVSS): Crítica (9.8)

Descripción: El usuario mitch tiene permisos para ejecutar el binario /usr/bin/vim como root sin necesidad de contraseña. Esto permite invocar una shell del sistema dentro del editor, heredando los permisos de superusuario.

Proceso de explotación con comandos de herramientas utilizadas:

```bash 
sudo nmap -p- --min-rate 5000 -n -vvv [IP_OBJETIVO] -oN escaneo_rapido.txt
```
Impacto: Fuga de información que facilitó el movimiento lateral y el descubrimiento de usuarios válidos para ataques de fuerza bruta.

Remediación:

Deshabilitar el acceso anónimo en la configuración del servicio FTP (ej: vsftpd.conf -> anonymous_enable=NO).
