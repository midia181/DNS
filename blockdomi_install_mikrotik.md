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

### Bloqueio de Domínios no Mikrotik V6

Para o bloqueio funcionar corretamente, o DNS Cache do mikrotik precisa esta habilitado. (Não é uma pratica recomendada)
Recomendo mudar o quanto antes para Unbound ou Bind9 em servidores Debian/Ubuntu.

1 - No winbox, vá para o item de menu IP > DNS. A janela configurações de DNS será exibida.

2 - Coloque os IPs fornecidos pelo ISP, ou utilize servidores publicos de DNS, (como google [8.8.8.8, 8.8.4.4], cloudfare [1.1.1.1, 1.0.0.1] etc...)

3 - Clique na caixa de seleção para permitir solicitações remotas,(Allow Remote Requests), Ele habilitará o recurso DNS do cache do MikroTik Router.

4 - Opcionalmente, você pode alterar o tamanho do cache colocando o tamanho personalizado na caixa de entrada Tamanho do cache. O tamanho padrão do cache é 2048 KiB ou 2MB.

5 - Clique no botão Aplicar e OK.

O DNS do MikroTik Caching agora está habilitado e você pode usar qualquer um dos seus MikroTik IP como DNS IP para o seu cliente de rede. Se tudo estiver OK, seu cliente receberá resposta do servidor DNS do cache MikroTik. Para verificar seu cache de DNS, vá para o item de menu IP> DNS e clique no botão Cache. Você encontrará o nome de domínio em cache na janela Cache DNS. Se desejar, você pode liberar o objeto em cache clicando no botão Liberar Cache.

Cache do MikroTik O DNS armazena a entrada de DNS dinamicamente sempre que recebe um novo domínio, Mas às vezes você pode precisar colocar a entrada de host estático, como seus servidores locais ou impressoras, MikroTik cache DNS é capaz de obter entrada de host estático. é assim que o blockdomi entra em ação.

Para integrar o Mikrotik ao BlockDomi com acesso via SSH ou Winbox (Terminal), Execute o comando abaixo:
```plaintext
/system script add name="BlockDomiImport" source="\
/log warning \"Iniciando limpeza de DNS Static\"; \
/system logging disable [find topics~\"info\"]; \
/ip dns static remove [find]; \
/system logging enable [find topics~\"info\"]; \
/log warning \"Limpeza de DNS Static concluida\"; \
/tool fetch url=\"https://api.blockdomi.com.br/commands/mikrotik/v6\" mode=http dst-path=\"blocked_domains.rsc\"; \
/log warning \"Iniciando importacao de dominios bloqueados\"; \
/system logging disable [find topics~\"info\"]; \
/import file-name=blocked_domains.rsc; \
/system logging enable [find topics~\"info\"]; \
/log warning \"Importacao de dominios bloqueados concluida\"; \
/file remove blocked_domains.rsc;"
```
Após a inserção do script, certifique-se de que ele está programado para rodar automaticamente todos os dias às 1h da madrugada. Para garantir a execução correta, verifique se o horário do seu Mikrotik está devidamente sincronizado.
```plaintext
/system scheduler add name="DailyBlockDomiImport" start-time=01:00:00 interval=1d on-event="BlockDomiImport"
```
siga os passos para garantir que o mikrotik irá sicronizar altomaticamente com o blockdomi

1º - Verifique em System > Scripts, veja se o script "BlockDomiImport" estará criado.

2º - Verifique em System > Scheduler, se a rotina "BlockDomiImport" foi criada.

3º - Verifique os dominios bloqueados em IP > DNS > Static

4º - Verifique as regras input que bloqueia invasões a porta 53 em IP > Firewall > Filter Rules irá conter uma regra com nome "Bloqueia Conexões Entrantes na Porta 53"


Validando as configurações acima tudo irá acontecer como esperado.
