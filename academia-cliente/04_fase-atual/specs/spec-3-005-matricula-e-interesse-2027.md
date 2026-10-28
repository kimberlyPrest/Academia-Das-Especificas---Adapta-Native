# SPEC-3-005 — Matrícula com checklist e retomada do Interesse 2027

**Fase:** 3  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** fase 3 (matrícula bloqueada sem checklist; Interesse 2027 com retomada em novembro); fluxos F7 e F8; RN-10, RN-12; AG-07 e AG-08; K3 (marco de conversão, GATE-12); CL-001, CL-009, CL-010  
**Degrau da solução:** reuso + construção mínima — reusa o checklist de matrícula e o bloqueio de `Novo aluno` da fase 1 (CA-1-15) e o orçamento da SPEC-3-002; acrescenta o registro da matrícula, `future_interests` e uma rotina diária no Supabase Cron.

## Contexto e decisões fechadas

- **Estado atual:** a fase 1 já recusa `Novo aluno` sem checklist, e o console tem os itens do checklist (módulo 9). Não há registro do que foi contratado nem do valor final. O lead de `Interesse 2027` fica parado, sem data de retomada; a retomada de novembro é lembrada de cabeça.
- **Estado desejado:** o atendente confere item por item, a central diz o que falta, e a matrícula fica com valor final, turma e o que foi contratado. O lead de 2027 guarda a turma-alvo e a data de retomada e volta sozinho para a fila nessa data.
- **Decisões já fechadas:**
  - Itens obrigatórios do checklist vêm do console; o padrão é o do Manual de Boas Práticas: dados do aluno, dados do responsável financeiro, documentos conferidos, cursos e específicas confirmados e valor final confirmado.
  - Documento não entra no sistema: o item só registra "conferido", por quem e quando. Contrato e pagamento seguem no processo administrativo de hoje.
  - Valor final: o total do orçamento salvo; se a negociação aprovou condição, o valor com a condição; qualquer outro valor exige motivo.
  - Marco de conversão, até a direção decidir: checklist concluído. A matrícula conta uma vez por lead e período letivo.
  - Matrícula presencial sem lead: o gestor cria o lead com origem "presencial" e registra a matrícula; ele entra nos indicadores, nunca fica de fora.
  - `Interesse 2027` exige ano-alvo e objetivo; a data de retomada padrão é o primeiro dia útil de novembro do ano anterior ao ano-alvo, configurável. Se essa data já passou, a retomada é no próximo dia útil. Dia útil = segunda a sexta, sem os feriados cadastrados no console.
  - Na retomada, a rotina das 7h devolve o lead para `Em atendimento` (etapa configurável) com a próxima ação "retomada" para quem estacionou o lead, ou para a gestão se a pessoa estiver desativada. Lead com opt-out recebe só a tarefa, sem modelo.
  - Massa de teste com `is_test = true`.
- **Bloqueios:** nenhum. Itens do checklist e marco de conversão (CL-009) e data de retomada (CL-010) mudam só a configuração.

## Resultado observável

Em `Em matrícula`, o cartão mostra o checklist com o que falta. Com tudo conferido e o valor final, o lead vira `Novo aluno` e a matrícula aparece com turma e específicas. Um lead posto em `Interesse 2027` mostra a data de retomada e reaparece na fila da unidade nesse dia.

## Limites e dependências

- **Inclui:** conferência do checklist pela tela; registro da matrícula; lead retroativo presencial; estacionamento em `Interesse 2027`; rotina de retomada.
- **Fora de escopo:** upload de documento; contrato, pagamento e integração financeira; gravar matrícula no MatriculaWeb; campanha automática de reengajamento.
- **Entradas e pré-condições:** SPECs 3-002 e 3-004 no ar.
- **Saídas/artefatos:** tabelas `enrollments` e `future_interests`; uso de `enrollment_checklist_items`; Edge Functions `enrollment-complete` e `reactivation-run`; agendamento diário no Cron.
- **Dependências e responsáveis:** direção decide marco e itens; atendentes conferem; Luiz testa.
- **Atores e permissões mínimas:** `attendant` confere e conclui da própria unidade; `unit_manager` cria lead presencial e corrige matrícula com motivo; `platform_admin` configura itens, marco e data de retomada; `service_role` roda a retomada.
- **Superfícies/arquivos/configurações afetadas:** cartão do lead, `stage_move`, console módulos 7 e 9, Cron.
- **Risco e plano B:** rotina de retomada parada: na volta, ela processa as datas vencidas sem duplicar e o painel mostra o atraso.
- **Rollback ou reversão:** tabelas aditivas; `retomada_ativa = false` no console para a rotina.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Tela → itens conferidos | Central | `lead_id`, `item_id`, `conferido_por`, `conferido_em`, `observacao` | `attendant`+ | conferir de novo não duplica | — |
| `enrollment-complete` → `enrollments` | Central | `lead_id`, `unit_id`, `quote_id`, `negotiation_id`, `turma_nome`, `periodo_letivo`, `itens_contratados`, `valor_final`, `motivo_valor`, `concluida_em`, `concluida_por`, `is_test` | `attendant`+ | uma por lead + período letivo | pendência: 409 com a lista |
| Tela → `future_interests` | Central | `lead_id`, `ano_alvo`, `objetivo`, `reactivate_at`, `status` (aguardando, retomado, cancelado), `estacionado_por`, `is_test` | `attendant`+ | um ativo por lead | sem ano-alvo: 400 |
| Cron → `reactivation-run` | Central | leads com `reactivate_at` ≤ hoje e `status = aguardando` | `service_role` | `origem_chave` = lead + `reactivate_at` | falha: dead-letter |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | `Novo aluno` com item pendente ou sem valor final | 409 com a lista do que falta | — | RN-12 do escopo |
| RN-02 | Valor final diferente do orçamento e da condição aprovada | Exige motivo | — | SPEC-3-002 e 3-004 |
| RN-03 | Checklist concluído | Grava a matrícula e o marco de conversão | Marco muda quando a direção decidir | GATE-12 |
| RN-04 | Segunda matrícula do mesmo lead no mesmo período letivo | Recusada | Correção pelo gestor com motivo | K3 |
| RN-05 | Entrada em `Interesse 2027` | Exige ano-alvo e objetivo; grava `reactivate_at`; pausa a cadência | — | RN-10 do escopo |
| RN-06 | `reactivate_at` chegou | Lead para a etapa de retomada com a ação "retomada" | Opt-out: só a tarefa | RN-10 e RN-17 do escopo |

## Fluxo e regras

1. O atendente marca os itens conferidos; a central mostra o que falta.
2. Ao concluir, `enrollment-complete` confere o checklist e o valor (RN-01, RN-02), grava a matrícula (RN-03) e move para `Novo aluno` pelo `stage_move`.
3. O gestor registra o lead presencial com a matrícula pelo mesmo caminho.
4. Para `Interesse 2027`, o atendente informa ano-alvo e objetivo (RN-05).
5. Às 7h, o Cron enfileira `reactivation-run`, que aplica RN-06 sem duplicar.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Lead `TESTE` com checklist completo e orçamento | Matrícula gravada; `Novo aluno` | — |
| Limite | Falta "documentos conferidos" | 409 listando o item | Conferir libera |
| Limite | `Interesse 2027` com ano-alvo 2027 em 28/10/2026 | Retomada no primeiro dia útil de novembro de 2026 | — |
| Falha | Rotina rodou duas vezes no dia | Uma retomada | — |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC; a SPEC-1-003 em `05_entregas/fase-1/specs/`; as SPECs 3-002 e 3-004; o bloqueio atual de `Novo aluno`.
2. **Alterar somente:** conferência do checklist, matrícula, lead presencial, `Interesse 2027`, rotina de retomada e a configuração ligada a eles.
3. **Não alterar:** orçamento; negociação; nada no MatriculaWeb.
4. **Executar nesta ordem:** checklist; matrícula; Interesse 2027; retomada.
5. **Parar e pedir validação quando:** alguém pedir para guardar documento ou dado de pagamento na central.
6. **Estado válido ao parar:** bloqueio de `Novo aluno` da fase 1 funcionando.
7. **Ritmo:** construir as tasks vinculadas em sequência, sem pedir autorização nem teste a cada task. O teste do cliente é o roteiro único de `## Teste humano do cliente`, quando todas as tasks vinculadas estiverem no ar. Ponto de aprovação antes disso: nenhum.

## Checklist de execução

- [ ] Checklist conferido pela tela, com o que falta.
- [ ] Matrícula com valor final, turma e contratado; motivo quando o valor muda.
- [ ] Lead presencial com matrícula.
- [ ] `Interesse 2027` com ano-alvo e data; retomada sem duplicar.

## Critérios de aceite

- [ ] **CA-3-18:** `Novo aluno` com item pendente ou sem valor final é recusado no servidor com a lista do que falta; nenhum documento é guardado, só "conferido", por quem e quando.
- [ ] **CA-3-19:** a matrícula grava valor final, turma, período letivo e o que foi contratado; valor diferente do orçamento e da condição aprovada exige motivo; conta uma vez por lead e período letivo; o lead presencial entra com origem "presencial".
- [ ] **CA-3-20:** `Interesse 2027` exige ano-alvo e objetivo, grava a data de retomada pela regra configurada, pausa a cadência e sai do denominador do K2.
- [ ] **CA-3-21:** na data de retomada, a rotina devolve o lead à etapa configurada com uma única ação "retomada" para quem estacionou ou para a gestão; lead com opt-out não recebe modelo.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P3.5-01: concluir com item pendente e sem valor | `enrollment-complete` | 409 com a lista | saída |
| GREEN | P3.5-02: checklist completo e valor do orçamento | `enrollment-complete` | matrícula gravada; `Novo aluno`; histórico | saída pgTAP |
| RED | P3.5-03: valor diferente sem motivo; segunda matrícula no período | função | 400; 409 | saída |
| GREEN | P3.5-04: lead presencial pelo gestor | tela + consulta | lead com origem "presencial" e matrícula | captura |
| RED | P3.5-05: `Interesse 2027` sem ano-alvo | `stage_move` | 400 | saída |
| GREEN | P3.5-06: ano-alvo 2027 em 28/10/2026 e 2028 | RPC | datas pela regra | saída |
| GREEN | P3.5-07: rotina com relógio simulado, rodando duas vezes | `reactivation-run` | uma retomada; ação certa; opt-out sem modelo | saída |
| REFACTOR/REGRESSÃO | P3.5-08: provas P3.1 a P3.4 e pgTAP da fase 1 | suíte | verdes | saída |

**Dados/fixtures:** leads `TESTE` em `Em matrícula` com e sem orçamento, um com condição aprovada; leads de 2027 e 2028; um com opt-out.  
**Caminhos de erro obrigatórios:** item pendente, valor sem motivo, matrícula duplicada, ano-alvo vazio, rotina repetida, responsável desativado.  
**Evidência exigida:** saídas das provas e capturas do checklist e da retomada na linha `> ` da task.

## Teste humano do cliente

- **Origem:** CL-001
- **Quando:** quando todas as tasks de `## Tasks vinculadas` estiverem no ar
- **Quem testa:** Luiz, com um atendente
- **Passos:** tentar concluir a matrícula de um lead `TESTE` com um item pendente; conferir tudo e concluir com o valor do orçamento; registrar um lead presencial; pôr outro lead em `Interesse 2027` e ver a data; adiantar o relógio de teste e ver o lead voltar para a fila.
- **Resultado esperado:** bloqueio com o que falta, matrícula registrada, presencial contado, data de retomada certa e lead de volta na fila.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Handoff e operação

- **Como demonstrar:** um lead `TESTE` de `Em matrícula` a `Novo aluno` e um de 2027 voltando.
- **Como operar depois:** a secretaria confere os itens; o gestor olha as retomadas do dia.
- **Como monitorar:** matrículas do dia e retomadas pendentes no painel.
- **Pendência conhecida:** marco definitivo de conversão, quando a direção decidir.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Conferir o checklist de matrícula antes de marcar Novo aluno | Academia das Específicas | SPEC-3-005 | CA-3-18 | P3.5-01, P3.5-02 | saída pgTAP | SPEC-3-004 no ar | ☐ |
| pendente | Registrar a matrícula com o valor final e o que foi contratado | Academia das Específicas | SPEC-3-005 | CA-3-19 | P3.5-03, P3.5-04 | saída + captura | Checklist no ar | ☐ |
| pendente | Guardar o lead de Interesse 2027 com a data de retomada | Academia das Específicas | SPEC-3-005 | CA-3-20 | P3.5-05, P3.5-06 | saída | — | ☐ |
| pendente | Trazer de volta os leads de 2027 na data de retomada | Academia das Específicas | SPEC-3-005 | CA-3-21 | P3.5-07, P3.5-08 | saída | Interesse 2027 no ar | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
