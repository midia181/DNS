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
Adicione a RPZ no arquivo de /etc/unbound/unbound.conf
```plaintext
nano /etc/unbound/unbound.conf
```
```plaintext
rpz:
    name: blockdomi.com.br
    zonefile: /etc/unbound/db.rpz.block.zone.hosts
    rpz-action-override: cname
    rpz-cname-override: "blockdomi.com.br."
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
