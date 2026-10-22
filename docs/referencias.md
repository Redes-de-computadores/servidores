# Referências oficiais

[← README](../README.md)

Use estas referências para aprofundar os assuntos e consultar opções que vão além das atividades. Ao abrir uma página, confira a versão do programa: endereços com `latest` ou `stable` podem apresentar versões diferentes da instalada na sua VM.

| Assunto | Fonte | Uso no guia |
| --- | --- | --- |
| Versões Debian | [Debian Releases](https://www.debian.org/releases/) | Debian 13 e situação do Debian 10 |
| Instalação | [Debian 13 — manual amd64](https://www.debian.org/releases/trixie/amd64/index.en.html) | Instalação, mídia, discos e boot |
| Autenticidade da mídia | [Verificação de imagens Debian](https://www.debian.org/CD/verify) | Hashes e assinaturas |
| Atualização de versão | [Notas de lançamento Trixie](https://www.debian.org/releases/trixie/release-notes/) | Planejamento de migração |
| Repositórios | [Debian Wiki — SourcesList](https://wiki.debian.org/SourcesList) | Formatos de fontes APT e assinatura |
| Rede | [Debian Reference — capítulo 5](https://www.debian.org/doc/manuals/debian-reference/ch05.en.html) | Gerenciadores de rede e resolução de nomes |
| SSH | [OpenSSH — sshd_config](https://man.openbsd.org/sshd_config) | Autenticação, precedência e configuração |
| nftables | [Wiki do projeto — servidor simples](https://wiki.nftables.org/wiki-nftables/index.php/Simple_ruleset_for_a_server) | Cadeias, estado e ICMPv6 |
| pfSense | [Configuração](https://docs.netgate.com/pfsense/en/latest/config/index.html) | Preparação do gateway |
| Administração pfSense | [Admin Access](https://docs.netgate.com/pfsense/en/latest/config/advanced-admin.html) | HTTPS, acesso, logs e SSH |
| DNS pfSense | [DNS Rebinding Protections](https://docs.netgate.com/pfsense/en/latest/services/dns/rebinding.html) | Proteções e exceções DNS |
| Rede pfSense | [Advanced Networking](https://docs.netgate.com/pfsense/en/latest/config/advanced-networking.html) | IPv6 e supressão de avisos ARP |
| ISC DHCP | [Fim de vida e substituição](https://www.isc.org/dhcp/) | Escolha de Kea para laboratório novo |
| Kea no Debian | [Pacote kea-dhcp4-server](https://packages.debian.org/stable/net/kea-dhcp4-server) | Disponibilidade no Debian 13 |
| Configuração Kea | [Manual DHCPv4, linha 2.6.3](https://kea.readthedocs.io/en/kea-2.6.3/arm/dhcp4-srv.html) | Interfaces, sub-redes e opções |
| Agendamento | [systemd.timer](https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html) | Calendário e persistência |
| rsyslog/MariaDB | [ommysql](https://docs.rsyslog.com/doc/configuration/modules/ommysql.html) | Ação de saída para banco |
| Escape SQL | [Opções de templates rsyslog](https://docs.rsyslog.com/doc/reference/templates/templates-options.html) | `sql`, `stdsql` e modo SQL |
| rsyslog no Debian | [Pacote rsyslog-mysql](https://packages.debian.org/trixie/rsyslog-mysql) | Módulo e integração de configuração |
| Backup MariaDB | [mariadb-dump no Debian 13](https://manpages.debian.org/trixie/mariadb-client/mariadb-dump.1.en.html) | Dump transacional e limites de consistência |
| Atualizações automáticas | [Debian Wiki — UnattendedUpgrades](https://wiki.debian.org/UnattendedUpgrades) | Instalação e revisão da política |
| UCS | [Quick start UCS 5.2](https://docs.software-univention.de/quickstart/5.2/en/index.html) | Instalação e funções de domínio |
| UCS completo | [Manual UCS 5.2 em PDF](https://docs.software-univention.de/manual/5.2/en/manual.pdf) | DNS, funções e associação |

## Manuais da versão instalada

As páginas upstream podem descrever versões mais novas. Consulte também os manuais locais, instalando `man-db` quando necessário:

```bash
man apt
man sources.list
man interfaces
man nmcli
man systemd.network
man chmod
man usermod
man sshd_config
man sftp
man nft
man systemd.timer
man rsyslog.conf
man mariadb-dump
```

Um manual pode não estar disponível se o pacote correspondente não estiver instalado. Consulte `dpkg -L nome-do-pacote` para localizar documentação e exemplos.
