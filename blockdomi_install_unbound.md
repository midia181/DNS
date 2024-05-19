# Implementar bloqueios de domínios
A restrição de domínios no sistema de DNS deve ser configurada no servidor DNS recursivo utilizado pelos clientes do provedor de Internet.
Bloqueio de domínios no Unbound
Crie um diretório onde irá ficar o script do BLOCKDOMI:
```plaintext
mkdir /etc/unbound/scripts
```
```plaintext
cd /etc/unbound/scripts
```
```plaintext
wget https://raw.githubusercontent.com/midia181/client_blockdomi/main/blockdomi_unbound.py
```
Como o script usa o python 3 precisaremos instalar os pacotes nescessários para executa-lo.
```plaintext
apt install python3 python3-requests tree python3-termcolor
```
No debian 12 irá precisar instalar o pacote unbound-anchor (caso nao tenha instalado) o mesmo não vem instalado por padrao.
```plaintext
apt install unbound-anchor
```
Execulte o script para sicronizar com a API do BLOCKDOMI (Caso utilize dominio para pagina de bloqueio, substitua localhost por seu dominio):
```plaintext
python3 /etc/unbound/scripts/blockdomi_unbound.py localhost
```
Ao rodar o script se tudo ocorrer bem a menssagem irá aparecer:
```plaintext
Diretório /etc/unbound/rpz criado com sucesso.
Arquivo de zona RPZ atualizado.
Permissões do diretório alteradas com sucesso.
Serviço Unbound reiniciado com sucesso.
```
No arquivo /etc/unbound/rpz/db.rpz.zone.hosts segue o exemplo de como irá ficar os dominios bloqueados
```plaintext
$TTL 1H
@       IN      SOA LOCALHOST. localhost. (
                2024051101      ; Serial
                1h              ; Refresh
                15m             ; Retry
                30d             ; Expire
                2h              ; Negative Cache TTL
        )
        NS  localhost.

sitequeprecisabloquear.com IN CNAME .
*.sitequeprecisabloquear.com IN CNAME .
```
A cada dominio bloqueado irá conter:
```plaintext
sitequeprecisabloquear.com IN CNAME .
*.sitequeprecisabloquear.com IN CNAME .
```
Seu diretório terá os seguintes arquivos
```plaintext
tree -h /etc/unbound/rpz/
```
```plaintext
[4.0K]  /etc/unbound/rpz/
├── [301K]  db.rpz.block.zone.hosts
├── [ 86K]  domain_all
└── [  10]  version

1 directory, 3 files
```
Se você executar o script novamente irá aparecer a seguinte menssagem:
```plaintext
Diretório /etc/unbound/rpz já existe.
Já está na versão mais atual.
```
Para que tenhamos nossa lista sempre atualizada, colocamos o script para ser executado todos os dias a meia noite.
(Caso utilize dominio para pagina de bloqueio, substitua localhost por seu dominio):
```plaintext
echo '00 00   * * *   root    python3 /etc/unbound/scripts/blockdomi_unbound.py localhost'\ >> /etc/crontab
```
Depois reinicie o cron
```plaintext
systemctl restart cron
```
Adicione a RPZ no final do arquivo /etc/unbound/unbound.conf
```plaintext
nano /etc/unbound/unbound.conf
```
```plaintext
module-config: "respip validator iterator"
rpz:
    name: localhost
    zonefile: /etc/unbound/rpz/db.rpz.block.zone.hosts
    rpz-action-override: cname
    rpz-cname-override: "localhost."
```
Feito isso reinicie o unbound para aplicar as configurações
```plaintext
systemctl restart unbound
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
