# STATUS — Academia das Específicas

**Fase atual:** 1 — Fundação determinística (sem WhatsApp produtivo)
**Data:** 2026-10-07

## Estado

- Escopo definitivo v1.0 aprovado pela consultora (DEC-EXEC-03 autorizou SPECs/tasks das 5 fases).
- 4 SPECs da Fase 1 publicadas em `04_fase-atual/specs/` com 24 critérios de aceite e TDD.
- 12 tasks da Fase 1 em `04_fase-atual/fase.md` (tabela operacional completa).
- **Execução em andamento pelo cliente (Hikari + Claude Code): T1.1 a T1.11 construídas e aprovadas em teste independente (v0.0.27 no Skip; 15 migrations no banco de desenvolvimento). Falta a T1.12 — sessão de aceite com a direção.**
- **2026-10-07: respondidos os pedidos técnicos de 02/10 (9 blocos). Publicados em `03_documentos/`: matriz de rastreabilidade (REQ-F1..F5), SPEC-2-001 e SPECs-contrato das Fases 3–5, quadros de tasks das Fases 2–5 (`03_documentos/fases/`) e 00-DMO.md.**
- Pendências de resposta ao cliente: itens 6.4 (Restaurar/Reverter), 7.2 (onde desligar o selo) e 7.3 (hospedagem/retenção) — a confirmar com o Skip. Itens 7.1/7.2 já respondidos com verificação do código do skip.js (pixel de visita + selo desligável via showBadge).

## Gates abertos que bloqueiam fases futuras

- GATE-03 (conta Meta/número/templates) e GATE-09 (LGPD) → Fase 2.
- GATE-04..08, 12 → Fase 3.
- GATE-13 → Fase 4. GATE-02 (baseline) → Fase 5.

## Próximo passo

Agendar e realizar a T1.12 (sessão de aceite da Fase 1 com a direção). Após a ata, executar o ritual de fechamento de fase (arquivar em `05_entregas/fase-1/`, promover Fase 2 a `04_fase-atual/`, atualizar STATUS/changelog/handoff-manifest).
