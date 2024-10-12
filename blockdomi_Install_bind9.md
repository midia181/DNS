# Bloqueios na Internet com BlockDomi

> **Confidencial**: Devido à natureza delicada das informações ou ao contexto de sua disseminação, os domínios e sites restritos são tratados como confidenciais. O acesso à API é exclusivo para empresas de telecomunicações devidamente registradas na Anatel.
>
> Confira seu CNPJ aqui: [ANATEL Outorga e Licenciamento](https://informacoes.anatel.gov.br/paineis/outorga-e-licenciamento)
>
> **Contatos para obter acesso à API**:
> - [WhatsApp](https://api.whatsapp.com/send/?phone=5584998667245&text=Como+obter+acesso+a+API%3F&type=phone_number&app_absent=0)
> - [Telegram](https://t.me/LucasMidia)

## Implementar Bloqueios de Domínios

A restrição de domínios no sistema de DNS deve ser configurada no servidor DNS recursivo utilizado pelos clientes do provedor de Internet.

### Bloqueio de Domínios no Bind9

#### 1. Adicionar Zona de Bloqueio
Adicione uma zona chamada `blockdomi.zone` em seu arquivo `/etc/bind/named.conf.default-zones`:

```sh
sudo sed -i '$a\zone "blockdomi.zone" {\n    type master;\n    file "/etc/bind/blockdomi/db.rpz.zone.hosts";\n};' /etc/bind/named.conf.default-zones
```

#### 2. Configuração da Zona de Bloqueio
Na pasta `/etc/bind/blockdomi/`, crie o arquivo `db.rpz.zone.hosts` e adicione os domínios a serem bloqueados:

```plaintext
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
```

A cada domínio bloqueado, adicione uma entrada como esta:

```plaintext
assistirseriesmp4.com IN CNAME localhost.
*.assistirseriesmp4.com IN CNAME localhost.
```

Assim, qualquer subdomínio (e.g. `*.dominio.com`) será sempre traduzido para o IP `localhost`.

#### 3. Configurar o Response Policy
Antes de criar o script, adicione o `response-policy` dentro do arquivo `/etc/bind/named.conf.options`:

```sh
sudo sed -i '/^};/i \    response-policy {\n        zone "blockdomi.zone";\n    };' /etc/bind/named.conf.options
```

## Criar o Script do BlockDomi

#### 1. Criar Diretório para o Script
Crie um diretório para armazenar o script do BlockDomi:

```sh
mkdir /etc/bind/scripts
cd /etc/bind/scripts
```

#### 2. Baixar o Script
Baixe o script do GitHub:

```sh
wget https://raw.githubusercontent.com/midia181/client_blockdomi/refs/heads/main/blockdomi-bind9.sh
```

#### 3. Permissão de Execução
Dê permissão de execução para o script bash:

```sh
chmod +x /etc/bind/scripts/blockdomi-bind9.sh
```

#### 4. Executar o Script
Execute o script para sincronizar com a API do BlockDomi (substitua `localhost` pelo seu domínio, se desejar):

```sh
/etc/bind/scripts/blockdomi-bind9.sh localhost
```

Ao rodar o script pela primeira vez, a seguinte mensagem deve aparecer:

```plaintext
Diretório /etc/bind/blockdomi criado com sucesso.
Versão local não encontrada, baixando a versão 2024101202.
Arquivo de zona RPZ atualizado.
Permissões do diretório alteradas com sucesso.
Serviço Bind9 recarregado com sucesso.
```

#### 5. Estrutura do Diretório
Verifique a estrutura do diretório criado:

```sh
tree -h /etc/bind/blockdomi/
```

Saída esperada:

```plaintext
/etc/bind/blockdomi/
├── [546K]  db.rpz.zone.hosts
├── [118K]  domain_all
└── [  10]  version

0 directories, 3 files
```

Caso não tenha o `tree` instalado:

```sh
sudo apt install tree
```

Se você executar o script novamente, a seguinte mensagem deve aparecer:

```plaintext
Diretório /etc/bind/blockdomi já existe.
Já está na versão mais atual: 2024101202.
```

## Agendar Execução Diária do Script
Para manter a lista sempre atualizada, agende a execução do script todos os dias à meia-noite:

```sh
echo '00 00   * * *   root    /etc/bind/scripts/blockdomi-bind9.sh localhost' >> /etc/crontab
```

Depois, reinicie o cron:

```sh
systemctl restart cron
```

## Verificar Domínios Bloqueados
Verifique os domínios bloqueados:

```sh
cat /etc/bind/blockdomi/domain_all
```

Apos rodar o script, você pode testar os domínios bloqueados. Substitua `dominiobloqueado.com` pelo domínio que deseja testar o bloqueio:

```sh
dig dominiobloqueado.com @localhost
```

Caso não tenha os pacotes `dnsutils` para testar com o `dig`:

```sh
apt install dnsutils
```

Exemplo de saída:

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
;dominiobloqueado.com.	IN	A

;; ANSWER SECTION:
dominiobloqueado.com.	5	IN	CNAME	localhost.
localhost. 10800	IN A	x.x.x.x

;; Query time: 479 msec
;; SERVER: ::1#53(::1)
;; WHEN: seg jan 22 11:35:55 -03 2024
;; MSG SIZE  rcvd: 137
```

