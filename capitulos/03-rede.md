# 03 · Configurações de rede

[← Pacotes](02-pacotes.md) · [README](../README.md) · [Usuários →](04-usuarios.md)

## Objetivo e conceitos

Dar ao servidor um endereço previsível. **IP** identifica a interface; **prefixo** delimita a rede local; **rota padrão** escolhe a saída para outras redes; **DNS** resolve nomes. Para esta prática: `192.168.50.10/24`, gateway e DNS `192.168.50.1`.

Pré-requisitos: [rede LAB montada](00-laboratorio.md), pfSense com LAN configurada segundo o capítulo 06 e resolução DNS ativa, ou outro gateway de laboratório com esses serviços. Você pode preparar o gateway antes de voltar a este capítulo.

> [!WARNING]
> Faça mudanças pelo console da VM. Alterar IP ou reiniciar o gerenciador de rede pode encerrar SSH. Tire um snapshot e copie os arquivos atuais antes de editar.

## Identificar o ambiente

```bash
ip -br link
ip -br address
ip route
systemctl is-active networking NetworkManager systemd-networkd
ls -l /etc/resolv.conf
cat /etc/resolv.conf
```

`enp0s3` é apenas o nome usado nos exemplos. Substitua-o em todas as configurações. Confira também `/etc/network/interfaces`, os perfis de NetworkManager e arquivos de `/etc/systemd/network/` existentes. Mais de um serviço pode estar ativo, administrando interfaces distintas.

Escolha **um** responsável pela interface. O Debian admite diferentes gerenciadores; não existe um arquivo universal para todas as instalações. Fonte: [Debian Reference — rede](https://www.debian.org/doc/manuals/debian-reference/ch05.en.html).

## Alternativa A: ifupdown

Use esta alternativa somente se a interface estiver sob responsabilidade de `ifupdown`. Garanta que o pacote esteja instalado e não configure a mesma interface nas alternativas B ou C. Faça cópia de `/etc/network/interfaces` e dos arquivos incluídos por ele.

Edite com `sudo nano /etc/network/interfaces`, preservando loopback e eventuais diretivas de inclusão. O trecho da interface deve ficar assim:

```ini
auto lo
iface lo inet loopback

auto enp0s3
iface enp0s3 inet static
    address 192.168.50.10/24
    gateway 192.168.50.1
```

Remova apenas a declaração DHCP anterior **da mesma interface**, inclusive em arquivos incluídos, para evitar duplicidade. Se `/etc/resolv.conf` for um arquivo comum sem gerenciamento automático, edite-o para conter:

```text
nameserver 192.168.50.1
```

Se for um link simbólico ou arquivo gerenciado, configure o DNS na ferramenta que o controla. A diretiva `dns-nameservers` em `interfaces` depende de integração adicional, como `resolvconf`; ela não configura DNS sozinha em qualquer instalação.

No console:

```bash
sudo systemctl restart networking
```

## Alternativa B: NetworkManager

Use esta alternativa para uma interface gerenciada pelo NetworkManager, comum em instalações gráficas:

```bash
nmcli device status
nmcli connection show
```

Anote o nome real do perfil e substitua `PERFIL_LAB` abaixo, mantendo aspas se houver espaços:

```bash
sudo nmcli connection modify 'PERFIL_LAB' ipv4.method manual ipv4.addresses 192.168.50.10/24 ipv4.gateway 192.168.50.1 ipv4.dns 192.168.50.1 ipv4.ignore-auto-dns yes
sudo nmcli connection up 'PERFIL_LAB'
```

Não altere IPv6 sem um plano específico; esta prática configura apenas o endereçamento IPv4.

## Alternativa C: systemd-networkd

Use apenas se a interface já for gerenciada por networkd. Identifique com `networkctl status enp0s3` o arquivo responsável; edite esse arquivo em vez de criar outro que possa perder a precedência. Exemplo de conteúdo `.network`:

```ini
[Match]
Name=enp0s3

[Network]
Address=192.168.50.10/24
Gateway=192.168.50.1
DNS=192.168.50.1
```

`DNS=` precisa de um resolvedor integrado, como `systemd-resolved`, para chegar às aplicações. Confira se ele está ativo e como `/etc/resolv.conf` é fornecido; sem essa integração, configure o resolvedor realmente utilizado. Aplique pelo console com `sudo networkctl reload` e `sudo networkctl reconfigure enp0s3`.

## Validar por camadas

```bash
ip -br address
ip route
ping -c 3 192.168.50.1
getent hosts debian.org
curl -I https://www.debian.org
```

| Sintoma | Investigue primeiro |
| --- | --- |
| Sem endereço esperado | Interface, cabo virtual, perfil aplicado e conflitos entre gerenciadores |
| Gateway inacessível | Segmento virtual, máscara, IP da LAN e firewall |
| IP funciona, nome falha | DNS e `/etc/resolv.conf` |
| Nome resolve, HTTPS falha | Rota, firewall, serviço remoto, relógio e certificados |

Ping pode ser bloqueado; combine evidências, não conclua por um único teste. Reinicie a VM e repita as verificações para confirmar persistência.

**Recuperação:** pelo console, restaure o arquivo ou perfil anterior e reaplique com o mesmo gerenciador. Se necessário, retorne ao snapshot. **Exercício:** explique por que trocar apenas o DNS não corrige um gateway ausente.
