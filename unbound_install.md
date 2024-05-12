# INSTALANDO DNS RECURSIVO (UNBOUND) BASICO
MASTER (NS1)
```plaintext
apt install unbound
```
Limpar o arquivo de configuração do unbound
```plaintext
>/etc/unbound/unbound.conf
```
Acessar o diretório /etc/unbound
```plaintext
cd /etc/unbound/
```
Baixe o root.servers
```plaintext
wget https://www.internic.net/domain/named.root
```
Renomeie o nome do arquivo named.root
```plaintext
mv named.root root.hints
```
Gere a chave
```plaintext
unbound-anchor -a /etc/unbound/root.key -v
```
Desative o systemd-resolve
```plaintext
systemctl disable systemd-resolved.service
```
```plaintext
systemctl stop systemd-resolved
```
Copiar e colar aquivo de config
```plaintext
nano /etc/unbound/unbound.conf 
```
Arquivo de Config (Edite de acordo com suas configurações)
```plaintext
# UNBOUND Recursivo + Roots + DNSSEC

 
server:

    # Ative o módulo respip se usar a API da BLOCKDOMI para bloqueio de dominios
    module-config: "respip validator iterator"
 
#usuario do daemon e permissoes ( chown unbound.unbound /etc/unbound -R )
directory: "/etc/unbound"
username: unbound
 
# habilita estatisticas detalhadas
extended-statistics: yes
 
# log file
logfile: "/etc/unbound/unbound.log"
 
# log verbosity
verbosity: 1
 
# log queries - em tempo real, nao deixar habilitado em producao.
log-queries: no
 
 
# log verbosity
verbosity: 1
 
# Endereço IP que vai "ouvir", default 127.0.0.1 e ::1
interface: 0.0.0.0
# Porta
port: 53
 
# Habilitar em IPv4
do-ip4: yes
 
# Caso utilize o IPv6 Habilite (yes) caso nao desative (no)
do-ip6: no
 
# Habilitar UDP
do-udp: yes
 
# Habiitar TCP
do-tcp: yes
 
# Controle de Acesso ( Default Refused ) Coloque sua Faixa
access-control: 127.0.0.1 allow
access-control: 10.0.0.0/8 allow
access-control: 192.168.0.0/16 allow
access-control: 100.64.0.0/10 allow
access-control: 172.16.0.0/12 allow
access-control: ::1 allow
 
# Arquivo root.hints ( http://www.internic.net/domain/named.root )
root-hints: "/etc/unbound/root.hints"
 
# Desabilitar id.server e hostname.bind queries.
hide-identity: no
 
# Desabilitar version.server e version.bind queries.
hide-version: yes
 
# Segurança
harden-glue: yes
harden-dnssec-stripped: yes
 
# http://tools.ietf.org/html/draft-vixie-dnsext-dns0x20-00
use-caps-for-id: yes
 
# TTL em cache, default 0
cache-min-ttl: 3600
cache-max-ttl: 86400
 
# Verificação de tempo em cache
prefetch: yes
 
# Número de threads . 1 desabilita. Deixa sempre com o numero de core que tem o teu processador
num-threads: 4
 
# Otimizações colocar 2 na 2 ou 2 na 3. Exemplo 2x2x2=8
msg-cache-slabs: 32
rrset-cache-slabs: 32
infra-cache-slabs: 32
key-cache-slabs: 32
 
# Limites de Memória 
# Primeiro parametro 10% ou 20% da memória
# Segundo parametro até 50% da memória  

rrset-cache-size: 1024m
msg-cache-size: 4096m
 
 
# Log de inundação e ações de limpeza
unwanted-reply-threshold: 10000
 
# Limitação de requests de localhost
do-not-query-localhost: no
 
# Uso de dns root ( RFC5011 )
auto-trust-anchor-file: "/etc/unbound/root.key"
 
# Segurança extra dados cache
val-clean-additional: yes

# Caso utilize API da BLOCKDOMI para bloquear dominios descomente o RPZ.
#rpz:
#   name: teste.blockdomi.com.br
#   zonefile: /etc/unbound/db.rpz.block.zone.hosts
#   rpz-action-override: cname
#   rpz-cname-override: "teste.blockdomi.com.br."
```
