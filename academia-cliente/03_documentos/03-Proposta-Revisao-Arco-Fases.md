# Proposta de revisão do arco de fases (§18 do Escopo Definitivo)

| Item | Informação |
|---|---|
| **Status** | Minuta para decisão com o champion (Luiz Fernando) e a direção — **não altera o escopo até aprovada** |
| **Data** | 19/10/2026 |
| **Autora** | Kim (consultoria) |
| **Afeta** | §18 (fases), §20 (gates), quadros `03_documentos/fases/fase_3.md` a `fase_5.md` |

## Por que revisar

O escopo definitivo fixa as Fases 1–3 concluídas antes da **Black Friday (27/11/2026)**. Os quadros atuais distribuem o restante assim:

| Fase | Datas atuais | Bloqueio real |
|---|---|---|
| 2 | T2.3–T2.4 até 30/10 | GATE-03 (conta Meta) e GATE-09 (LGPD) abertos — sem eles não há operação real |
| 3 | 03/11 a 12/11 (7 tasks) | Cinco gates abertos: 04, 05, 06, 07 e 12 (o 08 fechou em 08/10) |
| 4 | 17/11 a 24/11 | IA em produção na semana da Black Friday |
| 5 | 26/11 a 04/12 | T5.1 congela baseline em 26/11 — exigiria 30 dias de operação real desde ~27/10 |

O ritmo de execução está bom (T2.1 e T2.2 entregues antes do prazo), mas três premissas não fecham: (1) o baseline de 30 dias de operação real, (2) a Fase 3 inteira com cinco gates abertos, (3) IA entrando junto com o pico de vendas.

## Proposta

**Até a Black Friday (27/11) — operar de verdade o que ataca P1, P2 e P3:**

1. **Fase 2 completa em produção** (T2.3, T2.4, T2.5 + go-live). Pré-condição: GATE-01, GATE-03, GATE-09 e GATE-11 decididos até **31/10**.
2. **Recorte mínimo da Fase 3** (núcleo do funil):
   - T3.1 — qualificação obrigatória + próxima ação obrigatória (K2);
   - T3.2 — FUP 1/2 automáticos (o FUP 3 já virou lembrete na emenda de 10/10), com GATE-07;
   - T3.4 — checklist de matrícula + marco de conversão (GATE-12);
   - T3.6 — painel gerencial origem→matrícula.
3. **Convivência com o Kommo sem importação:** a partir do go-live, todo lead **novo** entra só no sistema novo; os leads em andamento continuam no Kommo até se encerrarem. Elimina a importação (T3.7) do caminho crítico da Black Friday.

**Depois da Black Friday (dez/2026–jan/2027):**

4. **Restante da Fase 3:** T3.3 (reuniões + negociação, GATE-05/06), T3.5 (Interesse 2027 — a retomada de novembro deste ano é feita no Kommo) e T3.7 (import Kommo, GATE-04).
5. **Fase 4 (IA)** após estabilização do ciclo da Black Friday, mantido o GATE-13.
6. **Fase 5** no ciclo jan–fev/2027, com retirada do Kommo depois dele.

**Baseline do K3 (GATE-02) — mudar a fonte:** medir a conversão qualificado→matrícula a partir do **histórico do Kommo** (export do GATE-04, período comparável do ano anterior), em vez de esperar 30 dias de operação no sistema novo. A comparação K3 passa a ser: ciclo jan–fev/2027 no sistema novo × mesmo ciclo no Kommo.

## Mudanças a registrar no escopo se aprovada

| Seção | Mudança |
|---|---|
| Cabeçalho (Horizonte) | "Fases 1–3 antes da Black Friday" → "Fase 2 em produção + núcleo da Fase 3 antes da Black Friday" |
| §18 Fase 3 | Dividir em 3a (T3.1, T3.2, T3.4, T3.6 — até 27/11) e 3b (T3.3, T3.5, T3.7 — dez/2026) |
| §18 Fase 4/5 | Datas para depois da Black Friday; Fase 5 no ciclo jan–fev/2027 |
| §20 GATE-02 | Fonte do baseline = histórico do Kommo (export) |
| §20 GATE-03/09 | Considerar as duas unidades como empresas distintas: controladores de dados e, possivelmente, contas/números Meta e orçamentos separados |
| DEC-09 | Migração por export passa a ser pós-Black Friday; leads novos nascem no sistema novo |

## Calendário de gates necessário

| Gate | Decisão | Dono | Prazo proposto |
|---|---|---|---|
| GATE-01 | Número do SLA de primeira resposta e régua de alerta | Direção | 24/10 |
| GATE-03 | Conta Meta Business, número(s), templates aprovados, orçamento | Academia | 31/10 |
| GATE-09 | Política LGPD (retenção, consentimento, menor, duas empresas) | DPO/direção | 31/10 |
| GATE-11 | Padrão de formulários de captação | Marketing | 31/10 |
| GATE-07 | Contagem de prazos de FUP (corridos × úteis × horário) | Academia | 31/10 |
| GATE-12 | Marco oficial de conversão | Direção | 07/11 |
| GATE-04 | Export do Kommo (para o baseline) | Admin Kommo | 07/11 |
| GATE-05/06 | Alçadas de desconto; agenda do diretor | Direção | dez/2026 |

## Decisão pedida

- [ ] Aprovar o novo arco (Fase 2 + núcleo da Fase 3 até 27/11; restante pós-Black Friday).
- [ ] Aprovar o baseline do K3 a partir do histórico do Kommo.
- [ ] Aprovar o calendário de gates com donos e datas.
