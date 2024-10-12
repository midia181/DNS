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

1. **Criar Diretório e Baixar Script**

   Crie um diretório onde o script do BLOCKDOMI será armazenado e baixe o script:

   ```plaintext
   mkdir /etc/bind/scripts
   cd /etc/bind/scripts
   wget https://raw.githubusercontent.com/midia181/client_blockdomi/refs/heads/main/blockdomi-bind9.sh
   ```


2. **Permissões de Execução para o Script**

   Dê permissão de execução ao script:

   ```plaintext
   sudo chmod +x /etc/bind/scripts/blockdomi-bind9.sh
   ```
   
3. **Adicionar Zona ao Bind9**

   Adicione uma zona chamada `blockdomi.zone` no arquivo `/etc/bind/named.conf.default-zones`:

   ```plaintext
   sudo sed -i '$a\zone "blockdomi.zone" {\n    type master;\n    file "/etc/bind/blockdomi/db.rpz.zone.hosts";\n};' /etc/bind/named.conf.default-zones
   ```

4. **Adicionar Response Policy**

   Adicione o `response-policy` no arquivo `/etc/bind/named.conf.options`:

   ```plaintext
   sudo sed -i '/^};/i \    response-policy {\n        zone "blockdomi.zone";\n    };' /etc/bind/named.conf.options
   ```

5. **Executar o Script**

   Execute o script para sincronizar com a API do BLOCKDOMI:

   ```plaintext
   sudo /etc/bind/scripts/blockdomi-bind9.sh localhost
   ```

   > Caso utilize um domínio para a página de bloqueio, substitua `localhost` pelo seu domínio.

6. **Verificar Mensagens do Script**

   Ao rodar o script pela primeira vez, se tudo ocorrer bem, a mensagem exibida será:

   <pre>
   Diretório /etc/bind/blockdomi criado com sucesso.
   Versão local não encontrada, baixando a versão 2024101202.
   Arquivo de zona RPZ atualizado.
   Permissões do diretório alteradas com sucesso.
   Serviço Bind9 recarregado com sucesso.
   </pre>


   O diretório `/etc/bind/blockdomi/` terá os seguintes arquivos:

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

   Caso não tenha o `tree` instalado:

   ```plaintext
   sudo apt install tree
   ```


7. **Executar o Script Novamente**

   Se o script for executado novamente com a mesma versão, a mensagem exibida será:

   <pre>
   Diretório /etc/bind/blockdomi já existe.
   Já está na versão mais atual: 2024101202.
   </pre>


8. **Automatizar a Execução do Script**

   Para manter a lista sempre atualizada, configure o script para ser executado diariamente à meia-noite:

   ```plaintext
   echo '00 00   * * *   root    /etc/bind/scripts/blockdomi-bind9.sh localhost' >> /etc/crontab
   ```

   > Caso utilize um domínio para a página de bloqueio, substitua `localhost` pelo seu domínio.

   Reinicie o cron:

   ```plaintext
   systemctl restart cron
   ```
9. **Domínios Bloqueados**

   No arquivo `/etc/bind/blockdomi/db.rpz.zone.hosts` segue o exemplo de como irá ficar os dominios bloqueados:

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


   Para cada domínio bloqueado, irá conter:

   <pre>
   assistirseriesmp4.com IN CNAME localhost.
   *.assistirseriesmp4.com IN CNAME localhost.
   </pre>
      

10. **Verificar Domínios Bloqueados**

    Para verificar os domínios bloqueados:

    ```plaintext
    cat /etc/bind/blockdomi/domain_all
    ```


11. **Testar Domínios Bloqueados**

    Após rodar o script, você poderá testar os domínios bloqueados substituindo `assistirseriesmp4.com` pelo domínio que deseja testar:

    ```plaintext
    dig assistirseriesmp4.com @localhost
    ```


    Caso não tenha os pacotes `dnsutils` instalados para testar com o `dig`:

    ```plaintext
    apt install dnsutils
    ```


    Exemplo de saída do comando `dig`:

    <pre>
      ; <<>> DiG 9.16.50-Debian <<>> assistirseriesmp4.com @localhost
      ;; global options: +cmd
      ;; Got answer:
      ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23555
      ;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 2
      
      ;; OPT PSEUDOSECTION:
      ; EDNS: version: 0, flags:; udp: 1232
      ; COOKIE: bb21e28a5d506ebe01000000670ac85874808e2b1b5e2c57 (good)
      ;; QUESTION SECTION:
      ;assistirseriesmp4.com.         IN      A
      
      ;; ANSWER SECTION:
      assistirseriesmp4.com.  5       IN      CNAME   localhost.
      localhost.              604800  IN      A       127.0.0.1
      
      ;; ADDITIONAL SECTION:
      blockdomi.zone.         1       IN      SOA     LOCALHOST. localhost. 2024101201 3600 900 2592000 7200
      
      ;; Query time: 552 msec
      ;; SERVER: ::1#53(::1)
      ;; WHEN: Sat Oct 12 16:04:56 -03 2024
      ;; MSG SIZE  rcvd: 176
    </pre>

    
    Observe que o domínio `assistirseriesmp4.com` está sendo redirecionado para o `localhost`:
    
    <pre>
      assistirseriesmp4.com.  5       IN      CNAME   localhost.
      localhost.              604800  IN      A       127.0.0.1
    </pre>
