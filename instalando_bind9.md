# INSTALANDO DNS RECURSIVO (BIND9) BASICO
MASTER (NS1)
```plaintext
apt install bind9 dnsutils
```
Pronto! O servidor recursivo já está funcionando, porém ele está aberto, e isso não é nada legal!
Você não vai querer “qualquer um” utilizando seu servidor para resolver nomes. Resolveremos isso mais a frente no arquivo named.conf.options.
Alteramos o DNS do servidor fazendo com que ele consulte em si próprio. Essa alteração deve ser feita no arquivo /etc/resolv.conf.
```plaintext
nano /etc/resolv.conf
```
```plaintext
nameserver 127.0.0.1
nameserver ::1
```
Para descobrir se seu servidor esta resolvendo nomes use o comando dig.
```plaintext
dig google.com @localhost
```
```plaintext
;; ANSWER SECTION:
google.com.             271     IN      A       142.250.78.206

;; Query time: 4 msec
;; SERVER: ::1#53(::1)
;; WHEN: Mon May 06 22:24:17 UTC 2024
;; MSG SIZE  rcvd: 83
```
Os arquivos de configuração do bind ficam no diretório /etc/bind/ e agora no Debian 10/11 também separando os root servers em /usr/share/dns/
```plaintext
cd /etc/bind
```
Iremos alterar o named.conf.options, o próprio nome já se auto descreve o que vamos encontrar nele.
Sempre gosto de preservar o arquivo original, então fizemos um backup antes de modifica-lo.
```plaintext
cp /etc/bind/named.conf.options /etc/bind/named.conf.options.bkp
```
```plaintext
echo > named.conf.options
```
Aqui vai uma dica para que usa Windows + Putty, tome cuidado ao colar, principalmente quem usa Windows 10 “ele copia caracteres imaginários que você não vê!”.
Recomento usar o Bitvise SSH Client que é muito superior.

Explicação comentada no arquivo.
```plaintext
nano /etc/bind/named.conf.options
```
```plaintext
// ACL "autorizados" vao ficar os ips que sao autorizados a fazer
// consultas recursivas neste servidor.
// Neste caso vou incluir os ips que foram nos delegados bem como de localhost e IPs privados
acl autorizados {
        127.0.0.1;
        ::1;
        192.168.0.0/16;
        172.16.0.0/12;
        100.64.0.0/10;
        10.0.0.0/8;
};
 
options {
    // O diretorio de trabalho do servidor
    // Quaisquer caminho nao informado sera tomado como  padrao este diretorio
    directory "/var/cache/bind";
 
    //Suporte a DNSSEC
    dnssec-validation auto;
 
    // Conforme RFC1035
    // https://www.ietf.org/rfc/rfc1035.txt
    auth-nxdomain no;
 
    // Respondendo para IPv4 e IPv6
    // Porta 53 estara aberta para ambos v4 e v6
    listen-on { any; };
    listen-on-v6 { any; };
 
    // Limitacao da taxa de resposta no sistema de nomes de dominio (https://kb.isc.org/docs/aa-00994)
    // Se voce esta montando um servidor apenas para autoritativo descomente as linhas a baixo.
    //rate-limit {
    //    responses-per-second 15;
    //    window 5;
    //};

    // Se usar bloqueios de dominios crie o rpz e descomente as linhas seguintes:
    //response-policy {
    //zone "rpz.zone" policy CNAME localhost;
    //};
 
    // Melhora o desempenho do servidor, reduzindo os volumes de dados de saída.
    // O padrao BIND é (no) nao.
    minimal-responses yes;
 
    // Especifica quais hosts estao autorizados a fazer consultas
    // recursivas atraves deste servidor.
    // Aqui que voce vai informar os IPs da sua rede que voce ira permitir consultar os DNS.
    allow-recursion {
        autorizados;
    };
 
    // Endereco estao autorizados a emitir consultas ao cache local,
    // sem acesso ao cache local as consultas recursivas sao inuteis.
    allow-query-cache {
        autorizados;
    };
 
    // Especifica quais hosts estao autorizados a fazer perguntas DNS comuns.
    allow-query { any; };
 
    // Especifica quais hosts estao autorizados a receber transferencias de zona a partir do servidor.
    // Seu servidor Secundario, no nosso ex vou deixar entao o ips dos dois servidores v4 e v6.
    // Caso nao utilize servidor secundario altere para allow-transfer { any; }; e remova also-notify.
    allow-transfer {
        45.80.48.3;
        2804:f123:bebe:cafe::3;
    };
    also-notify {
        45.80.48.3;
        2804:f123:bebe:cafe::3;
    };
 
    // Esta opcao faz com que o servidor slave ao fazer a transferencia de zonas
    // mastes deste servidor nao compile o arquivo, assim no outro servidor o arquivo
    // da zona tera um texto "puro"
    masterfile-format text;
 
    // Para evitar que vase a versao do Bind, definimos um nome
    // Reza a lenda que deixar RR DNS Server seu servidor nunca sofrera ataques.
    version "RR DNS Server";
};
```
Legal agora o servidor Recursivo já está funcionando e limitando os IPs que poderão realizar consultas ao mesmo.
Caso você não queria seu servidor sendo recursivo altere na ACL autorizados deixando apenas 127.0.0.1 e ::1.
Se seu servidor não tiver IPv6? (Que triste rsrsrs) Recomendo que desative o ipv6 no bind.

Altere listen-on-v6 para none.
```plaintext
nano /etc/bind/named.conf.options
```
```plaintext
listen-on-v6 { any; };
#para:
listen-on-v6 { none; };
```
Toda alteração feita no bind para ter efeito é necessário restartar o serviço.
```plaintext
systemctl restart bind9
```
Se desejar remover os comentários do named.conf.options execute:
```plaintext
cat /etc/bind/named.conf.options |grep -v "//" > /tmp/named.conf.options ; mv /tmp/named.conf.options /etc/bind/
```
Para verificar se seu arquivo tem algum erro use o comando named-checkconf
Se nada retornar é porque não tem nenhum erro.
```plaintext
named-checkconf /etc/bind/named.conf.options
```
Imprime todas as configurações
```plaintext
named-checkconf -p
```
Verifique também o status do bind para ver se o mesmo era rodando.
```plaintext
systemctl restart bind9
```
```plaintext
systemctl status bind9
```
Se tudo ocorrer como esperado seu DNS Recusivo esta finalizado e funcionando

# CREDITOS 
Rudimar Remontti
https://blog.remontti.com.br/5958
