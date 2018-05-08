#Cambiar el hostname
sudo nano /etc/hostname
mail

#Asignar IP estatica, y modificar el archivos hosts
sudo nano /etc/hosts
192.168.33.10 mail.ufg.dev

#Instalar Postfix y dependencias
sudo apt-get install postfix

#Confiurar Postfix
sudo dpkg-reconfigure postfix

#Configurar Postfix para SMTP-AUTH usando Dovecot SASL
sudo nano /etc/postfix/main.cf

#agregar la siguiente configuración
home_mailbox = Maildir/
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_local_domain =
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
smtp_tls_security_level = may
smtpd_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtpd_tls_loglevel = 1
smtpd_tls_received_header = yes

#Generar certificado digital
vagrant@mail:~$ openssl genrsa -des3 -out server.key 2048
vagrant@mail:~$ openssl rsa -in server.key -out server.key.insecure
vagrant@mail:~$ mv server.key server.key.secure
vagrant@mail:~$ mv server.key.insecure server.key
vagrant@mail:~$ openssl req -new -key server.key -out server.csr
vagrant@mail:~$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
vagrant@mail:~$ sudo cp server.crt /etc/ssl/certs
vagrant@mail:~$ sudo cp server.key /etc/ssl/private

#Configurar directorio del certificado
vagrant@mail:~$ sudo postconf -e 'smtpd_tls_key_file = /etc/ssl/private/server.key'
vagrant@mail:~$ sudo postconf -e 'smtpd_tls_cert_file = /etc/ssl/certs/server.crt'

#Hablilitar el envio SMTP, comentar las siguientes líneas del archivo /etc/postfix/master.cf
submission inet n       -       -       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING

#Instalar Dovecot SASL
vagrant@mail:~$ sudo apt-get install dovecot-common

#Abrir el archivo /etc/dovecot/conf.d/10-master.conf y buscar # Postfix smtp-auth, agregar las siguientes lineas
# Postfix smtp-auth
unix_listener /var/spool/postfix/private/auth {
mode = 0660
user = postfix
group = postfix
}

#Abrir /etc/dovecot/conf.d/10-auth.conf en la línea 100, reemplazar
auth_mechanisms = plain login

#Reiniciar servicios
vagrant@mail:~$ sudo service postfix restart
vagrant@mail:~$ sudo service dovecot restart

#Hacer pruebas
vagrant@mail:~$ telnet mail.ufg.dev smtp

#Proceder a instalar y configurar Dovecot
sudo apt-get install dovecot-imapd dovecot-pop3d

#Abrir /etc/dovecot/conf.d/10-mail.conf y remplazar
mail_location = maildir:~/Maildir

#Abrir /etc/dovecot/conf.d/20-pop3.conf y descomentar la linea
pop3_uidl_format = %08Xu%08Xv

#Abrir /etc/dovecot/conf.d/10-ssl.conf y descomentar la linea
ssl = yes

#Reiniciar Dovecot
vagrant@mail:~$ sudo service dovecot restart

#Hacer pruebas a los puertos, ej POP3 ó verificar los puertos de escucha con Netstat
vagranta@mail:~$ telnet mail.krizna.com 110

vagrant@mail:~$ netstat -nl4

#Crear usuarios y hacer pruebas
vagrant@mail:~$ sudo useradd -m jmiguel -s /sbin/nologin
vagrant@mail:~$ sudo passwd jmiguel

#WebAdmin
usuario mailadm admin
pass gavidia
