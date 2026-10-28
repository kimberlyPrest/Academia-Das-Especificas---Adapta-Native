# SPEC-3-004 — Reunião com o diretor e negociação com alçada

**Fase:** 3  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** fase 3 (reuniões, negociação com decisão registrada e retorno ao atendente); fluxos F5 e F6; RN-08, RN-15, RN-16; AG-06; GATE-05 e GATE-06; CL-001, CL-007, CL-008  
**Degrau da solução:** construção mínima — reunião e negociação registradas na própria central (`meetings`, `negotiations`, alçadas no console módulo 10), sem agenda externa; reusa a pausa da SPEC-3-003 e `next_actions`.

## Contexto e decisões fechadas

- **Estado atual:** pedido de desconto vira `Negociação` no Kommo e o atendente não concede condição na hora. A reunião com o diretor não tem registro ligado ao lead, e o follow-up continua correndo durante a reunião. As etapas `Negociação` e `Sem resposta pós negociação` já existem na central.
- **Estado desejado:** o atendente agenda a reunião com o diretor e o follow-up do lead para; o diretor registra o resultado e o lead segue o caminho certo. O pedido de desconto vai para a fila de negociação, quem tem alçada decide, e a decisão volta ao atendente com a próxima ação.
- **Decisões já fechadas:**
  - Reunião registrada na central: data e hora, presencial ou online, diretor responsável (usuário ativo com papel `director`). A ferramenta de agenda que a Academia usa não é integrada nesta fase.
  - Reunião agendada põe `cadencia_pausada_motivo = reuniao`, cancela os FUPs pendentes com o motivo "reunião agendada" e cria a próxima ação `reuniao` do diretor, com prazo no horário da reunião.
  - Resultados: compareceu e fechou → `Em matrícula`; compareceu e segue avaliando → próxima ação obrigatória e a cadência volta; não compareceu → próxima ação "remarcar reunião" de quem agendou e a cadência volta; desqualificou → motivo do catálogo e `Desqualificado`.
  - Reunião sem resultado até o fim do dia vira pendência de gestão e lembrete ao diretor.
  - Negociação: o atendente registra a objeção (desconto, parcelamento, outra) e a condição pedida. A central atribui ao negociador padrão configurado (ou outro `negotiator` ativo escolhido) e pausa a cadência (`negociacao`). Os negociadores veem a fila das duas unidades.
  - Alçada: faixas de desconto sobre o total final do orçamento, cada faixa com o papel ou a pessoa que aprova. Decisão acima da alçada do decisor é recusada no servidor. Sem alçada cadastrada, nenhuma decisão é aceita (falha fechada). Os valores reais vêm da direção; até lá só existem faixas `TESTE`.
  - A condição aprovada fica na negociação e no lead. O MatriculaWeb não é alterado.
- **Bloqueios:** nenhum para construir. Alçadas reais (CL-007) e lista de diretores e negociadores (CL-008) entram no console quando chegarem.

## Resultado observável

No cartão do lead, "Agendar reunião" para o follow-up e aparece na agenda do diretor dentro da central. O diretor registra o resultado e o lead muda de etapa. Na fila de negociação, o negociador vê o pedido com o orçamento e a objeção, decide dentro da alçada e o atendente recebe a decisão como tarefa.

## Limites e dependências

- **Inclui:** agendar, remarcar e cancelar reunião; registrar resultado; pendência de reunião sem resultado; fila de negociação; cadastro de alçadas; decisão com condição; devolução ao atendente; `Sem resposta pós negociação`.
- **Fora de escopo:** integração com agenda externa; envio automático de convite; política de desconto (a direção define); alteração de preço no MatriculaWeb.
- **Entradas e pré-condições:** SPECs 3-002 e 3-003 no ar; usuários de teste com papel `director` e `negotiator`.
- **Saídas/artefatos:** tabelas `meetings`, `negotiations`, `negotiation_authority`; Edge Functions `meetings-upsert`, `meeting-result`, `negotiation-decide`; telas Agenda do diretor e Fila de negociação; console módulo 10.
- **Dependências e responsáveis:** direção define alçadas e quem são diretores e negociadores; Luiz testa.
- **Atores e permissões mínimas:** `attendant` agenda e encaminha da própria unidade; `director` vê e registra só as reuniões dele; `negotiator` vê a fila das duas unidades e decide na alçada; `unit_manager` vê a unidade; `platform_admin` cadastra alçadas.
- **Superfícies/arquivos/configurações afetadas:** cartão do lead, `stage_move`, pausa da SPEC-3-003, RLS das tabelas novas.
- **Risco e plano B:** diretor sem acesso à central: o gestor da unidade registra o resultado em nome dele, com o nome do diretor no registro.
- **Rollback ou reversão:** tabelas aditivas; esconder as telas não afeta o funil.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Tela → `meetings` | Central | `lead_id`, `unit_id`, `director_id`, `data_hora`, `tipo` (presencial, online), `status` (agendada, realizada, nao_compareceu, cancelada), `resultado` (fechou, segue_avaliando, desqualificou), `registrado_por`, `registrado_em`, `is_test` | `attendant`+ agenda; `director` ou `unit_manager` registra | uma reunião agendada por lead | data no passado: 400 |
| Tela → `negotiations` | Central | `lead_id`, `unit_id`, `quote_id`, `objecao`, `condicao_pedida` (% ou valor), `negociador_id`, `status` (aberta, decidida, devolvida, sem_resposta), `decisao` (aprovada, recusada, contraproposta), `condicao_aprovada`, `alcada_id`, `justificativa`, `decidido_por`, `decidido_em`, `devolvida_em`, `is_test` | `attendant`+ abre; `negotiator` decide | uma negociação aberta por lead | 400 com o campo |
| Console → `negotiation_authority` | Central | `desconto_min`, `desconto_max` (% sobre o total final), `aprovador_papel` ou `aprovador_id`, `ativo` | `platform_admin` | faixas sem sobreposição | sobreposição: 400 |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | Reunião agendada | Pausa a cadência, cancela FUPs pendentes, cria a ação `reuniao` do diretor | — | RN-15 do escopo |
| RN-02 | Resultado registrado | Aplica o caminho do resultado e tira a pausa | — | RN-15 do escopo |
| RN-03 | Reunião passada sem resultado no fim do dia | Pendência de gestão e lembrete ao diretor | — | Escopo F5 |
| RN-04 | Pedido de desconto | Lead para `Negociação` com objeção e condição pedida; cadência pausada | O atendente nunca concede condição | RN-08 do escopo |
| RN-05 | Decisão acima da alçada do decisor | 403 "fora da sua alçada; encaminhe para <aprovador>" | — | GATE-05 |
| RN-06 | Nenhuma alçada cadastrada | Decisão recusada com "alçada não cadastrada" | — | Falha fechada |
| RN-07 | Decisão registrada | Volta ao atendente que encaminhou com a próxima ação "comunicar a decisão" (prazo do FUP 2) | Atendente desativado: gestão | Escopo F6 |
| RN-08 | Lead não responde no prazo depois da devolução | `Sem resposta pós negociação` com a ação "retomar contato"; sem resposta de novo, lembrete com destino, como na SPEC-3-003 | — | Escopo F6 |

## Fluxo e regras

1. O atendente agenda a reunião (`meetings-upsert`); RN-01 aplica a pausa.
2. O diretor vê as reuniões dele, registra o resultado (`meeting-result`) e o lead segue RN-02. Sem resultado, RN-03.
3. Para desconto, o atendente move o lead para `Negociação` com objeção e condição (RN-04); a fila mostra orçamento, objeção, histórico e quanto tempo o pedido espera.
4. O negociador decide (`negotiation-decide`) dentro da alçada (RN-05, RN-06); a decisão volta ao atendente (RN-07).
5. O atendente comunica e registra a resposta do lead: aceitou → `Em matrícula`; recusou → destino como no lembrete da SPEC-3-003; sem resposta → RN-08.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Lead `TESTE` em `Fup 1` agenda reunião, comparece e fecha | FUP cancelado; lead em `Em matrícula` | — |
| Limite | Reunião ontem sem resultado | Pendência e lembrete | Gestor registra pelo diretor |
| Falha | Negociador tenta aprovar 25% com alçada de 10% | 403 com o aprovador certo | Encaminhar |
| Falha | Nenhuma faixa cadastrada | Decisão recusada | Admin cadastra |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC; as SPECs 3-002 e 3-003; `stage_move` e a pausa da cadência.
2. **Alterar somente:** tabelas, funções e telas desta SPEC, o console módulo 10 e a ligação com a pausa.
3. **Não alterar:** cálculo do orçamento; regras de matrícula; nada no MatriculaWeb.
4. **Executar nesta ordem:** agendar reunião com pausa; resultado da reunião; fila e alçadas; decisão e devolução.
5. **Parar e pedir validação quando:** alguém pedir uma alçada real sem a decisão da direção registrada no grupo ou no checklist.
6. **Estado válido ao parar:** funil e cadência funcionando como antes.
7. **Ritmo:** construir as tasks vinculadas em sequência, sem pedir autorização nem teste a cada task. O teste do cliente é o roteiro único de `## Teste humano do cliente`, quando todas as tasks vinculadas estiverem no ar. Ponto de aprovação antes disso: nenhum.

## Checklist de execução

- [ ] Reunião pausa e cancela FUP; resultado aplica o caminho certo.
- [ ] Reunião sem resultado vira pendência.
- [ ] Fila de negociação das duas unidades para `negotiator`.
- [ ] Alçada no servidor, com falha fechada sem cadastro.
- [ ] Decisão volta ao atendente; `Sem resposta pós negociação` funcionando.

## Critérios de aceite

- [ ] **CA-3-14:** agendar reunião com diretor ativo e data futura pausa a cadência, cancela os FUPs pendentes com motivo e cria a ação do diretor; o diretor vê só as reuniões dele.
- [ ] **CA-3-15:** cada resultado leva ao caminho definido (fechou → `Em matrícula`; segue avaliando → próxima ação obrigatória; não compareceu → remarcar; desqualificou → motivo) e tira a pausa; reunião sem resultado no fim do dia vira pendência de gestão.
- [ ] **CA-3-16:** o pedido de desconto leva o lead a `Negociação` com objeção e condição pedida e pausa a cadência; a fila aparece para os negociadores das duas unidades; decisão fora da alçada ou sem alçada cadastrada é recusada no servidor.
- [ ] **CA-3-17:** a decisão registrada (aprovada, recusada ou contraproposta, com condição e justificativa) volta ao atendente com a próxima ação; sem resposta do lead no prazo, ele vai para `Sem resposta pós negociação` com a ação de retomar contato.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P3.4-01: reunião com data passada e com usuário sem papel `director` | chamada da função | 400 nos dois | saída |
| GREEN | P3.4-02: reunião agendada para lead em `Fup 1` | função + worker da SPEC-3-003 | FUP cancelado; nenhum FUP novo; ação do diretor criada | saída pgTAP |
| GREEN | P3.4-03: os quatro resultados | `meeting-result` | etapa e próxima ação certas; pausa retirada | saída |
| RED | P3.4-04: diretor A lê reunião do diretor B | consulta com sessão de teste | nada retornado | saída pgTAP |
| RED | P3.4-05: decisão de 25% com faixa máxima de 10%; decisão sem faixa | `negotiation-decide` | 403; recusa "alçada não cadastrada" | saída |
| GREEN | P3.4-06: decisão dentro da alçada | `negotiation-decide` | negociação `devolvida`; ação do atendente criada | saída |
| GREEN | P3.4-07: sem resposta depois da devolução | relógio simulado | lead em `Sem resposta pós negociação` com a ação | saída |
| REFACTOR/REGRESSÃO | P3.4-08: provas P3.1 a P3.3 | suíte | verdes | saída |

**Dados/fixtures:** leads `TESTE`; dois diretores e um negociador de teste; faixas `TESTE` de 0 a 10% (negociador) e de 10 a 30% (`platform_admin`).  
**Caminhos de erro obrigatórios:** data passada, papel errado, reunião de outro diretor, fora da alçada, sem alçada, atendente desativado.  
**Evidência exigida:** saídas das provas e capturas da agenda e da fila na linha `> ` da task.

## Teste humano do cliente

- **Origem:** CL-001
- **Quando:** quando todas as tasks de `## Tasks vinculadas` estiverem no ar
- **Quem testa:** Luiz como diretor e negociador de teste, com um atendente
- **Passos:** agendar uma reunião para um lead `TESTE` em follow-up e ver o FUP sumir; registrar "não compareceu" e ver a tarefa de remarcar; em outro lead, registrar "fechou"; encaminhar um pedido de desconto, tentar aprovar acima da faixa de teste, aprovar dentro dela e ver a tarefa voltar ao atendente.
- **Resultado esperado:** follow-up parado durante a reunião, caminhos certos por resultado, alçada respeitada e decisão de volta ao atendente.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Handoff e operação

- **Como demonstrar:** agenda do diretor e fila de negociação com leads `TESTE`.
- **Como operar depois:** o diretor registra o resultado no mesmo dia; os negociadores olham a fila duas vezes por dia.
- **Como monitorar:** reuniões sem resultado e tempo de decisão da negociação no painel.
- **Pendência conhecida:** alçadas reais e integração com a agenda, quando a Academia decidir.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Agendar a reunião com o diretor e pausar o follow-up | Academia das Específicas | SPEC-3-004 | CA-3-14 | P3.4-01, P3.4-02, P3.4-04 | saída pgTAP | SPEC-3-003 no ar | ☐ |
| pendente | Registrar o resultado da reunião e seguir o caminho de cada resultado | Academia das Específicas | SPEC-3-004 | CA-3-15 | P3.4-03 | saída | Reunião no ar | ☐ |
| pendente | Abrir a fila de negociação com as alçadas de desconto | Academia das Específicas | SPEC-3-004 | CA-3-16 | P3.4-05 | saída + captura | — | ☐ |
| pendente | Devolver a decisão da negociação ao atendente com a próxima ação | Academia das Específicas | SPEC-3-004 | CA-3-17 | P3.4-06, P3.4-07, P3.4-08 | saída | Fila no ar | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
