# Fase 4 — Atendimento com IA e virada para a operação real

Liberada pela consultora em 10/11/2026, com a **Emenda 01** (`03_documentos/04-Emenda-IA-no-Atendimento.md`). A fase 4 leva a central para o WhatsApp de verdade antes da Black Friday:
- **Meta e MatriculaWeb de verdade:** testes que ficaram da fase 3, mídia recebida e enviada, conversa marcada como lida.
- **Conversa com dono:** assumir, devolver e pausar; a IA desligada em três níveis; transbordo com etiqueta e fila por janela e temperatura.
- **Área Assistente no admin:** nome, horários, jeito de atender, FAQ, exemplos e textos do orçamento, com simulação e versões.
- **Atendente virtual (Gemini):** responde fora do horário e na abertura, qualifica, nunca fala preço e passa para a equipe com o resumo.
- **Contatos da mesma compra:** mãe, pai e aluno no mesmo lead; indicação é lead novo.
- **Orçamento:** gravado no MatriculaWeb, em PDF no modelo da Academia, enviado com a pergunta final.
- **Métricas e virada:** K1a, K1b, números por atendente e da IA; limpeza do teste, ensaio e virada dos números.

Tudo é construído com massa `TESTE`. A virada para os números reais é feita com a consultora, depois do GATE-03 e do GATE-04 (checklist da fase).

Em `04_fase-atual/` o executor só marca o estado das tarefas, registra o resultado na linha `> ` e preenche o checklist de execução das SPECs. Tarefa, SPEC, critério, emenda ou id novo vão para a consultora (AGENTS.md).

| SPEC | Entrega | O que o cliente vê | Teste do cliente |
|---|---|---|---|
| [SPEC-4-001](spec-4-001-meta-real-midia-e-matriculaweb.md) | Meta de verdade, mídia e MatriculaWeb de verdade | Áudio, foto e PDF abrindo; PDF enviado; tiques azuis; valores reais | Luiz, com o Hikari |
| [SPEC-4-002](spec-4-002-assumir-transbordo-e-fila.md) | Assumir a conversa, transbordo e fila de prioridade | Filtro Transbordo ordenado; dono na conversa; IA desligável | Luiz, com um atendente de cada unidade |
| [SPEC-4-003](spec-4-003-area-assistente-no-admin.md) | Área "Assistente" no admin | Tudo da IA editável, simulável e com versões | Luiz |
| [SPEC-4-004](spec-4-004-atendente-virtual.md) | Atendente virtual fora do horário e na abertura | Resposta em até 1 minuto, no jeito do Manual, sem preço | Luiz, com o Hikari e atendentes |
| [SPEC-4-005](spec-4-005-contatos-da-mesma-compra.md) | Contatos da mesma compra | Mãe, pai e aluno no mesmo lead | Luiz, com um atendente |
| [SPEC-4-006](spec-4-006-orcamento-matriculaweb-e-pdf.md) | Orçamento no MatriculaWeb, em PDF, pelo WhatsApp | Número oficial, PDF da Academia, mensagem com pergunta | Luiz, com um atendente |
| [SPEC-4-007](spec-4-007-metricas-e-virada.md) | Métricas e virada para a operação real | K1a, K1b, equipe e IA no painel; números reais na central | Luiz, gestores e atendentes |

## Ordem e dependências

As SPECs seguem esta ordem: 4-001, 4-002, 4-003, 4-004, 4-005, 4-006 e 4-007.
- **4-001 primeiro:** a IA precisa ouvir áudio e a equipe precisa abrir mídia; o orçamento precisa do envio de PDF.
- **4-002 antes da 4-004:** a IA só fala quando a conversa está com ela, e passa a conversa pela etiqueta de transbordo.
- **4-003 antes da 4-004:** a IA usa a versão publicada da área Assistente. A tarefa "Simular" fecha depois que a função da IA existir.
- **4-005 depois da 4-004:** a IA usa as ferramentas de contato.
- **4-006 pode correr junto da 4-005:** depende só da 4-001 e dos textos da 4-003.
- **4-007 por último:** mede tudo e faz a virada.

Os prazos de `fase.md` são uma proposta para chegar à virada antes da Black Friday (por volta de 25/11). Se alguma SPEC atrasar, a virada pode acontecer com a IA só fora do horário, e o resto liga depois.

## Fora desta fase

- **Fase 5 (ciclo jan–fev/2027):** K3 contra a linha de base (histórico do Kommo), sugestor de resposta para a atendente e classificador de perda, retirada do Kommo.
- **IA falando preço, dando desconto ou fechando matrícula.**
- **Escrita no Kommo** e **disparo em massa.**
- **Liberação de fase:** só a consultora libera a fase 5 (AGENTS.md).
