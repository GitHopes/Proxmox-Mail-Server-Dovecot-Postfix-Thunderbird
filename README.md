# Proxmox-Mail-Server-Dovecot-Postfix-Thunderbird
Creacion de un servidor de mail local con Dovecot, Postfix y Thunderbird dentro de una VM (Debian Trixie) de Proxmox.

## Primeros Pasos
### Uso de Repositorios
Descargamos los paquetes necesarios para la creacion y configuracion de nuestro servidor de correo dentro de Debian Trixie.
Esta es la lista de repositorios necesarios para instalar.
```bash
sudo apt install postfix dovecot-core dovecot-imapd mailutils thunderbird -y
```
Este comando sirve para instalar el servicio de dovecot, que es para el protocolo IMAP, el servicio postfix, que se encarga de enviar, recibir y enrutar correos electronicos, la herramienta de linea de comandos mailutils, que es para gestionar correos por la linea de comandos en Linux, y el cliente grafico thunderbird para gestionar los correos de manera grafica y sencilla.
### Configuraciones necesarias
Para empezar la configuracion de nuestro servidor cambiamos el nombre de dominio porque sera necesario para comunicarnos en la red y si quieres establecer el servidor de correo en una red LAN sera necesario tener un servidor de DNS propio que nos permita comunicarnos con el servidor.
Para cambiar el nombre de dominio del servidor:
```
sudo hostnamectl set-hostname mail.server
```
En nuestro caso nuestro servidor lo pusimos como "mosfraayo.local".
Editamos en el archivo /etc/hosts:
```127.0.0.1   mail.server```
Esto nos permite comunicarnos con nosotros mismos desde el servidor para el dominio mail.server, util para realizar pruebas de funcionamiento desde nuestro propio servidor.
Ahora ajustamos nuestro Postfix para que concuerde con dovecot y con nuestro nombre de servidor.
Este es un ejemplo de la configuracion de Postfix en /etc/postfix/main.cf:
```
myhostname = mail.server
mydomain = server
myorigin = /etc/mailname
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, localhost
home_mailbox = Maildir/
```
Ahora terminando con la configuracion de Postfix pasamos a la configuracion de Dovecot para que coincida con Postfix y que use Maildir como buzon para funcionar.
Para eso editamos el archivo /etc/dovecot/conf.d/10-mail.conf:
```
mail_location = maildir:~/Maildir
```
Si estas usando las versiones mas recientes de Dovecot es posible que esta configuracion ya no sea funcional porque se ha cambiado a una nueva manera de configuracion que deja al comando mail_location como un comando no encontrado en la configuracion. [Migrating 2.3 to 2.4 Dovecot Tips](https://doc.dovecot.org/main/installation/upgrade/2.3-to-2.4.html#upgrade-tips)
Para ese caso se tiene que utilizar la siguiente configuracion:
```
mail_path = ~/Maildir
mail_driver = maildir
```
Ademas si se tiene que desactivar el SSL es solo para pruebas pero para mantenerlo activo se tiene que crear unos certificados dentro de la carpet private de dovecot.
Para desactivar el ssl en el archivo /etc/dovecot/conf.d/10-ssl.conf, editamos:
```
ssl = no
```
Y para tener la autenticacion simple de dovecot en el archivo /etc/dovecot/conf.d/10-auth.conf, editamos:
```
disable_plaintext_auth = no
auth_mechanisms = plain login
```
### Creacion de usuarios
Para que el correo funcione se necesitan que existan usuarios que se autenticaran con su propia clave.
```
sudo adduser mossab
sudo adduser franco
sudo adduser ayoub
```
### Reinicio de servicios
Ahora reiniciamos los servicios para que la configuracion se aplique cuando se vuelvan a iniciar y despues probaremos que todo haya funcionado.
```systemctl restart postfix dovecot```
Y tambien para que se inicien cuando el sistema se vuelva a encender
```systemctl enable postfix dovecot```
### Pruebas finales
Ahora abrimas la interfaz grafica de thunderbird y realizamos las siguiente configuraciones para conectarnos a el servidor.
Usar siempre el nombre de dominio del servidor para cualquier configuracion y para el protocolo imap usar el puerto 143 sin aplicar seguridad y con autenticacion simple, y para SMTP usar el puerto 25 sin seguridad y sin autenticacion.
Te logueas usando uno de los usuarios del sistema de correo y envias un mensaje hacia ti mismo o hacia otro para ver que funciona.
### Comentarios Finales
Puede que haya errores cuando se configure dovecot para que se envie correos de un lado a otro pero el servidor se puede testear usando telnet para ver el que el protocolo de envio funcione correctamente pero si algo falla en dovecot puede que sea porque el sistema no sabe donde dejar los correos y los deja en un lugar por defecto.
