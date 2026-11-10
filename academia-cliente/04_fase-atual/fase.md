# Fase 4 — Tarefas

<!-- fase-format:2 -->

Cada linha é uma tarefa da Jornada de Execução. **Tudo que cabe num card cabe nesta linha** — se um
campo não estiver aqui, ele não tem como ser preenchido, porque é este arquivo que cria a tarefa.

```
- [ ] Título da tarefa @responsável !30/09/2026 #projeto [interno]   <!-- id:… -->
      > descrição da tarefa, uma ou mais linhas
  - [ ] subtarefa (basta indentar 2 espaços)                         <!-- id:… -->
    - [ ] sub-subtarefa (indente mais 2)                             <!-- id:… -->
```

| marcador | o que define | se você não escrever |
|---|---|---|
| `- [ ]` / `- [/]` / `- [x]` | a fazer / em andamento / concluída | a fazer |
| `@nome` | responsável (`@"Nome Composto"` com aspas) | fica **sem responsável** |
| `!dd/mm/aaaa` | prazo | fica **sem prazo** |
| `#projeto` / `#aculturamento` | tipo | Projeto de IA |
| `[interno]` | o cliente **não** vê esta tarefa | o cliente vê |
| `> texto` na linha de baixo | descrição (aparece ao abrir o card) | sem descrição |
| indentar 2 espaços | vira subtarefa da tarefa acima (vale em qualquer profundidade) | tarefa de topo |

Os marcadores só valem **no fim da linha** — `Revisar #3 do contrato` continua sendo um título.
Um título que TERMINA na forma de um marcador sai escapado com `\\` (`Ligar para \\@joao`); a barra é
só para o parser e nunca aparece no card. Você não precisa escrever isso à mão.
Marque `[x]` para concluir e adicione linhas novas à vontade: elas entram no quadro na próxima
sincronização e voltam aqui com o `<!-- id:… -->` preenchido. **Não apague o marcador de id** das
tarefas que já têm um.

- [ ] Dar dono à conversa com Assumir, Devolver e Pausar @"Academia das Específicas" !11/11/2026
  > A conversa fica com a IA, com uma pessoa (com nome) ou pausada; duas atendentes não assumem juntas. SPEC-4-002 · CA-4-05.
- [ ] Ligar e desligar a IA na conversa, na unidade e em tudo @"Academia das Específicas" !11/11/2026
  > Atendente na conversa, gestor na unidade, admin em tudo; desligada, nenhuma resposta automática sai. SPEC-4-002 · CA-4-06.
- [ ] Mostrar o transbordo com etiqueta, motivo e fila por janela e temperatura @"Academia das Específicas" !12/11/2026
  > Filtro Transbordo ordenado por janela perto de fechar, temperatura e espera, com o resumo da IA. SPEC-4-002 · CA-4-07.
- [ ] Avisar mensagem nova, criar respostas prontas e cobrir a caixa com E2E @"Academia das Específicas" !12/11/2026
  > Som e notificação para a atendente; respostas prontas da unidade; teste E2E da caixa (dívida de 19/10). SPEC-4-002 · CA-4-08, CA-4-09.
- [ ] Criar a área Assistente com rascunho, versões e permissões @"Academia das Específicas" !13/11/2026
  > Só o admin edita e publica; cada publicação vira versão e dá para voltar. SPEC-4-003 · CA-4-10, CA-4-11.
- [ ] Editar identidade, horários e o jeito de atender em blocos @"Academia das Específicas" !13/11/2026
  > Nome da atendente por unidade, expediente, instruções em blocos e regras fixas travadas. SPEC-4-003 · CA-4-12.
- [ ] Montar o FAQ com anexos, os exemplos e os textos do orçamento @"Academia das Específicas" !14/11/2026
  > Carga inicial tirada do Manual; exemplos sem telefone nem e-mail; textos que acompanham o PDF. SPEC-4-003 · CA-4-12, CA-4-13.
- [ ] Decidir quando a IA pode responder e agrupar as mensagens do lead @"Academia das Específicas" !14/11/2026
  > Fora do horário e na abertura, nunca depois de uma pessoa; uma resposta para mensagens seguidas. SPEC-4-004 · CA-4-14, CA-4-15.
- [ ] Responder com o Gemini, preencher a ficha e enviar anexos do FAQ @"Academia das Específicas" !15/11/2026
  > Gemini 2.5 Flash pela API paga, chave só no servidor; ficha completa leva a Qualificado. SPEC-4-004 · CA-4-15, CA-4-16, CA-4-19.
- [ ] Barrar no Guardião o que a IA não pode dizer @"Academia das Específicas" !15/11/2026
  > Preço, percentual, desconto, parcela, data, outra unidade e textos longos não saem. SPEC-4-004 · CA-4-17.
- [ ] Transferir para a equipe com motivo e resumo, e avisar o horário @"Academia das Específicas" !16/11/2026
  > Pedido de pessoa, preço, fechar, objeção, imagem, "é robô?" e falta de avanço viram transbordo. SPEC-4-004 · CA-4-18.
- [ ] Desligar a IA sozinha em caso de erro e passar na bateria de avaliação @"Academia das Específicas" !17/11/2026
  > Desligamento automático por falhas; 20 conversas do Manual sem nenhum preço e com o transbordo certo. SPEC-4-004 · CA-4-19.
- [ ] Simular uma conversa antes de publicar @"Academia das Específicas" !17/11/2026
  > O admin testa o rascunho ou a versão publicada sem gravar lead nem enviar nada. SPEC-4-003 · CA-4-13.
- [ ] Guardar vários números por lead, com o papel de cada um @"Academia das Específicas" !18/11/2026
  > Aluno, mãe, pai e responsável financeiro no mesmo lead; mensagem de qualquer um cai na mesma conversa. SPEC-4-005 · CA-4-20.
- [ ] Ligar só quando for a mesma compra, e separar indicação e família @"Academia das Específicas" !18/11/2026
  > Mesmo aluno, compra aberta e mesma unidade; aluno diferente vira indicação. SPEC-4-005 · CA-4-21, CA-4-22.
- [ ] Dar à IA e à atendente o jeito de ligar e desfazer, com a contagem única @"Academia das Específicas" !19/11/2026
  > A IA pergunta o número de quem falou antes; a atendente liga ou desfaz; K3 conta uma vez. SPEC-4-005 · CA-4-22.
- [ ] Gravar o orçamento no MatriculaWeb e guardar o número @"Academia das Específicas" !19/11/2026
  > Interesse e orçamento gravados pela API pública, sem gravar duas vezes; erros com saída manual. SPEC-4-006 · CA-4-23, CA-4-26.
- [ ] Gerar o PDF do orçamento no modelo da Academia @"Academia das Específicas" !20/11/2026
  > Uma página A4, logo e cores, valores da gravação e textos do admin. SPEC-4-006 · CA-4-24.
- [ ] Enviar o PDF com a mensagem, a validade e a pergunta @"Academia das Específicas" !20/11/2026
  > Mensagem que termina em pergunta; lead em Orçamento enviado com o FUP 1. SPEC-4-006 · CA-4-25.
- [ ] Testar texto e modelo com o número de teste da Meta @"Academia das Específicas" !21/11/2026
  > Envio, entrega e leitura voltando pelo webhook com o número de teste do app da Academia. Pendência da fase 3. SPEC-4-001 · CA-4-01.
- [ ] Receber e abrir áudio, imagem e documento na conversa @"Academia das Específicas" !21/11/2026
  > A mídia recebida é baixada para o armazenamento privado e abre na conversa; outra unidade não vê. SPEC-4-001 · CA-4-02.
- [ ] Enviar imagem e PDF e marcar a conversa como lida @"Academia das Específicas" !22/11/2026
  > Arquivo só dentro da janela de 24 horas; ao responder, o lead vê a mensagem como lida. SPEC-4-001 · CA-4-03.
- [ ] Testar o orçamento com a API real do MatriculaWeb @"Academia das Específicas" !22/11/2026
  > Unidades, turmas e simulação do MatriculaWeb de verdade, conferidas com a página pública de orçamento. Pendência da fase 3. SPEC-4-001 · CA-4-04.
- [ ] Medir o K1a e o K1b no painel @"Academia das Específicas" !23/11/2026
  > Primeira resposta em até 1 minuto e resposta humana ao transbordo dentro do SLA, conferidas com a consulta. SPEC-4-007 · CA-4-27.
- [ ] Mostrar os números de cada atendente e da IA @"Academia das Específicas" !23/11/2026
  > Abas Equipe e Assistente no painel gerencial, com custo por conversa. SPEC-4-007 · CA-4-28.
- [ ] Limpar a massa de teste e ensaiar a virada com volume @"Academia das Específicas" !24/11/2026
  > 300 leads de mentira em 1 hora sem perder nada; plano de volta executado. SPEC-4-007 · CA-4-29, CA-4-30.
- [ ] Importar o histórico real do Kommo e virar os números para a central @"Academia das Específicas" !25/11/2026
  > Com a consultora: importação, SalesBot desligado, número no app da Academia, IA fora do horário no primeiro dia. Depende do GATE-03 e do GATE-04. SPEC-4-007 · CA-4-31.
