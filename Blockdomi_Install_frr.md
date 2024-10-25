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

Antes de começar não esqueça de virar root da forma correta e atualizar os pacotes, e instalar alguns pacotes que serão necessários.

```plaintext
su -
apt update
apt upgrade
apt install curl apt-transport-https gnupg2 lsb-release tree net-tools
```

1. **Instalação do Free Range Routing**

Pacotes presente do FRR no repositório do Debian 10 estão na versão 6.x.x, e Debian 11 na 7.5.x vou ir além e usar o repositório mais
atualizado do FRR disponíveis em https://deb.frrouting.org, mas fica a seu critério se desejar usar o repositório “default”.
  
Para usar pacotes do repositório FRR faça:
  
```plaintext
curl -s https://deb.frrouting.org/frr/keys.asc | apt-key add -
```

2. **Instale o frr mas fique a seu critério/necessidade para instalar mais pacotes.**

```plaintext
apt install frr
```
   
3. **Após instalação o diretorio /etc/ffr/ será criado juntamente com seus arquivos de configuração. Arquitetura do diretório:**

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


4. **Como de costume vamos fazer um backup dos arquivos originais caso cometermos alguma “cagada” **


```plaintext
mkdir /etc/frr/backups
cp /etc/frr/daemons /etc/frr/backups/
cp /etc/frr/frr.conf /etc/frr/backups/
cp /etc/frr/vtysh.conf /etc/frr/backups/
cp /etc/frr/support_bundle_commands.conf /etc/frr/backups/
```

5. **Ative o daemon BGP**

Para ativar o daemon BGP, edite o arquivo /etc/frr/daemons e habilite o daemon desejado configurando com "yes", da seguinte forma:

```plaintext
nano /etc/frr/daemons
```
<pre>
   # Para habilitar um daemon específico, simplesmente altere o 'no' correspondente para 'yes', e será necessário reiniciar o serviço.
   bgpd=yes 	# BGP
   ospfd=no 	# OSPFD (OSPFv2 - IPv4)
   ospf6d=no 	# OSPF6D (OSPFv3 - IPv6)
   ripd=no 	# RIPD (RIPv2 - IPv4)
   ripngd=no 	# RIPNGD (RIPv3 - IPv6)
   isisd=no 	# ISISD (IS-IS - Protocolo igp Cisco)
   pimd=no 	# PIMD (PIM - Roteamento Multicast)
   ldpd=no 	# LDPD (LDP - Labels MPLS para rotas igp)
   nhrpd=no 	# NHRPD (NHRP - roteamento entre tuneis)
   eigrpd=no 	# EIGRPD (EIGRP - protocolo igp Cisco)
   babeld=no 	# BABELD (BABEL - protocolo igp dual-stack)
   sharpd=no 	# SHARPD (SHARP - protocolo exemplo zclient)
   pbrd=no 	# PBRD (PBR - gestao de regras para Police Based Routing)
   bfdd=no 	# BFDD (BFD - protocolo de adjacencia instantanea)
   fabricd=no 	# FABRICD (OpenFabric)
   vrrpd=no 	# VRRPD (Virtual Router Redundancy Protocol Deamon)
   
   # Como o nome diz, isso faz com que o VTYSH aplique a configuração ao iniciar aos daemons. 
   vtysh_enable=yes 
   
   # O próximo conjunto de linhas controla quais opções são passadas aos daemons quando iniciados. 
   zebra_options="  -A 127.0.0.1 -s 90000000"
   bgpd_options="   -A 127.0.0.1"
   ospfd_options="  -A 127.0.0.1"
   ospf6d_options=" -A ::1"
   ripd_options="   -A 127.0.0.1"
   ripngd_options=" -A ::1"
   isisd_options="  -A 127.0.0.1"
   pimd_options="   -A 127.0.0.1"
   ldpd_options="   -A 127.0.0.1"
   nhrpd_options="  -A 127.0.0.1"
   eigrpd_options=" -A 127.0.0.1"
   babeld_options=" -A 127.0.0.1"
   sharpd_options=" -A 127.0.0.1"
   pbrd_options="   -A 127.0.0.1"
   staticd_options="-A 127.0.0.1"
   bfdd_options="   -A 127.0.0.1"
   fabricd_options="-A 127.0.0.1"
   vrrpd_options="  -A 127.0.0.1"
</pre>
 

6. **Alterações feitas no /etc/frr/daemons vamos reiniciar o FRR**

```plaintext
systemctl restart frr
```
   
Verifique se o mesmo está ok

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


7. **Entrado no Shell VTY e configurando BGP**

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


9. **Criar a pasta e baixar o script para altomatizar os bloqueios dos ips**


```plaintext
mkdir /etc/frr/block
mkdir /etc/frr/block/script
cd /etc/frr/block/script
curl -O https://raw.githubusercontent.com/midia181/client_blockdomi/refs/heads/main/sync-frr-block.sh
```


11. **Testar o script**

Substitua o `<AS>` pelo numero de AS configurado no IBGP:

```plaintext
bash /etc/frr/block/script/sync-frr-block.sh <AS>
```


12. **Você pode adicionar o script ao cron usando o comando echo para criar uma nova entrada. Abaixo está o exemplo de como adicionar o script sync-frr-block.sh ao cron para ser executado diariamente:

```plaintext
echo "0 0 * * * /bin/bash /etc/frr/block/script/sync-frr-block.sh" >> /etc/crontab
```

13. 
