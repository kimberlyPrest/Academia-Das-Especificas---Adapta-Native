# Fase 3 — CRM operacional, da qualificação à matrícula (fechada)

Fechada pela consultora (Kim) em 10/11/2026.

- `fase.md`: as 25 tarefas `[x]`, com o resultado na linha `> Resultado:`.
- `specs/`: as SPECs 3-001 a 3-007 como foram executadas, e o índice da fase.
- `checklist-do-cliente.md`: o checklist CL-001 a CL-016 de 28/10, como histórico. Os itens que continuam valendo foram para o checklist da fase 4.

## O que foi entregue

As sete SPECs foram construídas entre 28/10 e 09/11 e estão na versão **v0.0.77** (consolidada no Skip, não publicada), com as provas automáticas (pgTAP e scripts) verdes no teste independente do executor:

| SPEC | Entrega | Construção |
|---|---|---|
| 3-001 | Atendimento pelo WhatsApp em teste: envio pelo servidor, boas-vindas, caixa com janela de 24h, SLA, K1, painel do dia | 28–29/10 |
| 3-002 | Ficha de qualificação, próxima ação, orçamento (MatriculaWeb só simulando, desligado na nuvem) e K2 | 30–31/10 |
| 3-003 | Follow-up depois do orçamento: FUP 1, FUP 2, tarefa e lembrete | 01–02/11 |
| 3-004 | Reunião com o diretor e negociação com alçada | até 03/11 |
| 3-005 | Matrícula com checklist e retomada do Interesse 2027 | 04/11 |
| 3-006 | Painel gerencial da origem à matrícula e auditoria de conversas | 05–06/11 |
| 3-007 | Importação do histórico do Kommo, só com arquivo de teste | 07–09/11 |

O teste humano das SPECs foi dado como feito pela consultora em 10/11/2026. Isso vale também como o aceite da fase 1 (CL-002).

## O que passou para a fase 4

- **Teste com o número de teste da Meta** (envio e recebimento de verdade pela Cloud API): SPEC-4-001.
- **Teste com a API real do MatriculaWeb** (unidades, turmas e simulação): SPEC-4-001. A gravação do orçamento no MatriculaWeb entra na SPEC-4-006.
- **Importação real do Kommo** (chave nova, data de corte): SPEC-4-007.
- **Teste E2E da caixa de conversas**: a dívida de 19/10 continua aberta (o projeto ainda não tem teste de frontend) e passa para a SPEC-4-002.
