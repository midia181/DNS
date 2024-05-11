# Bloquear sites pelo DNS (UNBOUND) Response Policy Zones
# API BlockDomi Client UNBOUND
Obtenha acesso a API:
https://wa.me/5584998667245?text=Como+obter+acesso+a+API%3F
# DNS PRIMARIO (MASTER)
Crie as pastas /var/cache/unbound e /var/cache/unbound/rpz.
```plaintext
mkdir /var/cache/unbound
```
```plaintext
mkdir /var/cache/unbound/rpz
```
Crie um arquivo de zona vazio.
```plaintext
touch /var/cache/unbound/rpz/db.rpz.zone.hosts
```
Crie um atalho do arquivo db.rpz.block.zone.hosts em /etc/unbound .
```plaintext
ln -s /var/cache/unbound/rpz/db.rpz.block.zone.hosts /etc/unbound/db.rpz.block.zone.hosts
```
Adicione a RPZ no final do arquivo /etc/unbound/unbound.conf
```plaintext
nano /etc/unbound/unbound.conf
```
```plaintext
rpz:
    name: bloqueados.blockdomi.com.br
    zonefile: /etc/unbound/db.rpz.block.zone.hosts
    rpz-action-override: cname
    rpz-cname-override: "bloqueados.blockdomi.com.br."
```
Crie um diretório onde irá ficar o script do BLOCKDOMI:
```plaintext
mkdir /etc/unbound/scripts
```
```plaintext
cd /etc/unbound/scripts
```
```plaintext
wget https://raw.githubusercontent.com/midia181/DNS/main/blockdomi_unbound.py
```
Como o script usa o python 3 precisaremos instalar os pacotes nescessários para executa-lo.
```plaintext
apt install python3 python3-requests tree
```
Execulte o script para sicronizar com a API do BLOCKDOMI:
```plaintext
python3 /etc/bind/scripts/blockdomi_bind9.py bloqueados.blockdomi.com.br
```
Ao rodar o script se tudo ocorrer bem a menssagem irá aparecer:
```plaintext
Arquivo de zona RPZ atualizado.
Permissões do diretório alteradas com sucesso.
Serviço Bind9 reiniciado com sucesso.
```
Seu diretório terá os seguintes arquivos
```plaintext
tree -h /var/cache/unbound/rpz/
```
```plaintext
/var/cache/unbound/rpz/
|-- [299K]  db.rpz.block.zone.hosts
|-- [   0]  db.rpz.zone.hosts
|-- [ 86K]  domain_all
`-- [  10]  version

0 directories, 4 files
```
Se você executar o script novamente nada irá acontecer até que uma nova versão seja lancada. Para que tenhamos nossa lista sempre atualizada, colocamos o script para ser executado todos os dias a meia noite.
```plaintext
echo '00 00   * * *   root    python3 /etc/unbound/scripts/blockdomi_unbound.py bloqueados.blockdomi.com.br'\ >> /etc/crontab
```
Depois reinicie o cron
```plaintext
systemctl restart cron
```
Verifique os dominios bloqueados:
```plaintext
cat /etc/unbound/rpz/domain_all
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
