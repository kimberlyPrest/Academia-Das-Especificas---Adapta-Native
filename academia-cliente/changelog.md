# Changelog — Academia das Específicas

## 2026-11-10 — Fase 3 fechada, Emenda 01 e fase 4 liberada

- Fase 3 fechada pela consultora: SPECs 3-001 a 3-007 construídas entre 28/10 e 09/11 (v0.0.77), provas automáticas verdes, teste humano dado como feito; vale também como aceite da fase 1. Arquivada em `05_entregas/fase-3/` (`fase.md` com as 25 tarefas `[x]` e o resultado, SPECs, índice e checklist CL-001 a CL-016).
- Pendências da fase 3 levadas para a fase 4: teste com o número de teste da Meta e com a API real do MatriculaWeb (SPEC-4-001), importação real do Kommo (SPEC-4-007), E2E da caixa (SPEC-4-002).
- Publicada a **Emenda 01** ao escopo (`03_documentos/04-Emenda-IA-no-Atendimento.md`): IA atende fora do horário e na abertura (DEC-04), persona com nome (DEC-13), transbordo como etiqueta com fila (DEC-14), tudo da IA editável no admin (DEC-15), Gemini 2.5 Flash pela API paga (DEC-16), ficha só na central (DEC-17), orçamento gravado no MatriculaWeb e enviado em PDF (DEC-18), contatos da mesma compra (DEC-19); novo AG-14; K1a e K1b; GATE-09 dispensado; GATE-13 fechado. Substitui a minuta de revisão do arco (19/10).
- Fase 4 publicada em `04_fase-atual/`: SPECs 4-001 a 4-007, 27 tarefas sem id (os ids chegam pelo portal), critérios CA-4-01 a CA-4-31, checklist CL-017 a CL-028 e anexos (modelo do PDF de orçamento e carga inicial da área Assistente).
- `AGENTS.md` e `CLAUDE.md`: teste do cliente por SPEC da fase 4; dado real liberado na virada com a consultora (GATE-09 dispensado).
- `06_notas/dividas.md`: dívidas da fase 3 atualizadas e novas linhas da fase 4.
- Matriz de rastreabilidade atualizada e `handoff-manifest.json` regenerado para a fase 4.

## 2026-10-28 — Fases 1 e 2 fechadas e fase 3 liberada

- Fase 1 fechada pelo consultor: T1.1 a T1.11 com o resultado de 07/10 (v0.0.27) e as emendas de 10/10; a demonstração e o aceite da direção (T1.12) passam para o teste da SPEC-3-001. Arquivada em `05_entregas/fase-1/` (`fase.md` com os ids do portal, quadro antigo e SPECs 1-001 a 1-004).
- Fase 2 fechada pelo consultor com T2.1 e T2.2 (sandbox, v0.0.33 e v0.0.36). T2.3, T2.4 e T2.5 passam para a SPEC-3-001. Arquivada em `05_entregas/fase-2/` (`fase.md` com os ids do portal e SPEC-2-001 com a emenda de 28/10).
- Fase 3 publicada em `04_fase-atual/`: SPECs 3-001 a 3-007, 25 tarefas sem id (os ids chegam pelo portal), critérios CA-3-01 a CA-3-27 e checklist CL-001 a CL-016.
- Orçamento da fase 3 usa a API pública de orçamento do MatriculaWeb só para consultar e simular; nada é gravado lá (resposta ao pedido de 16/10).
- Contratos de 07/10 das fases 3 a 5 movidos para `05_entregas/historico-contratos-fases-3-a-5/`.
- `AGENTS.md`: só o consultor libera fase; "pode continuar" não libera fase; o executor não cria SPEC, tarefa nem id; massa de teste `TESTE`; segredos só no servidor.
- `06_notas/dividas.md`: registro de próxima ação pela tela e E2E da caixa entram na fase 3; novas linhas para massa de teste no banco da operação e orçamento só simulado.
- `handoff-manifest.json` regenerado para a fase 3.

## 2026-10-19 — Sincronização com o sistema do cliente (v0.0.36) e proposta de revisão do arco

- Fase 1: T1.1–T1.11 marcadas como concluídas; registradas as emendas T1.13 (14 etapas, FUP 3 arquivada, 7 motivos — GATE-08 fechado) e T1.14 (quadro com arrastar, pendência clicável); CA-1-13 passa a 14 etapas; checklist reinclui a política LGPD provisória.
- Fase 2: T2.1 e T2.2 concluídas em sandbox; SPEC-2-001 recebe as emendas de 12 e 13/10 (contato sem adivinhação, deduplicação só dentro da unidade, fila única sem dono, nomes longos) e a correção AG-01..05 → AG-01..03 + AG-05; criada a T2.5 (envio + boas-vindas) e os critérios CA-2-08 e CA-2-09; GATE-01 e GATE-11 explicitados.
- `06_notas/dividas.md`: GATE-08 quitado; registradas FUP 3 como lembrete, AG-03 simplificado, `next_actions` sem registro pela tela e ausência de testes de frontend.
- Matriz de rastreabilidade atualizada (T1.13, T1.14, T2.5, CA-2-08/09).
- Publicada a minuta `03_documentos/03-Proposta-Revisao-Arco-Fases.md` (não vigente até aprovação).

## 2026-10-07 — Resposta aos pedidos técnicos + pacote das Fases 2–5

- Respondidos os 9 blocos de pedidos técnicos de 02/10 (documento do Hikari).
- Publicados em `03_documentos/`: `matriz-de-rastreabilidade.md` (REQ-F1-01 a REQ-F5-01) e `00-DMO.md` (29/08).
- Publicadas as SPECs das Fases 2–5 em `03_documentos/fases/specs/` (SPEC-2-001 completa; SPEC-3-001, SPEC-4-001 e SPEC-5-001 como contrato de fase, conforme DEC-EXEC-03).
- Publicados os quadros de tasks das Fases 2–5 em `03_documentos/fases/` (fase_2.md a fase_5.md).
- STATUS atualizado: execução da Fase 1 pelo cliente (T1.1–T1.11 aprovadas em teste independente, v0.0.27); T1.12 (sessão de aceite) pendente.

## 2026-09-21 — Handoff da Fase 1

- Escopo definitivo v1.0 publicado (22 seções, 12 DEC, 13 agentes, 14 gates, 5 fases).
- DEC-EXEC-01 (Supabase), DEC-EXEC-02 (React+TS+Vite) e DEC-EXEC-03 (liberação das 5 fases) registradas.
- SPECs da Fase 1 (4) com 24 critérios de aceite e TDD publicadas em `04_fase-atual/specs/`.
- 12 tasks da Fase 1 publicadas em `04_fase-atual/fase.md`.
- Pasta do cliente criada e validada.
