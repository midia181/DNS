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
apt install certbot python3-certbot-apache php sudo
```
Entre na pasta /var/www/ baixe a pagina web e descompacte
```plaintext
cd /var/www/
```
```plaintext
wget https://github.com/midia181/Client_blockdomi/raw/main/bloqueadonobrasil.tar.gz
```
```plaintext
tar -vxzf bloqueadonobrasil.tar.gz
```
```plaintext
rm bloqueadonobrasil.tar.gz
```
Crie o arquivo de configuração da pagina web
```plaintext
nano /etc/apache2/sites-available/bloqueadonobrasil.conf
```
Adicione ao arquivo de configuração:
```plaintext
<virtualhost *:80>
        Protocols h2 http/1.1
        ServerName bloqueados.blockdomi.com.br
        #ServerAlias x.xxx.xxx.x
        #ServerAlias [xxxx:xxxx:xxxx:xxxx::x]
 
        ServerAdmin seuemail@email.com
 
        ErrorDocument 404 /index.php
 
        DocumentRoot /var/www/bloqueadonobrasil
 
        <Directory /var/www/bloqueadonobrasil/>
            Options FollowSymLinks
            AllowOverride All
            # Restringe o acesso apenas para os IPs do seu ISP
            #Require ip 127.0.0.1 ::1 100.64.0.0/10 x.xx.xxx.x/22 xxxx:xxxx::/32
        </Directory>
 
        LogLevel warn
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</virtualhost>
```
Habilite a configuração e reinicie o Apache
```plaintext
a2ensite bloqueadonobrasil.conf
```
Instale o certificado SSL para o dominio
```plaintext
certbot
```
Coloque no contrab para rodar as 2 da manhã para renovar o certificado altomaticamente
```plaintext
echo "0 2 * * * certbot renew --quiet" | sudo tee -a /etc/crontab > /dev/null
```
Reinicie o cron
```plaintext
service cron restart
```
Agora reinicie o apache2 para aplicar as configurações
```plaintext
systemctl restart apache2
```
