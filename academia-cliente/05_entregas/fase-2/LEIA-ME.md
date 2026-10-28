# Fase 2 — Canal WhatsApp, caixa e entrada de leads (fechada)

Fechada pelo consultor em 28/10/2026 com o que estava registrado no repositório até 19/10.

- `fase.md`: as 4 tarefas com os ids do portal, todas `[x]`, com o resultado na linha `> `.
- `specs/SPEC-2-001.md`: a SPEC da fase, com a emenda de 28/10.

Concluído em sandbox:
- webhooks do WhatsApp e dos formulários, com assinatura conferida, idempotência e dead-letter (12/10, v0.0.33);
- validação do contato, deduplicação dentro da unidade e distribuição em fila única (12 e 13/10, v0.0.36).

Passaram para a SPEC-3-001 da fase 3:
- caixa de conversas por unidade, janela de 24 horas e modelos com custo (antiga T2.3);
- SLA, K1 com resposta humana e painel do dia (antiga T2.4);
- envio pelo servidor e boas-vindas (antiga T2.5, que não chegou a virar cartão no portal).

Se essas partes já foram construídas depois de 19/10, as tasks da SPEC-3-001 fecham rodando as provas na versão atual, sem reconstruir.
