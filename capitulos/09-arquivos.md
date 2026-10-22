# 09 · Transferência de arquivos: SFTP e FTP

[← Backup](08-backup.md) · [README](../README.md) · [Logs e MariaDB →](10-logs-mariadb.md)

## Entender antes de escolher

| Protocolo | Proteção de transporte | Portas e operação |
| --- | --- | --- |
| FTP | Sem cifragem no modo tradicional | TCP 21 e conexões adicionais para dados |
| FTPS | FTP com TLS | Controle e dados; requer certificado e planejamento de portas |
| SFTP | Canal SSH cifrado | Normalmente TCP 22; protocolo distinto de FTP |

O índice original inclui FTP. Para a prática principal, usaremos **SFTP**, aproveitando SSH já configurado. Não é necessário instalar um servidor FTP ou abrir outra porta. Uma aplicação que exige FTP não passa a funcionar com SFTP apenas por mudar a porta: ambos os lados precisam suportar o protocolo escolhido.

## Pré-requisitos

Conexão SSH por chave funcionando do cliente para `aluno@192.168.50.10`. A conta terá suas permissões Linux normais; este exercício não cria um serviço de upload isolado nem um confinamento por `chroot`.

## Transferir e conferir

No cliente Linux, prepare um arquivo conhecido:

```bash
printf 'Primeira transferencia do laboratorio\n' > anotacoes.txt
sha256sum anotacoes.txt
sftp aluno@192.168.50.10
```

No prompt `sftp>`, digite apenas os comandos abaixo:

```text
pwd
lpwd
mkdir envio-lab
cd envio-lab
put anotacoes.txt
ls -l
get anotacoes.txt copia-anotacoes.txt
bye
```

`pwd` mostra o diretório remoto; `lpwd`, o local. `put` envia; `get` baixa. Se `envio-lab` já existir, entre nele sem recriá-lo.

De volta ao terminal do cliente:

```bash
sha256sum anotacoes.txt copia-anotacoes.txt
ssh aluno@192.168.50.10 'sha256sum ~/envio-lab/anotacoes.txt'
```

Os três hashes devem coincidir. No Windows, use `Get-FileHash` para calcular SHA-256 de arquivos locais. Em um cliente gráfico como WinSCP ou FileZilla, selecione explicitamente **SFTP**, usuário, IP e a chave apropriada; confira a identidade do servidor como no primeiro acesso SSH.

## Se precisar estudar FTP legado

Faça o exercício em uma VM separada e rede interna sem dados reais:

1. Instale `vsftpd` pelo repositório Debian e salve a configuração anterior.
2. Leia `man vsftpd.conf` da versão instalada e planeje acesso autenticado, proibição de anonimato, diretório de escrita e faixa de portas passivas.
3. Antes de transportar credenciais ou dados reais, configure TLS, certificado verificável e exigência de cifragem tanto no controle quanto nos dados.
4. Configure cliente, firewall do servidor e NAT para as portas planejadas; testar somente TCP 21 não comprova transferência.
5. Confirme que o cliente rejeita transporte sem TLS e certificados inválidos. Ao terminar, pare o serviço que não será utilizado.

Não há um arquivo FTPS universal seguro para qualquer rede: certificados, modo passivo e endereços anunciados dependem da topologia. A prática funcional deste capítulo é SFTP. Para consulta de comportamento, use os manuais locais `man sftp`, `man ssh` e `man vsftpd.conf`.

## Validação e exercício

- [ ] Upload e download funcionam e os hashes coincidem.
- [ ] Não foi necessário liberar TCP 21 para usar SFTP.
- [ ] A conta não escreve em diretórios que não lhe foram autorizados.

**Exercício:** tente enviar para um diretório de teste sem permissão de escrita, registre o erro e explique por que cifragem e autorização são problemas diferentes.
