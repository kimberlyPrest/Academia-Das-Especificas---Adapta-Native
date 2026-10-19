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
