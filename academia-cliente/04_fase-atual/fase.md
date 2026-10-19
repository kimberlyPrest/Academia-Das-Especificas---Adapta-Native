# Fase 1 — Fundação determinística (sem WhatsApp produtivo)

**Projeto:** Academia das Específicas — Central de Atendimento e Conversão Comercial
**Fase:** 1 de 5 · **Status:** construída (T1.1–T1.11, T1.13, T1.14 ☑, v0.0.29 no Skip) — aguardando aceite formal (T1.12) e política LGPD provisória
**Resultado da fase:** base Supabase operando com autenticação, papéis, RLS por unidade, funil configurável e demonstração com fixture sintética.
**Fora desta fase:** canal WhatsApp real, dados reais do Kommo, agentes de IA, migração histórica.

## Tasks

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| T1.1 | Bootstrap do projeto Supabase (repo, config, ambientes) | Consultoria | SPEC-1-001 | CA-1-01 | Reset em clone limpo | Log de reset sem erro | Escopo aprovado | ☑ |
| T1.2 | Migrations de auditoria append-only + filas duráveis + dead-letter | Consultoria | SPEC-1-001 | CA-1-04, CA-1-05 | pgTAP de append-only e dead-letter | Testes pgTAP verdes | T1.1 | ☑ |
| T1.3 | Seed sintético idempotente sem PII + varredura de segredos | Consultoria | SPEC-1-001 | CA-1-02, CA-1-03 | Reset duplo + busca de segredos | Saída idêntica; zero segredos | T1.2 | ☑ |
| T1.4 | Migrations de identidade (units, users, role_assignments) + convite | Consultoria | SPEC-1-002 | CA-1-11 | Convite de usuário de teste | Usuário criado com papel/unidade | T1.3 | ☑ |
| T1.5 | Policies RLS por papel/unidade + testes pgTAP | Consultoria | SPEC-1-002 | CA-1-06, CA-1-07, CA-1-08 | Suite pgTAP de RLS | Testes verdes | T1.4 | ☑ |
| T1.6 | MFA de admin + desativação de usuário (provas negativas) | Consultoria | SPEC-1-002 | CA-1-09, CA-1-10 | Fluxo manual com contas de teste | Capturas de bloqueio | T1.5 | ☑ |
| T1.7 | Tabelas de configuração (stages, catálogos, checklist) + seed do processo real | Consultoria | SPEC-1-003 | CA-1-13 | Inspeção do funil populado | 15 etapas + catálogos | T1.6 | ☑ |
| T1.8 | RPC de transição de etapa com validação server-side (motivo/checklist) | Consultoria | SPEC-1-003 | CA-1-14, CA-1-15 | RPC de teste negativa | Bloqueios com erro claro | T1.7 | ☑ |
| T1.9 | Console administrativo (funil, catálogos, checklist) + auditoria de configuração | Consultoria | SPEC-1-003 | CA-1-12, CA-1-16, CA-1-17, CA-1-18 | Demonstração guiada | Vídeo + auditoria | T1.8 | ☑ |
| T1.10 | Fixture sintética rica (20 leads, transições, desqualificações) | Consultoria | SPEC-1-004 | CA-1-19 | Conferência numérica consulta×seed | Números batem | T1.9 | ☑ |
| T1.11 | Painel-base com filtros e pendências de gestão | Consultoria | SPEC-1-004 | CA-1-20, CA-1-21, CA-1-22, CA-1-23 | Fluxo manual + consulta de conferência | Capturas + consulta | T1.10 | ☑ |
| T1.12 | Roteiro de demonstração + sessão de aceite com champion/direção | Consultoria | SPEC-1-004 | CA-1-24 | Demonstração guiada | Ata de aceite | T1.11, T1.13, T1.14 | ☐ |
| T1.13 | Etapas e motivos decididos pela direção em 08/10 (14 etapas; FUP 3 arquivada; 7 motivos do GATE-08) | Cliente (Hikari + Skip) | SPEC-1-003 | CA-1-13 | pgTAP `etapas_motivos_decididos_test.sql` + reset duplo idêntico | Testes verdes; audit_log da mudança | T1.9 | ☑ |
| T1.14 | Quadro do funil com arrastar e pendência de gestão clicável | Cliente (Hikari + Skip) | SPEC-1-004 | CA-1-21 | Fluxo manual no quadro e no painel | Capturas | T1.11 | ☑ |

## Checklist de aceite da fase

- [x] Login com MFA para admin; usuários por papel/unidade
- [x] RLS impede atendente de outra unidade (teste de policy)
- [x] Funil com 14 etapas configuráveis sem deploy (emenda de 10/10; era 15)
- [x] Catálogos editáveis (origens, motivos, produtos, checklist)
- [x] Fixture sintética sem PII real carregada
- [x] Trilha de auditoria append-only em toda transição
- [ ] Política LGPD provisória documentada (GATE-09 aberto) — item do escopo §18 que faltava neste checklist
- [ ] Demonstração realizada e aceite registrado em ata (CA-1-24)

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| 2026-10-10 | Decisões da direção em 08/10 (atendimento, itens 5 a 8) | T1.13 → SPEC-1-003 / CA-1-13 | Funil passa a 14 etapas como no Kommo de hoje; FUP 3 arquivada (3º retorno vira lembrete, sem coluna); 7 motivos de desqualificação validados (GATE-08 fechado) |
| 2026-10-10 | Feedback da demonstração de 08/10 | T1.14 → SPEC-1-004 / CA-1-21 | Quadro com arrastar e pendência de gestão clicável |
| 2026-10-19 | Revisão da consultoria | Checklist da fase | Reincluído o item de política LGPD provisória (escopo §18) |

## Próximas fases (arco)

1. **Fase 1** — Fundação determinística (esta).
2. **Fase 2** — Canal WhatsApp + inbox + entrada de leads (após GATE-03/09/11).
3. **Fase 3** — CRM ponta a ponta + migração Kommo (após GATE-04..08/12).
4. **Fase 4** — IA assistida supervisionada (após GATE-13).
5. **Fase 5** — Validação integral, KPIs e retirada do Kommo (após GATE-02).
