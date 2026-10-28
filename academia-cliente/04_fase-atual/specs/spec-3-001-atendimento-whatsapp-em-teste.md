# SPEC-3-001 — Atendimento pelo WhatsApp em teste (o que ficou da fase 2)

**Fase:** 3  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** fase 2 (inbox, janela de 24h, modelos com custo, SLA, painel diário); SPEC-2-001 (CA-2-04, CA-2-05, CA-2-08, CA-2-09); antigas T2.3, T2.4 e T2.5, que passaram para a fase 3 na liberação de 28/10/2026; aceite da fase 1 (CA-1-24); CL-001, CL-002  
**Degrau da solução:** reuso + construção mínima — reaproveita webhooks, agentes AG-01 a AG-03, filas e dead-letter da fase 2 (v0.0.36) e acrescenta envio, caixa de conversas e painel do dia em Edge Functions e telas que já existem.

## Contexto e decisões fechadas

- **Estado atual:** até 19/10 (v0.0.36, em sandbox) estavam prontos os webhooks do WhatsApp (assinatura conferida) e dos formulários, com reprocessamento sem duplicar e fila de falhas, e a validação, deduplicação e distribuição do lead (com as emendas de 12 e 13/10). O registro do repositório parou em 19/10. Em 27/10 a equipe informou que estava finalizando a etapa 2: o que já estiver no ar entra como entrega a validar ou completar, sem reconstruir.
- **Estado desejado:** o atendente responde pela central, dentro da janela de 24 horas, a fila de cada unidade aparece por tempo de espera, a boas-vindas sai sozinha e o gestor vê o SLA e o painel do dia. Tudo com números de teste da equipe.
- **Decisões já fechadas:**
  - Canal: API oficial da Meta (decisão da Academia em 19/10). Em teste, usa o número de teste do app da Academia no Meta for Developers. O número de hoje continua preso ao app do Kommo até a virada.
  - Token da Meta só nos segredos das Edge Functions. Nunca no navegador, no repositório ou em log. Logs sem texto de mensagem e com telefone mascarado.
  - Boas-vindas é modelo aprovado, sai no primeiro contato e não conta como atendimento: não grava `first_reply_at`.
  - Fora da janela de 24h, só modelo aprovado, com confirmação de quem envia e custo registrado em `cost_log`.
  - SLA da primeira resposta: 10 minutos (referência do Manual de Boas Práticas), configurável no console, até a direção definir o número.
  - Sem dono do lead: fila única por unidade (emenda de 12/10).
  - Os testes rodam no mesmo banco que vai receber a operação (orientação de 20/10). Todo registro de teste leva `is_test = true` e nome começando com `TESTE`. Painéis e indicadores excluem `is_test` por padrão; só o `platform_admin` vê o filtro "incluir dados de teste". Se a marca ainda não existir, esta SPEC cria a coluna `is_test boolean not null default false` em `leads`, `conversations` e `messages`; as SPECs seguintes usam a mesma regra nas suas tabelas.
- **Bloqueios:** nenhum para construir. Lead real e envio real dependem da conta Meta aprovada e da política de LGPD (checklist da fase) e não fazem parte desta SPEC.

## Resultado observável

Uma mensagem de um celular da equipe para o número de teste vira um lead `TESTE` na fila da unidade, recebe a boas-vindas e mostra o cronômetro da janela. O atendente responde pela central. O painel do dia mostra quem espera, quem passou do SLA e o K1 da unidade.

## Limites e dependências

- **Inclui:** `message-send` (envio pelo servidor) e boas-vindas; caixa de conversas por unidade, ordenada por tempo de espera; cronômetro da janela; escolha de modelo com custo fora da janela; cron de SLA; K1; painel operacional do dia por unidade; marca de teste; um teste E2E do fluxo da caixa.
- **Fora de escopo:** cadência de follow-up (SPEC-3-003); IA; disparo em massa; troca de número; operação com lead real.
- **Entradas e pré-condições:** fase 2 em sandbox (v0.0.36 ou posterior); app da Meta com número de teste; modelos de teste aprovados na Meta (boas-vindas e um de Utilidade).
- **Saídas/artefatos:** Edge Function `message-send`; telas Caixa de conversas e Painel do dia; tabela `sla_breaches`; registros em `cost_log`; teste E2E.
- **Dependências e responsáveis:** Academia mantém o app da Meta e os modelos de teste; Hikari executa; Luiz testa.
- **Atores e permissões mínimas:** `attendant` lê e responde só a própria unidade; `unit_manager` lê a unidade e o painel; `platform_admin` vê as duas unidades e configura SLA e custos; `marketing` não lê mensagens.
- **Superfícies/arquivos/configurações afetadas:** `supabase/functions/message-send`, `wa-webhook` (status de entrega), `cron-tick`; tabelas `conversations`, `messages`, `templates`, `cost_log`, `sla_breaches`; console (módulos 3 e 5); telas de caixa e painel.
- **Risco e plano B:** Meta fora do ar ou modelo recusado: retry com backoff, depois dead-letter com contexto e aviso no painel; a operação real continua no Kommo.
- **Rollback ou reversão:** desligar o envio no console (`envio_ativo = false`); webhooks continuam recebendo; nada muda no Kommo.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Central → Meta (`POST /{phone-number-id}/messages`) | Meta Cloud API | texto livre ou `template` (nome, idioma, variáveis) | Token no segredo da Edge Function; papel mínimo `attendant` | 10 s; 3 tentativas com backoff e jitter; chave `client_message_id` por envio | Falha esgotada: dead-letter + aviso no cartão |
| Meta → Central (webhook de status) | Meta Cloud API | `meta_message_id`, status (sent, delivered, read, failed) | Assinatura HMAC | Upsert por `meta_message_id` | Assinatura inválida: 401 sem processar |
| Central → `cost_log` | Central | `message_id`, `template_id`, categoria (Marketing, Utilidade), custo estimado, unidade, `is_test` | Servidor | Uma linha por envio de modelo | Custo da categoria não cadastrado: grava "custo a definir" e avisa o admin |
| Cron → `sla_breaches` | Central | `lead_id`, `unit_id`, `entered_at`, `sla_min`, `breached_at` | `service_role` | Único por lead + entrada | — |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | Primeiro contato do lead distribuído | Boas-vindas por modelo aprovado, uma vez por entrada | Lead em "Enviar e-mail" não recebe | Escopo §6.9 |
| RN-02 | Mensagem do lead recebida | `window_expires_at` = horário da mensagem + 24h | — | Meta |
| RN-03 | Janela fechada | Só modelo aprovado, com confirmação e custo | Opt-out bloqueia modelo de reengajamento (SPEC-3-003) | Escopo §14.3 |
| RN-04 | Primeira mensagem enviada por usuário humano | Grava `first_reply_at`; boas-vindas e mensagens de sistema não gravam | — | K1 |
| RN-05 | Lead sem primeira resposta humana depois do SLA | Abre `sla_breaches` e aparece em "fora do SLA" | Lead em "Enviar e-mail" com exceção registrada sai do denominador do K1 | Escopo §1 |
| RN-06 | Registro de teste | `is_test = true`; nome começa com `TESTE`; fora dos indicadores por padrão | — | Orientação de 20/10 |

## Fluxo e regras

1. O webhook recebe a mensagem, os agentes da fase 2 validam, deduplicam e distribuem; a boas-vindas sai (RN-01).
2. A conversa entra na caixa da unidade, ordenada pelo tempo de espera, com o cronômetro da janela (RN-02).
3. O atendente responde pela central; `message-send` confere papel, unidade e janela e envia; a primeira resposta humana grava `first_reply_at` (RN-04).
4. Fora da janela, a tela mostra só os modelos aprovados, com categoria e custo; o envio pede confirmação e grava `cost_log` (RN-03).
5. O `cron-tick` enfileira a varredura de SLA a cada minuto; o painel do dia mostra recebidos, aguardando primeira resposta, fora do SLA, exceções de contato, duplicatas a revisar, mediana e p90 da primeira resposta e K1.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Lead `TESTE` novo escreve | Fila da unidade, boas-vindas, cronômetro; resposta humana grava `first_reply_at` | — |
| Limite | Última mensagem do lead há 25 horas | Só modelo, com custo e confirmação | Sem modelo aprovado: envio bloqueado com o motivo |
| Falha | Atendente de outra unidade chama `message-send` | 403, nada enviado | — |
| Falha | Meta devolve erro | 3 tentativas, depois dead-letter com contexto | Reenvio manual pelo cartão |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC; a SPEC-2-001 em `05_entregas/fase-2/specs/`; o código atual das funções `wa-webhook`, `lead-webhook` e dos agentes AG-01 a AG-03. Se envio, caixa ou painel já existirem numa versão depois de 19/10, rode as provas desta SPEC nessa versão e complete só o que faltar.
2. **Alterar somente:** `message-send`, status de entrega no `wa-webhook`, varredura de SLA no `cron-tick`, as tabelas listadas, as telas de caixa e painel e a marca de teste.
3. **Não alterar:** regras de validação, deduplicação e distribuição da fase 2; RLS das tabelas da fase 1; nada no Kommo.
4. **Executar nesta ordem:** envio e boas-vindas; caixa com janela; SLA, K1, painel do dia, marca de teste e E2E.
5. **Parar e pedir validação quando:** alguém pedir envio para número que não seja da equipe ou cadastro do número de produção.
6. **Estado válido ao parar:** webhooks recebendo; envio desligado ou só para números de teste.
7. **Ritmo:** construir as tasks vinculadas em sequência, sem pedir autorização nem teste a cada task. O teste do cliente é o roteiro único de `## Teste humano do cliente`, quando todas as tasks vinculadas estiverem no ar. Ponto de aprovação antes disso: nenhum.

## Checklist de execução

- [ ] Envio só pelo servidor; token fora do navegador; papel sem permissão recusado.
- [ ] Boas-vindas sai uma vez e não grava `first_reply_at`.
- [ ] Caixa por unidade, ordenada por espera, com cronômetro e modelo com custo fora da janela.
- [ ] SLA configurável, `sla_breaches`, K1 e painel do dia conferidos com consulta direta.
- [ ] Marca de teste nas tabelas e indicadores sem teste por padrão.
- [ ] E2E da caixa e provas das fases 1 e 2 verdes.

## Critérios de aceite

- [ ] **CA-3-01:** mensagem só sai por `message-send` (papel mínimo `attendant`, da própria unidade); nenhum token aparece no navegador; outro papel ou outra unidade recebe 403; a boas-vindas sai uma vez por entrada por modelo aprovado e não grava `first_reply_at`.
- [ ] **CA-3-02:** a caixa de cada unidade lista as conversas por tempo de espera, com o cronômetro da janela; fora da janela só aparecem modelos aprovados, o envio pede confirmação e grava uma linha em `cost_log`.
- [ ] **CA-3-03:** o SLA vem do console (10 minutos até a decisão da direção); lead sem primeira resposta humana no prazo abre `sla_breaches` e aparece em "fora do SLA"; o K1 conta só resposta humana e exclui a exceção "Enviar e-mail" registrada.
- [ ] **CA-3-04:** o painel do dia por unidade (recebidos, aguardando primeira resposta, fora do SLA, exceções de contato, duplicatas a revisar, mediana e p90) bate com a consulta direta no mesmo período.
- [ ] **CA-3-05:** registros de teste têm `is_test = true` e nome `TESTE`; painel e K1 os excluem por padrão; o E2E da caixa e as provas das fases 1 e 2 passam na versão entregue.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P3.1-01: chamada a `message-send` sem sessão e com atendente de outra unidade | teste de integração da função | 401 e 403; nada enviado | saída do teste |
| GREEN | P3.1-02: lead `TESTE` novo escreve pelo número de teste | webhook de sandbox + consulta | boas-vindas enviada uma vez; `first_reply_at` vazio | log sanitizado + consulta |
| GREEN | P3.1-03: atendente responde dentro da janela | tela + consulta | `first_reply_at` gravado; K1 do lead dentro do SLA | captura + consulta |
| RED | P3.1-04: conversa com última mensagem há 25 horas | fixture + tentativa de texto livre | texto livre recusado; só modelo | saída do teste |
| GREEN | P3.1-05: envio de modelo fora da janela | tela com confirmação | `cost_log` com categoria e custo | consulta |
| GREEN | P3.1-06: lead sem resposta por 11 minutos | cron de teste | `sla_breaches` aberto; painel mostra "fora do SLA" | consulta + captura |
| REFACTOR/REGRESSÃO | P3.1-07: painel do dia × consulta direta; E2E da caixa (login, abrir conversa, responder); `supabase test db` das fases 1 e 2 | Playwright + pgTAP | números iguais; E2E e pgTAP verdes | saída dos testes |

**Dados/fixtures:** celulares da equipe como leads `TESTE`; conversa de fixture com última mensagem há 25 horas; usuários de teste por papel nas duas unidades.  
**Caminhos de erro obrigatórios:** sem sessão, outra unidade, janela fechada, modelo não aprovado, Meta com erro, custo não cadastrado.  
**Evidência exigida:** saídas dos testes, logs sanitizados e capturas da caixa e do painel registrados na linha `> ` da task.

## Teste humano do cliente

- **Origem:** CL-001 (aceite da entrega) e CL-002 (aceite da fase 1)
- **Quando:** quando todas as tasks de `## Tasks vinculadas` estiverem no ar
- **Quem testa:** Luiz, com o Hikari conduzindo; um atendente de cada unidade, se puder
- **Passos:** mandar mensagem de um celular da equipe para o número de teste e ver o lead na fila da unidade com a boas-vindas; responder pela central e ver o cronômetro e o K1; abrir uma conversa de teste fora da janela e enviar um modelo com confirmação; conferir o painel do dia; entrar como atendente da outra unidade e ver que o lead não aparece; percorrer o funil de 14 etapas, os motivos e o console da fase 1.
- **Resultado esperado:** lead na fila certa, boas-vindas sem contar como atendimento, resposta medida, modelo com custo, painel batendo, unidades separadas e a base da fase 1 reconhecida pela direção.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro; vale também como aceite da fase 1.

## Handoff e operação

- **Como demonstrar:** caixa da unidade e painel do dia, com dois leads `TESTE`.
- **Como operar depois:** o gestor abre o painel do dia de manhã e à tarde; o admin ajusta SLA e custos no console.
- **Como monitorar:** "fora do SLA", dead-letter e falhas de envio no painel.
- **Pendência conhecida:** operação com lead real só depois da conta Meta aprovada e da política de LGPD.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Enviar mensagens só pelo servidor, com a boas-vindas automática | Academia das Específicas | SPEC-3-001 | CA-3-01 | P3.1-01, P3.1-02 | saída do teste + log sanitizado | Número de teste da Meta | ☐ |
| pendente | Abrir a caixa de conversas de cada unidade com o cronômetro de 24 horas | Academia das Específicas | SPEC-3-001 | CA-3-02 | P3.1-03, P3.1-04, P3.1-05 | capturas + `cost_log` | Envio no ar | ☐ |
| pendente | Medir a primeira resposta humana e mostrar o painel do dia | Academia das Específicas | SPEC-3-001 | CA-3-03, CA-3-04, CA-3-05 | P3.1-06, P3.1-07 | consulta × painel; E2E e pgTAP | Caixa no ar | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
