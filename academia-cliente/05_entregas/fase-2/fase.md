# Fase 2 — Tarefas

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

- [x] Receber mensagens e formulários sem perder nem duplicar o lead @"Academia das Específicas" !22/10/2026 <!-- id:064c607e-877a-40bf-9ed3-c80489dbdf9e -->
  > Webhooks do WhatsApp (assinatura conferida) e dos formulários; reprocessar não duplica; falha vai para a fila com contexto. SPEC-2-001 · CA-2-01, CA-2-06, CA-2-07 · antiga T2.1.
  > Resultado (12/10, v0.0.33, sandbox): 2 rodadas, sem segredos. Fase 2 fechada em 28/10/2026.
- [x] Validar o contato, juntar duplicatas e distribuir o lead para a unidade @"Academia das Específicas" !26/10/2026 <!-- id:9bdb21dc-ac5a-4374-a0f7-f085bf56cf66 -->
  > Telefone inválido vai para Enviar e-mail sem adivinhar; duplicata exata junta na unidade; conflito vai para revisão. SPEC-2-001 · CA-2-02, CA-2-03 · antiga T2.2.
  > Resultado (12 e 13/10, v0.0.36, sandbox): 3 rodadas, com as emendas. Fase 2 fechada em 28/10/2026.
- [x] Abrir a caixa de conversas com a janela de 24 horas e o custo dos modelos @"Academia das Específicas" !28/10/2026 <!-- id:002289e1-d530-44c6-8b8d-f7d036885f57 -->
  > Caixa por unidade com o cronômetro da janela; fora dela só modelo aprovado com custo. SPEC-2-001 · CA-2-04 · antiga T2.3.
  > Passou para a fase 3 (SPEC-3-001) na liberação de 28/10/2026; sem registro de construção no repositório até 19/10.
- [x] Medir o SLA, a primeira resposta humana e o painel do dia @"Academia das Específicas" !30/10/2026 <!-- id:7bb35268-1e93-4f21-966d-9bc14e4be95d -->
  > Cron de SLA, K1 só com resposta humana e painel operacional do dia por unidade. SPEC-2-001 · CA-2-05, CA-2-09 · antiga T2.4.
  > Passou para a fase 3 (SPEC-3-001) em 28/10/2026, junto com o envio e a boas-vindas (antiga T2.5, que não chegou a virar cartão).
