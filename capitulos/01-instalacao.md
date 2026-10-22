# 01 · Instalação do Debian

[← Guia Completo com Imagens (Item 00)](00-instalacao.md) · [README](../README.md) · [Pacotes →](02-pacotes.md)

> [!TIP]
> Para o guia visual completo de instalação passo a passo do Debian 13 com capturas de tela e explicações de cada tela do instalador, consulte o **[Item 00 · Instalação do Debian 13 no Laboratório](00-instalacao.md)**.

## Objetivo e pré-requisitos

Instalar o sistema que será administrado nos próximos capítulos. Prepare a VM e leia o [planejamento de rede](00-laboratorio.md). Vamos usar o **Debian 13 (Trixie)**. Ao baixar a imagem, confira a versão e a arquitetura para acompanhar os exemplos com a mesma base.

## Obter e verificar a imagem

1. Baixe a imagem de instalação pela [página oficial do Debian](https://www.debian.org/distrib/). Para PCs Intel/AMD de 64 bits, escolha `amd64`.
2. Baixe também os arquivos de somas e assinaturas disponibilizados junto à imagem.
3. Calcule o SHA-256 do arquivo baixado. No Windows, substitua o caminho abaixo pelo arquivo real:

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath 'C:\Downloads\debian-netinst.iso'
```

Compare o resultado inteiro com a linha correspondente em `SHA256SUMS`. Isso detecta corrupção; a autenticidade depende também de verificar a assinatura da lista conforme as [instruções de verificação do Debian](https://www.debian.org/CD/verify).

Na VM, conecte a ISO ao leitor virtual. Para um computador físico, grave a ISO em um pendrive com uma ferramenta de gravação de imagem e selecione esse dispositivo no menu de boot UEFI.

> [!WARNING]
> Gravar uma imagem no pendrive apaga seu conteúdo. O particionamento também pode apagar discos. Confirme o dispositivo e use apenas o disco virtual reservado ao laboratório.

## Instalação guiada

1. Inicie a VM pela ISO e escolha a instalação gráfica ou textual.
2. Selecione idioma, teclado e fuso horário apropriados.
3. Use temporariamente DHCP na interface NAT, se ainda não houver gateway do laboratório.
4. Defina o nome `srv01`. Deixe o domínio vazio nesta instalação independente; o domínio UCS terá planejamento próprio.
5. Crie o usuário `aluno`. No instalador Debian, deixar a senha de root vazia desativa seu login por senha e permite ao primeiro usuário usar `sudo`. Se optar por uma senha de root, configure o sudo posteriormente.
6. Para a VM descartável, selecione particionamento guiado usando todo o **disco virtual correto**. Confirme a proposta antes de gravar.
7. Configure o espelho de pacotes. Na seleção de programas, mantenha utilitários padrão; SSH pode ser instalado aqui ou no capítulo 05. Para o servidor, não selecione ambiente gráfico.
8. Conclua a instalação do carregador de boot conforme o instalador, remova a ISO virtual e reinicie.

Veja as opções de firmware, discos e inicialização no [manual de instalação do Debian 13](https://www.debian.org/releases/trixie/amd64/index.en.html).

## Primeiro acesso

Entre como `aluno` e confira:

```bash
cat /etc/os-release
hostnamectl
lsblk -f
ip -br address
sudo -v
```

`lsblk` mostra discos e partições; `sudo -v` valida sua autorização administrativa. Se sudo não estiver disponível, entre no console como root com `su -` e execute:

```bash
apt update
apt install sudo
usermod -aG sudo aluno
exit
```

Encerre a sessão do aluno e entre novamente para receber o grupo. Esses comandos pressupõem que uma senha de root foi configurada; se você não tem acesso administrativo, use o método de recuperação documentado pela distribuição antes de prosseguir.

## Interface gráfica opcional

Um servidor pode ser administrado por terminal. Para estudar ferramentas gráficas, prefira a VM cliente, pois a interface consome memória e instala mais componentes.

```bash
sudo apt update
sudo apt install task-xfce-desktop
sudo reboot
```

O comando instala um conjunto de programas e poderá solicitar a escolha de um gerenciador de login. Após instalar uma interface gráfica, confira novamente quem gerencia a rede no [capítulo 03](03-rede.md).

## Validação e exercício

- [ ] Reinicie sem a ISO e confirme que o sistema inicia pelo disco virtual.
- [ ] Confirme Debian 13 em `/etc/os-release` e autorização no sudo.
- [ ] Anote a interface de rede real e crie um snapshot da instalação limpa.

**Exercício:** explique por que uma ISO é montada como mídia de instalação e por que isso não significa que o sistema já foi instalado no disco.
