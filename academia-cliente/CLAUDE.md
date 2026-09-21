# CLAUDE.md — Academia das Específicas

## Objetivo do projeto

Construir a **Central de Atendimento e Conversão Comercial** da Academia das Específicas: sistema próprio (Supabase + React/TypeScript/Vite) que estrutura e automatiza o atendimento e follow-up comercial — lead → atendimento → qualificação → orçamento → follow-up → negociação → matrícula/perda — preservando o atendimento humano e consultivo.

## Contexto

Cursinho pré-vestibular (ENEM, UnB, PAS/UnB, particulares de medicina do DF) com unidades Asa Sul e Taguatinga. O gargalo não é geração de demanda (60–65% das matrículas vêm de indicação), e sim a conversão: leads sem resposta rápida, sem próxima ação e com cadência manual. O funil atual roda no Kommo; a migração acontece por export na Fase 3, com operação paralela até a Fase 5.

## Champion

Luiz Fernando (direção) — responsável pelo projeto do lado da Academia.

## Consultor

Kim (consultoria técnica Adapta Native).

## Limites do champion

- Executar as tarefas da fase atual conforme as SPECs de `04_fase-atual/specs/`.
- Registrar dúvidas e bloqueios em `06_notas/` — não improvisar fora do contrato da SPEC.
- Não ativar integrações reais (WhatsApp/Meta), não importar dados reais e não operar fora de sandbox sem o gate correspondente aprovado.

## Linha vermelha

Nenhum dado real de aluno/responsável entra no sistema antes da política de LGPD aprovada (GATE-09). Nenhuma mensagem real é enviada antes do gate do canal (GATE-03). Aprovar escopo/SPEC autoriza execução da fase; **não** autoriza integrações, envios ou migração.

## Formato da dívida

Débitos e simplificações ficam registrados em `06_notas/dividas.md` com origem, motivo e condição de pagamento — nunca silenciosos no código.
