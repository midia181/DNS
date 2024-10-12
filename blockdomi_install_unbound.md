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

# Bloqueio de domínios no Unbound
Crie um diretório onde irá ficar o script do BLOCKDOMI:
```plaintext
mkdir /etc/unbound/scripts
```
```plaintext
cd /etc/unbound/scripts
```
```plaintext
wget https://raw.githubusercontent.com/midia181/client_blockdomi/refs/heads/main/blockdomi-unbound.sh
```
No debian 12 irá precisar instalar o pacote unbound-anchor (caso nao tenha instalado) o mesmo não vem instalado por padrao.
```plaintext
apt install unbound-anchor
```
Execulte o script para sicronizar com a API do BLOCKDOMI (Caso utilize dominio para pagina de bloqueio, substitua 127.0.0.1 por seu dominio):
```plaintext
. /etc/unbound/scripts/blockdomi-unbound.sh 127.0.0.1
```
Ao rodar o script pela primeira vez se tudo ocorrer bem a menssagem irá aparecer:
```plaintext
Diretório /etc/unbound/blockdomi criado com sucesso.
Versão local não encontrada, baixando a versão 2024101104.
Arquivo de configuração do Unbound atualizado para bloqueio.
Permissões do diretório alteradas com sucesso.
unbound-checkconf: no errors in /etc/unbound/unbound.conf
Serviço Unbound recarregado com sucesso.
```
No arquivo /etc/unbound/blockdomi/blockdomi.conf segue o exemplo de como irá ficar os dominios bloqueados
```plaintext
local-zone: "sitequeprecisabloquear.com" redirect
local-data: "sitequeprecisabloquear.com A 127.0.0.1"
```
A cada dominio bloqueado irá conter:
```plaintext
local-zone: "sitequeprecisabloquear.com" redirect
local-data: "sitequeprecisabloquear.com A 127.0.0.1"
```
Seu diretório terá os seguintes arquivos
```plaintext
tree -h /etc/unbound/blockdomi/
```
```plaintext
[4.0K]  /etc/unbound/blockdomi/
├── [597K]  blockdomi.conf
├── [118K]  domain_all
└── [  10]  version

1 directory, 3 files
```
Se você executar o script novamente irá aparecer a seguinte menssagem:
```plaintext
Diretório /etc/unbound/blockdomi já existe.
Já está na versão mais atual: 2024101104.
```
Para que tenhamos nossa lista sempre atualizada, colocamos o script para ser executado todos os dias a meia noite.
(Caso utilize dominio para pagina de bloqueio, substitua 127.0.0.1 por seu dominio):
```plaintext
echo '00 00   * * *   root    . /etc/unbound/scripts/blockdomi-unbound.sh 127.0.0.1' >> /etc/crontab
```
Depois reinicie o cron
```plaintext
systemctl restart cron
```
Adicione o arquivo de configuração do blockdomi no parametro server: do arquivo /etc/unbound/unbound.conf
```plaintext
nano /etc/unbound/unbound.conf
```
```plaintext
server:
	include: /etc/unbound/blockdomi/blockdomi.conf
```
Feito isso verifique se o unbound não contem erros de configurações
```plaintext
unbound-checkconf
```
Se não houver erros, o comando retornará:
```plaintext
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```
Feito isso reinicie o unbound para aplicar as configurações
```plaintext
systemctl restart unbound
```
Verifique os dominios bloqueados:
```plaintext
cat /etc/unbound/blockdomi/domain_all
```
Apos rodar o script poderá testar os dominios bloqueados, substitua o dominiobloqueado.com pelo dominio que deseja testar o bloqueio:
```plaintext
dig dominiobloqueado.com @localhost
```
<pre>
🚨 ;; communications error to ::1#53: connection refused
🚨 ;; communications error to ::1#53: connection refused
🚨 ;; communications error to ::1#53: connection refused

📘 ; <<>> DiG 9.18.24-1-Debian <<>> dominiobloqueado.com @localhost
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8506
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

📘 ;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
📘 ;; QUESTION SECTION:
;dominiobloqueado.com.           IN      A

📘 ;; ANSWER SECTION:
dominiobloqueado.com.    3600    IN      A       🟢 127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(localhost) (UDP)
;; WHEN: Sat Oct 12 10:55:25 -03 2024
;; MSG SIZE  rcvd: 64
</pre>

