## 1. Introducción
El hardening consiste en aplicar una serie de medidas de seguridad destinadas a reducir la superficie de ataque de un sistema. El objetivo es minimizar los riesgos de acceso no autorizado, explotación de vulnerabilidades y movimientos laterales dentro de una infraestructura.
En este laboratorio se ha realizado el endurecimiento básico de un servidor Ubuntu mediante:

Configuración segura de SSH.

Implementación de firewall UFW.

Instalación y configuración de Fail2ban.

Creación de usuarios sin privilegios administrativos.


## 2. Configuración de usuarios
Creación de usuario sin privilegios
Se creó un usuario específico para la administración diaria del sistema:
sudo adduser user
Verificación:		id user

## 3. SSH seguro
SSH es uno de los servicios más atacados en servidores ya que permite acceso remoto. Por eso es recomendable hacer algunos cambios en el fichero de configuración de sshd
sudo nano /etc/ssh/sshd_config
Configuración aplicada:
Port 2260 
Cambia el puerto por defecto 22, ya que suele ser muy atacado, por otro menos común

PermitRootLogin no
Impide acceder con la cuenta del sistema Root, también muy utilizada en los ataques
PasswordAuthentication no
	Evita ataques de fuerza bruta sobre contraseñas.

MaxAuthTries 3
	Configurar a 3 los intentos máximos de contraseñas
Después de realizar los cambios, reinicio del servicio:	sudo systemctl restart ssh
Comprobación:	sudo systemctl status ssh

<img width="917" height="902" alt="Captura de pantalla 2026-06-03 172740" src="https://github.com/user-attachments/assets/6c50d909-33d9-48e3-b18a-097e6fd5f607" />


Por lo que ahora, para acceder por ssh, se debe añadir -p + nuevo puerto:
ssh alvaro@192.168.1.131 -p 2260

## 4. Instalación y configuración de Fail2ban
**Instalación**
sudo apt install fail2ban -y
Comprobar estado:
sudo systemctl status fail2ban
**Configuración**
Crear archivo:
sudo nano /etc/fail2ban/jail.local
Configuración básica:
[sshd]
port = 2260 (puerto configurado en sshd)
maxretry = 3
findtime = 10m
bantime = 10m

<img width="1021" height="197" alt="Captura de pantalla 2026-06-03 120556" src="https://github.com/user-attachments/assets/2e88ebcb-6fb9-40bd-9835-5d1a48f2b7c3" />
<img width="466" height="285" alt="Captura de pantalla 2026-06-03 173015" src="https://github.com/user-attachments/assets/cf3dbbe1-2568-4717-ba5e-31fe211f3136" />


Reiniciar servicio:	sudo systemctl restart fail2ban
Comprobación:	sudo fail2ban-client status sshd

**¿Por qué mejora la seguridad?**
Fail2ban monitoriza los registros del sistema.
Cuando detecta varios intentos fallidos consecutivos:
Identifica la IP atacante.
La bloquea automáticamente mediante reglas de firewall.
Reduce ataques de fuerza bruta.
En la configuración actual una IP que falle 3 veces en 10 minutos quedará bloqueada durante 10 minutos.


## 5. Configuración del Firewall UFW
El firewall controla qué conexiones pueden salir y entrar al servidor.
Las funciones del firewall ufw :
Bloquea accesos no autorizados.
Reduce servicios expuestos.
Limita posibles vectores de ataque.
**Instalación**
sudo apt update
sudo apt install ufw -y
Configuración inicial:
**sudo ufw default deny incoming**
	Cualquier conexión que intente entrar al servidor será bloqueada por defecto.
Solo se permitirá el tráfico para el que hayas creado una regla explícita.

**sudo ufw default allow outgoing**

El servidor puede iniciar conexiones hacia Internet o hacia otros equipos sin restricciones. 
Permitir SSH:
**sudo ufw allow 2260**
Activación:
**sudo ufw enable**
Verificación:
**sudo ufw status**

<img width="644" height="231" alt="Captura de pantalla 2026-06-03 184212" src="https://github.com/user-attachments/assets/b843bf35-3786-47b4-a29c-a411d27f8b0f" />


## 6. Verificación de servicios
Comprobación general:
sudo systemctl status ssh
sudo systemctl status fail2ban
sudo ufw status
