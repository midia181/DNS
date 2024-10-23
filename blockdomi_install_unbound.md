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

## Dependencias para rodar o script bash
   
   **No Debian 12, é necessário instalar o pacote `unbound-anchor` (caso ainda não esteja instalado), pois ele não vem incluído por padrão**
   
   ```plaintext
   apt install unbound-anchor
   ```
   
   **No Debian 11, é necessário instalar o pacote `curl` (caso ainda não esteja instalado), pois ele não vem incluído por padrão**
   
   ```plaintext
   apt install curl
   ```

## Bloqueio de domínios no Unbound
2. **Criar Diretório e Baixar Script**

   Crie um diretório onde o script do BLOCKDOMI será armazenado e baixe o script:

   ```plaintext
   mkdir /etc/unbound/scripts
   cd /etc/unbound/scripts
   wget https://raw.githubusercontent.com/midia181/client_blockdomi/refs/heads/main/blockdomi-unbound.sh
   ```

   
3. **Permissões de Execução para o Script**

   Dê permissão de execução ao script:

   ```plaintext
   chmod +x /etc/unbound/scripts/blockdomi-unbound.sh
   ```


4. **Adicione o arquivo de configuração do blockdomi no parametro server: do arquivo /etc/unbound/unbound.conf**
   
    ```plaintext
    nano /etc/unbound/unbound.conf
    ```
    
    ```plaintext
    include: /etc/unbound/blockdomi/blockdomi.conf
    ```
    
   <pre>
   (conteudo atual)
       
   server:
      include: /etc/unbound/blockdomi/blockdomi.conf
   </pre>


    Verifique se o unbound não contem erros de configurações
    ```plaintext
    unbound-checkconf
    ```
    Se não houver erros, o comando retornará:

   <pre>
   unbound-checkconf: no errors in /etc/unbound/unbound.conf
   </pre>


5. **Executar o Script**

   Execute o script para sincronizar com a API do BLOCKDOMI:
   
   ```plaintext
   /etc/unbound/scripts/blockdomi-unbound.sh 127.0.0.1
   ```

   > Caso utilize um domínio para a página de bloqueio, substitua `127.0.0.1` pelo seu domínio.

6. **Verificar Mensagens do Script**

   Ao rodar o script pela primeira vez, se tudo ocorrer bem, a mensagem exibida será:

   <pre>
   Diretório /etc/unbound/blockdomi criado com sucesso.
   Versão local não encontrada, baixando a versão 2024101104.
   Arquivo de configuração do Unbound atualizado para bloqueio.
   Permissões do diretório alteradas com sucesso.
   unbound-checkconf: no errors in /etc/unbound/unbound.conf
   Serviço Unbound recarregado com sucesso.
   </pre>

   O diretório `/etc/bind/blockdomi/` terá os seguintes arquivos:

   ```plaintext
   tree -h /etc/unbound/blockdomi/
   ```

   <pre>
   [4.0K]  /etc/unbound/blockdomi/
   ├── [597K]  blockdomi.conf
   ├── [118K]  domain_all
   └── [  10]  version
    
   1 directory, 3 files
   </pre>
   
   Caso não tenha o `tree` instalado:

   ```plaintext
   sudo apt install tree
   ```


7. **Executar o Script Novamente**

   Se o script for executado novamente com a mesma versão, a mensagem exibida será:

   <pre>
   Diretório /etc/unbound/blockdomi já existe.
   Já está na versão mais atual: 2024101104.
   </pre>


8. **Automatizar a Execução do Script**

   Para manter a lista sempre atualizada, configure o script para ser executado diariamente à meia-noite:

   ```plaintext
   echo '00 00   * * *   root    /etc/unbound/scripts/blockdomi-unbound.sh 127.0.0.1' >> /etc/crontab
   ```
   
   > Caso utilize um domínio para a página de bloqueio, substitua `l27.0.0.1` pelo seu domínio.

   Reinicie o cron:

   ```plaintext
   systemctl restart cron
   ```

9. **Domínios Bloqueados**


   No arquivo /etc/unbound/blockdomi/blockdomi.conf segue o exemplo de como irá ficar os dominios bloqueados

   <pre>
   local-zone: "sitequeprecisabloquear.com" redirect
   local-data: "sitequeprecisabloquear.com A 127.0.0.1"
   </pre>

   Para cada domínio bloqueado, irá conter:
   
   <pre>
   local-zone: "sitequeprecisabloquear.com" redirect
   local-data: "sitequeprecisabloquear.com A 127.0.0.1"
   </pre>


10. **Verificar Domínios Bloqueados**

    Para verificar os domínios bloqueados:

   ```plaintext
   cat /etc/unbound/blockdomi/domain_all
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
    ;; communications error to ::1#53: connection refused
    ;; communications error to ::1#53: connection refused
    ;; communications error to ::1#53: connection refused
    
    ; <<>> DiG 9.18.24-1-Debian <<>> assistirseriesmp4.com @localhost
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8506
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
    
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ;; QUESTION SECTION:
    ;assistirseriesmp4.com.           IN      A
    
    ;; ANSWER SECTION:
    assistirseriesmp4.com.    3600    IN      A       127.0.0.1
    
    ;; Query time: 0 msec
    ;; SERVER: 127.0.0.1#53(localhost) (UDP)
    ;; WHEN: Sat Oct 12 10:55:25 -03 2024
    ;; MSG SIZE  rcvd: 64
    </pre>

    
    Observe que o domínio `assistirseriesmp4.com` está sendo redirecionado para o `127.0.0.1`:
    
    <pre>
    assistirseriesmp4.com.    3600    IN      A       127.0.0.1
    </pre>
