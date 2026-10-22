# Registro de atendimento — falha de DNS

[← Passo a passo e evidências](README.md) · [Todos os laboratórios](../README.md)

**Registro didático de um incidente provocado no laboratório.** Não representa atendimento profissional nem chamado aberto em uma ferramenta ITSM.

| Campo | Registro |
| --- | --- |
| Categoria | Redes / resolução de nomes |
| Equipamento | PC-FIN-01 — Windows 11 |
| Conta utilizada | jdl-lab-test\ana.souza |
| Escopo observado | Um cliente e a consulta do nome do servidor |
| Sintoma | Consulta DNS encerrada por tempo limite, com conectividade ICMP ao servidor preservada |
| Alteração que provocou o incidente | DNS preferencial do cliente alterado de 192.168.50.10 para 192.168.50.254 |
| Causa da falha simulada | Cliente consultando o endereço DNS incorreto .254, que não respondeu ao teste |
| Correção | Restaurar o DNS preferencial para 192.168.50.10 |
| Situação | Consulta do registro A recuperada; aviso inicial de timeout permanece |

## Investigação

1. Registrei identidade e configuração de rede com hostname, whoami e ipconfig /all.
2. Confirmei a resposta ao ping e a resolução do nome antes da alteração.
3. Configurei o DNS .254 no cliente para reproduzir a falha.
4. Comparei os testes: ping para .10 respondeu sem perdas; consulta ao .254 expirou.
5. Direcionei a mesma consulta ao DNS .10 e obtive o endereço esperado.
6. Restaurei .10 nas propriedades IPv4 e repeti a consulta usando o DNS configurado.

## Evidências principais

- [Falha: ping responde, consulta DNS expira](imagens/05-ping-funciona-dns-falha.png).
- [Isolamento: consulta direta ao servidor correto](imagens/06-consulta-direta-dns-correto.png).
- [Correção: consulta pelo DNS configurado volta a responder](imagens/07-validacao-apos-correcao.png).

## Validação e pendências

Após a correção, o ping teve quatro respostas e nenhuma perda; o nslookup retornou WIN-8AGKJPGKIM3.jdl-lab-test.local com IP 192.168.50.10.

O aviso inicial de timeout e a identificação UnKnown já existiam antes da simulação e permaneceram. A causa não foi comprovada. A diferença entre o sufixo DNS do cliente e o nome consultado também está registrada nas [observações do laboratório](README.md#observacoes). Não houve validação de aplicações ou autenticação.
