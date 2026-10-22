# 04 · Usuários, grupos e permissões

[← Rede](03-rede.md) · [README](../README.md) · [SSH →](05-ssh.md)

## Conceitos e objetivo

Um usuário identifica uma pessoa ou serviço. Um grupo reúne usuários para compartilhar permissões. Cada arquivo tem dono, grupo e permissões para dono, grupo e outros. `r` permite leitura, `w` escrita e `x` execução; em diretórios, `x` permite atravessar o caminho e acessar suas entradas.

Pré-requisito: acesso ao sudo. Nesta prática, `aluno` é a conta administrativa, `estudante` é uma conta comum e `laboratorio` é um grupo de colaboração.

## Criar e conferir contas

```bash
sudo adduser estudante
sudo groupadd laboratorio
sudo usermod -aG laboratorio aluno
sudo usermod -aG laboratorio estudante
id estudante
getent group laboratorio
```

`adduser` solicita senha e cria a pasta pessoal. Em `usermod`, `-aG` **acrescenta** grupos suplementares; usar somente `-G` pode substituir a lista existente. Encerre e abra novamente as sessões dos usuários para receber os novos grupos.

## Compartilhar um diretório

```bash
sudo install -d -o root -g laboratorio -m 2770 /srv/laboratorio
ls -ld /srv/laboratorio
```

`2770` combina `rwx` para dono e grupo, nenhuma permissão para outros e o bit **setgid** (`2`). Em um diretório, setgid faz novos arquivos herdarem seu grupo. Ele não garante sozinho escrita de grupo nos arquivos: a máscara `umask` e as permissões solicitadas pelo programa continuam valendo.

Entre como estudante em uma nova sessão e teste:

```bash
id
umask 0007
touch /srv/laboratorio/anotacoes.txt
ls -l /srv/laboratorio/anotacoes.txt
```

Em outra sessão, como aluno, escreva no arquivo. Neste exemplo, `umask 0007` vale apenas para o shell atual e retira permissões de outros. A permissão final também depende do modo de criação do aplicativo.

| Modo | Uso didático |
| --- | --- |
| `600` | Arquivo privado: dono lê e escreve |
| `640` | Dono lê/escreve, grupo lê |
| `750` | Diretório acessível pelo dono e grupo |
| `2770` | Diretório colaborativo com herança de grupo |

> [!TIP]
> Ao receber “Permission denied”, confira `id`, `ls -ld` e as permissões dos diretórios superiores com `namei -l /srv/laboratorio/anotacoes.txt`. Aplicar `chmod 777` não explica nem resolve corretamente a necessidade de acesso.

## Administração e ciclo de vida

Conceda sudo somente a quem administra o sistema. Para bloquear a **senha** de uma conta de teste:

```bash
sudo passwd -l estudante
sudo passwd -S estudante
```

Isso não revoga automaticamente chaves SSH, sessões abertas ou outros mecanismos de autenticação. Para desativar uma conta por completo, revise todos os acessos, sessões e tarefas associados. Para reabilitar a senha bloqueada durante o exercício, use `sudo passwd -u estudante`.

## Validação e exercício

- [ ] Ambos os integrantes conseguem trabalhar no arquivo compartilhado.
- [ ] Um usuário fora do grupo não consegue listar o diretório.
- [ ] A conta comum não recebeu sudo inadvertidamente.

**Exercício:** explique a diferença entre dono e grupo e por que um arquivo legível pode continuar inacessível quando um diretório superior não permite travessia. Consulte `man chmod`, `man umask` e `man usermod` na versão instalada.
