# INSTALANDO PAGINA WEB (APACHE2) DE BLOQUEIO DE DOMINIOS (DEBIAN)
Atualize o debian
```plaintext
apt update
```
```plaintext
apt upgrade
```
Instale o certbot para gerar o certificado SSL para o dominio
```plaintext
apt install certbot python3-certbot-apache
```
Crie a pasta bloqueadosnobrasil em /var/www
```plaintext
mkdir /var/www/bloqueadonobrasil
```
