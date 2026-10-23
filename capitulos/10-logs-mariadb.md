# 10 · Logs e MariaDB

[← 09 · Backup](09-backup.md) · [README](../README.md) · [Próximo: Segurança →](11-seguranca.md)

## Conceitos e objetivo

Um **log** registra eventos. O journal do systemd permite consultá-los por serviço, data ou prioridade. `rsyslog` recebe e encaminha mensagens syslog. MariaDB é um sistema de banco de dados relacional: organiza dados em tabelas consultadas por SQL.

São três componentes distintos. Nem todo evento do journal será encaminhado automaticamente ao rsyslog em qualquer instalação. Vamos provar o caminho de uma mensagem de teste local até uma tabela, sem expor o banco à rede.

Pré-requisitos: sudo, espaço em disco e acesso aos repositórios.

## Começar pelo journal

```bash
sudo journalctl -b -p warning --no-pager
sudo journalctl -u ssh --since '1 hour ago' --no-pager
journalctl --disk-usage
logger -t guia-linux 'Teste inicial de registro'
sudo journalctl -t guia-linux -n 10 --no-pager
```

`-b` limita ao boot atual, `-u` ao serviço e `-t` ao identificador. Se o journal não for persistente, os registros podem desaparecer após reinicialização. Confira `man journald.conf` antes de configurar retenção e persistência.

## Instalar o banco e o módulo de saída

```bash
sudo apt update
sudo apt install mariadb-server rsyslog rsyslog-mysql
sudo systemctl enable --now mariadb rsyslog
sudo mariadb
```

Se a instalação perguntar sobre configuração automática do banco pelo `dbconfig-common`, **recuse para este exercício**, pois criaremos banco e usuário manualmente. Confira arquivos já existentes em `/etc/rsyslog.d/` para não carregar `ommysql` duas vezes nem manter uma ação antiga apontando para outro banco.

Em uma instalação Debian padrão, `sudo mariadb` costuma autenticar o administrador pelo socket local. Se isso tiver sido alterado, use o método administrativo autorizado; não remova a autenticação para prosseguir.

## Criar uma tabela didática e um usuário restrito

No prompt MariaDB, substitua `SUBSTITUA_POR_SENHA_EXCLUSIVA` por uma senha forte. Se ela contiver aspas ou barras, aplique o escape SQL correto; não copie segredos para o repositório.

```sql
CREATE DATABASE LogsLab CHARACTER SET utf8mb4;
USE LogsLab;
CREATE TABLE Eventos (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    recebido_em TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    host VARCHAR(255) NOT NULL,
    programa VARCHAR(255) NOT NULL,
    mensagem TEXT NOT NULL
) ENGINE=InnoDB;
CREATE USER 'logwriter'@'localhost' IDENTIFIED BY 'SUBSTITUA_POR_SENHA_EXCLUSIVA';
GRANT INSERT ON LogsLab.Eventos TO 'logwriter'@'localhost';
SHOW GRANTS FOR 'logwriter'@'localhost';
SELECT @@GLOBAL.sql_mode;
EXIT;
```

Não são necessários privilégios globais nem `FLUSH PRIVILEGES` após esses comandos de gerenciamento de contas. O usuário da aplicação pode inserir, mas não apagar tabelas ou ler todos os dados. Consultas administrativas serão feitas por outro acesso.

## Configurar o rsyslog

Crie um arquivo restrito e edite-o:

```bash
sudo install -m 600 /dev/null /etc/rsyslog.d/30-lab-mariadb.conf
sudo nano /etc/rsyslog.d/30-lab-mariadb.conf
```

Use a mesma senha cadastrada no banco:

```text
module(load="ommysql")

template(name="LabInsert" type="string" option.sql="on"
  string="INSERT INTO Eventos (host, programa, mensagem) VALUES ('%hostname%', '%programname%', '%msg%')")

if ($programname == "guia-linux") then {
    action(type="ommysql"
           server="localhost"
           db="LogsLab"
           uid="logwriter"
           pwd="SUBSTITUA_POR_SENHA_EXCLUSIVA"
           template="LabInsert")
}
```

Somente mensagens com identificador `guia-linux` entram nessa ação. Não há `stop`, portanto as demais regras locais continuam sendo processadas. O filtro limita o volume do exercício e evita enviar todos os eventos ao banco sem planejamento.

`option.sql="on"` faz escape compatível com MariaDB/MySQL. Confira que `NO_BACKSLASH_ESCAPES` **não** está habilitado no modo SQL usado pela conexão; se estiver, adapte o template à opção `stdsql` conforme a documentação. Não remova o escape. Fontes: [módulo ommysql](https://docs.rsyslog.com/doc/configuration/modules/ommysql.html) e [opções de templates](https://docs.rsyslog.com/doc/reference/templates/templates-options.html).

```bash
sudo rsyslogd -N1
sudo systemctl restart rsyslog
logger -t guia-linux "Validacao SQL: texto com 'aspas'"
sudo journalctl -u rsyslog -n 50 --no-pager
sudo mariadb -e "SELECT id, recebido_em, host, programa, mensagem FROM LogsLab.Eventos ORDER BY id DESC LIMIT 10;"
```

O resultado deve conter a mensagem com aspas preservadas. Uma configuração sintaticamente válida não comprova conexão com o banco: a consulta é a evidência final.

Se não houver linha, confira identificador, recebimento local pelo rsyslog, carregamento do módulo, usuário/senha, modo SQL e permissões. Verifique também `systemctl status mariadb rsyslog` e os respectivos journals. Não abra TCP 3306 para corrigir uma conexão local por socket. Confira com `sudo ss -lntp` se o banco está restrito a loopback, quando há listener TCP.

## Backup do banco e limites do exercício

Use `mariadb-dump`, não uma cópia do diretório do banco em execução. Exemplo para esta tabela InnoDB:

```bash
sudo sh -c 'umask 077; mariadb-dump --single-transaction LogsLab > /var/backups/LogsLab.sql'
sudo ls -lh /var/backups/LogsLab.sql
```

O nome fixo sobrescreve a cópia anterior: faça isso apenas para o teste ou adote versionamento e retenção. `--single-transaction` não oferece a mesma garantia para tabelas não transacionais e exige evitar alterações estruturais durante a cópia. Confira `man mariadb-dump` da versão instalada e o [manual do cliente no Debian 13](https://manpages.debian.org/trixie/mariadb-client/mariadb-dump.1.en.html).

Para validar a restauração sem tocar na tabela original:

```bash
sudo mariadb -e 'CREATE DATABASE LogsLabRestauracao CHARACTER SET utf8mb4;'
sudo sh -c 'mariadb LogsLabRestauracao < /var/backups/LogsLab.sql'
sudo mariadb -e 'SELECT COUNT(*) FROM LogsLabRestauracao.Eventos;'
```

Use um banco de restauração ainda inexistente; não repita o `CREATE DATABASE` sobre um banco em uso. Compare o total e uma mensagem conhecida. O dump desse banco não inclui automaticamente contas e privilégios globais.

Este pipeline didático não oferece fila persistente, garantia de entrega ou política de retenção. Antes de centralizar logs reais, planeje armazenamento, índices, filas, comportamento em falhas, transporte protegido, acesso e prazo de retenção. Logs podem conter dados pessoais e credenciais expostas por aplicações.

**Exercício:** pare o banco brevemente, envie uma mensagem e observe o rsyslog. Reinicie o banco e registre o que aconteceu, sem pressupor que mensagens foram preservadas.

---

[← Voltar para 09 · Backup](09-backup.md) · [README](../README.md) · [Avançar para 11 · Segurança →](11-seguranca.md)
