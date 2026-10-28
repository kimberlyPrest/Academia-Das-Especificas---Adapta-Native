# Fase 3 — Tarefas

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

- [ ] Enviar mensagens só pelo servidor, com a boas-vindas automática @"Academia das Específicas" !29/10/2026
  > Atendente responde pela central sem token no navegador; a boas-vindas sai sozinha no primeiro contato e não conta como atendimento. Antiga T2.5. SPEC-3-001 · CA-3-01.
- [ ] Abrir a caixa de conversas de cada unidade com o cronômetro de 24 horas @"Academia das Específicas" !29/10/2026
  > Fila de cada unidade por tempo de espera, com o cronômetro da janela; fora dela, só modelo aprovado, com confirmação e custo registrado. Antiga T2.3. SPEC-3-001 · CA-3-02.
- [ ] Medir a primeira resposta humana e mostrar o painel do dia @"Academia das Específicas" !30/10/2026
  > SLA de 10 minutos configurável, alerta de quem passou do prazo, K1 e painel do dia por unidade; dados de teste ficam fora. Antiga T2.4. SPEC-3-001 · CA-3-03, CA-3-04, CA-3-05.
- [ ] Exigir a ficha de qualificação antes do orçamento @"Academia das Específicas" !30/10/2026
  > O lead só entra em Qualificado com objetivo, etapa do PAS (quando houver), turno e origem; o admin muda a lista no console. SPEC-3-002 · CA-3-06.
- [ ] Registrar pela tela a próxima ação do lead, com responsável e prazo @"Academia das Específicas" !03/11/2026
  > Criar, concluir com resultado e cancelar com motivo a próxima ação de cada lead, com tudo no histórico. SPEC-3-002 · CA-3-07.
- [ ] Montar o orçamento com as turmas e os preços do MatriculaWeb @"Academia das Específicas" !03/11/2026
  > Escolher unidade, turma e específicas e ver os valores com desconto calculados pelo MatriculaWeb; com a API fora, orçamento manual marcado. SPEC-3-002 · CA-3-08.
- [ ] Salvar o orçamento enviado e abrir o retorno do lead @"Academia das Específicas" !04/11/2026
  > O lead vai para Orçamento enviado com o orçamento guardado e uma tarefa de retorno; nada é gravado no MatriculaWeb. SPEC-3-002 · CA-3-09.
- [ ] Mostrar ao gestor o K2 e os leads sem próxima ação @"Academia das Específicas" !04/11/2026
  > Percentual de leads qualificados com próxima ação por unidade e a lista de quem está sem, conferidos com a contagem direta. SPEC-3-002 · CA-3-10.
- [ ] Criar sozinho as tarefas de FUP 1 e FUP 2 nos prazos configurados @"Academia das Específicas" !05/11/2026
  > Sem resposta, FUP 1 em 20 horas e FUP 2 em 24 horas, com a mudança de etapa; resposta encerra; reunião e negociação pausam; nada é enviado. SPEC-3-003 · CA-3-11.
- [ ] Mostrar na tarefa de follow-up se cabe mensagem livre ou só modelo @"Academia das Específicas" !05/11/2026
  > A tarefa mostra a janela aberta com o horário-limite ou os modelos aprovados com o custo; quem pediu para não receber fica bloqueado. SPEC-3-003 · CA-3-12.
- [ ] Lembrar o terceiro retorno e pedir a decisão sobre o lead @"Academia das Específicas" !06/11/2026
  > Depois do FUP 2 sem resposta, um lembrete, sem coluna nova, que só fecha com negociação, 2027, matrícula, desqualificação ou nova ação. SPEC-3-003 · CA-3-13.
- [ ] Agendar a reunião com o diretor e pausar o follow-up @"Academia das Específicas" !06/11/2026
  > Reunião com data, formato e diretor registrada no lead; o follow-up para e o diretor vê a reunião na agenda dele na central. SPEC-3-004 · CA-3-14.
- [ ] Registrar o resultado da reunião e seguir o caminho de cada resultado @"Academia das Específicas" !09/11/2026
  > Fechou, segue avaliando, não compareceu ou desqualificou levam o lead ao passo certo; reunião sem resultado no dia vira pendência. SPEC-3-004 · CA-3-15.
- [ ] Abrir a fila de negociação com as alçadas de desconto @"Academia das Específicas" !09/11/2026
  > Pedido de desconto vai para a fila dos negociadores das duas unidades; quem decide só aprova dentro da faixa cadastrada. SPEC-3-004 · CA-3-16.
- [ ] Devolver a decisão da negociação ao atendente com a próxima ação @"Academia das Específicas" !10/11/2026
  > A decisão volta ao atendente como tarefa; sem resposta do lead no prazo, ele vai para Sem resposta pós negociação. SPEC-3-004 · CA-3-17.
- [ ] Conferir o checklist de matrícula antes de marcar Novo aluno @"Academia das Específicas" !10/11/2026
  > O atendente confere item por item e a central diz o que falta; documento não é guardado, só a marca de conferido. SPEC-3-005 · CA-3-18.
- [ ] Registrar a matrícula com o valor final e o que foi contratado @"Academia das Específicas" !11/11/2026
  > Matrícula com turma, específicas e valor final, que conta uma vez por lead; matrícula presencial entra com origem presencial. SPEC-3-005 · CA-3-19.
- [ ] Guardar o lead de Interesse 2027 com a data de retomada @"Academia das Específicas" !11/11/2026
  > Ano-alvo e objetivo obrigatórios; a data de retomada sai da regra do console e o follow-up para até lá. SPEC-3-005 · CA-3-20.
- [ ] Trazer de volta os leads de 2027 na data de retomada @"Academia das Específicas" !12/11/2026
  > Rotina diária devolve o lead para a fila da unidade com uma tarefa de retomada, sem duplicar. SPEC-3-005 · CA-3-21.
- [ ] Mostrar o funil da origem à matrícula por unidade e campanha @"Academia das Específicas" !12/11/2026
  > Leads, qualificados, orçamentos, negociações e matrículas com a conversão de cada passo, por unidade, origem, campanha e período. SPEC-3-006 · CA-3-22.
- [ ] Abrir os painéis ao marketing e à diretoria sem acesso às conversas @"Academia das Específicas" !13/11/2026
  > Marketing vê só números, sem conversa nem dado pessoal; gestor vê a unidade; diretoria vê cada unidade e a soma. SPEC-3-006 · CA-3-23.
- [ ] Deixar o gestor auditar uma amostra de conversas da unidade @"Academia das Específicas" !13/11/2026
  > O gestor sorteia conversas, marca qualificação completa e pergunta no final, e o painel mostra os percentuais. SPEC-3-006 · CA-3-24.
- [ ] Ler o arquivo de exportação do Kommo e mapear etapas, motivos e campos @"Academia das Específicas" !16/11/2026
  > Com arquivo de teste no formato do Kommo, a central mapeia as 14 etapas, os 7 motivos e os campos e lista o que ficou sem mapa. SPEC-3-007 · CA-3-25.
- [ ] Guardar o histórico importado sem permitir alteração @"Academia das Específicas" !16/11/2026
  > O histórico entra marcado como do Kommo, não duplica ao reimportar e o banco recusa edição ou exclusão. SPEC-3-007 · CA-3-26.
- [ ] Gerar o relatório de conferência da importação @"Academia das Específicas" !17/11/2026
  > Totais por etapa, unidade e ano, lacunas e possíveis duplicatas, batendo com o arquivo; dado real só depois da LGPD aprovada. SPEC-3-007 · CA-3-27.
