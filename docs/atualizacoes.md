# Origem, decisões e atualizações

[← README](../README.md) · [Referências](referencias.md)

## O que foi possível consultar

A base local é [Servidores Linux.html](../Servidores%20Linux.html), uma página salva do Google Sites. Ela contém o índice de assuntos, orientações curtas sobre pfSense e sete capturas na pasta `Servidores Linux_files`.

Os capítulos externos são links de navegação, não conteúdo incorporado. Tentativas de leitura de páginas sobre DHCP, logs/MariaDB, FTP e UCS pela ferramenta web não obtiveram seu conteúdo. Portanto, este guia **não afirma ter revisado os procedimentos completos de 2024**. Ele cobre os temas do índice com procedimentos novos, fundamentados nas referências atuais consultadas. A data de 2024 foi informada pelo solicitante, não confirmada como data editorial do HTML.

## Mapa entre o índice antigo e o guia

| Entrada do material original | Destino |
| --- | --- |
| 00 — Instalação — Debian 10; Boot USB; Interface Gráfica | [01 — Instalação](../capitulos/01-instalacao.md) |
| 01 — Configurações de rede | [03 — Rede](../capitulos/03-rede.md) |
| 02 — Gerenciamento de pacotes | [02 — Pacotes](../capitulos/02-pacotes.md) |
| 03 — Acesso Remoto SSH | [05 — SSH](../capitulos/05-ssh.md) |
| 04 — Usuários e Grupos | [04 — Usuários](../capitulos/04-usuarios.md) |
| 6 — DHCP | [07 — DHCP com Kea](../capitulos/07-dhcp.md) |
| 10 — Firewall; PfSense; General Setup; NFT | [06 — Firewall](../capitulos/06-firewall.md) |
| Backup e Agendamento | [08 — Backup](../capitulos/08-backup.md) |
| 15 — FTP | [09 — SFTP e FTP](../capitulos/09-arquivos.md) |
| 11 — B — Registro de Logs — MariaDB | [10 — Logs e MariaDB](../capitulos/10-logs-mariadb.md) |
| 19 — Segurança | [11 — Segurança](../capitulos/11-seguranca.md) |
| 23 — UCS DC | [12 — UCS](../capitulos/12-ucs.md) |

A numeração foi reorganizada para ensino progressivo. O capítulo 00, as validações e os exercícios são acréscimos didáticos.

## Mudanças de referência técnica

| Tema | Decisão | Motivo ou limite |
| --- | --- | --- |
| Debian 10 | Basear instalações novas em Debian 13 | A linha 10 está arquivada e fora do LTS público; não é uma migração automática |
| Rede | Identificar ifupdown, NetworkManager ou networkd antes de editar | O gerenciador depende da instalação; não presumir um arquivo universal |
| Repositórios | Reconhecer `.sources` e `.list` | Evitar duplicidades e receitas antigas de confiança global |
| DHCP | Usar Kea | Fim de vida do ISC DHCP no projeto de origem; arquivos de configuração distintos |
| FTP | Fazer a prática principal com SFTP | Reutilizar canal SSH cifrado; FTPS e SFTP são protocolos diferentes |
| Firewall | Usar nftables no Debian e pfSense em VM separada | Distinguir proteção do host de proteção entre redes |
| pfSense IPv6 | Corrigir o efeito de `Allow IPv6` | Filtragem não equivale a desativar a pilha |
| pfSense ARP | Não recomendar supressão indiscriminada | Ocultar aviso não corrige a causa |
| Proteções do painel pfSense | Manter proteções e tratar exceções específicas | Capturas antigas desabilitam verificações que merecem ser preservadas |
| Agendamento | Mostrar timer systemd e cron como alternativas | Evitar duas execuções do mesmo agendamento |
| Logs em banco | Tabela e usuário próprios de laboratório | Não presumir conteúdo do procedimento antigo nem acesso administrativo pela aplicação |
| UCS | Usar documentação 5.2 e explicar Primary Directory Node | Atualizar terminologia e separar o sistema de identidade do servidor Debian |

As evidências oficiais estão ligadas nos capítulos e em [referências](referencias.md). A escolha de uma topologia, nomes, intervalos e recursos das VMs é uma decisão didática deste guia.

## Estado de validação

- Revisão documental de versões e componentes: realizada em **22/10/2026**.
- Revisão estrutural de Markdown, navegação e arquivos locais: realizada durante a elaboração.
- Instalação e testes de serviços em VMs: **não executados neste ambiente**.
- Renderização dos diagramas no GitHub: preparada com sintaxe Mermaid; requer visualização na plataforma para confirmação visual final.

Antes de aplicar em produção, reproduza os testes de cada capítulo em ambiente equivalente ao real. Uma atualização de versão pode mudar nomes, menus, padrões e permissões; confira as páginas oficiais e os manuais dos pacotes efetivamente instalados.
