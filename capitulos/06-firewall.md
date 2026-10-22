# 06 · Firewall com nftables e pfSense

[← Usuários](04-usuarios.md) · [README](../README.md) · [DHCP →](07-dhcp.md)

## Dois papéis diferentes

O **firewall do host** protege o próprio Debian. O **firewall de borda** controla tráfego entre redes. Aqui, nftables protege `srv01`; pfSense atua como gateway em uma VM separada. pfSense é baseado em FreeBSD, não é uma distribuição Linux; não execute nele comandos APT ou nftables.

Uma regra combina condições e uma ação, como permitir ou descartar. Um firewall **com estado** reconhece respostas de conexões já autorizadas. Uma política de entrada restritiva exige abrir explicitamente os serviços desejados.

## Parte A · Preparar o gateway pfSense

Esta parte pode ser feita antes do capítulo de rede. Use a imagem compatível com seu hipervisor obtida pelo [canal oficial da Netgate](https://www.pfsense.org/download/), seguindo as condições da edição escolhida.

1. Crie uma VM com duas interfaces: WAN em NAT e LAN na rede interna `LAB`. Anote os endereços MAC para não inverter as interfaces.
2. Instale no disco virtual e atribua WAN e LAN pelo console.
3. Configure LAN como `192.168.50.1/24`. Se ativar DHCP provisório, reserve o intervalo `.100`–`.199` e mantenha o servidor `.10` fora dele.
4. Coloque o cliente na LAN, usando DHCP ou IP estático `.100/24`, gateway `.1` e DNS `.1` se DHCP ainda não estiver disponível.
5. Acesse `https://192.168.50.1` pela LAN, confira o certificado no contexto da instalação e conclua o assistente. Troque imediatamente a senha administrativa inicial.
6. Em **System > General Setup**, configure nome `fw01`, domínio de laboratório e fuso `America/Sao_Paulo`. Configure DNS de saída válido para o ambiente. Em **Services > DNS Resolver**, confirme que o resolvedor atende a LAN e consegue resolver nomes.
7. Em **Interfaces > WAN**, a WAN virtual atrás de NAT terá IP privado. Avalie a opção que bloqueia redes privadas na WAN, pois ela pode bloquear tráfego do segmento de testes; não replique essa exceção indiscriminadamente em uma WAN real.
8. Verifique as regras em **Firewall > Rules > LAN** e o modo de NAT de saída em **Firewall > NAT > Outbound**. Para começar, o NAT automático costuma atender a saída da LAN nesse cenário.

O IP LAN de fábrica pode ser diferente do adotado aqui; este guia o altera deliberadamente. Consulte a [documentação de configuração do pfSense](https://docs.netgate.com/pfsense/en/latest/config/index.html).

Antes de restringir a LAN, crie uma regra que permita do cliente administrativo para o endereço LAN do firewall TCP 443. As regras de interface são avaliadas no tráfego que **entra por aquela interface**, e sua ordem importa. Teste navegação, DNS e administração antes e depois da mudança. Não publique a interface administrativa na WAN.

### Proteger o acesso administrativo

Antes de concluir a configuração, confira as opções abaixo. Algumas caixas desativam uma proteção quando são marcadas; leia o nome completo de cada opção.

| Opção | Como configurar e o que observar |
| --- | --- |
| HTTPS na administração | Manter HTTPS e planejar certificado confiável |
| Desabilitar logs de logins bem-sucedidos | Manter registro para auditoria |
| Desabilitar DNS Rebinding Checks | Manter proteção; tratar exceções necessárias por domínio/nome |
| Desabilitar HTTP_REFERER enforcement | Manter verificação; usar nomes alternativos autorizados quando necessário |
| SSH habilitado com senha ou chave | Habilitar só se necessário; testar chaves antes de restringir a autenticação |
| Desmarcar Allow IPv6 | Isso controla filtragem, não desativa toda a pilha IPv6 |
| Marcar Suppress ARP messages | Suprime mensagens; não corrige conflitos de rede |

**System > Advanced > Admin Access** concentra opções administrativas. As recomendações sobre HTTPS, logs, nomes alternativos e SSH seguem o [manual de acesso administrativo](https://docs.netgate.com/pfsense/en/latest/config/advanced-admin.html); exceções DNS devem ser específicas, conforme [DNS Rebinding Protections](https://docs.netgate.com/pfsense/en/latest/services/dns/rebinding.html).

Em **System > Advanced > Networking**, não use supressão ARP como correção de topologia nem trate `Allow IPv6` como chave geral de desativação de IPv6. Consulte [Advanced Networking](https://docs.netgate.com/pfsense/en/latest/config/advanced-networking.html). Este laboratório usa IPv4 para os exemplos, mas o planejamento de segurança também deve considerar IPv6.

Exporte uma cópia da configuração em **Diagnostics > Backup & Restore** após validar o gateway. Proteja esse arquivo: ele pode conter informações sensíveis. Não desative a regra anti-lockout antes de ter outra regra de acesso testada e console disponível.

## Parte B · Firewall local com nftables

Pré-requisitos: Debian com IP estático e SSH funcional. A regra abaixo é para um **servidor independente** sem Docker, VPN ou outro gerenciador de firewall. Ela substitui as regras existentes e não configura roteamento/NAT.

> [!WARNING]
> Aplique pelo console da VM. `flush ruleset` remove todas as regras nftables existentes, inclusive de outros programas. Tire snapshot, confira `sudo nft list ruleset` e não misture esta configuração com UFW, firewalld ou regras geradas por contêineres.

```bash
sudo apt install nftables
sudo sh -c 'nft list ruleset > /root/nftables-anterior.nft'
sudo cp -a /etc/nftables.conf /root/nftables.conf.anterior
sudo nano /etc/nftables.conf
```

Conteúdo para o laboratório com IP estático:

```nft
#!/usr/sbin/nft -f
flush ruleset

table inet lab_host {
    chain input {
        type filter hook input priority 0; policy drop;
        iifname "lo" accept
        ct state invalid drop
        ct state established,related accept
        meta l4proto { icmp, ipv6-icmp } accept
        ip saddr 192.168.50.0/24 tcp dport 22 counter accept
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

A família `inet` atende IPv4 e IPv6. A regra ICMP permite diagnóstico e mensagens necessárias à rede, incluindo descoberta de vizinhos IPv6. A entrada SSH fica autorizada apenas a partir da rede IPv4 LAB. A saída está liberada neste exercício. A organização das cadeias pode ser comparada ao [exemplo oficial do nftables](https://wiki.nftables.org/wiki-nftables/index.php/Simple_ruleset_for_a_server).

Valide sintaxe antes de aplicar:

```bash
sudo nft -c -f /etc/nftables.conf
sudo nft -f /etc/nftables.conf
sudo systemctl enable nftables
sudo nft list ruleset
```

`-c` verifica a configuração, mas não comprova que as regras permitem o acesso desejado. No cliente da LAB, abra **outra** sessão SSH. Uma sessão antiga pode sobreviver pela regra `established`, mesmo quando novas conexões estão bloqueadas.

### Ajustar para novos serviços

O servidor DHCP do próximo capítulo precisa de uma regra adicional **antes de aplicar o bloqueio de entrada**. Inclua-a na cadeia `input` de `/etc/nftables.conf`, substituindo a interface real:

```nft
iifname "enp0s3" udp sport 68 udp dport 67 counter accept
```

Não limite essa regra a `ip saddr 192.168.50.0/24`: clientes sem endereço podem enviar a partir de `0.0.0.0`. Esse caso é para clientes diretamente conectados; relay DHCP exige regras e configuração próprias. DHCP com sockets raw pode interagir de forma diferente com os filtros do host: a regra não substitui isolamento da rede e teste real de concessão. Se o próprio servidor usar DHCP como cliente, a configuração restritiva também precisará acomodar esse tráfego; o cenário aqui usa IP fixo.

Para qualquer alteração: edite o arquivo, execute `nft -c -f` e depois `nft -f`. Não abra porta MariaDB quando somente programas locais acessam o banco. SFTP usa a mesma porta SSH.

## Validação e recuperação

- [ ] Cliente resolve nomes e alcança a internet pelo pfSense.
- [ ] Administração funciona pela LAN e não foi publicada na WAN.
- [ ] Novas conexões SSH da LAB funcionam com nftables ativo.
- [ ] Após reiniciar o Debian, `nft list ruleset` mostra as regras esperadas.

Se bloquear acesso ao Debian, restaure o arquivo anterior pelo console. Para restaurar também o estado de regras salvo, no console deste laboratório execute `sudo nft flush ruleset` seguido de `sudo nft -f /root/nftables-anterior.nft`; isso deixa um intervalo sem filtragem e só deve ser feito na rede isolada. No pfSense, use o console e a cópia de configuração conforme o manual de recuperação.

**Exercício:** explique por que uma regra no gateway não controla necessariamente a comunicação entre dois computadores no mesmo segmento LAN.
