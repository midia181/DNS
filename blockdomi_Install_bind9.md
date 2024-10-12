## Bloqueios na Internet

> **Confidencial**: Devido à natureza delicada das informações ou ao contexto de sua disseminação, os domínios e sites restritos são tratados como confidenciais. O acesso à API é exclusivo para empresas de telecomunicações devidamente registradas na Anatel.
> 
> Confira seu CNPJ aqui: <a href="https://informacoes.anatel.gov.br/paineis/outorga-e-licenciamento" target="_blank">ANATEL Outorga e Licenciamento</a>
> 
> Contatos para obter acesso à API:
> - <a href="https://api.whatsapp.com/send/?phone=5584998667245&text=Como+obter+acesso+a+API%3F&type=phone_number&app_absent=0" target="_blank">WhatsApp</a>
> - <a href="https://t.me/LucasMidia" target="_blank">Telegram</a>


# Implementar bloqueios de domínios
A restrição de domínios no sistema de DNS deve ser configurada no servidor DNS recursivo utilizado pelos clientes do provedor de Internet.

# Bloqueio de domínios no Bind9
Adicione uma zona chamada blockdomi.zone em seu /etc/bind/named.conf.default-zones.
```plaintext
sudo sed -i '$a\zone "blockdomi.zone" {\n    type master;\n    file "/etc/bind/rpz/db.rpz.zone.hosts";\n};' /etc/bind/named.conf.default-zones
```
Na pasta /etc/bind/rpz/db.rpz.zone.hosts segue o exemplo de como irá ficar os dominios bloqueados
<then>
$TTL 1H
@       IN      SOA LOCALHOST. localhost. (
                2024101201      ; Serial
                1h              ; Refresh
                15m             ; Retry
                30d             ; Expire
                2h              ; Negative Cache TTL
        )
        NS  localhost.

assistirseriesmp4.com IN CNAME bloqueadosnobrasil.blockdomi.com.
*.assistirseriesmp4.com IN CNAME bloqueadosnobrasil.blockdomi.com.
</then>
A cada dominio bloqueado irá conter:
```plaintext
sitequeprecisabloquear.com        IN CNAME .
*.sitequeprecisabloquear.com      IN CNAME .
```
Assim qualquer subdomínio (*).domino.com seja traduzido sempre irá ser apontado para seu IP ou localhost.
Antes de criar o script, iremos adicionar o response-policy dentro do seu /etc/bind/named.conf.options
```plaintext
sudo sed -i '/^};/i \    response-policy {\n        zone "blockdomi.zone";\n    };' /etc/bind/named.conf.options
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
tree -h /etc/bind/rpz/
```
```plaintext
/etc/bind/rpz/
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
