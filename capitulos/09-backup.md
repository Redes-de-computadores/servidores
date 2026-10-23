# 09 · Backup e agendamento

[← 08 · DHCP](08-dhcp.md) · [README](../README.md) · [Próximo: Logs e MariaDB →](10-logs-mariadb.md)

## Conceitos e objetivo

Backup é uma cópia recuperável. **RPO** representa quanto dado você admite perder em tempo; **RTO**, quanto tempo pode levar para recuperar o serviço. Uma cópia no mesmo disco ajuda a estudar, mas não protege da perda desse disco. Planeje cópias em outro equipamento e uma cópia isolada ou externa.

Pré-requisitos: diretório `/srv/empresa` do capítulo 04, acesso ao sudo e espaço livre. Aqui será feito um arquivo compactado diário; bases de dados terão procedimento próprio.

## Preparar e executar um script

Instale `gzip` e `tar` caso não estejam disponíveis. Crie o arquivo com `sudo nano /usr/local/sbin/backup-lab`:

```bash
#!/bin/bash
set -euo pipefail
umask 077

destino=/var/backups/lab
install -d -m 0700 "$destino"
exec 9>/run/lock/backup-lab.lock
flock -n 9 || exit 0
nome="lab-$(date +%Y%m%d-%H%M%S).tar.gz"
temporario=$(mktemp "$destino/.backup-XXXXXX")
trap 'rm -f -- "$temporario"' EXIT

tar -czf "$temporario" -C /srv laboratorio
gzip -t "$temporario"
mv -- "$temporario" "$destino/$nome"
printf 'Backup criado: %s\n' "$destino/$nome"
```

`umask 077` restringe novas cópias ao dono. `flock` evita duas execuções simultâneas. O arquivo temporário só recebe o nome definitivo após sucesso; o `trap` remove apenas o temporário desta execução. O pacote `util-linux` fornece `flock`.

```bash
sudo chmod 750 /usr/local/sbin/backup-lab
sudo /usr/local/sbin/backup-lab
sudo ls -lh /var/backups/lab
```

Este exemplo não congela arquivos enquanto mudam nem preserva todos os atributos avançados possíveis. Para aplicações em uso, planeje quiescência ou ferramentas específicas. Não use cópia direta de arquivos de MariaDB em execução como backup lógico consistente.

## Agendar com systemd timer

Crie `/etc/systemd/system/backup-lab.service`:

```ini
[Unit]
Description=Backup dos arquivos do laboratorio

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/backup-lab
```

Crie `/etc/systemd/system/backup-lab.timer`:

```ini
[Unit]
Description=Agendamento diario do backup do laboratorio

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
timedatectl
sudo systemctl daemon-reload
sudo systemctl start backup-lab.service
sudo systemctl enable --now backup-lab.timer
systemctl list-timers --all
sudo journalctl -u backup-lab.service -n 30 --no-pager
```

O horário segue o fuso da VM, verificado com `timedatectl`; configure `America/Sao_Paulo` se for a sua referência. `Persistent=true` permite recuperar uma ativação de calendário perdida enquanto o timer estava inativo, não reproduz cada execução perdida. Fonte: [systemd.timer](https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html).

<details>
<summary>Alternativa: agendar com cron</summary>

Se preferir cron, instale e ative o serviço `cron`, execute `sudo crontab -e` e acrescente:

```cron
0 2 * * * /usr/local/sbin/backup-lab
```

Os campos são minuto, hora, dia do mês, mês e dia da semana. A linha pertence ao crontab do usuário root e não leva uma coluna de usuário. Não use cron e timer simultaneamente para o mesmo agendamento.

</details>

## Restaurar antes de considerar concluído

Escolha um arquivo real listado anteriormente e substitua `ARQUIVO.tar.gz`. Extraia primeiro em um diretório de teste **vazio**, nunca sobre o serviço em funcionamento:

```bash
sudo tar -tzf /var/backups/lab/ARQUIVO.tar.gz
sudo mkdir -p /srv/restauracao-teste
sudo tar -xzf /var/backups/lab/ARQUIVO.tar.gz -C /srv/restauracao-teste
sudo diff -r /srv/laboratorio /srv/restauracao-teste/laboratorio
```

Sem diferenças de conteúdo, `diff` não imprime nada e retorna zero. Compare também donos e permissões com `ls -ln`. Se os originais mudaram depois da cópia, diferenças são esperadas; faça o primeiro teste com arquivos estáveis.

> [!IMPORTANT]
> O exemplo acumula arquivos. Monitore `df -h /var/backups` e defina retenção antes de deixá-lo ativo por muito tempo. Retenção exige decidir quais datas preservar e verificar uma cópia externa antes de excluir versões antigas.

**Exercício:** crie um arquivo, faça o backup, altere o original e recupere a versão antiga em outro diretório. Registre quanto tempo a recuperação levou e onde ficaria a cópia que sobreviveria à perda da VM.

---

[← Voltar para 08 · DHCP](08-dhcp.md) · [README](../README.md) · [Avançar para 10 · Logs e MariaDB →](10-logs-mariadb.md)
