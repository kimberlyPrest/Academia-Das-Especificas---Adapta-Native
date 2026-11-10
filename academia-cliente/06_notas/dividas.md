# Dividas e simplificações

Registro append-only de débitos técnicos e simplificações aceitas. Nunca silenciosos no código.

| Data | Origem | Dívida/simplificação | Condição de pagamento |
|---|---|---|---|
| 2026-09-21 | Handoff F1 | Score do Kommo importado como dado, sem uso em decisão (regra de cálculo desconhecida) | GATE-10 decidir manter/recalcular/descartar |
| 2026-09-21 | Handoff F1 | Réguas de SLA/FUP com valores atuais (20h/24h) até GATE-07 confirmar contagem | GATE-07 |
| 2026-09-21 | Handoff F1 | Catálogo de motivos de desqualificação inicial (6 itens) até GATE-08 homologar | GATE-08 |
| 2026-10-19 | Emenda T1.13 (10/10) | **Paga:** catálogo de motivos homologado com 7 itens em 08/10 (GATE-08 fechado) — quita a linha de 2026-09-21 sobre o catálogo inicial | — |
| 2026-10-19 | Emenda T1.13 (10/10) | FUP 3 deixa de ser coluna e vira lembrete; régua de 20h/24h segue provisória | GATE-07 (contagem de prazos e destino pós-FUP 3) |
| 2026-10-19 | Emenda SPEC-2-001 (12/10) | AG-03 simplificado: fila única por unidade, sem dono (sem round-robin/responsável) | Revisar na Fase 3 se a operação pedir responsável por lead |
| 2026-10-19 | T1.11 | `next_actions` só gravada por fixture/servidor; registro pela tela aguarda a equipe informar como marca a próxima ação hoje no Kommo | Antes da T3.1 (K2) |
| 2026-10-19 | Revisão da consultoria | Projeto sem testes de frontend (Vitest/Playwright, DEC-EXEC-02); provas são pgTAP + roteiros bash | Antes do canal produtivo (Fase 2), ao menos E2E do fluxo de inbox |
| 2026-10-28 | Liberação da fase 3 | **Em pagamento:** registro da próxima ação pela tela entra na SPEC-3-002 (linha de 2026-10-19 sobre `next_actions`) | Aceite da SPEC-3-002 |
| 2026-10-28 | Liberação da fase 3 | **Em pagamento:** E2E do fluxo da caixa entra na SPEC-3-001 (linha de 2026-10-19 sobre testes de frontend) | Aceite da SPEC-3-001 |
| 2026-10-28 | Orientação de 20/10 | Testes no mesmo banco da operação, com `is_test` e nome `TESTE`; a auditoria é só de inclusão, então a massa de teste fica registrada | SPEC própria de limpeza antes do primeiro dado real (GATE-03 e GATE-09) |
| 2026-10-28 | SPEC-3-002 | Orçamento só consulta e simula no MatriculaWeb; não grava orçamento nem lead lá | Decisão do consultor sobre integração de escrita |
| 2026-10-28 | SPEC-3-004 | Reunião registrada na central, sem integração com a agenda do diretor | GATE-06 |
| 2026-11-10 | Fechamento da fase 3 | **Paga:** registro da próxima ação pela tela (SPEC-3-002) — quita a linha de 2026-10-19 sobre `next_actions` | — |
| 2026-11-10 | Fechamento da fase 3 | **Continua aberta:** o projeto ainda não tem teste de frontend (`package.json` sem Vitest nem Playwright); o E2E da caixa passa para a SPEC-4-002 | Aceite da SPEC-4-002 |
| 2026-11-10 | Fechamento da fase 3 | Fase fechada sem o teste com o número de teste da Meta e com a API real do MatriculaWeb | Aceite da SPEC-4-001 |
| 2026-11-10 | Emenda 01 | **Paga em parte:** orçamento passa a ser gravado no MatriculaWeb (SPEC-4-006) — quita a linha de 2026-10-28 sobre orçamento só simulado | Aceite da SPEC-4-006 |
| 2026-11-10 | Emenda 01 | **Em pagamento:** fila sem dono (AG-03 simplificado, 12/10) — a conversa passa a ter dono ao assumir (SPEC-4-002); a distribuição continua por fila única da unidade | Aceite da SPEC-4-002 |
| 2026-11-10 | Emenda 01 | Política LGPD formal dispensada pela consultora; ficam as proteções técnicas (RLS, segredos no servidor, logs sem texto, mídia privada com prazo de guarda, IA pela API paga) | Revisar se a Academia pedir política formal |
| 2026-11-10 | Emenda 01 | A IA usa um nome de atendente; se perguntarem, ela não nega que é atendimento virtual e passa para uma pessoa | Revisar com a Academia depois da Black Friday |
| 2026-11-10 | SPEC-4-006 | A API pública do MatriculaWeb não gera PDF; o PDF é gerado pela central com os valores da gravação | Rota de PDF no próprio MatriculaWeb, se a Academia quiser o arquivo idêntico ao do sistema interno |
| 2026-11-10 | SPEC-4-006 | A API pública do MatriculaWeb não tem login e limita por IP | Liberação de limite para o servidor da central (CL-025) |
