# 05 · Acesso remoto com SSH

[← Usuários](04-usuarios.md) · [README](../README.md) · [Firewall →](06-firewall.md)

## Conceitos e pré-requisitos

SSH cria um canal cifrado entre cliente e servidor. O cliente inicia a conexão; `sshd` atende no servidor, normalmente na porta TCP 22. A chave do **servidor** permite conferir sua identidade; a chave do **usuário** comprova quem está entrando. Não são a mesma chave.

Prepare IP fixo, uma conta com sudo e um cliente na rede LAB. Mantenha acesso ao console até terminar todos os testes.

## Instalar no servidor

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable --now ssh
systemctl status ssh --no-pager
sudo ss -lntp
sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

No **cliente**, conecte:

```bash
ssh aluno@192.168.50.10
```

Compare a impressão digital apresentada com a obtida no console. Aceitá-la grava a identidade em `known_hosts`. Se ela mudar posteriormente, investigue reinstalação, troca de chaves ou destino incorreto antes de remover o registro.

## Criar uma chave no cliente

```bash
ssh-keygen -t ed25519 -C 'aluno-laboratorio'
```

Escolha uma senha para proteger a chave privada e não sobrescreva uma chave existente sem compreender o impacto. O arquivo `.pub` é público; o outro é privado e deve permanecer no cliente.

Em um cliente Linux, envie a chave pública enquanto o login por senha ainda funciona:

```bash
ssh-copy-id aluno@192.168.50.10
ssh -o PreferredAuthentications=publickey -o PasswordAuthentication=no aluno@192.168.50.10
```

No Windows com OpenSSH, `ssh` e `ssh-keygen` podem estar disponíveis, mas `ssh-copy-id` normalmente não está. Copie o conteúdo de `id_ed25519.pub` e acrescente uma linha em `/home/aluno/.ssh/authorized_keys` no servidor, como o próprio aluno:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Confira que o diretório e o arquivo pertencem a `aluno`. Nunca envie a chave privada.

## Restringir autenticação depois de testar

> [!WARNING]
> Só desative senha depois que uma **segunda sessão** autenticar por chave. Mantenha a sessão atual e o console disponíveis para recuperação.

Copie a configuração atual. Confirme a diretiva `Include /etc/ssh/sshd_config.d/*.conf` no arquivo principal. Crie `/etc/ssh/sshd_config.d/00-laboratorio.conf` com:

```text
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

Valide e confira o resultado efetivo:

```bash
sudo /usr/sbin/sshd -t
sudo /usr/sbin/sshd -T | grep -E 'permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication'
sudo systemctl reload ssh
```

Em geral, OpenSSH usa o primeiro valor encontrado para cada opção; arquivos anteriores, configurações principais e blocos `Match` podem alterar o resultado. Para regras condicionais, confira também com `sshd -T -C user=aluno,addr=192.168.50.100,host=cliente`. Não presuma que um arquivo adicional sempre sobrescreve os demais. Fonte: [manual sshd_config](https://man.openbsd.org/sshd_config).

## Validação e recuperação

Abra outra conexão exigindo chave, execute `whoami` e `sudo -v`. Teste também uma conexão com `ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password aluno@192.168.50.10`: ela deve falhar após a restrição. Consulte `sudo journalctl -u ssh -n 50 --no-pager` para diagnóstico.

Se perder acesso, use o console, restaure a configuração anterior, execute `sshd -t` e recarregue o serviço. Uma chave perdida exige outro meio de acesso autorizado.

**Exercício:** descreva o que permanece protegido pela senha da chave privada e por que trocar a porta SSH não substitui autenticação e firewall.
