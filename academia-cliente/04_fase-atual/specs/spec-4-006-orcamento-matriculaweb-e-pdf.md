# SPEC-4-006 — Orçamento gravado no MatriculaWeb, em PDF, enviado pelo WhatsApp

**Fase:** 4  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** Emenda 01 (DEC-18); SPEC-3-002 (orçamento simulado, `quotes`); Manual de Boas Práticas p. 5 ("não envie o arquivo do orçamento sem dizer nada depois") e p. 9 (nome completo, e-mail e matérias antes do orçamento); exemplo bom de Asa Sul 1 (mensagem, validade ligada à condição de pagamento e pergunta)  
**Degrau da solução:** construção sobre a SPEC-3-002 — acrescenta a gravação no MatriculaWeb, o PDF e o envio do arquivo.

## Contexto e decisões fechadas

- **Estado atual (v0.0.77):** a atendente monta o orçamento com unidade, turma e específicas; a simulação vem do MatriculaWeb e o retrato fica em `quotes`, sem poder ser alterado. Nada é gravado no MatriculaWeb e não existe arquivo: hoje a equipe gera o PDF "ORCAMENTO_…" no sistema interno e envia pelo Kommo.
- **Estado desejado:** quatro botões na mesma tela: **Montar → Gravar no MatriculaWeb → Gerar PDF → Enviar**. O lead recebe o PDF com o número oficial do orçamento, a mensagem padrão, a validade e uma pergunta. O lead vai para `Orçamento enviado` e o FUP 1 nasce.
- **Decisões já fechadas:**
  - **Gravação no MatriculaWeb** pela API pública: `POST /api/OrcamentoPublico/interesse` (nome, e-mail, celular → `interesseId`, guardado no lead para não repetir) e `POST /api/OrcamentoPublico` (`interesseId`, `nome`, `email`, `celular`, `alunoNome`, `unidadeId`, `turmaId`, `itens`, `itensRecusados` → `id` do aluno com status Orçamento, `idOrcamento`, `orcamento`). **Os valores do PDF são os que o MatriculaWeb devolve na gravação.**
  - Nome completo e e-mail são obrigatórios para gravar (Manual p. 9). A IA (SPEC-4-004) já os pede quando o lead quer preço.
  - **O PDF segue o modelo visual** de `04_fase-atual/anexos/orcamento-modelo.html` (logo, cores da marca, fonte Archivo, uma página A4). O HTML é a referência visual; o PDF é gerado **no servidor** (Edge Function `quote-pdf`) por uma biblioteca de PDF em JavaScript, a partir do retrato salvo, nunca recalculando valor. Texto selecionável, não imagem.
  - **Textos do admin** (SPEC-4-003, tela Orçamento): mensagem que acompanha o PDF (com `{nome}` e `{turma}`), condição de pagamento, validade e chamada do próximo passo.
  - **A mensagem tem que terminar com pergunta.** A tela não deixa enviar se o texto final não terminar em "?".
  - **Orçamento manual** (MatriculaWeb fora, já existe na SPEC-3-002): o PDF sai sem número oficial e com a frase "valores informados pela equipe".
  - O envio do arquivo segue a janela de 24h: fora dela, primeiro um modelo aprovado (regra da fase 3).
- **Bloqueios:** a gravação real usa a API pública de produção do MatriculaWeb. A liberação de limite de requisições para o servidor da central é pedida à Academia no checklist; até lá vale o limite público, com a nova tentativa manual.

## Resultado observável

A atendente abre o lead que a IA qualificou, escolhe turma e específicas, clica em Gravar (aparece "Orçamento nº 4812 no MatriculaWeb"), Gerar PDF (prévia ao lado) e Enviar. No celular chegam o PDF e a mensagem: "Segue o orçamento solicitado referente ao ENEM Anual 😊 … A condição de pagamento vale até 30/11… Ficou alguma dúvida sobre o curso?".

## Limites e dependências

- **Inclui:** botões e estados na tela de orçamento; chamadas `interesse` e `OrcamentoPublico` na `quote-simulate` (ou função irmã `quote-save-mw`); colunas `mw_interesse_id` (lead), `mw_aluno_id`, `mw_orcamento_id`, `pdf_path` (quote); Edge Function `quote-pdf`; envio de documento pelo `message-send` (subir o arquivo na Meta e mandar `type: document`, com o contrato da SPEC-4-001), provado com a Meta de mentira; envio do documento com a mensagem; trava contra gravação dupla.
- **Fora de escopo:** IA enviando orçamento; contrato, pagamento ou matrícula no MatriculaWeb; outras rotas do MatriculaWeb.
- **Entradas e pré-condições:** textos da tela Orçamento publicados (SPEC-4-003). Não depende da SPEC-4-001: a Meta é de mentira nas provas, e o envio para o celular é testado na 4-001.
- **Atores e permissões mínimas:** `attendant` e `unit_manager` da unidade do lead; os demais não veem.
- **Risco e plano B:** MatriculaWeb com 429 ou 5xx: "Tentar de novo" ou orçamento manual; nada é enviado pela metade.
- **Rollback:** desligar a gravação no console volta ao fluxo da SPEC-3-002 (só simulação).

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Central → `POST /api/OrcamentoPublico/interesse` | MatriculaWeb | `nome`, `email`, `celular` → `interesseId` | Público; chamado pelo servidor | 10 s; sem repetição automática; uma vez por lead | 400: mostra o campo; 429: tentar de novo |
| Central → `POST /api/OrcamentoPublico` | MatriculaWeb | `interesseId`, `nome`, `email`, `celular`, `alunoNome`, `unidadeId`, `turmaId`, `itens`, `itensRecusados` → `id`, `idOrcamento`, `orcamento` | Idem | 10 s; sem repetição automática; trava por orçamento (estado `gravando`) para não gravar duas vezes | Idem |
| `quote-pdf` → Storage `midias/orcamentos/` | Central | PDF A4 de 1 página | `service_role`; leitura por link assinado da unidade | Um PDF por orçamento (regerar substitui e guarda o anterior) | Falha: botão "Gerar de novo" |
| Central → Meta (documento) | Meta | `document` com `filename` = `Orcamento-{numero}-Academia.pdf` e a mensagem em seguida | Igual à SPEC-4-001 | Igual à SPEC-4-001 | Igual à SPEC-4-001 |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-4-24 | Gravar | Exige nome completo, e-mail e celular; usa o `interesseId` salvo se houver | — | Manual p. 9 |
| RN-4-25 | Valores | O PDF usa a resposta da gravação; se diferir da simulação, a tela mostra a diferença antes de gerar | — | DEC-18 |
| RN-4-26 | Enviar | PDF + mensagem com texto final em "?" | — | Manual p. 5 |
| RN-4-27 | Enviado | `Orçamento enviado` + FUP 1 (regra da fase 3) | — | SPEC-3-003 |

## Critérios de aceite

- [ ] **CA-4-23:** "Gravar" chama `interesse` (uma vez por lead) e `OrcamentoPublico`, guarda `mw_aluno_id` e `mw_orcamento_id` e mostra o número; clique duplo ou repetição não grava dois orçamentos; sem nome completo ou e-mail o botão fica travado com o motivo.
- [ ] **CA-4-24:** "Gerar PDF" produz uma página A4 com texto selecionável, no modelo do anexo (logo, cores, Archivo), com os valores da gravação e os textos publicados do admin; o orçamento manual sai sem número e com a frase de valores informados pela equipe.
- [ ] **CA-4-25:** "Enviar" manda o PDF e a mensagem; texto final sem "?" é recusado; com a janela fechada o envio é recusado; depois do envio o lead está em `Orçamento enviado` com o FUP 1.
- [ ] **CA-4-26:** com o MatriculaWeb devolvendo 400, 429 ou 5xx, nada é enviado, a tela mostra o motivo e oferece tentar de novo ou o orçamento manual.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P4.6-01: gravar sem e-mail; gravar duas vezes em paralelo | MatriculaWeb de mentira (`scripts/matriculaweb-falsa.mjs`, agora aceitando as duas rotas de gravação) | recusado; uma gravação só | saída do teste |
| GREEN | P4.6-02: gravar, gerar e enviar um orçamento `TESTE` | MatriculaWeb de mentira + Meta de mentira | número salvo; PDF de 1 página; documento e mensagem enviados; etapa e FUP 1 | consulta + PDF |
| RED | P4.6-03: mensagem sem "?" e janela fechada | teste | recusados | saída do teste |
| RED | P4.6-04: MatriculaWeb com 400, 429 e 500 | modos do servidor de mentira | nada enviado; motivo na tela | saída do teste |
| GREEN | P4.6-05: um orçamento real de teste no MatriculaWeb de produção (com nome `TESTE`) | tela | número oficial e valores iguais à página pública | captura + número |

**Observação:** o orçamento `TESTE` gravado no MatriculaWeb de produção fica lá como aluno com status Orçamento. Avisar a secretaria antes e apagar pelo sistema interno depois.

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC, a SPEC-3-002, `quote-simulate`, o anexo `orcamento-modelo.html` e a tela de orçamento.
2. **Não alterar:** a simulação e o retrato de `quotes` da SPEC-3-002 (só acrescentar colunas).
3. **Executar nesta ordem:** gravação com trava; PDF; envio com mensagem; erros; teste real.
4. **Parar e pedir validação quando:** o MatriculaWeb devolver valores diferentes da simulação de forma recorrente.
5. **Ritmo:** tasks em sequência; teste do cliente no fim.

## Checklist de execução

- [ ] Gravação no MatriculaWeb com trava.
- [ ] PDF no modelo, vetorial, 1 página.
- [ ] Envio com mensagem terminando em pergunta.
- [ ] Caminhos de erro.
- [ ] Um orçamento `TESTE` real gravado e conferido.

## Teste humano do cliente

- **Quando:** quando todas as tasks vinculadas estiverem no ar
- **Quem testa:** Luiz, com um atendente
- **Passos:** abrir um lead `TESTE` qualificado; gravar o orçamento; gerar o PDF e comparar com o modelo; enviar (para o celular da equipe, se o número de teste da SPEC-4-001 já estiver no ar; antes disso, conferir o envio na conversa); tentar enviar sem pergunta no fim; conferir o número no MatriculaWeb.
- **Resultado esperado:** PDF bonito, valores certos, número oficial, mensagem com pergunta, lead em `Orçamento enviado`.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Gravar o orçamento no MatriculaWeb e guardar o número | Academia das Específicas | SPEC-4-006 | CA-4-23, CA-4-26 | P4.6-01, P4.6-04 | saída do teste | SPEC-4-003 | ☐ |
| pendente | Gerar o PDF do orçamento no modelo da Academia | Academia das Específicas | SPEC-4-006 | CA-4-24 | P4.6-02 | PDF | Task anterior | ☐ |
| pendente | Enviar o PDF com a mensagem, a validade e a pergunta | Academia das Específicas | SPEC-4-006 | CA-4-25 | P4.6-02, P4.6-03, P4.6-05 | consulta (captura do celular na SPEC-4-001) | Task anterior | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| 2026-11-10 | Consultora | Ordem da fase | Deixa de depender da SPEC-4-001; o envio de documento passa a ser construído aqui |
