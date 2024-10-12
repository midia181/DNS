## Bloqueios na Internet

> **Confidencial**: Devido à natureza delicada das informações e ao contexto de sua disseminação, os domínios e sites restritos são tratados como confidenciais. O acesso à API é exclusivo para empresas de telecomunicações devidamente registradas na Anatel.
>
> **Verifique seu CNPJ aqui**: [ANATEL Outorga e Licenciamento](https://informacoes.anatel.gov.br/paineis/outorga-e-licenciamento)
>
> **Contatos para obter acesso à API**:
> - [WhatsApp](https://api.whatsapp.com/send/?phone=5584998667245&text=Como+obter+acesso+a+API%3F&type=phone_number&app_absent=0)
> - [Telegram](https://t.me/LucasMidia)

---

# Implementação de Bloqueios de Domínios

A restrição de domínios no sistema DNS deve ser configurada no servidor DNS recursivo utilizado pelos clientes do provedor de Internet.

### Bloqueio de Domínios no Bind9

1. **Adicionar Zona ao Bind9**

   Adicione uma zona chamada `blockdomi.zone` no arquivo `/etc/bind/named.conf.default-zones`:

   </pre>bash
   sudo sed -i '$a\zone "blockdomi.zone" {\n    type master;\n    file "/etc/bind/blockdomi/db.rpz.zone.hosts";\n};' /etc/bind/named.conf.default-zones
   </pre>

2. **Configurar os Domínios Bloqueados**

   Crie o arquivo `/etc/bind/blockdomi/db.rpz.zone.hosts` para definir os domínios bloqueados:

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

   Para cada domínio bloqueado, inclua estas linhas:

   <pre>
   assistirseriesmp4.com IN CNAME localhost.
   *.assistirseriesmp4.com IN CNAME localhost.
   </pre>

3. **Adicionar Response Policy**

   Adicione o `response-policy` no arquivo `/etc/bind/named.conf.options`:

   </pre>bash
   sudo sed -i '/^};/i \    response-policy {\n        zone "blockdomi.zone";\n    };' /etc/bind/named.conf.options
   </pre>

4. **Criar Diretório e Baixar Script**

   Crie um diretório onde o script do BLOCKDOMI será armazenado e baixe o script:

   </pre>bash
   mkdir /etc/bind/scripts
   cd /etc/bind/scripts
   wget https://raw.githubusercontent.com/midia181/client_blockdomi/refs/heads/main/blockdomi-bind9.sh
   </pre>

5. **Permissões de Execução para o Script**

   Dê permissão de execução ao script:

   </pre>bash
   chmod +x /etc/bind/scripts/blockdomi-bind9.sh
   </pre>

6. **Executar o Script**

   Execute o script para sincronizar com a API do BLOCKDOMI:

   </pre>bash
   /etc/bind/scripts/blockdomi-bind9.sh localhost
   </pre>

   > Caso utilize um domínio para a página de bloqueio, substitua `localhost` pelo seu domínio.

7. **Verificar Mensagens do Script**

   Ao rodar o script pela primeira vez, se tudo ocorrer bem, a mensagem exibida será:

   <pre>
   Diretório /etc/bind/blockdomi criado com sucesso.
   Versão local não encontrada, baixando a versão 2024101202.
   Arquivo de zona RPZ atualizado.
   Permissões do diretório alteradas com sucesso.
   Serviço Bind9 recarregado com sucesso.
   </pre>

   O diretório `/etc/bind/blockdomi/` terá os seguintes arquivos:

   </pre>bash
   tree -h /etc/bind/blockdomi/
   </pre>

   <pre>
   /etc/bind/blockdomi/
   ├── [546K]  db.rpz.zone.hosts
   ├── [118K]  domain_all
   └── [  10]  version

   0 directories, 3 files
   </pre>

   Caso não tenha o `tree` instalado:

   </pre>bash
   sudo apt install tree
   </pre>

8. **Executar o Script Novamente**

   Se o script for executado novamente, a mensagem exibida será:

   <pre>
   Diretório /etc/bind/blockdomi já existe.
   Já está na versão mais atual: 2024101202.
   </pre>

9. **Automatizar a Execução do Script**

   Para manter a lista sempre atualizada, configure o script para ser executado diariamente à meia-noite:

   </pre>bash
   echo '00 00   * * *   root    /etc/bind/scripts/blockdomi-bind9.sh localhost' >> /etc/crontab
   </pre>

   Reinicie o cron:

   </pre>bash
   systemctl restart cron
   </pre>

10. **Verificar Domínios Bloqueados**

    Para verificar os domínios bloqueados:

    </pre>bash
    cat /etc/bind/blockdomi/domain_all
    </pre>

11. **Testar Domínios Bloqueados**

    Após rodar o script, você poderá testar os domínios bloqueados substituindo `dominiobloqueado.com` pelo domínio que deseja testar:

    </pre>bash
    dig dominiobloqueado.com @localhost
    </pre>

    Caso não tenha os pacotes `dnsutils` instalados para testar com o `dig`:

    </pre>bash
    apt install dnsutils
    </pre>

    Exemplo de saída do comando `dig`:

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
    ;dominiobloqueado.com.	IN	A

    ;; ANSWER SECTION:
    dominiobloqueado.com.    5   IN  CNAME   localhost.
    localhost.               10800   IN  A   x.x.x.x

    ;; Query time: 479 msec
    ;; SERVER: ::1#53(::1)
    ;; WHEN: seg jan 22 11:35:55 -03 2024
    ;; MSG SIZE  rcvd: 137
    </pre>

