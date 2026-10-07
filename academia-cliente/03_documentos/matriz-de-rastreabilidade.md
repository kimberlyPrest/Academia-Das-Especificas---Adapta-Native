# Matriz de rastreabilidade

| Fase | Requisito | Origem | Resultado observável | SPEC | Critério de aceite | Prova | Tasks |
|---|---|---|---|---|---|---|---|
| 1 | REQ-F1-01 — Ambiente Supabase reproduzível e auditável | DEC-01, DEC-09, DEC-10; Fase 1 | Banco local recriado por migrations e seed sem PII | SPEC-1-001 | CA-1-01 a CA-1-05 | `supabase db reset`, testes pgTAP e inspeção de segredos | T1.1, T1.2, T1.3 |
| 1 | REQ-F1-02 — Identidade, papéis e segregação por unidade | DEC-06; §4; Fase 1 | Usuários de teste acessam apenas dados permitidos | SPEC-1-002 | CA-1-06 a CA-1-11 | pgTAP de RLS + demonstração autenticada | T1.4, T1.5, T1.6 |
| 1 | REQ-F1-03 — Funil e catálogos configuráveis sem deploy | DEC-07, DEC-11; §6.8 | Admin altera etapas, SLA e catálogos por interface | SPEC-1-003 | CA-1-12 a CA-1-18 | testes de domínio/UI e demonstração | T1.7, T1.8, T1.9 |
| 1 | REQ-F1-04 — Demonstração com fixture sintética e painel-base | DEC-09, DEC-12; §17; Fase 1 | Gestor navega funil sintético e vê indicadores-base | SPEC-1-004 | CA-1-19 a CA-1-24 | reset/seed, testes de métricas e roteiro de demo | T1.10, T1.11, T1.12 |
| 2 | REQ-F2-01 — Entrada real de leads e inbox WhatsApp | DEC-03; Fase 2 | Capacidade prevista; detalhamento em onda | SPEC-2-001 | CA-2-01 a CA-2-07; Gates 03, 09 e 11 | ver SPEC-2-001 (TDD) | T2.1, T2.2, T2.3, T2.4 |
| 3 | REQ-F3-01 — CRM ponta a ponta e migração Kommo | DEC-02, DEC-08, DEC-09; Fase 3 | Capacidade prevista; detalhamento em onda | SPEC-3-001 | CA-3-01 a CA-3-08; Gates 04–08 e 12 | ver SPEC-3-001 (TDD) | T3.1 a T3.7 |
| 4 | REQ-F4-01 — IA assistida supervisionada | DEC-04; Fase 4 | Capacidade prevista; detalhamento em onda | SPEC-4-001 | CA-4-01 a CA-4-06; GATE-13 | ver SPEC-4-001 (TDD) | T4.1, T4.2, T4.3 |
| 5 | REQ-F5-01 — Validação integral e retirada do Kommo | DEC-12; Fase 5 | Capacidade prevista; detalhamento em onda | SPEC-5-001 | CA-5-01 a CA-5-06; GATE-02 | ver SPEC-5-001 (TDD) | T5.1, T5.2, T5.3 |
