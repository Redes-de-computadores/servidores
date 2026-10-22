# 02 · Gerenciamento de pacotes

[← Rede](01-rede.md) · [README](../README.md) · [SSH →](03-ssh.md)

## Conceitos

Um **pacote** contém arquivos de um programa e informações sobre suas dependências. Um **repositório** publica pacotes e metadados assinados. O APT consulta repositórios e resolve dependências; o `dpkg` trabalha com os pacotes locais. Um **serviço** é um processo administrado em segundo plano, como o servidor SSH. O nome do pacote pode ser diferente do nome do serviço.

Pré-requisitos: Debian instalado, acesso ao sudo e conectividade temporária ou definitiva com a internet.

## Atualizar sem confundir as etapas

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
```

`update` atualiza o índice local, sem atualizar os programas instalados. `upgrade` apresenta as atualizações e pede confirmação. Leia a proposta. Reinicializações ou reinícios de serviços podem ser necessários.

`apt full-upgrade` pode remover pacotes para resolver dependências. Use-o quando compreender a proposta, especialmente em atualizações de versão. Trocar a versão principal exige seguir as [notas de lançamento](https://www.debian.org/releases/trixie/release-notes/); não basta substituir nomes nos repositórios.

## Encontrar e instalar ferramentas

```bash
apt search openssh-server
apt show openssh-server
apt policy openssh-server
sudo apt install nano man-db curl dnsutils rsync
dpkg -l openssh-server
dpkg -L openssh-server
```

Os dois últimos comandos só encontrarão o pacote se ele estiver instalado. `apt policy` mostra a versão candidata e sua origem. `dpkg -L` lista os arquivos fornecidos pelo pacote, útil para localizar exemplos de configuração.

| Ação | Comando | Consequência |
| --- | --- | --- |
| Remover programa | `sudo apt remove nome-do-pacote` | Pode manter configurações do pacote |
| Remover e expurgar configuração | `sudo apt purge nome-do-pacote` | Remove também configurações gerenciadas pelo pacote |
| Rever dependências dispensáveis | `sudo apt autoremove` | Apresenta uma lista; confira antes de aceitar |

Os nomes `nome-do-pacote` são marcadores: substitua-os por um pacote real. Não expurgue programas de acesso remoto durante uma sessão sem console de recuperação.

## Conferir repositórios

```bash
ls /etc/apt/sources.list.d/
cat /etc/apt/sources.list
```

Um dos arquivos pode não existir: instalações diferentes usam `/etc/apt/sources.list` ou arquivos `.sources`/`.list` no diretório complementar. Leia os que existirem. Configurações modernas podem usar o formato Deb822 (`.sources`); não acrescente entradas duplicadas nem misture versões Debian. A confiança deve ser vinculada ao repositório por `Signed-By` quando aplicável, conforme o manual `man sources.list` e o [guia APT do Debian](https://wiki.debian.org/SourcesList).

> [!IMPORTANT]
> Falhas de assinatura, relógio incorreto e falhas de DNS exigem diagnóstico. Não desative a verificação de assinatura para fazer a instalação continuar.

## Serviços e validação

Quando instalar SSH no capítulo 05, compare:

```bash
systemctl status ssh --no-pager
systemctl is-enabled ssh
journalctl -u ssh -n 30 --no-pager
```

`status` mostra o estado atual; `is-enabled` informa se o serviço está configurado para iniciar automaticamente. Um pacote instalado não garante que seu serviço esteja ativo.

**Exercício:** instale `tree`, execute `tree -L 1 /etc`, localize o executável com `command -v tree` e descubra qual pacote o fornece com `dpkg -S /usr/bin/tree`. Explique o que mudou entre atualizar o índice e instalar o programa.
