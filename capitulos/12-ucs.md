# 12 · Domínio com Univention Corporate Server

[← Segurança](11-seguranca.md) · [README](../README.md)

## Conceitos e objetivo

Um **domínio de identidade** reúne contas e políticas administradas centralmente. O usuário pode autenticar em diferentes máquinas sem que cada uma mantenha uma conta local independente. Isso é diferente de apenas configurar um sufixo DNS.

UCS é uma plataforma de servidores com diretório, console de administração e integração de identidade. Para serviços compatíveis com Active Directory, utiliza componentes como Samba. **LDAP** organiza dados de diretório; **Kerberos** participa da autenticação; **DNS** permite localizar servidores e serviços. Falhas em DNS e horário frequentemente impedem entrada no domínio.

O primeiro servidor de um domínio UCS recebe o papel de **Primary Directory Node**. Neste capítulo, vamos usar o UCS 5.2, seguindo o [guia oficial de início rápido](https://docs.software-univention.de/quickstart/5.2/en/index.html). Use a atualização de manutenção suportada no momento da instalação.

## Planejar uma VM separada

Não transforme `srv01` em UCS copiando repositórios ou instalando pacotes aleatórios. Use a imagem UCS em uma VM própria, na rede interna LAB, com recursos sugeridos no capítulo 00.

| Parâmetro | Escolha do laboratório |
| --- | --- |
| Nome completo do servidor | `ucs01.ad.example.test` |
| Domínio DNS | `ad.example.test` |
| Nome curto do domínio | `LAB` |
| IP fixo | `192.168.50.20/24` |
| Gateway | `192.168.50.1` |
| Encaminhamento DNS externo | Resolvedor alcançável, como o pfSense `.1` |
| DNS dos clientes de domínio | `192.168.50.20` |

`example.test` é um nome para testes, não um domínio público a ser publicado. Em produção, planeje um subdomínio de um domínio sob controle da organização. A escolha do nome de domínio é estrutural; não presuma que poderá ser alterada depois da instalação.

## Instalação e primeiro domínio

1. Obtenha a imagem pelo [site da Univention](https://www.univention.com/products/download/) e consulte os requisitos e notas da versão escolhida.
2. Conecte a VM somente à LAB, configure seu IP fixo e confira saída para repositórios pelo gateway.
3. No assistente de configuração de domínio, escolha criar um **novo domínio UCS**, com o primeiro servidor como Primary Directory Node. Não escolha associação ou tomada de um domínio real existente.
4. Informe nome completo, organização e credenciais administrativas exclusivas. Revise os nomes antes de confirmar.
5. Habilite o componente de serviços de domínio compatíveis com Active Directory oferecido pelo instalador ou App Center da versão usada, conforme o manual. Ter apenas um diretório UCS não comprova que o serviço AD necessário ao cliente está pronto.
6. Conclua o assistente e acesse a página HTTPS do UCS a partir do cliente LAB. Verifique a identidade do servidor e abra o **Univention Management Console (UMC)**.
7. Aplique atualizações pela ferramenta administrativa do UCS e acompanhe os diagnósticos. Não substitua os repositórios UCS pelos do Debian 13 deste guia.

As funções de domínio e o DNS necessário à associação são detalhados no [manual UCS 5.2](https://docs.software-univention.de/manual/5.2/en/manual.pdf).

## Configurar DNS dos clientes

Máquinas do domínio devem consultar o DNS do domínio, capaz de responder aos registros de serviço. No [Kea](07-dhcp.md), altere o valor da opção `domain-name-servers` para `192.168.50.20`, teste a configuração, reinicie o serviço e renove a concessão dos clientes.

Não anuncie um DNS público como alternativa indiferenciada: ele não conhece o diretório interno. A resolução externa deve ser encaminhada pelo DNS do domínio. Evite laço de encaminhamento: UCS não deve encaminhar para pfSense se o pfSense encaminhar todas as consultas de volta para UCS.

De um cliente Linux com `dnsutils`:

```bash
dig @192.168.50.20 ucs01.ad.example.test A
dig @192.168.50.20 _ldap._tcp.ad.example.test SRV
dig @192.168.50.20 _kerberos._tcp.ad.example.test SRV
getent hosts ucs01.ad.example.test
timedatectl
```

As consultas diretas testam o DNS do UCS; `getent` verifica a resolução realmente usada pelo cliente. Registros LDAP/Kerberos dependem da função de domínio configurada. Se faltarem, confira a instalação dos componentes antes de tentar associar máquinas.

## Criar conta e testar associação

1. No UMC, crie o grupo `TurmaLinux` e um usuário de teste `estudante.dominio` com os atributos necessários à função de domínio; adicione-o ao grupo.
2. Confira que o cliente usa `.20` como DNS e está com horário sincronizado.
3. Em uma VM Windows com edição que suporte associação a domínio, abra as opções de alteração do nome/domínio do computador, escolha domínio e informe `ad.example.test`.
4. Use uma conta autorizada a associar computadores, reinicie quando solicitado e teste login com a conta de estudante usando `LAB\estudante.dominio` ou o nome de login compatível configurado no domínio.
5. Confirme a conta com `whoami` no cliente e o objeto do computador no UMC. Não use a conta administrativa do domínio para o trabalho diário.

O cliente precisa alcançar os serviços de domínio, não somente HTTPS ou SSH. Na LAB, as VMs compartilham o segmento; se adicionar firewall ao UCS ou segmentação entre elas, consulte a matriz de portas e os serviços da versão instalada antes de criar regras. Não aplique ao UCS o firewall restritivo feito para `srv01`.

Para clientes Linux, use o método de associação explicitamente suportado para a distribuição e versão do cliente no manual UCS. Não copie comandos de associação de outra versão, pois integração de diretório, PAM e resolução de identidades precisa ser coerente.

## Validação, limites e exercício

- [ ] O nome do UCS e os registros SRV resolvem no cliente.
- [ ] Uma conta de domínio autentica no cliente associado.
- [ ] A conta é reconhecida com sua identidade de domínio.
- [ ] O objeto da máquina aparece no diretório e não há falhas pendentes nos diagnósticos.
- [ ] Existe plano de backup próprio para identidade, dados e recuperação do domínio.

**Exercício:** explique a diferença entre conta local e conta de domínio. Em um cliente de teste, configure temporariamente um DNS que não conhece o domínio, observe a falha de localização e depois restaure `.20`. Não use restauração arbitrária de snapshot de um controlador integrado a outros servidores como estratégia de recuperação do domínio.
