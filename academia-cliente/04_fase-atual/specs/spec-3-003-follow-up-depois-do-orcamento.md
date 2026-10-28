# SPEC-3-003 — Follow-up depois do orçamento

**Fase:** 3  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** fase 3 (cadência FUP automática, envio humano); DEC-08; fluxo F4; RN-03, RN-07, RN-16, RN-17; AG-04; emenda de 10/10 (FUP 3 arquivada, 3º retorno vira lembrete); GATE-07; CL-001, CL-004  
**Degrau da solução:** nativo da plataforma — Supabase Cron enfileira na fila `cadence_scheduler` e o worker cria as próximas ações; reusa `next_actions`, `stage_move` e a janela de 24h da SPEC-3-001.

## Contexto e decisões fechadas

- **Estado atual:** hoje o FUP é arrastado à mão no Kommo e os prazos ficam de memória. Na central já existem as etapas `Fup 1` e `Fup 2`; a coluna FUP 3 foi arquivada em 10/10 e o terceiro retorno passou a ser lembrete. A SPEC-3-002 cria a próxima ação "aguardar resposta do orçamento".
- **Estado desejado:** sem resposta do lead, a central cria sozinha a tarefa de FUP 1 e depois a de FUP 2, move o lead de etapa e diz ao atendente que tipo de mensagem cabe. Depois do FUP 2 vem o lembrete do terceiro retorno, que só fecha com uma decisão sobre o lead. Nenhuma mensagem sai sozinha.
- **Decisões já fechadas:**
  - Prazos no console (módulo 3): FUP 1 = 20 horas depois do orçamento enviado (`fup1_horas`, a mesma configuração usada pela SPEC-3-002); FUP 2 = 24 horas depois da tarefa de FUP 1 (`fup2_horas`); lembrete do terceiro retorno = 24 horas depois da tarefa de FUP 2 (`lembrete3_horas`), até a Academia definir outro prazo. Contagem padrão em horas corridas, como hoje; a opção "horário comercial" usa os dias e horários que o admin cadastrar.
  - "Sem resposta" = nenhuma mensagem do lead desde o início do prazo e nenhuma tarefa do lead concluída com o resultado "lead respondeu" (para contato fora da central).
  - A tarefa de FUP é um registro de `next_actions` (`tipo` = `fup_1`, `fup_2`, `lembrete_3_retorno`). O responsável é quem enviou o orçamento; se estiver desativado, a tarefa vai para a gestão reatribuir.
  - A ação automática só cria tarefa e move o lead entre `Orçamento enviado`, `Fup 1` e `Fup 2`. Nunca envia mensagem, nunca desqualifica e nunca promete condição. Cada execução fica em `agent_executions` com `execution_id`.
  - Pausa: lead em `Negociação`, com reunião agendada (SPEC-3-004), em `Interesse 2027` ou em etapa terminal não recebe FUP. Ao sair da pausa, a contagem recomeça da saída.
  - Opt-out ("não quero receber mensagens"): registro permanente no lead; bloqueia no servidor o envio de modelo de reengajamento; a resposta livre a uma mensagem do próprio lead, dentro da janela, continua permitida.
- **Bloqueios:** nenhum. A contagem definitiva e o destino depois do terceiro retorno (CL-004) mudam só a configuração.

## Resultado observável

Um lead `TESTE` que não responde ao orçamento ganha a tarefa de FUP 1 e vai para `Fup 1`; sem resposta, ganha a de FUP 2 e vai para `Fup 2`. Cada tarefa mostra se a janela está aberta ou se só cabe modelo, com o custo. Depois vem o lembrete do terceiro retorno, que pede a decisão.

## Limites e dependências

- **Inclui:** cadência FUP 1 e FUP 2; lembrete do terceiro retorno com decisão; pausa e retomada; tipo de mensagem permitido na tarefa; registro e bloqueio de opt-out.
- **Fora de escopo:** envio automático de qualquer mensagem; disparo em massa; IA sugerindo texto (fase 4); reunião e negociação (SPEC-3-004, que só liga a pausa).
- **Entradas e pré-condições:** SPECs 3-001 e 3-002 no ar.
- **Saídas/artefatos:** agendamento no `cron-tick`; consumidor da fila `cadence_scheduler`; campos `cadencia_pausada_motivo`, `cadencia_retomada_em`, `opt_out_em`, `opt_out_por` no lead; configuração dos prazos.
- **Dependências e responsáveis:** Academia define a contagem e o destino depois do terceiro retorno; atendentes enviam; Luiz testa.
- **Atores e permissões mínimas:** `attendant` vê e conclui as tarefas da unidade e registra opt-out; `unit_manager` reatribui; `platform_admin` configura prazos; `service_role` roda o worker.
- **Superfícies/arquivos/configurações afetadas:** `cron-tick`, `queue-worker`, `message-send` (bloqueio de opt-out), cartão do lead, lista de tarefas, console módulo 3.
- **Risco e plano B:** worker parado: a fila acumula, o painel mostra o atraso e nenhuma tarefa se perde; ao voltar, cria as atrasadas com a data certa, sem duplicar.
- **Rollback ou reversão:** `cadencia_ativa = false` no console para a criação; tarefas criadas continuam valendo.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Cron → `cadence_scheduler` | Central | evento `cadence_check` com `lead_id`, `etapa`, `referencia_em` | `service_role` | o cron só enfileira; `event_id` por lead + etapa + referência | job falho: backoff e dead-letter |
| Worker → `next_actions` | Central | `tipo` (`fup_1`, `fup_2`, `lembrete_3_retorno`), `responsavel_id`, `prazo`, `contexto` (turma, total do orçamento, dias desde o envio), `origem` = `AG-04`, `origem_chave` = lead + tipo + `quote_id` | `service_role` | `origem_chave` única | conflito: ignora sem erro |
| Worker → `stage_move` | Central | `Orçamento enviado` → `Fup 1` → `Fup 2`, como sistema | `service_role` | transição repetida é ignorada | etapa diferente da esperada: não move, registra |
| Tela → lead (opt-out) | Central | `opt_out_em`, `opt_out_por`, motivo | `attendant`+ | gravar de novo não muda a data | — |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | 20h sem resposta depois do orçamento enviado | Fecha "aguardar resposta" com "sem resposta"; cria tarefa `fup_1`; lead para `Fup 1` | Pausa ativa | DEC-08 |
| RN-02 | 24h sem resposta depois da tarefa de FUP 1 | Fecha a `fup_1` pendente com "sem resposta"; cria `fup_2`; lead para `Fup 2` | Pausa ativa | DEC-08 |
| RN-03 | 24h sem resposta depois da tarefa de FUP 2 | Cria `lembrete_3_retorno`, sem mudar de etapa | Pausa ativa | Emenda de 10/10 |
| RN-04 | Mensagem do lead ou tarefa concluída com "lead respondeu" | Encerra a cadência; o atendente registra a próxima ação | — | Escopo F4 |
| RN-05 | Concluir o lembrete do terceiro retorno | Exige um destino: `Negociação`, `Interesse 2027`, `Desqualificado` com motivo, `Em matrícula` ou "segue conversando" com nova próxima ação | — | Escopo F4 |
| RN-06 | Janela de 24h aberta | Tarefa mostra "mensagem livre até HH:MM" | — | SPEC-3-001 |
| RN-07 | Janela fechada | Tarefa mostra "só modelo aprovado", com os modelos e o custo | Opt-out: "não enviar modelo" | SPEC-3-001 |
| RN-08 | Lead com opt-out | `message-send` recusa modelo de reengajamento com 409 | Resposta livre dentro da janela | RN-17 do escopo |

## Fluxo e regras

1. O `cron-tick` enfileira a checagem dos leads em `Orçamento enviado`, `Fup 1` e `Fup 2` sem pausa.
2. O worker confere "sem resposta" e o prazo pela contagem configurada e aplica RN-01 a RN-03.
3. O atendente abre a tarefa, vê o contexto e o tipo de mensagem permitido (RN-06, RN-07), envia pela caixa e conclui com o resultado.
4. Resposta do lead encerra a cadência (RN-04). O lembrete do terceiro retorno fecha com o destino (RN-05).
5. Pausa e retomada seguem `cadencia_pausada_motivo`; a SPEC-3-004 liga e desliga a pausa de reunião e de negociação.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Orçamento enviado há 21h, sem resposta | Uma tarefa `fup_1`; lead em `Fup 1` | — |
| Limite | Worker roda duas vezes no mesmo minuto | Uma tarefa só | — |
| Limite | Lead respondeu às 19h | Sem FUP 1; cadência encerrada | — |
| Falha | Atendente tenta enviar modelo a lead com opt-out | 409 com o motivo | — |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC; as SPECs 3-001 e 3-002; o `cron-tick` e o `queue-worker` atuais.
2. **Alterar somente:** cadência, lembrete, pausa, opt-out, configuração dos prazos e a regra de opt-out em `message-send`.
3. **Não alterar:** regras de envio da SPEC-3-001 além do opt-out; RLS; etapas e motivos da fase 1.
4. **Executar nesta ordem:** cadência com pausa; tipo de mensagem e opt-out na tarefa; lembrete com decisão.
5. **Parar e pedir validação quando:** a regra pedir envio automático de qualquer mensagem.
6. **Estado válido ao parar:** cadência desligável no console; nenhuma mensagem enviada pelo sistema.
7. **Ritmo:** construir as tasks vinculadas em sequência, sem pedir autorização nem teste a cada task. O teste do cliente é o roteiro único de `## Teste humano do cliente`, quando todas as tasks vinculadas estiverem no ar. Ponto de aprovação antes disso: nenhum.

## Checklist de execução

- [ ] FUP 1 e FUP 2 criados nos prazos, sem duplicar, com mudança de etapa.
- [ ] Pausa e retomada conferidas.
- [ ] Tipo de mensagem e custo visíveis na tarefa; opt-out bloqueia modelo.
- [ ] Lembrete do terceiro retorno só fecha com destino.
- [ ] Nenhuma mensagem enviada pelo sistema.

## Critérios de aceite

- [ ] **CA-3-11:** sem resposta, a central cria uma única tarefa de FUP 1 no prazo configurado e move o lead para `Fup 1`, e depois uma única de FUP 2 com o lead em `Fup 2`; resposta do lead encerra a cadência; lead em pausa não recebe FUP e a contagem recomeça ao sair dela; nada é enviado.
- [ ] **CA-3-12:** a tarefa mostra a janela aberta com o horário-limite ou "só modelo aprovado" com o custo; lead com opt-out mostra "não enviar modelo" e `message-send` recusa o modelo com 409.
- [ ] **CA-3-13:** 24 horas depois da tarefa de FUP 2 sem resposta, aparece o lembrete do terceiro retorno, sem coluna nova; ele só fecha com um destino, e cada destino passa pelas regras da etapa.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P3.3-01: orçamento enviado há 21h, antes da cadência | worker de teste | nenhuma tarefa (falha esperada) | saída |
| GREEN | P3.3-02: mesmo caso com a cadência | worker de teste com relógio simulado | uma `fup_1`; lead em `Fup 1`; `agent_executions` gravado | saída pgTAP |
| GREEN | P3.3-03: FUP 2 e lembrete do terceiro retorno | relógio simulado +24h e +48h | uma `fup_2`, lead em `Fup 2`; um lembrete | saída |
| RED | P3.3-04: worker duas vezes; lead que respondeu; lead em pausa | worker de teste | sem duplicar; sem FUP para quem respondeu ou está em pausa | saída |
| GREEN | P3.3-05: tarefa com janela aberta e com janela fechada | tela + consulta | horário-limite; lista de modelos com custo | captura |
| RED | P3.3-06: modelo para lead com opt-out | chamada a `message-send` | 409 | saída |
| GREEN | P3.3-07: concluir lembrete sem destino e com cada destino | RPC | sem destino: 400; destinos válidos aplicam as regras da etapa | saída |
| REFACTOR/REGRESSÃO | P3.3-08: provas P3.1 e P3.2; contagem "horário comercial" num fim de semana | suíte + relógio simulado | provas verdes; prazo pula o período fechado | saída |

**Dados/fixtures:** leads `TESTE` com orçamento enviado em horários diferentes; relógio simulado no worker; um lead com opt-out; um lead em `Negociação`.  
**Caminhos de erro obrigatórios:** execução repetida, resposta no limite do prazo, responsável desativado, opt-out, pausa.  
**Evidência exigida:** saídas das provas e capturas das tarefas na linha `> ` da task.

## Teste humano do cliente

- **Origem:** CL-001
- **Quando:** quando todas as tasks de `## Tasks vinculadas` estiverem no ar
- **Quem testa:** Luiz, com um atendente
- **Passos:** usar dois leads `TESTE` com orçamento enviado e o relógio de teste adiantado; ver a tarefa de FUP 1 e a mudança para `Fup 1`; responder por um deles e ver a cadência parar; no outro, avançar até o FUP 2 e o lembrete; ver na tarefa o tipo de mensagem e o custo; marcar opt-out num lead e tentar mandar um modelo; fechar o lembrete escolhendo um destino.
- **Resultado esperado:** tarefas certas nos prazos, nada enviado sozinho, opt-out bloqueando modelo e lembrete pedindo decisão.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Handoff e operação

- **Como demonstrar:** a lista de tarefas do dia com FUP 1, FUP 2 e lembretes.
- **Como operar depois:** o atendente trabalha a lista de tarefas; o gestor reatribui as de quem saiu.
- **Como monitorar:** atraso da fila `cadence_scheduler` e tarefas vencidas no painel.
- **Pendência conhecida:** contagem definitiva e destino depois do terceiro retorno, quando a Academia decidir.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Criar sozinho as tarefas de FUP 1 e FUP 2 nos prazos configurados | Academia das Específicas | SPEC-3-003 | CA-3-11 | P3.3-01 a P3.3-04 | saída pgTAP | SPEC-3-002 no ar | ☐ |
| pendente | Mostrar na tarefa de follow-up se cabe mensagem livre ou só modelo | Academia das Específicas | SPEC-3-003 | CA-3-12 | P3.3-05, P3.3-06 | captura + saída | Cadência no ar | ☐ |
| pendente | Lembrar o terceiro retorno e pedir a decisão sobre o lead | Academia das Específicas | SPEC-3-003 | CA-3-13 | P3.3-07, P3.3-08 | saída | — | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
