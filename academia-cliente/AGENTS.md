# AGENTS.md — Academia das Específicas

Agente de execução do projeto **Central de Atendimento e Conversão Comercial** (Academia das Específicas). Vale para qualquer executor: Maestro, Claude Code ou outro agente usado pela equipe.

## Regras

1. Leia `CLAUDE.md`, `STATUS.md` e `04_fase-atual/specs/00-INDICE.md` antes de qualquer trabalho.
2. Execute somente tarefas de `04_fase-atual/fase.md`, pela SPEC de origem em `04_fase-atual/specs/`.
3. Construa as tasks de cada SPEC em sequência, uma em construção por vez, com as provas de cada uma, sem pedir autorização nem teste entre elas. Chame o teste do cliente da SPEC (`## Teste humano do cliente`) quando todas as tasks dela estiverem no ar.
4. Critérios de aceite e provas (TDD) vivem na SPEC; a task não cria contrato paralelo.
5. Toda ação externa real (lead real, envio real pela Meta com o número de produção, leitura do Kommo real, importação de dado real) acontece na virada da SPEC-4-007, com a consultora. Exceções da fase 4: o número de teste da Meta (SPEC-4-001), a API real do MatriculaWeb para consultar, simular e gravar orçamento `TESTE` (SPECs 4-001 e 4-006) e o Gemini pela API paga (SPEC-4-004). Nada é escrito no Kommo.
6. Débitos e simplificações vão para `06_notas/dividas.md`, nunca silenciosos.

## Teste do cliente por SPEC — fase 4

Um teste por SPEC, quando todas as tasks dela estiverem no ar, pelo roteiro de `## Teste humano do cliente`. Task sem nada para o cliente ver fecha pela prova automática. Enquanto o teste de uma SPEC não acontece, a construção segue pela SPEC seguinte.

| SPEC | Quem testa | Observação |
|---|---|---|
| SPEC-4-001 | Luiz, com o Hikari | Número de teste da Meta e MatriculaWeb real |
| SPEC-4-002 | Luiz, com um atendente de cada unidade | — |
| SPEC-4-003 | Luiz | — |
| SPEC-4-004 | Luiz, com o Hikari e atendentes | Só depois da bateria de avaliação verde |
| SPEC-4-005 | Luiz, com um atendente | — |
| SPEC-4-006 | Luiz, com um atendente | Um orçamento `TESTE` real no MatriculaWeb |
| SPEC-4-007 | Luiz, gestores e atendentes | Ensaio e virada, com a consultora |

## Arquivo de tarefas lido pelo portal

`04_fase-atual/fase.md` espelha o quadro do portal. Ao concluir uma task:
- marque `[x]` e registre o resultado (versão, provas e teste do cliente) numa linha `> ` embaixo da task;
- não escreva nada no título depois do prazo `!dd/mm/aaaa`;
- não apague o `<!-- id:… -->`. Linha sem id vira cartão novo no portal; os ids chegam pelo consultor.

## Fases: só o consultor libera — 28/10/2026

Vale para todas as fases.

- Só o consultor libera fase. Ele publica `04_fase-atual/` com SPECs, tarefas e checklist; os ids das tarefas vêm do portal.
- Em `04_fase-atual/` o executor só marca o estado das tarefas (`[ ]`, `[/]`, `[x]`), registra o resultado na linha `> ` e preenche o checklist de execução das SPECs. Não cria, renomeia, apaga nem reescreve tarefa, SPEC, critério, emenda, índice ou checklist.
- **O executor não cria SPEC, tarefa nem id.** Nunca cria UUID nem id de checklist (`CL-…`). Pedido novo do dono vai para o consultor.
- **"Pode continuar" não libera fase.** "Pode continuar", "pode seguir", "passou" ou "testei" do dono valem como aceite só quando respondem ao roteiro de `## Teste humano do cliente` de uma SPEC. Fora disso, significam seguir para a próxima tarefa da fase atual.
- Ao terminar a última SPEC da fase: registre `etapa: fase_N_aguardando_liberacao` em `06_notas/estado-atual.md`, diga ao dono "a fase N terminou; a próxima fase é liberada pelo consultor" e pare. Não escreva SPEC, tarefa, id, prazo, checklist nem `fase.md` de fase nova.
- O recorte executável está nas SPECs e no índice da fase liberada. Os contratos antigos das fases 3 a 5 em `05_entregas/historico-contratos-fases-3-a-5/` são histórico: não definem fase, tarefa nem ordem.
- Divergência entre SPEC, escopo e pedido do dono vai para o consultor. Registre em `06_notas/` e siga com o que a SPEC já fecha.
- Não marque `[x]` sem as provas da task. A SPEC só fecha com o aceite explícito do roteiro de teste dela.

## Dados e segredos

- Testes rodam no mesmo banco da operação: todo registro de teste leva `is_test = true` e nome começando com `TESTE`.
- Nenhum dado real de aluno ou responsável antes da virada (SPEC-4-007). A política LGPD formal foi dispensada (Emenda 01); RLS, segredos no servidor, logs sem texto de mensagem e mídia privada continuam obrigatórios.
- Chave, token ou senha só no segredo do servidor (Edge Functions). Nunca no repositório, no navegador, em log ou no grupo.
- Não escreva no repositório nome, telefone, e-mail ou documento de aluno ou responsável.
