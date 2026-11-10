# STATUS — Academia das Específicas

**Fase atual:** 4 — Atendimento com IA e virada para a operação real (liberada em 10/11/2026)
**Data:** 2026-11-10

## Estado

- **Fase 3 fechada** pela consultora em 10/11: as 7 SPECs (3-001 a 3-007) construídas entre 28/10 e 09/11, na v0.0.77 (consolidada, não publicada), com as provas automáticas verdes e o teste humano dado como feito. Isso fecha também o aceite da fase 1. Arquivo em `05_entregas/fase-3/`.
- **Emenda 01 ao escopo** (`03_documentos/04-Emenda-IA-no-Atendimento.md`): a IA atende fora do horário e na abertura, com nome por unidade, sem falar preço, e passa para a equipe pela etiqueta de transbordo. Gemini 2.5 Flash pela API paga. Orçamento gravado no MatriculaWeb e enviado em PDF. GATE-09 (política LGPD) dispensado; GATE-13 fechado.
- **Fase 4 liberada:** 7 SPECs, 27 tarefas, critérios CA-4-01 a CA-4-31 e checklist da Academia (CL-017 a CL-028). Construção com massa `TESTE`; a virada dos números é feita com a consultora.
- A minuta `03_documentos/03-Proposta-Revisao-Arco-Fases.md` foi substituída pela Emenda 01.

## Gates

- **Fechados:** GATE-08 (motivos, 08/10), GATE-13 (IA, 10/11). **Dispensado:** GATE-09 (10/11).
- **Travam só a virada:** GATE-03 (conta Meta, números no app da Academia, modelos) e GATE-04 (exportação do Kommo com chave nova).
- **Não travam:** GATE-01 (SLA), GATE-05 (alçadas), GATE-06 (agenda), GATE-07 (prazos de follow-up), GATE-11 (formulários), GATE-12 (marco de conversão). Valem os padrões das SPECs.
- GATE-02 (baseline) → fase 5, medido pelo histórico do Kommo.

## Pendências que vieram da fase 3

- Teste com o número de teste da Meta e com a API real do MatriculaWeb → SPEC-4-001.
- Importação real do Kommo → SPEC-4-007.
- E2E da caixa de conversas (dívida de 19/10) → SPEC-4-002.

## Próximo passo

1. SPECs na ordem 4-002, 4-003, 4-004, 4-005, 4-006, 4-001 e 4-007. Os testes com o número de teste da Meta (SPEC-4-001) ficam por último na construção, para não travar o resto.
2. Teste do cliente uma vez por SPEC, no fim dela.
3. Virada com a consultora, pelo roteiro da SPEC-4-007.
4. Só a consultora libera a fase 5.
