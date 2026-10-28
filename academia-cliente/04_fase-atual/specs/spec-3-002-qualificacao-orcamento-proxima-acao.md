# SPEC-3-002 — Qualificação, orçamento e próxima ação (K2)

**Fase:** 3  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** fase 3 (roteiro de qualificação, orçamento, próxima ação obrigatória, K2); fluxo F3; RN-04, RN-05, RN-13; dívida de 19/10 (próxima ação só por fixture); pedido da Academia de 16/10 (orçamento pela API do MatriculaWeb); CL-001, CL-005, CL-006, CL-011  
**Degrau da solução:** reuso + dependência existente — usa `stage_move`, catálogos, campos por etapa (console módulo 8) e `next_actions` da fase 1, e a API pública de orçamento do MatriculaWeb, que a Academia já mantém, para turmas e preços; constrói só a ficha de qualificação, o orçamento e a tela da próxima ação.

## Contexto e decisões fechadas

- **Estado atual:** o funil tem 14 etapas (emenda de 10/10) e a transição já é validada no servidor (motivo para desqualificar, checklist para matrícula). `next_actions` existe, mas só é gravada por fixture ou servidor. Não há ficha de qualificação nem orçamento. Os preços e as turmas vivem no MatriculaWeb, que tem uma API pública de orçamento (sem login, com limite por IP, valores sempre calculados pelo servidor dela).
- **Estado desejado:** o atendente preenche a ficha, monta o orçamento com a turma e as específicas, vê os valores calculados pelo MatriculaWeb, registra o envio e deixa uma próxima ação com responsável e prazo. O gestor vê o K2 e quem está sem próxima ação.
- **Decisões já fechadas:**
  - Campos obrigatórios para entrar em `Qualificado` (ou em qualquer etapa depois dela): objetivo (ENEM, PAS, UnB, particulares de medicina; pode ser mais de um), etapa do PAS quando o objetivo inclui PAS, turno e origem ou campanha conhecida. Ano escolar, experiência anterior e maior dificuldade são opcionais. O admin muda a lista no console (módulo 8), sem deploy.
  - Orçamento só depois da qualificação. Valores só do MatriculaWeb (`/simular`); o navegador nunca calcula total ou desconto.
  - Nesta fase a central **não grava nada no MatriculaWeb**: não chama `POST /api/OrcamentoPublico/interesse` nem `POST /api/OrcamentoPublico`. Usa só a lista de unidades, a lista de turmas e a simulação.
  - Toda tarefa de lead é um registro de `next_actions` (a tabela do K2). Se a versão atual tiver também uma tabela `tasks`, não crie tarefa de lead nela.
  - Sem dono do lead (fila única por unidade): a próxima ação sempre tem um responsável explícito, usuário ativo da unidade.
  - Massa de teste com `is_test = true` e nome `TESTE`, fora do K2 por padrão (regra da SPEC-3-001).
- **Bloqueios:** nenhum. Os tipos de próxima ação que a equipe usa no Kommo (CL-005) e a confirmação dos campos obrigatórios (CL-006) ajustam o catálogo e a lista no console quando chegarem.

## Resultado observável

O atendente abre o lead, preenche a ficha e só então consegue movê-lo para `Qualificado`. Monta o orçamento escolhendo unidade, turma e específicas e vê os valores com desconto vindos do MatriculaWeb. Ao salvar o orçamento enviado, o lead vai para `Orçamento enviado` com o retorno agendado. No painel, o gestor vê o K2 da unidade e a lista de leads sem próxima ação.

## Limites e dependências

- **Inclui:** ficha de qualificação; tela da próxima ação (criar, concluir com resultado, cancelar com motivo); orçamento com consulta e simulação no MatriculaWeb; orçamento manual quando a API falhar; K2 e pendências.
- **Fora de escopo:** gravar orçamento ou lead no MatriculaWeb; IA montando orçamento (fase 4); desconto de negociação (SPEC-3-004); cadência automática (SPEC-3-003).
- **Entradas e pré-condições:** SPEC-3-001 no ar; mapa das unidades da central para os ids de unidade do MatriculaWeb, cadastrado pelo admin.
- **Saídas/artefatos:** tabelas `qualification_data`, `quotes`, `matriculaweb_unit_map`; Edge Functions `qualification-save`, `quote-simulate`, `quote-save`; tela da próxima ação; cartão K2 no painel.
- **Dependências e responsáveis:** Academia mantém turmas e preços no MatriculaWeb; atendentes usam; Luiz testa.
- **Atores e permissões mínimas:** `attendant` e `unit_manager` da unidade preenchem ficha, orçamento e próxima ação; `platform_admin` configura campos, catálogo de ações e mapa de unidades; `marketing` não vê ficha nem orçamento individual.
- **Superfícies/arquivos/configurações afetadas:** cartão do lead, quadro do funil, painel; console (módulos 2, 7 e 8); `stage_move`.
- **Risco e plano B:** API do MatriculaWeb fora ou limite de requisições atingido: orçamento manual com valor digitado, marcado `manual` com motivo, para não travar o atendimento.
- **Rollback ou reversão:** tabelas novas são aditivas; desligar `orcamento_matriculaweb_ativo` no console volta ao orçamento manual.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Central → MatriculaWeb `GET /api/OrcamentoPublico/unidades` | MatriculaWeb | `id`, `nome`, `enderecoCompleto` | API pública; chamada pela Edge Function, nunca pelo navegador | 10 s; 1 nova tentativa | Falha: orçamento manual |
| Central → `GET /api/OrcamentoPublico/unidades/{unidadeId}/turmas` | MatriculaWeb | `id`, `nome`, `periodoLetivo`, `dataInicio` | Idem | 10 s; 1 nova tentativa | Lista vazia: "sem turma aberta nesta unidade" |
| Central → `POST /api/OrcamentoPublico/simular` | MatriculaWeb | envia `unidadeId`, `turmaId`, `itens` (códigos `E123`/`S45`), `itensRecusados`; recebe `linhas` (`codigo`, `tipo`, `nome`, `valor`, `percDesconto`, `valorFinal`, `selecionado`, `bloqueado`, `motivoBloqueio`, `selecionadoAutomaticamente`, `marcadoPadrao`), `matriculaValor`, `produtosVinculadosValor`, `totalBruto`, `totalDesconto`, `totalFinal`, `semFaixaCadastrada`, `itensRecusados` | Idem | 10 s; sem repetição automática; uma chamada por mudança na seleção | 429 ou 5xx: mensagem "tente de novo" e opção de orçamento manual |
| Tela → `quotes` | Central | `lead_id`, `unit_id`, `mw_unidade_id`, `mw_turma_id`, `turma_nome`, `periodo_letivo`, `itens` (jsonb com código, nome, tipo, valor, valor final), `matricula_valor`, `total_bruto`, `total_desconto`, `total_final`, `sem_faixa`, `origem_valores` (`matriculaweb`, `manual`), `motivo_manual`, `simulado_em`, `enviado_em`, `created_by`, `is_test` | `attendant`+ da unidade | Um orçamento ativo por lead; novo orçamento substitui o anterior e guarda o histórico | 400 com o campo |
| Tela → `next_actions` | Central | `lead_id`, `tipo` (catálogo), `descricao`, `responsavel_id`, `prazo`, `status` (pendente, feita, cancelada), `resultado`, `cancel_motivo`, `origem`, `origem_chave` (única), `unit_id`, `is_test` | `attendant`+ da unidade | `origem_chave` impede duplicar ação automática | 400 com o campo |

Catálogo inicial de tipos de próxima ação (editável no console): ligar, mandar mensagem, enviar orçamento, aguardar resposta do orçamento, reunião, negociação, matrícula, retomada, outro.

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | Entrada em `Qualificado` ou etapa posterior | Exige os campos obrigatórios da ficha; o erro cita o campo | Admin muda a lista no console | RN-04 do escopo |
| RN-02 | Item com `bloqueado = true` | Não pode ser marcado; mostra "contratar na secretaria" (`SECRETARIA`) ou "depende de outra específica" (`RELACIONADA`) | — | API do MatriculaWeb |
| RN-03 | Item com `selecionadoAutomaticamente = true` | Aparece como "incluído" | — | API do MatriculaWeb |
| RN-04 | `semFaixaCadastrada = true` | Aviso "sem faixa de desconto para essa quantidade" no orçamento | — | API do MatriculaWeb |
| RN-05 | Entrada em `Orçamento enviado` | Exige orçamento salvo com `enviado_em`; cria a próxima ação "aguardar resposta do orçamento" do autor, com prazo de 20 horas (configuração `fup1_horas` do console; se ainda não existir, esta SPEC a cria com 20) | — | RN-13 do escopo |
| RN-06 | Lead em etapa ativa de `Qualificado` em diante sem próxima ação pendente | Pendência de gestão "sem próxima ação" | `Interesse 2027` e etapas terminais | RN-13 do escopo |
| RN-07 | Concluir próxima ação | Exige resultado e, se o lead segue em etapa ativa, a próxima | Cancelar exige motivo | RN-13 do escopo |
| RN-08 | Responsável desativado | Ações dele vão para a gestão reatribuir | — | Fase 1 |

## Fluxo e regras

1. O atendente preenche a ficha (`qualification-save`) e move o lead para `Qualificado`; `stage_move` confere os campos (RN-01).
2. No orçamento, a central lista as turmas da unidade do lead pelo mapa de unidades e chama `quote-simulate` a cada mudança; a tela mostra as linhas e os totais como vieram (RN-02 a RN-04).
3. O atendente envia o orçamento ao lead (pela caixa ou fora dela) e marca "enviado"; `quote-save` grava o retrato do orçamento; o lead vai para `Orçamento enviado` e ganha a próxima ação de retorno (RN-05).
4. A qualquer momento o atendente cria, conclui ou cancela próximas ações pela tela (RN-07).
5. O painel calcula o K2 da unidade: leads de `Qualificado` em diante, em etapa ativa e sem `is_test`, com próxima ação pendente com responsável ativo e prazo ÷ total desses leads; `Interesse 2027` fica fora. A lista "sem próxima ação" abre cada lead.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Lead `TESTE` com ficha completa, turma e duas específicas | Valores do MatriculaWeb; lead em `Orçamento enviado` com retorno agendado | — |
| Limite | Objetivo PAS sem etapa do PAS | Entrada em `Qualificado` recusada citando "etapa do PAS" | Preencher libera |
| Falha | `/simular` devolve 429 | Aviso; orçamento manual marcado `manual` com motivo | Tentar de novo depois |
| Falha | Próxima ação sem responsável ou prazo | 400 com o campo | — |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC; as SPECs 1-003 e 1-004 em `05_entregas/fase-1/specs/`; a documentação pública da API em `https://api.academiadasespecificas.com.br/docs-publico/index.html`; `stage_move`, `next_actions` e o console atual.
2. **Alterar somente:** ficha, orçamento, tela da próxima ação, as Edge Functions e tabelas listadas, campos obrigatórios da etapa `Qualificado` e o cartão K2.
3. **Não alterar:** regras de desqualificação e de matrícula da fase 1; caixa de conversas; nada no MatriculaWeb além das três leituras.
4. **Executar nesta ordem:** ficha; próxima ação pela tela; orçamento com simulação; orçamento enviado; K2.
5. **Parar e pedir validação quando:** a simulação exigir login, chave ou qualquer chamada que grave no MatriculaWeb.
6. **Estado válido ao parar:** funil funcionando como antes; orçamento manual disponível.
7. **Ritmo:** construir as tasks vinculadas em sequência, sem pedir autorização nem teste a cada task. O teste do cliente é o roteiro único de `## Teste humano do cliente`, quando todas as tasks vinculadas estiverem no ar. Ponto de aprovação antes disso: nenhum.

## Checklist de execução

- [ ] Ficha com campos obrigatórios configuráveis e bloqueio no servidor.
- [ ] Próxima ação pela tela, com resultado ao concluir e motivo ao cancelar.
- [ ] Orçamento com turmas e valores do MatriculaWeb pelo servidor; manual quando a API falhar.
- [ ] Nenhuma chamada de gravação no MatriculaWeb.
- [ ] K2 e lista "sem próxima ação" conferidos com consulta direta.

## Critérios de aceite

- [ ] **CA-3-06:** a entrada em `Qualificado` (ou etapa posterior) sem objetivo, etapa do PAS quando há PAS, turno ou origem é recusada no servidor citando o campo; o admin muda a lista no console e vale sem deploy.
- [ ] **CA-3-07:** o atendente cria pela tela a próxima ação com tipo, responsável ativo da unidade e prazo; concluir exige resultado e, em etapa ativa, a próxima; cancelar exige motivo; tudo entra no histórico do lead.
- [ ] **CA-3-08:** o orçamento lista as turmas da unidade e mostra linhas e totais exatamente como o `/simular` devolveu, com itens bloqueados e incluídos sinalizados; com a API fora, o orçamento manual fica marcado `manual` com motivo.
- [ ] **CA-3-09:** a entrada em `Orçamento enviado` exige orçamento salvo com data de envio e cria uma única próxima ação de retorno; nenhuma requisição de gravação sai para o MatriculaWeb.
- [ ] **CA-3-10:** o K2 da unidade e a lista "sem próxima ação" batem com a consulta direta, sem `Interesse 2027`, etapas terminais e massa de teste.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P3.2-01: `stage_move` para `Qualificado` sem turno e com PAS sem etapa | RPC de teste | 400 citando os campos | saída pgTAP |
| GREEN | P3.2-02: ficha completa | RPC de teste | transição aceita; histórico gravado | saída pgTAP |
| RED | P3.2-03: próxima ação sem responsável; conclusão sem resultado | chamada da função | 400 nos dois | saída do teste |
| GREEN | P3.2-04: `quote-simulate` com turma e duas específicas de uma unidade aberta | chamada da função | linhas e totais iguais aos do `/simular` | resposta registrada (sem dado pessoal) |
| RED | P3.2-05: `/simular` simulado com 429 | teste com resposta falsa | aviso e orçamento manual marcado `manual` | saída do teste |
| GREEN | P3.2-06: `quote-save` e entrada em `Orçamento enviado` | RPC + consulta | uma próxima ação de retorno; nenhuma chamada a `POST /api/OrcamentoPublico` ou `/interesse` no log de saída | consulta + log |
| REFACTOR/REGRESSÃO | P3.2-07: K2 × consulta direta; provas P3.1 e pgTAP da fase 1 | consulta + `supabase test db` | números iguais; provas verdes | saída |

**Dados/fixtures:** leads `TESTE` nas duas unidades em todas as etapas ativas; uma turma aberta de cada unidade do MatriculaWeb; resposta falsa de 429 e de 500.  
**Caminhos de erro obrigatórios:** campo obrigatório vazio, item bloqueado, API fora, limite de requisições, responsável desativado.  
**Evidência exigida:** saídas das provas e capturas da ficha, do orçamento e do K2 na linha `> ` da task.

## Teste humano do cliente

- **Origem:** CL-001
- **Quando:** quando todas as tasks de `## Tasks vinculadas` estiverem no ar
- **Quem testa:** Luiz, com um atendente de cada unidade, se puder
- **Passos:** tentar qualificar um lead `TESTE` sem turno; completar a ficha e qualificar; montar o orçamento com uma turma real e duas específicas e comparar o total com a página de orçamento do site; marcar enviado; ver o lead em `Orçamento enviado` com o retorno agendado; abrir o painel e conferir o K2 e a lista sem próxima ação.
- **Resultado esperado:** bloqueio com o campo certo; total igual ao do site; retorno agendado; K2 coerente com o quadro.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Handoff e operação

- **Como demonstrar:** um lead `TESTE` da ficha ao orçamento enviado.
- **Como operar depois:** o atendente fecha cada conversa com uma próxima ação; o gestor olha a lista "sem próxima ação" todo dia.
- **Como monitorar:** K2 diário por unidade e falhas do `/simular` no painel.
- **Pendência conhecida:** gravar o orçamento no MatriculaWeb fica para decisão do consultor.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Exigir a ficha de qualificação antes do orçamento | Academia das Específicas | SPEC-3-002 | CA-3-06 | P3.2-01, P3.2-02 | saída pgTAP | SPEC-3-001 no ar | ☐ |
| pendente | Registrar pela tela a próxima ação do lead, com responsável e prazo | Academia das Específicas | SPEC-3-002 | CA-3-07 | P3.2-03 | saída do teste + captura | — | ☐ |
| pendente | Montar o orçamento com as turmas e os preços do MatriculaWeb | Academia das Específicas | SPEC-3-002 | CA-3-08 | P3.2-04, P3.2-05 | resposta registrada + captura | Mapa de unidades cadastrado | ☐ |
| pendente | Salvar o orçamento enviado e abrir o retorno do lead | Academia das Específicas | SPEC-3-002 | CA-3-09 | P3.2-06 | consulta + log de saída | Orçamento no ar | ☐ |
| pendente | Mostrar ao gestor o K2 e os leads sem próxima ação | Academia das Específicas | SPEC-3-002 | CA-3-10 | P3.2-07 | consulta × painel | — | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
