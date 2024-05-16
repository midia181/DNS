# Bloquear sites pelo DNS (BIND9) Response Policy Zones
# API BlockDomi Client BIND9
Obtenha acesso a API:
https://wa.me/5584998667245?text=Como+obter+acesso+a+API%3F
# DNS PRIMARIO (MASTER)
Crie uma zona chamada rpz.zone em seu /etc/bind/named.conf.local.
```plaintext
nano /etc/bind/named.conf.local
```
```plaintext
zone "rpz.zone" {
    type master;
    file "/var/cache/bind/rpz/db.rpz.zone.hosts";
    allow-query { none; };
};
```
Crie o diretório “rpz” e os arquivos de hosts que serão bloqueados.
```plaintext
mkdir /var/cache/bind/rpz/
```
Crie um atalho em /etc/bind para facilitar o acesso a pasta rpz
```plaintext
ln -s /var/cache/bind/rpz/ /etc/bind/rpz
```
Na pasta /var/cache/bind/rpz/db.rpz.zone.hosts segue o exemplo de como irá ficar os dominios bloqueados
```plaintext
$TTL 1H
@       IN      SOA LOCALHOST. localhost. (
                2024012201      ; Serial  
                1h              ; Refresh
                15m             ; Retry
                30d             ; Expire 
                2h              ; Negative Cache TTL
        )
        NS  localhost.
;       ou
;       NS  localhost.
sitequeprecisabloquear.com     IN CNAME .
*.sitequeprecisabloquear.com   IN CNAME .
elesmandamnosfaz.bo.bo         IN CNAME .
*.elesmandamnosfaz.bo.bo       IN CNAME .
```
A cada dominio bloqueado irá conter:
```plaintext
sitequeprecisabloquear.com        IN CNAME .
*.sitequeprecisabloquear.com      IN CNAME .
```
Assim qualquer subdomínio (*).domino.com seja traduzido sempre irá ser apontado para seu IP ou localhost.
Antes de criar o script, ajuste o response-policy dentro do seu /etc/bind/named.conf.options
```plaintext
nano /etc/bind/named.conf.options
```
```plaintext
    response-policy {
      zone "rpz.zone" policy CNAME localhost;
    };
```
Crie um diretório onde irá ficar o script do BLOCKDOMI:
```plaintext
mkdir /etc/bind/scripts
```
```plaintext
cd /etc/bind/scripts
```
```plaintext
wget https://raw.githubusercontent.com/midia181/client_blockdomi/main/blockdomi_bind9.py
```
Como o script usa o python 3 precisaremos instalar os pacotes nescessários para executa-lo.
```plaintext
apt install python3 python3-requests tree python3-termcolor
```
Execulte o script para sicronizar com a API do BLOCKDOMI:
```plaintext
python3 /etc/bind/scripts/blockdomi_bind9.py localhost
```
Ao rodar o script se tudo ocorrer bem a menssagem irá aparecer:
```plaintext
Arquivo de zona RPZ atualizado.
Permissões do diretório alteradas com sucesso.
Serviço Bind9 reiniciado com sucesso.
```
Seu diretório terá os seguintes arquivos
```plaintext
tree -h /var/cache/bind/rpz/
```
```plaintext
/var/cache/bind/rpz/
|-- [299K]  db.rpz.zone.hosts
|-- [ 86K]  domain_all
`-- [  10]  version

0 directories, 3 files
```
Se você executar o script novamente nada irá acontecer até que uma nova versão seja lancada.
Para que tenhamos nossa lista sempre atualizada, colocamos o script para ser executado todos os dias a meia noite.

```plaintext
echo '00 00   * * *   root    python3 /etc/bind/scripts/blockdomi_bind9.py localhost'\ >> /etc/crontab
```
Depois reinicie o cron
```plaintext
systemctl restart cron
```
Verifique os dominios bloqueados:
```plaintext
cat /etc/bind/rpz/domain_all
```
Apos rodar o script poderá testar os dominios bloqueados, substitua o dominiobloqueado.com pelo dominio que deseja testar o bloqueio:
```plaintext
dig dominiobloqueado.com @localhost
```
```plaintext
; <<>> DiG 9.11.5-P4-5.1+deb10u9-Debian <<>> dominiobloqueado.com @localhost
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42124
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
 
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: be484fb71dfe219dd9e6544465ae7d4b7e142ef750ed0d80 (good)
;; QUESTION SECTION:
;dominiobloqueado.com.		IN	A
 
;; ANSWER SECTION:
dominiobloqueado.com.	5	IN	CNAME	localhost.
localhost. 10800	IN A	x.x.x.x
 
;; Query time: 479 msec
;; SERVER: ::1#53(::1)
;; WHEN: seg jan 22 11:35:55 -03 2024
;; MSG SIZE  rcvd: 137
```
