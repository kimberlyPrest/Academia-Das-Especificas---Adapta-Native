# Fase 1 — Fundação determinística (sem WhatsApp produtivo)

**Projeto:** Academia das Específicas — Central de Atendimento e Conversão Comercial
**Fase:** 1 de 5 · **Status:** pronta para execução
**Resultado da fase:** base Supabase operando com autenticação, papéis, RLS por unidade, funil configurável e demonstração com fixture sintética.
**Fora desta fase:** canal WhatsApp real, dados reais do Kommo, agentes de IA, migração histórica.

## Tasks

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| T1.1 | Bootstrap do projeto Supabase (repo, config, ambientes) | Consultoria | SPEC-1-001 | CA-1-01 | Reset em clone limpo | Log de reset sem erro | Escopo aprovado | ☐ |
| T1.2 | Migrations de auditoria append-only + filas duráveis + dead-letter | Consultoria | SPEC-1-001 | CA-1-04, CA-1-05 | pgTAP de append-only e dead-letter | Testes pgTAP verdes | T1.1 | ☐ |
| T1.3 | Seed sintético idempotente sem PII + varredura de segredos | Consultoria | SPEC-1-001 | CA-1-02, CA-1-03 | Reset duplo + busca de segredos | Saída idêntica; zero segredos | T1.2 | ☐ |
| T1.4 | Migrations de identidade (units, users, role_assignments) + convite | Consultoria | SPEC-1-002 | CA-1-11 | Convite de usuário de teste | Usuário criado com papel/unidade | T1.3 | ☐ |
| T1.5 | Policies RLS por papel/unidade + testes pgTAP | Consultoria | SPEC-1-002 | CA-1-06, CA-1-07, CA-1-08 | Suite pgTAP de RLS | Testes verdes | T1.4 | ☐ |
| T1.6 | MFA de admin + desativação de usuário (provas negativas) | Consultoria | SPEC-1-002 | CA-1-09, CA-1-10 | Fluxo manual com contas de teste | Capturas de bloqueio | T1.5 | ☐ |
| T1.7 | Tabelas de configuração (stages, catálogos, checklist) + seed do processo real | Consultoria | SPEC-1-003 | CA-1-13 | Inspeção do funil populado | 15 etapas + catálogos | T1.6 | ☐ |
| T1.8 | RPC de transição de etapa com validação server-side (motivo/checklist) | Consultoria | SPEC-1-003 | CA-1-14, CA-1-15 | RPC de teste negativa | Bloqueios com erro claro | T1.7 | ☐ |
| T1.9 | Console administrativo (funil, catálogos, checklist) + auditoria de configuração | Consultoria | SPEC-1-003 | CA-1-12, CA-1-16, CA-1-17, CA-1-18 | Demonstração guiada | Vídeo + auditoria | T1.8 | ☐ |
| T1.10 | Fixture sintética rica (20 leads, transições, desqualificações) | Consultoria | SPEC-1-004 | CA-1-19 | Conferência numérica consulta×seed | Números batem | T1.9 | ☐ |
| T1.11 | Painel-base com filtros e pendências de gestão | Consultoria | SPEC-1-004 | CA-1-20, CA-1-21, CA-1-22, CA-1-23 | Fluxo manual + consulta de conferência | Capturas + consulta | T1.10 | ☐ |
| T1.12 | Roteiro de demonstração + sessão de aceite com champion/direção | Consultoria | SPEC-1-004 | CA-1-24 | Demonstração guiada | Ata de aceite | T1.11 | ☐ |

## Checklist de aceite da fase

- [ ] Login com MFA para admin; usuários por papel/unidade
- [ ] RLS impede atendente de outra unidade (teste de policy)
- [ ] Funil com 15 etapas configuráveis sem deploy
- [ ] Catálogos editáveis (origens, motivos, produtos, checklist)
- [ ] Fixture sintética sem PII real carregada
- [ ] Trilha de auditoria append-only em toda transição
- [ ] Demonstração realizada e aceite registrado

## Próximas fases (arco)

1. **Fase 1** — Fundação determinística (esta).
2. **Fase 2** — Canal WhatsApp + inbox + entrada de leads (após GATE-03/09/11).
3. **Fase 3** — CRM ponta a ponta + migração Kommo (após GATE-04..08/12).
4. **Fase 4** — IA assistida supervisionada (após GATE-13).
5. **Fase 5** — Validação integral, KPIs e retirada do Kommo (após GATE-02).
