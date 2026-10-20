# Laboratório de Suporte N1

Prática de suporte de TI em máquinas virtuais, com foco em Active Directory, contas de usuário, permissões de acesso, compartilhamentos e políticas de grupo.

**Autor:** [Jardel](https://github.com/jardelluizoficial) · **Realização:** outubro de 2026

## Projeto: Active Directory e Windows

Simulei uma empresa com os setores Financeiro e RH. Executei os exercícios com orientação passo a passo, alternando entre a administração no Windows Server e a experiência do usuário no Windows 11. Registrei as configurações e os resultados em capturas de tela.

O objetivo foi praticar atividades relacionadas ao suporte N1 e entender como validar o atendimento: confirmar a identidade da sessão, testar acessos e verificar o resultado de uma correção.

## Ambiente

| Máquina | Sistema | Função | Endereço IPv4 |
|---|---|---|---|
| WIN-8AGKJPGKIM3 | Windows Server 2022 Standard Evaluation | Controlador de domínio, DNS e compartilhamento de arquivos | 192.168.50.10 |
| PC-FIN-01 | Windows 11 Pro | Estação de trabalho dos usuários de teste | 192.168.50.20 |

- **Virtualização:** Oracle VirtualBox.
- **Rede:** rede interna `intnet`, com máscara `255.255.255.0` e sem gateway configurado.
- **Domínio:** `jdl-lab-test.local`.
- **DNS da estação:** `192.168.50.10`.
- **Compartilhamento:** `\\192.168.50.10\Financeiro`.
- **Pasta no servidor:** `C:\Compartilhamentos\Financeiro`.

Parti de uma VM de servidor com AD DS, DNS e domínio já configurados. Este registro cobre a configuração de rede usada no laboratório, o ingresso da estação no domínio e os exercícios de administração e suporte descritos abaixo. Os usuários e os setores são fictícios.

## Exercícios realizados

### 1. Organização de usuários e acesso ao domínio

Organizei as contas em unidades organizacionais (OUs) e usei a estação PC-FIN-01 para testar o acesso ao domínio.

| Usuário | Conta | OU | Participação no GG_Financeiro |
|---|---|---|---|
| Ana Souza | ana.souza | Financeiro | Membro |
| Bruno Lima | bruno.lima | RH | Não é membro |

Configurei a troca da senha temporária no primeiro logon e validei a sessão com `whoami` e `hostname`.

<details>
<summary>Ver evidências de cadastro e login</summary>

**Ana na OU Financeiro**

![Ana Souza na OU Financeiro](active-directory/imagens/01-usuario-ana-ou-financeiro.png)

**Troca de senha no primeiro acesso**

![Troca de senha no primeiro logon](active-directory/imagens/02-troca-senha-primeiro-logon.png)

**Validação da conta de domínio e da estação**

![Login de Ana no domínio](active-directory/imagens/03-login-ana-dominio.png)

**Cadastro de Bruno, do RH**

![Cadastro do usuário Bruno Lima](active-directory/imagens/13-criacao-usuario-bruno-rh.png)

</details>

### 2. Redefinição de senha

Simulei o chamado **“Esqueci minha senha”** usando a conta da Ana.

1. Localizei a conta no Active Directory Users and Computers.
2. Defini uma nova senha temporária e exigi alteração no próximo logon.
3. Encerrei a sessão da Ana no Windows 11 e entrei com a senha temporária.
4. Alterei a senha e confirmei o acesso com `whoami` e `hostname`.

**Resultado:** acesso validado após a redefinição. Em um atendimento real, a confirmação da identidade e a entrega da senha temporária devem seguir o procedimento da organização.

<details>
<summary>Ver evidências da redefinição e da validação</summary>

![Redefinição da senha no AD](active-directory/imagens/04-redefinicao-senha-ad.png)

![Confirmação da redefinição](active-directory/imagens/05-confirmacao-redefinicao-senha.png)

![Novo logon após a redefinição](active-directory/imagens/06-novo-logon-apos-redefinicao.png)

![Validação do acesso após a redefinição](active-directory/imagens/07-validacao-login-apos-redefinicao.png)

</details>

### 3. Grupo de segurança e permissões de arquivos

Criei o grupo global de segurança `GG_Financeiro`, adicionei a Ana e configurei o compartilhamento Financeiro.

| Camada | Configuração para GG_Financeiro |
|---|---|
| Compartilhamento | Change e Read |
| NTFS | Modify |

Removi Everyone da lista de permissões do compartilhamento. Preservei as entradas administrativas e do sistema nas permissões NTFS. A permissão Modify permite criar, alterar e excluir arquivos.

**Validação positiva:** Ana acessou o compartilhamento e criou o arquivo `teste-ana.txt`.

**Validação negativa:** Bruno, fora do GG_Financeiro, recebeu acesso negado ao tentar abrir o mesmo compartilhamento pela rede.

<details>
<summary>Ver evidências do grupo, das permissões e dos testes</summary>

![Criação do grupo Financeiro](active-directory/imagens/08-criacao-grupo-financeiro.png)

![Ana como membro do grupo](active-directory/imagens/09-ana-membro-grupo-financeiro.png)

![Permissões do compartilhamento](active-directory/imagens/10-permissoes-compartilhamento.png)

![Permissões NTFS](active-directory/imagens/11-permissoes-ntfs.png)

![Criação de arquivo por Ana no compartilhamento](active-directory/imagens/12-teste-criacao-arquivo-compartilhado.png)

![Acesso negado para Bruno no Financeiro](active-directory/imagens/14-acesso-negado-bruno-financeiro.png)

</details>

### 4. Mapeamento automático por GPO

Criei a GPO `GPO_Financeiro_Unidade_F`, vinculada à OU Financeiro. Em **User Configuration → Preferences → Windows Settings → Drive Maps**, configurei:

| Opção | Valor |
|---|---|
| Action | Update |
| Location | `\\192.168.50.10\Financeiro` |
| Reconnect | Habilitado |
| Label as | Financeiro |
| Drive Letter | F: |

Na sessão da Ana, atualizei as políticas com `gpupdate /target:user /force` e consultei o resultado com `gpresult /r /scope:user`.

**Resultado:** GPO listada como aplicada, unidade F: disponível e acesso ao arquivo de teste. A GPO configura o mapeamento; as permissões do compartilhamento e NTFS controlam o acesso aos arquivos.

<details>
<summary>Ver evidências da configuração e da aplicação da GPO</summary>

![Configuração da GPO de mapeamento](active-directory/imagens/15-configuracao-gpo-unidade-f.png)

![Unidade F mapeada](active-directory/imagens/16-unidade-f-mapeada.png)

![GPO aplicada à sessão de Ana](active-directory/imagens/17-gpo-aplicada-ana.png)

![Confirmação da unidade F](active-directory/imagens/18-confirmacao-unidade-f.png)

</details>

## Incidente simulado: unidade de rede ausente

**Relato simulado:** “Minha unidade Financeiro sumiu do computador.”

A falha foi provocada intencionalmente na sessão da Ana, removendo o mapeamento com `net use F: /delete`. Essa ação desconecta a unidade de rede e mantém os arquivos no servidor.

| Etapa | Ação e resultado |
|---|---|
| Reprodução | Removi o mapeamento e confirmei a ausência da unidade F: em Este Computador. |
| Diagnóstico | Abri `\\192.168.50.10\Financeiro` diretamente e acessei o arquivo de teste. O compartilhamento continuava acessível nessa sessão. |
| Correção | Executei `gpupdate /target:user /force` para reaplicar a configuração do usuário. |
| Validação | Confirmei o retorno da unidade F: e abri o arquivo por ela. |

**Causa neste exercício:** remoção manual do mapeamento. Em um chamado real, o mesmo sintoma exigiria investigação; esta simulação não demonstra todas as causas possíveis.

<details>
<summary>Ver evidências do incidente, diagnóstico e recuperação</summary>

![Incidente: unidade F ausente](active-directory/imagens/19-incidente-unidade-f-ausente.png)

![Diagnóstico: acesso direto ao compartilhamento](active-directory/imagens/20-diagnostico-acesso-direto.png)

![Resolução: unidade F restaurada](active-directory/imagens/21-resolucao-unidade-f-restaurada.png)

</details>

## Comandos praticados

| Comando | Finalidade |
|---|---|
| `ipconfig /all` | Consultar a configuração de rede, incluindo IPv4 e DNS. |
| `ping 192.168.50.10` | Testar respostas ICMP do servidor; isso não valida sozinho todos os serviços. |
| `nslookup jdl-lab-test.local` | Consultar a resolução DNS do domínio. |
| `whoami` | Identificar a conta da sessão atual. |
| `hostname` | Identificar o computador. |
| `gpupdate /target:user /force` | Reaplicar as políticas do usuário. |
| `gpresult /r /scope:user` | Consultar o resumo das políticas aplicadas ao usuário. |
| `net use F: /delete` | Remover o mapeamento F: usado na simulação. |

## O que aprendi

- Diferenciar conta local e conta de domínio e conferir a sessão usada em um teste.
- Usar OUs para organizar objetos e vincular políticas, e grupos de segurança para administrar permissões.
- Validar uma redefinição de senha com um novo acesso do usuário.
- Considerar as permissões de compartilhamento e NTFS no acesso pela rede.
- Testar acesso autorizado e acesso negado com contas diferentes.
- Diferenciar a aplicação da GPO do funcionamento efetivo da unidade mapeada.
- Registrar o sintoma, os testes, a correção e a validação de um incidente.

## Referências técnicas

- [Microsoft Learn — Gerenciar contas de usuário no Active Directory](https://learn.microsoft.com/pt-br/windows-server/identity/ad-ds/manage-user-accounts-in-windows-server)
- [Microsoft Learn — Compartilhamento e permissões NTFS](https://learn.microsoft.com/pt-br/iis/web-hosting/configuring-servers-in-the-windows-web-platform/configuring-share-and-ntfs-permissions)
- [Microsoft Learn — Preferências de mapeamento de unidades](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn789188%28v%3Dws.11%29)
- [Microsoft Learn — gpupdate](https://learn.microsoft.com/pt-br/windows-server/administration/windows-commands/gpupdate)
- [Microsoft Learn — gpresult](https://learn.microsoft.com/pt-br/windows-server/administration/windows-commands/gpresult)
- [Microsoft Learn — net use](https://learn.microsoft.com/en-gb/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/gg651155%28v%3Dws.11%29)
