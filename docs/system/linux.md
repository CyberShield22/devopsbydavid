# 1. Configuración inicial
## ¿Por qué hacer esto?
Los servidores recién creados vienen con paquetes obsoletos. Las actualizaciones de seguridad corrigen fallas que los atacantes activamente. Hacer esto primero garantiza una base limpia.

Además, trabajar siempre como `root` arriesgado - cualquier error o comando malicioso tiene poder total sobre el sistema. Crear un usuario común con sudo es una práctica de seguridad básica.
## 1.1 Actualizar el sistema
```
apt update && apt upgrade -y
```
- `apt update` -> recupera la lista de paquetes disponibles
- `apt upgrade -y` -> instala actualizaciones sin solicitar confirmación
## 1.2 Crear usuario no root
```
adduser usuario
```
El terminal le pedirá una contraseña y algunos datos opcionales (puede presionar Enter para omitir)
Añadir al grupo sudo:
```
usermod -aG sudo usuario
```
## 1.3 Comprueba si sudo funciona
```
su - usuario
sudo whoami
```
Esto confirma que el usuario puede ejecutar comandos administrativos cuando sea necesario.
# 2. Hardening SSH
## ¿Por qué hacer esto?
SSH es la puerta de enlace a su servidor. De forma predeterminada, el usuario root puede iniciar sesión a través de SSH y los bots en Internet lo intentan las 24 horas del día, los 7 días de la semana. Bloquear la raíz elimina el objetivo más obvio.
## 2.1 Editar el archivo de configuración SSH
```
nano /etc/ssh/sshd_config
```
Busque la línea a continuación y cámbiela a no:
```
PermitRootLogin no
```
Guardar: `Ctrl+O` → `Enter` → `Ctrl+X`
## 2.2 Verificar anulaciones
Ubuntu 24.04 tiene una carpeta que puede sobrescribir sshd_config:
```
cat /etc/ssh/sshd_config.d/*.conf
```
Si algún archivo tiene autenticación de contraseña sí, cámbielo a no ahora (lo necesitará en el paso 05).
## 2.3 Reiniciar SSH
```
systemctl restart ssh
```

> [!WARNING] Atención
> Antes de cerrar la sesión actual, abra una nueva terminal y pruebe iniciar sesión con su usuario: 
> ```
> ssh seunome@IP_DA_VPS
> ```
> Si funciona puede cerrar la sesión raíz de forma segura. Si no realiza pruebas de antemano y algo sale mal, puede perder el acceso al servidor.

## 2.4 Resultado esperado
Intento de inicio de sesión como root bloqueado:
```
Permission denied, please try again.
```
# 3. Firewall com UFW
## ¿Por qué hacer esto?
Un servidor sin firewall tiene todos los puertos expuestos a Internet. El principio correcto es el contrario: bloquea todo por defecto y libera sólo lo que necesitas.

UFW (Uncomplicated Firewall) es la forma más sencilla de hacer esto en Ubuntu.

## 3.1 Instalar UFW
```
apt install -y ufw
```
## 3.2
**IMPORTANTE:** Libere el puerto SSH antes de activar el firewall. Si lo activas sin liberarlo, pierdes el acceso al servidor:
```
ufw allow ssh
```
## 3.3 Activar firewall
```
ufw enable
```
Va preguntar si quieres continuar, en este caso escribimos: `y`.
## 3.4 Verificar estado
```
ufw status verbose
```
Salida esperada:
```
Status: active
Default: deny (incoming), allow (outgoing)

To                  Action      From
22/tcp              ALLOW IN    Anywhere
```
## 3.5 Cómo abrir puertas en el futuro
Al instalar aplicaciones, libere solo los puertos necesarios:
```
# Servidor web
ufw allow 80/tcp   # HTTP
ufw allow 443/tcp  # HTTPS

# Porta específica
ufw allow 3000/tcp

# Bloquear uma porta
ufw deny 3306/tcp
```

# 4. Fail2ban
## ¿Por qué hacer esto?
Incluso con root bloqueado, los bots continúan intentando hackear su servidor con otros usuarios y contraseñas. Fail2ban monitorea los registros y prohíbe automáticamente las IP que pierden la contraseña muchas veces seguidas.

En menos de 3 minutos después de la configuración, ya puedes ver cómo se prohíben las IP.
## 4.1 Instalación
```
apt install -y fail2ban
```
## 4.2 Crear archivo de configuración local
Nunca edite jail.conf directamente — puede sobrescribirse en las actualizaciones. Utilice .local:
```
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
nano /etc/fail2ban/jail.local
```
## 4.3 Configurar la sección [sshd]
Busque la sección [sshd] en el archivo. Tenga en cuenta que puede aparecer dos veces — una versión incompleta en la parte superior y una versión completa al final. Deje solo la versión completa con la siguiente configuración:
```
[sshd]
enabled = true
port    = ssh
maxretry = 3
findtime = 10m
bantime = 1h
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```
Qué hace cada opción:

| Opción   | Valor | Significado                   |
| -------- | ----- | ----------------------------- |
| maxretry | 3     | Máximo de intentos fallidos   |
| findtime | 10m   | Ventana de tiempo monitoreada |
| bantime  | 1h    | Hora en que se prohíbe la IP  |
## 4.4 Activar e iniciar
```
systemctl enable fail2ban
systemctl restart fail2ban
```
## 4.5 Verificar funcionamiento
```
fail2ban-client status sshd
```
Resultado esperado después de unos minutos:
```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 2
|  |- Total failed:     24
`- Actions
   |- Currently banned: 4
   `- Banned IP list: 45.x.x.x 172.x.x.x ...
```

## 4.6 Problema común
**Fail2ban no aparece con el error "la sección 'sshd' ya existe"**

Causa: La sección `[sshd]` está duplicada en `jail.local`. Elimine la versión incompleta en la parte superior del archivo y conserve solo la versión completa.
