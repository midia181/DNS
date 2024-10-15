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

### Bloqueio de Domínios no Pi-Hole

1. **Adicione a lista de dominios**
   
```plaintext
 https://api.blockdomi.com.br/domain/all
```

<img src="https://i.imgur.com/qGUjsPU.png" alt="image" class="img-custom">


2. **No terminal atualize a lista**

Após adicionar a lista de dominios, é necessário atualizar a "gravity" do Pi-hole para aplicar as mudanças:

```plaintext
pihole -g
```


3. **Adicionar uma Tarefa ao crontab do Root Diretamente com echo**

Você pode adicionar uma linha ao crontab do root para rodar o comando do Pi-hole diariamente às 3 da manhã. Para isso, execute o seguinte comando:

```plaintext
echo "0 3 * * * /usr/local/bin/pihole -g > /tmp/pihole_update.log 2>&1" | sudo tee -a /var/spool/cron/crontabs/root > /dev/null
```


4. **Conferir Dominios Bloqueados**

Execute o script para verificar os dominios bloqueados:

```plaintext
cat /etc/pihole/gravity.db
```


5. **Testar Domínios Bloqueados**

Você poderá testar os domínios bloqueados substituindo `assistirseriesmp4.com` pelo domínio que deseja testar:

```plaintext
dig assistirseriesmp4.com @localhost
```

Caso não tenha os pacotes `dnsutils` instalados para testar com o `dig`:

```plaintext
apt install dnsutils
```


Exemplo de saída do comando `dig`:

<pre>
; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> assistirseriesmp4.com @localhost
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55630
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;assistirseriesmp4.com.         IN      A

;; ANSWER SECTION:
assistirseriesmp4.com.  2       IN      A       0.0.0.0

;; Query time: 4 msec
;; SERVER: ::1#53(localhost) (UDP)
;; WHEN: Tue Oct 15 13:13:36 UTC 2024
;; MSG SIZE  rcvd: 66
</pre>


Observe que o domínio `assistirseriesmp4.com` está sendo redirecionado para o `0.0.0.0`:
    
<pre>
;; ANSWER SECTION:
assistirseriesmp4.com.  2       IN      A       0.0.0.0
</pre>
