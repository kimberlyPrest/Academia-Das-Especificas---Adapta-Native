# SPEC-3-006 — Painel gerencial da origem à matrícula

**Fase:** 3  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** fase 3 (painel gerencial completo origem→matrícula; gestor audita amostras; diretoria vê funil consolidado); §17 (indicadores); papéis `marketing` e `viewer`; AG-09; CL-001  
**Degrau da solução:** reuso — estende o painel-base da fase 1 e o painel do dia da SPEC-3-001 com o rollup `metrics_daily` (AG-09); sem BI externo.

## Contexto e decisões fechadas

- **Estado atual:** o painel-base da fase 1 mostra volume por etapa, unidade e origem, pendências de gestão e desqualificações por motivo. A SPEC-3-001 traz o painel do dia e o K1; a SPEC-3-002 traz o K2. Não há visão da origem até a matrícula, nem acesso do marketing e da diretoria, nem auditoria de conversas.
- **Estado desejado:** direção, gestores e marketing veem, por unidade, origem, campanha e período, quantos leads viraram qualificados, orçamentos, negociações e matrículas, com a conversão de cada passo. O gestor audita uma amostra de conversas da unidade.
- **Decisões já fechadas:**
  - Indicadores: leads recebidos; qualificados; orçamentos enviados; negociações abertas e decididas, com o tempo de decisão; matrículas por turma e por específica; conversão de cada etapa e de qualificado para matrícula; desqualificações por motivo; custo dos modelos pagos (`cost_log`); K1 e K2.
  - As duas unidades são empresas distintas: `unit_manager` vê só a unidade dele; `viewer` (diretoria) e `platform_admin` veem cada unidade e a soma.
  - `marketing` vê só números agregados: nenhum nome, telefone, e-mail ou conversa, nem por API.
  - Os números vêm de `stage_transitions`, `quotes`, `negotiations`, `enrollments` e `cost_log`, consolidados em `metrics_daily` de hora em hora; o painel mostra a hora da última consolidação.
  - Massa de teste fora por padrão; o histórico migrado do Kommo (SPEC-3-007) aparece só no filtro "histórico do Kommo", nunca somado à operação.
  - Auditoria: o gestor sorteia uma amostra de conversas da unidade no período (ele escolhe de 5 a 20) e marca, em cada uma, "qualificação completa" e "terminou com pergunta"; o painel mostra os dois percentuais e quem auditou.
- **Bloqueios:** nenhum.

## Resultado observável

O painel gerencial abre com o funil da origem à matrícula, filtrável por unidade, origem, campanha e período. O usuário de marketing vê os mesmos números sem nenhuma conversa ou dado pessoal. O gestor sorteia conversas, marca os dois itens e vê os percentuais.

## Limites e dependências

- **Inclui:** painel gerencial; ampliação do AG-09; acesso de `marketing` e `viewer`; auditoria de amostras; exportação agregada sem dado pessoal.
- **Fora de escopo:** comparação com baseline e K3 contra meta (fase 5); IA analisando conversa (fase 4); BI externo.
- **Entradas e pré-condições:** SPECs 3-001 a 3-005 no ar; usuários de teste com papel `marketing` e `viewer`.
- **Saídas/artefatos:** visões e RPCs do painel; `metrics_daily` ampliada; tabela `conversation_audits`; Edge Function `reports-export` (agregada).
- **Dependências e responsáveis:** direção e marketing usam; Luiz testa.
- **Atores e permissões mínimas:** conforme as decisões acima; `marketing` sem acesso a `messages`, `conversations` e campos pessoais de `leads`.
- **Superfícies/arquivos/configurações afetadas:** painel-base, `cron-rollup`, RLS das visões, tela Auditoria.
- **Risco e plano B:** rollup atrasado: o painel calcula direto de `stage_transitions` e avisa "consolidação atrasada".
- **Rollback ou reversão:** visões e tabela aditivas; o painel-base da fase 1 continua.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Cron → `metrics_daily` | Central | por dia, unidade, origem, campanha e etapa: entradas, saídas, orçamentos, negociações, matrículas, custo; `consolidado_em` | `service_role` | upsert por chave do dia | falha: painel calcula direto e avisa |
| Gestor → `conversation_audits` | Central | `conversation_id`, `unit_id`, `amostra_id`, `qualificacao_completa`, `terminou_com_pergunta`, `nota`, `auditado_por`, `auditado_em` | `unit_manager` da unidade | uma auditoria por conversa e amostra | — |
| `reports-export` | Central | só agregados do filtro atual | `unit_manager`+, `viewer`, `marketing` | — | pedido de campo pessoal: 403 |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | Usuário `marketing` | Só agregados | — | §4 e §15 do escopo |
| RN-02 | Usuário `unit_manager` | Só a própria unidade | — | DEC-06 |
| RN-03 | Registro `is_test` ou `migrated` | Fora dos números por padrão | Filtros próprios | SPECs 3-001 e 3-007 |
| RN-04 | Matrícula | Conta pelo marco da SPEC-3-005, uma vez por lead e período letivo | — | K3 |

## Fluxo e regras

1. O `cron-rollup` consolida `metrics_daily` de hora em hora.
2. O painel aplica os filtros e as regras de acesso (RN-01 a RN-03) e mostra a hora da consolidação.
3. Cada número abre a lista de leads correspondente, menos para `marketing`.
4. O gestor sorteia a amostra, audita e o painel mostra os percentuais.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Massa `TESTE` com o filtro de teste ligado pelo admin | Funil bate com a contagem direta | — |
| Limite | Filtro sem resultado | Estado vazio claro | — |
| Falha | `marketing` consulta `messages` pela API | Nada retornado | — |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC; a SPEC-1-004 em `05_entregas/fase-1/specs/`; o painel-base e o `cron-rollup` atuais.
2. **Alterar somente:** painel gerencial, rollup, RLS das visões, auditoria e exportação agregada.
3. **Não alterar:** regras do funil; cálculo do K1 e do K2 já entregues, além de exibi-los.
4. **Executar nesta ordem:** funil e rollup; acesso de marketing e diretoria; auditoria.
5. **Parar e pedir validação quando:** algum indicador pedir dado pessoal para o marketing.
6. **Estado válido ao parar:** painel-base funcionando.
7. **Ritmo:** construir as tasks vinculadas em sequência, sem pedir autorização nem teste a cada task. O teste do cliente é o roteiro único de `## Teste humano do cliente`, quando todas as tasks vinculadas estiverem no ar. Ponto de aprovação antes disso: nenhum.

## Checklist de execução

- [ ] Funil da origem à matrícula com filtros e hora da consolidação.
- [ ] Números conferidos com a contagem direta.
- [ ] Acesso de marketing sem dado pessoal, provado por API.
- [ ] Auditoria de amostra com os dois percentuais.

## Critérios de aceite

- [ ] **CA-3-22:** o painel mostra, por unidade, origem, campanha e período, leads, qualificados, orçamentos, negociações com tempo de decisão, matrículas por turma e específica, conversão de cada passo, desqualificações por motivo, custo de modelos, K1 e K2; os números batem com a contagem direta; teste e histórico do Kommo ficam fora por padrão.
- [ ] **CA-3-23:** `marketing` vê só agregados e não lê conversa, nome, telefone ou e-mail nem pela API; `unit_manager` vê só a unidade; `viewer` vê cada unidade e a soma.
- [ ] **CA-3-24:** o gestor sorteia uma amostra da unidade, marca os dois itens em cada conversa e o painel mostra os percentuais e quem auditou.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P3.6-01: rollup antes da ampliação × contagem direta | consulta | divergência (falha esperada) | saída |
| GREEN | P3.6-02: rollup ampliado × contagem direta, massa `TESTE` com filtro | consulta | números iguais | saída |
| RED | P3.6-03: `marketing` lê `messages`, `conversations` e campos pessoais de `leads` | pgTAP com sessão de teste | nada retornado | saída pgTAP |
| GREEN | P3.6-04: `unit_manager` de Taguatinga e `viewer` | pgTAP | só a unidade; cada unidade e a soma | saída pgTAP |
| GREEN | P3.6-05: amostra de 5 conversas auditada | tela + consulta | percentuais certos | captura |
| REFACTOR/REGRESSÃO | P3.6-06: provas P3.1 a P3.5; `reports-export` sem dado pessoal | suíte + exportação | verdes; arquivo só com agregados | saída |

**Dados/fixtures:** massa `TESTE` nas duas unidades cobrindo todas as etapas; usuários de teste `marketing`, `viewer` e `unit_manager`.  
**Caminhos de erro obrigatórios:** filtro vazio, rollup atrasado, acesso indevido por API, exportação com campo pessoal.  
**Evidência exigida:** saídas das provas e capturas do painel na linha `> ` da task.

## Teste humano do cliente

- **Origem:** CL-001
- **Quando:** quando todas as tasks de `## Tasks vinculadas` estiverem no ar
- **Quem testa:** Luiz como diretoria; quem cuida do marketing, com o usuário de teste dele
- **Passos:** abrir o painel com a massa de teste e filtrar por unidade e campanha; abrir um número e ver os leads; entrar como marketing e ver que não há conversa nem dado pessoal; como gestor, sortear 5 conversas, auditar e ver os percentuais.
- **Resultado esperado:** funil coerente com o quadro, marketing só com números e auditoria registrada.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Handoff e operação

- **Como demonstrar:** painel com filtros e a auditoria de uma amostra.
- **Como operar depois:** a direção olha o painel na semana de campanha; o gestor audita uma amostra por semana.
- **Como monitorar:** hora da última consolidação no painel.
- **Pendência conhecida:** comparação com baseline na fase 5.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Mostrar o funil da origem à matrícula por unidade e campanha | Academia das Específicas | SPEC-3-006 | CA-3-22 | P3.6-01, P3.6-02 | consulta × painel | SPEC-3-005 no ar | ☐ |
| pendente | Abrir os painéis ao marketing e à diretoria sem acesso às conversas | Academia das Específicas | SPEC-3-006 | CA-3-23 | P3.6-03, P3.6-04 | saída pgTAP | Funil no ar | ☐ |
| pendente | Deixar o gestor auditar uma amostra de conversas da unidade | Academia das Específicas | SPEC-3-006 | CA-3-24 | P3.6-05, P3.6-06 | captura + saída | — | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
