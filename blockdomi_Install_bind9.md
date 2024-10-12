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
sudo sed -i '$a\zone "blockdomi.zone" {\n    type master;\n    file "/etc/bind/blockdomi/db.rpz.zone.hosts";\n};' /etc/bind/named.conf.default-zones
```
Na pasta /etc/bind/blockdomi/db.rpz.zone.hosts segue o exemplo de como irá ficar os dominios bloqueados
<pre>
$TTL 1H
@       IN      SOA LOCALHOST. localhost. (
                2024101201      ; Serial
                1h              ; Refresh
                15m             ; Retry
                30d             ; Expire
                2h              ; Negative Cache TTL
        )
        NS  localhost.

assistirseriesmp4.com IN CNAME localhost.
*.assistirseriesmp4.com IN CNAME localhost.
</pre>
A cada dominio bloqueado irá conter:
<pre>
assistirseriesmp4.com IN CNAME localhost.
*.assistirseriesmp4.com IN CNAME localhost.
</pre>
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
wget https://raw.githubusercontent.com/midia181/client_blockdomi/refs/heads/main/blockdomi-bind9.sh
```
Dê permissão de execução para o script bash.
```plaintext
chmod +x /etc/bind/scripts/blockdomi-bind9.sh
```
Execulte o script para sicronizar com a API do BLOCKDOMI (Caso utilize dominio para pagina de bloqueio, substitua localhost por seu dominio):
```plaintext
/etc/bind/scripts/blockdomi-bind9.sh localhost
```
Ao rodar o script a primeira vez se tudo ocorrer bem a menssagem irá aparecer:
<pre>
Diretório /etc/bind/blockdomi criado com sucesso.
Versão local não encontrada, baixando a versão 2024101202.
Arquivo de zona RPZ atualizado.
Permissões do diretório alteradas com sucesso.
Serviço Bind9 recarregado com sucesso.
</pre>
Seu diretório terá os seguintes arquivos
```plaintext
tree -h /etc/bind/blockdomi/
```
<pre>
/etc/bind/blockdomi/
├── [546K]  db.rpz.zone.hosts
├── [118K]  domain_all
└── [  10]  version

0 directories, 3 files
</pre>
Caso não tenha o tree instalado
```plaintext
sudo apt install tree
```
Se você executar o script novamente:
<pre>
Diretório /etc/bind/blockdomi já existe.
Já está na versão mais atual: 2024101202.
</pre>
Para que tenhamos nossa lista sempre atualizada, colocamos o script para ser executado todos os dias a meia noite.

```plaintext
echo '00 00   * * *   root    /etc/bind/scripts/blockdomi-bind9.sh localhost' >> /etc/crontab
```
Depois reinicie o cron
```plaintext
systemctl restart cron
```
Verifique os dominios bloqueados:
```plaintext
cat /etc/bind/blockdomi/domain_all
```
Apos rodar o script poderá testar os dominios bloqueados, substitua o dominiobloqueado.com pelo dominio que deseja testar o bloqueio:
```plaintext
dig dominiobloqueado.com @localhost
```
Caso não tenha os pacotes dnsutils para testar com o dig:
```plaintext
apt install dnsutils
```
<pre>
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
</pre>
