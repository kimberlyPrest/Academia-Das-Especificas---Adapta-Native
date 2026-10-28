# STATUS — Academia das Específicas

**Fase atual:** 3 — CRM operacional, da qualificação à matrícula (liberada em 28/10/2026)
**Data:** 2026-10-28

## Estado

- **Fase 1 fechada** pelo consultor em 28/10: T1.1 a T1.11 aprovadas em teste independente (v0.0.27, 07/10), com as emendas de 10/10 (14 etapas, 7 motivos, quadro com arrastar). O aceite da direção (antiga T1.12) passou para o teste da SPEC-3-001. Arquivo em `05_entregas/fase-1/`.
- **Fase 2 fechada** pelo consultor em 28/10 com o que estava registrado até 19/10: webhooks com idempotência e dead-letter (v0.0.33) e validação, deduplicação e distribuição (v0.0.36), em sandbox. Caixa de conversas, envio com boas-vindas e SLA com painel do dia (antigas T2.3, T2.4 e T2.5) passaram para a SPEC-3-001; o que já estiver construído fecha rodando as provas na versão atual. Arquivo em `05_entregas/fase-2/`.
- **Fase 3 liberada:** 7 SPECs, 25 tarefas, critérios CA-3-01 a CA-3-27 e checklist da Academia (CL-001 a CL-016). Tudo com massa de teste `TESTE`; nenhuma gravação no MatriculaWeb ou no Kommo.
- Os contratos de 07/10 das fases 3 a 5 foram arquivados em `05_entregas/historico-contratos-fases-3-a-5/`. As fases 4 e 5 serão geradas na liberação de cada uma.
- A minuta `03_documentos/03-Proposta-Revisao-Arco-Fases.md` continua não vigente. A ordem das SPECs da fase 3 já põe primeiro o que ataca a Black Friday e a importação do Kommo por último.

## Gates

- **Fechado:** GATE-08 (7 motivos, 08/10).
- **Não travam a construção da fase 3:** GATE-01 (SLA), GATE-05 (alçadas), GATE-06 (agenda do diretor), GATE-07 (prazos de follow-up), GATE-11 (formulários), GATE-12 (marco de conversão). Até a decisão, valem os padrões das SPECs.
- **Travam só a operação real:** GATE-03 (conta Meta, número, modelos; API oficial escolhida em 19/10), GATE-09 (LGPD) e GATE-04 (exportação do Kommo, com chave nova).
- GATE-13 → fase 4. GATE-02 (baseline) → fase 5.

## Próximo passo

1. SPEC-3-001, começando pelo envio pelo servidor com a boas-vindas.
2. Teste do cliente uma vez por SPEC, no fim dela. O teste da SPEC-3-001 vale também como aceite da fase 1.
3. Só o consultor libera a fase 4.
