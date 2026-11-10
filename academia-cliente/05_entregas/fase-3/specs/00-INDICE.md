# Fase 3 — CRM operacional, da qualificação à matrícula

Liberada pelo consultor em 28/10/2026. A fase 3 começa pelo que ficou da fase 2 (caixa de conversas, envio, boas-vindas e painel do dia) e depois leva o lead pelo funil inteiro na central:
- **Atendimento:** caixa por unidade, janela de 24 horas, SLA e K1, com números de teste.
- **Qualificação e orçamento:** ficha obrigatória, orçamento com as turmas e os preços do MatriculaWeb e próxima ação com responsável (K2).
- **Follow-up:** FUP 1 e FUP 2 automáticos, lembrete do terceiro retorno, envio sempre humano.
- **Reunião e negociação:** reunião com o diretor pausando o follow-up e negociação dentro da alçada.
- **Matrícula e 2027:** checklist antes de `Novo aluno` e retomada automática do Interesse 2027.
- **Painel gerencial:** funil da origem à matrícula para direção, gestores e marketing.
- **Histórico do Kommo:** importação pronta e testada com arquivo de teste, travada para dado real.

Tudo roda com massa de teste marcada `TESTE` no mesmo banco da operação. Lead real, envio real pela Meta e importação real do Kommo dependem da conta Meta aprovada e da política de LGPD e são ligados pelo consultor, fora das tasks. Nenhuma chamada grava no MatriculaWeb ou no Kommo nesta fase.

Em `04_fase-atual/` o executor (Maestro ou Claude Code) só marca o estado das tarefas, registra o resultado na linha `> ` e preenche o checklist de execução das SPECs. Tarefa, SPEC, critério, emenda ou id novo vão para o consultor (AGENTS.md).

| SPEC | Entrega | O que o cliente vê | Teste do cliente |
|---|---|---|---|
| [SPEC-3-001](spec-3-001-atendimento-whatsapp-em-teste.md) | Atendimento pelo WhatsApp em teste (o que ficou da fase 2) | Fila da unidade com cronômetro; boas-vindas; modelo com custo; painel do dia | Luiz, com o Hikari; vale também como aceite da fase 1 |
| [SPEC-3-002](spec-3-002-qualificacao-orcamento-proxima-acao.md) | Qualificação, orçamento e próxima ação | Ficha obrigatória; orçamento com os valores do MatriculaWeb; K2 | Luiz, com um atendente de cada unidade |
| [SPEC-3-003](spec-3-003-follow-up-depois-do-orcamento.md) | Follow-up depois do orçamento | Tarefas de FUP 1 e FUP 2; tipo de mensagem; lembrete com decisão | Luiz, com um atendente |
| [SPEC-3-004](spec-3-004-reuniao-e-negociacao.md) | Reunião com o diretor e negociação com alçada | Agenda do diretor; fila de negociação; decisão de volta ao atendente | Luiz como diretor e negociador de teste |
| [SPEC-3-005](spec-3-005-matricula-e-interesse-2027.md) | Matrícula com checklist e retomada do Interesse 2027 | O que falta para matricular; matrícula registrada; lead de 2027 voltando | Luiz, com um atendente |
| [SPEC-3-006](spec-3-006-painel-gerencial.md) | Painel gerencial da origem à matrícula | Funil com conversão; marketing só com números; auditoria de conversas | Luiz e quem cuida do marketing |
| [SPEC-3-007](spec-3-007-historico-do-kommo.md) | Importação do histórico do Kommo, travada para dado real | Mapas de etapas e motivos; relatório da importação de teste | Luiz, com quem administra o Kommo |

## Ordem e dependências

As SPECs seguem esta ordem: 3-001, 3-002, 3-003, 3-004, 3-005, 3-006 e 3-007.
- **3-001 primeiro:** a caixa e a janela de 24 horas são a base do tipo de mensagem do follow-up. Se a equipe já construiu essa parte depois de 19/10, as tasks fecham rodando as provas na versão atual, sem reconstruir.
- **3-002 antes da 3-003:** o follow-up começa no orçamento enviado.
- **3-003 antes da 3-004:** reunião e negociação usam a pausa da cadência.
- **3-005 depois da 3-004:** o valor final usa a condição aprovada na negociação.
- **3-006 depois da 3-005:** o painel conta as matrículas registradas.
- **3-007 por último:** é a única que depende de gate para ter efeito real, e não trava nada antes dela.

O executor constrói as tasks de cada SPEC em sequência e chama o teste do cliente quando todas as tasks daquela SPEC estiverem no ar.

## Dados da Academia que chegam nesta fase

Nenhum destes itens trava a construção. Cada um muda só a configuração quando chegar (detalhes no checklist da fase):
- **Número do SLA e régua de alerta:** o console troca os 10 minutos (SPEC-3-001).
- **Campos obrigatórios da ficha e tipos de próxima ação usados hoje no Kommo:** o console ajusta (SPEC-3-002).
- **Contagem dos prazos de follow-up e destino depois do terceiro retorno:** o console ajusta (SPEC-3-003).
- **Alçadas de desconto, diretores e negociadores:** o admin cadastra; até lá só existem faixas de teste (SPEC-3-004).
- **Itens do checklist, marco de conversão e data de retomada de 2027:** o console ajusta (SPEC-3-005).
- **Mapas do Kommo e exportação real:** a importação real só depois da LGPD aprovada e da chave nova (SPEC-3-007).

## Fora desta fase

- **Operação real:** lead real no canal, envio real e importação real, ligados pelo consultor depois da conta Meta aprovada e da política de LGPD.
- **Gravar no MatriculaWeb:** orçamento e matrícula não são enviados para lá; fica para decisão do consultor.
- **IA assistida (fase 4):** sugestão de resposta, resumo e classificação de perda, sempre com revisão humana.
- **Validação e virada (fase 5):** K1, K2 e K3 contra a linha de base e retirada do Kommo.
- **Liberação de fase:** só o consultor libera a fase 4 (AGENTS.md).
