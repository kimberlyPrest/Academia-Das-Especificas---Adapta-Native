# STATUS — Academia das Específicas

**Fase atual:** 1 — Fundação determinística (construída; aguardando aceite formal) · Fase 2 em execução em sandbox
**Data:** 2026-10-19

## Estado

- **Fase 1 construída:** T1.1–T1.11 ☑ e as emendas T1.13 (14 etapas, FUP 3 arquivada, 7 motivos — decisões da direção em 08/10) e T1.14 (quadro com arrastar, pendência clicável) ☑. GATE-08 fechado.
- **Falta para fechar a Fase 1:**
  1. T1.12 / CA-1-24 — ata de aceite com champion/direção (a demonstração de 08/10 gerou as emendas, mas o aceite não foi registrado).
  2. Política LGPD provisória documentada (checklist do escopo §18; reincluída no `fase.md`).
  3. Ritual de fechamento: arquivar em `05_entregas/fase-1/`, promover a Fase 2 a `04_fase-atual/`, atualizar handoff-manifest.
- **Fase 2 (sandbox, v0.0.36):** T2.1 ☑ (webhooks Meta HMAC + formulários, idempotência, dead-letter; funções publicadas e fechadas sem segredos) e T2.2 ☑ (AG-01..03 + revisão de duplicatas; emendas de 12 e 13/10). Restam T2.3 (inbox + janela 24h), T2.4 (SLA + K1 + painel diário) e a nova **T2.5** (envio `message-send` + boas-vindas — não tinha task).
- **Proposta de revisão do arco** publicada em `03_documentos/03-Proposta-Revisao-Arco-Fases.md` (minuta, não vigente): Fase 2 + núcleo da Fase 3 até a Black Friday; restante da Fase 3, IA e retirada do Kommo depois; baseline do K3 a partir do histórico do Kommo.
- Pendências de resposta ao cliente: 6.4 (Restaurar/Reverter) e 7.3 (hospedagem/retenção) — a confirmar com o Skip.

## Gates

- **Fechado:** GATE-08 (7 motivos, 08/10).
- **Caminho crítico (Fase 2 produtiva):** GATE-01 (número do SLA), GATE-03 (conta Meta/número/templates — considerar duas empresas), GATE-09 (LGPD — dois controladores), GATE-11 (formulários).
- GATE-04..07, 12 → Fase 3. GATE-13 → Fase 4. GATE-02 (baseline) → Fase 5.

## Próximo passo

1. Reunião com o Luiz Fernando: aceite da Fase 1 (ata), decisão sobre a proposta de revisão do arco e calendário de gates.
2. Em paralelo, a execução segue com T2.5 → T2.3 → T2.4 em sandbox.
