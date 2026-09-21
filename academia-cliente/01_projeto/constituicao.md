# Constituição do projeto

## Papéis

| Papel | Quem | Responsabilidade |
|---|---|---|
| Champion | Luiz Fernando (direção) | Prioridade, validações e decisões de negócio |
| Consultoria | Kim | Implementação, SPECs e acompanhamento técnico |
| Gestores de unidade | Asa Sul / Taguatinga | Validação operacional das telas e fluxos |
| Atendentes | Operação comercial | Uso diário e feedback |

## Stack permitida

- **Backend/banco:** Supabase (Postgres, Auth, RLS, Edge Functions, Realtime, Storage, Vault, Cron).
- **Frontend:** React + TypeScript + Vite (testes com Vitest; E2E com Playwright).
- **Canal:** WhatsApp via Meta Cloud API (oficial).

## Limites do champion

- Executar tarefas conforme as SPECs; dúvidas e bloqueios em `06_notas/`.
- Sem integrações reais, dados reais ou envio de mensagens sem o gate correspondente aprovado.

## Linha vermelha

Nenhum dado real de aluno/responsável antes da política de LGPD aprovada. Nenhuma mensagem real antes do gate do canal. Aprovar documentos autoriza execução da fase — não autoriza operação real.

## Formato da dívida

Débitos registrados em `06_notas/dividas.md` com origem, motivo e condição de pagamento.
