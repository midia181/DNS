# blockinstall
Cliente install para blockdomi 

# DNS PRIMARIO (MASTER)
nano /etc/bind/named.conf.local
# Caso não utilize DNS SLAVES (SECUNDARIO), retire a parte allow-transfer e also-notify, caso utilize substitua os ips 10.0.0.1 pelo ip do DNS SLAVES (SECUNDARIO)

```plaintext
zone "rpz.zone" {
type master;
file "/var/cache/bind/rpz/db.rpz.zone.hosts";
allow-query { none; };
allow-transfer { 10.0.0.1; };
also-notify { 10.0.0.1; };
}
```

# Crie o diretório “rpz” e os arquivos de hosts que serão bloqueados.
```plaintext
mkdir /var/cache/bind/rpz/
```
# Crie um atalho em /etc/bind para facilitar o acesso a pasta rpz
ln -s /var/cache/bind/rpz/ /etc/bind/rpz
# Na pasta /var/cache/bind/rpz/db.rpz.zone.hosts segue o exemplo de como irá ficar os dominios bloqueados
$TTL 1H
@       IN      SOA LOCALHOST. bloqueiosana.provedor.com. (
                2024012201      ; Serial  
                1h              ; Refresh
                15m             ; Retry
                30d             ; Expire 
                2h              ; Negative Cache TTL
        )
        NS  bloqueiosana.provedor.com.
;       ou
;       NS  localhost.
 
sitequeprecisabloquear.com     IN CNAME .
*.sitequeprecisabloquear.com   IN CNAME .
elesmandamnosfaz.bo.bo         IN CNAME .
*.elesmandamnosfaz.bo.bo       IN CNAME .

# A cada dominio bloqueado irá conter:
sitequeprecisabloquear.com        IN CNAME .
*.sitequeprecisabloquear.com      IN CNAME .
# Assim qualquer subdomínio (*).domino.com seja traduzido sempre irá ser apontado para seu IP ou localhost.

# Antes de criar o script, ajuste o response-policy dentro do seu /etc/bind/named.conf.options
options {
//...
    response-policy {
      zone "rpz.zone" policy CNAME bloqueiosana.provedor.com;
    };
//...
# se você irá apontar para localhost use:

options {
//...
response-policy {
zone "rpz.zone" policy CNAME localhost;
};
//...
1
2
3
4
5
6
options {
//...
    response-policy {
      zone "rpz.zone" policy CNAME localhost;
    };
//...
# Crie um diretório onde irá ficar o script do BLOCKDOMI:
mkdir /etc/bind/scripts
cd /etc/bind/scripts
wget https://raw.githubusercontent.com/midia181/blockinstall/main/blockdomi_bind9.py
# Como o script usa o python 3 precisaremos instalar os pacotes nescessários para executa-lo.
apt install python3 python3-requests tree
# Execulte o script para sicronizar com a API do BLOCKDOMI:
python3 /etc/bind/scripts/blockdomi_bind9.py bloqueiosana.provedor.com
# Caso venha utilizar localhost:
python3 /etc/bind/scripts/blockdomi_bind9.py localhost
# Ao rodar o script se tudo ocorrer bem a menssagem irá aparecer:
Arquivo de zona RPZ atualizado.
Permissões do diretório alteradas com sucesso.
Serviço Bind9 reiniciado com sucesso.
# Seu diretório terá os seguintes arquivos
tree -h /var/cache/bind/rpz/

[4.0K]  /var/cache/bind/rpz/
├── [299K]  db.rpz.zone.hosts
├── [ 86K]  domain_all
├── [  20]  rpz -> /var/cache/bind/rpz/
└── [  10]  version
# Se você executar o script novamente nada irá acontecer até que uma nova versão seja lancada.
# Para que tenhamos nossa lista sempre atualizada, colocamos o script para ser executado todos os dias a meia noite. Caso utilize localhost mudar o dominio para localhost
echo '00 00   * * *   root    python3 /etc/bind/scripts/blockdomi_bind9.py bloqueiosana.provedor.com'\ >> /etc/crontab
# ou
echo '00 00   * * *   root    python3 /etc/bind/scripts/blockdomi_bind9.py localhost'\ >> /etc/crontab
# Depois reinicie o cron
systemctl restart cron
# Apos rodar o script poderá testar os dominios bloqueados:
dig 2filmestorrent.top @localhost
