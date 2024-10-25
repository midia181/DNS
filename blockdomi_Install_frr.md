## Bloqueios na Internet

> **Confidencial**: Devido à natureza delicada das informações e ao contexto de sua disseminação, os domínios e sites restritos são tratados como confidenciais. O acesso à API é exclusivo para empresas de telecomunicações devidamente registradas na Anatel.
>
> **Verifique seu CNPJ aqui**: [ANATEL Outorga e Licenciamento](https://informacoes.anatel.gov.br/paineis/outorga-e-licenciamento)
>
> **Contatos para obter acesso à API**:
> - [WhatsApp](https://api.whatsapp.com/send/?phone=5584998667245&text=Como+obter+acesso+a+API%3F&type=phone_number&app_absent=0)
> - [Telegram](https://t.me/LucasMidia)

---

# Implementação de Bloqueios de Ips

A restrição de IPs pode ser implementada no roteador de borda ou no servidor BGP, como o FRR, sendo anunciada aos roteadores de borda via BGP ou configurada de forma estática, redirecionando para blackhole.

### Bloqueio de Ips no BGP FRR

Antes de começar, certifique-se de trocar para o usuário root corretamente, atualizar os pacotes e instalar as dependências necessárias:

```plaintext
su -
apt update
apt upgrade
apt install curl apt-transport-https gnupg2 lsb-release tree net-tools
```

**Instalação do Free Range Routing (FRR)**

Os pacotes do FRR disponíveis no repositório padrão do Debian 10 estão na versão 6.x.x e, no Debian 11, na versão 7.5.x. Vamos utilizar o repositório mais atualizado do FRR disponível em https://deb.frrouting.org, mas você pode optar por usar o repositório padrão, se preferir.
  
Para usar os pacotes do repositório oficial do FRR:
  
```plaintext
curl -s https://deb.frrouting.org/frr/keys.asc | apt-key add -
```

**Agora, instale o FRR:**

```plaintext
apt install frr
```
   
**Após a instalação, o diretório /etc/frr/ será criado juntamente com seus arquivos de configuração. A estrutura do diretório será a seguinte:**

```plaintext
tree /etc/frr/
```

<pre>
   /etc/frr/
   ├── daemons
   ├── frr.conf
   ├── support_bundle_commands.conf
   └── vtysh.conf
</pre>


**Como precaução, faça um backup dos arquivos de configuração originais:**


```plaintext
mkdir /etc/frr/backups
cp /etc/frr/daemons /etc/frr/backups/
cp /etc/frr/frr.conf /etc/frr/backups/
cp /etc/frr/vtysh.conf /etc/frr/backups/
cp /etc/frr/support_bundle_commands.conf /etc/frr/backups/
```

**Ative o daemon BGP**

Para ativar o daemon BGP, edite o arquivo /etc/frr/daemons e altere a opção bgpd para "yes":

```plaintext
nano /etc/frr/daemons
```

Habilite o daemon BGP configurando as opções da seguinte maneira:

<pre>
bgpd=yes 	# BGP
ospfd=no 	# OSPFD (OSPFv2 - IPv4)
ospf6d=no 	# OSPF6D (OSPFv3 - IPv6)
ripd=no 	# RIPD (RIPv2 - IPv4)
ripngd=no 	# RIPNGD (RIPv3 - IPv6)
isisd=no 	# ISISD (IS-IS - Protocolo IGP da Cisco)
pimd=no 	# PIMD (Roteamento Multicast)
ldpd=no 	# LDPD (MPLS para rotas IGP)
nhrpd=no 	# NHRPD (Roteamento entre túneis)
eigrpd=no 	# EIGRPD (Protocolo IGP da Cisco)
babeld=no 	# BABELD (Protocolo IGP Dual-Stack)
sharpd=no 	# SHARPD (Protocolo exemplo zclient)
pbrd=no 	# PBRD (Gestão de Regras de PBR)
bfdd=no 	# BFDD (Protocolo de Adjacência Instantânea)
fabricd=no 	# FABRICD (OpenFabric)
vrrpd=no 	# VRRPD (Protocolo de Redundância Virtual)
vtysh_enable=yes
</pre>
 

**Após as alterações, reinicie o serviço FRR:**

```plaintext
systemctl restart frr
```
   
Verifique se o serviço está ativo e rodando corretamente:

```plaintext
systemctl status frr
```
<pre>
     ● frr.service - FRRouting
      Loaded: loaded (/lib/systemd/system/frr.service; enabled; vendor preset: enabled)
      Active: active (running) since Tue 2020-08-11 14:00:29 -03; 41s ago
        Docs: https://frrouting.readthedocs.io/en/latest/setup.html
     Process: 665 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
      Status: "FRR Operational"
       Tasks: 18 (limit: 1150)
      Memory: 24.2M
      CGroup: /system.slice/frr.service
              ├─674 /usr/lib/frr/watchfrr -d -F traditional zebra bgpd ospfd ospf6d babeld staticd
              ├─699 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
              ├─704 /usr/lib/frr/bgpd -d -F traditional -A 127.0.0.1
              ├─712 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
              ├─716 /usr/lib/frr/ospf6d -d -F traditional -A ::1
              ├─720 /usr/lib/frr/babeld -d -F traditional -A 127.0.0.1
              └─724 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1
   
   ago 11 14:00:29 frr zebra[699]: client 47 says hello and bids fair to announce only static routes vrf=0
   set 23 15:44:11 accel watchfrr[1220]: zebra state -> up : connect succeeded
   set 23 15:44:11 accel watchfrr[1220]: bgpd state -> up : connect succeeded
   set 23 15:44:11 accel watchfrr[1220]: ospfd state -> up : connect succeeded
   set 23 15:44:11 accel watchfrr[1220]: ospf6d state -> up : connect succeeded
   set 23 15:44:11 accel watchfrr[1220]: staticd state -> up : connect succeeded
   set 23 15:44:11 accel watchfrr[1220]: all daemons up, doing startup-complete notify
   ago 11 14:00:29 frr frrinit.sh[665]: Started watchfrr.
   ago 11 14:00:29 frr systemd[1]: Started FRRouting.
</pre>

### Acesso ao Shell VTY e Configuração do BGP

**Entrado no Shell VTY e configurando BGP**

Vtysh fornece um frontend combinado para todos os daemons FRR em uma única sessão combinada.:

```plaintext
vtysh
```
   
Aqui iremos bloquear rotas entrantes no ibgp frr e exportar somente a lista de ips nescessarias para o bloqueio,
Exemplo de configuração basica, Ajuste de acordo com oque precisa:

```plaintext
   configure terminal
    !
    router bgp 65530
     neighbor <ipv4-remoto> remote-as <as-remoto>
     neighbor <ipv4-remoto> description "iBGP_BLOCKDOMI_IPv4"
     neighbor <ipv6-remoto> remote-as <as-remoto>
     neighbor <ipv6-remoto> description "iBGP_BLOCKDOMI_IPv6"
     !
     address-family ipv4 unicast
      neighbor <ipv4-remoto> prefix-list FILTRO-BLOCKDOMI-IPV4-IN in
      neighbor <ipv4-remoto> prefix-list FILTRO-BLOCKDOMI-IPV4-OUT out
     exit-address-family
     !
     address-family ipv6 unicast
      neighbor <ipv6-remoto> activate
      neighbor <ipv6-remoto> prefix-list FILTRO-BLOCKDOMI-IPV6-IN in
      neighbor <ipv6-remoto> prefix-list FILTRO-BLOCKDOMI-IPV6-OUT out
     exit-address-family
    !
    ip prefix-list FILTRO-BLOCKDOMI-IPV4-IN seq 5 deny any
    !
    ipv6 prefix-list FILTRO-BLOCKDOMI-IPV6-IN seq 5 deny any
    !
    line vty
    !
    end
    write
```


**Automatizando o Bloqueio de IPs**

Crie o diretório para os scripts e baixe o script de bloqueio:

```plaintext
mkdir /etc/frr/block
mkdir /etc/frr/block/script
cd /etc/frr/block/script
curl -O https://raw.githubusercontent.com/midia181/client_blockdomi/refs/heads/main/sync-frr-block.sh
```


**Testando o Script**

Execute o script, substituindo `<AS>` pelo número do AS configurado no iBGP:

```plaintext
bash /etc/frr/block/script/sync-frr-block.sh <AS>
```


**Adicionando o Script ao Cron**

Para agendar a execução diária do script, adicione-o ao cron:

```plaintext
echo "0 0 * * * /bin/bash /etc/frr/block/script/sync-frr-block.sh" >> /etc/crontab
```

Isso irá garantir que o script seja executado automaticamente todos os dias à meia-noite.

**Arquivos de IPs Bloqueados**

As listas de IPs bloqueados, tanto para IPv4 quanto para IPv6, estão localizadas nos seguintes arquivos dentro do diretório /etc/frr/block/:

ipv4_list.txt: Contém os IPs bloqueados para IPv4.
ipv6_list.txt: Contém os IPs bloqueados para IPv6.
version.txt: Contém a versão atual da lista de bloqueios.

Esses arquivos são atualizados conforme necessário para refletir as alterações no bloqueio de IPs.
