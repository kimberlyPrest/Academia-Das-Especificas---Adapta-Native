# Fase 1 — Tarefas

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

- [x] Preparar o projeto no Supabase com ambientes e reset reproduzível @"Academia das Específicas" !05/10/2026 <!-- id:6c131263-deab-4e37-84d4-b4b449b28c9d -->
  > SPEC-1-001 · CA-1-01 · antiga T1.1.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Guardar a auditoria sem edição e as filas com tratamento de falha @"Academia das Específicas" !06/10/2026 <!-- id:610c4c2a-dfa7-4e37-9743-02fe618fa664 -->
  > SPEC-1-001 · CA-1-04, CA-1-05 · antiga T1.2.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Carregar dados de teste sem dado pessoal e sem segredo no código @"Academia das Específicas" !06/10/2026 <!-- id:8ddf6250-b03b-4c92-8116-727aa33cd5bd -->
  > SPEC-1-001 · CA-1-02, CA-1-03 · antiga T1.3.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Cadastrar unidades, usuários e papéis, com convite por e-mail @"Academia das Específicas" !07/10/2026 <!-- id:a057c395-f6b6-4626-97c1-a21d999b9ad5 -->
  > SPEC-1-002 · CA-1-11 · antiga T1.4.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Separar o acesso por papel e por unidade no banco @"Academia das Específicas" !08/10/2026 <!-- id:ba20e009-0d34-4cf9-95de-240f0b3666f9 -->
  > SPEC-1-002 · CA-1-06 a CA-1-08 · antiga T1.5.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Exigir segundo fator do admin e desativar usuário sem apagar @"Academia das Específicas" !09/10/2026 <!-- id:04ebfdfe-d197-47ba-8367-1f3e2c472d31 -->
  > SPEC-1-002 · CA-1-09, CA-1-10 · antiga T1.6.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Montar o funil e os catálogos com o processo real @"Academia das Específicas" !13/10/2026 <!-- id:a326adb5-5f49-42df-9fca-ea1209b570e4 -->
  > SPEC-1-003 · CA-1-13 · antiga T1.7.
  > Aprovada em teste independente (v0.0.27); emenda de 10/10 com 14 etapas e 7 motivos (antiga T1.13). Fase 1 fechada em 28/10/2026.
- [x] Validar no servidor a mudança de etapa, com motivo e checklist @"Academia das Específicas" !14/10/2026 <!-- id:7e83b069-d713-4d0f-a530-c494fbc15298 -->
  > SPEC-1-003 · CA-1-14, CA-1-15 · antiga T1.8.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Abrir o console do admin para funil, catálogos e checklist @"Academia das Específicas" !15/10/2026 <!-- id:aca527ef-1286-4573-8ab0-0b6100babcd5 -->
  > SPEC-1-003 · CA-1-12, CA-1-16 a CA-1-18 · antiga T1.9.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Criar a massa de demonstração com 20 leads @"Academia das Específicas" !16/10/2026 <!-- id:6749e779-6b81-4372-a633-843bf04f02b3 -->
  > SPEC-1-004 · CA-1-19 · antiga T1.10.
  > Aprovada em teste independente (v0.0.27, 07/10). Fase 1 fechada pelo consultor em 28/10/2026.
- [x] Mostrar o painel-base com filtros e pendências de gestão @"Academia das Específicas" !16/10/2026 <!-- id:0557069a-8ca1-4767-bd68-3f2ce5221af0 -->
  > SPEC-1-004 · CA-1-20 a CA-1-23 · antiga T1.11.
  > Aprovada em teste independente (v0.0.27); quadro com arrastar e pendência clicável em 10/10 (antiga T1.14). Fase 1 fechada em 28/10/2026.
- [x] Demonstrar a base à direção e registrar o aceite @"Academia das Específicas" !19/10/2026 <!-- id:c9686863-97bd-41bd-916b-5ca06f534c90 -->
  > SPEC-1-004 · CA-1-24 · antiga T1.12.
  > Passou para a fase 3 em 28/10/2026: a demonstração e o aceite da fase 1 entram no teste da SPEC-3-001.
