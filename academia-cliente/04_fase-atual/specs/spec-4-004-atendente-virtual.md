# SPEC-4-004 — Atendente virtual (IA) fora do horário e na abertura

**Fase:** 4  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** Emenda 01 (DEC-04, DEC-05, DEC-13, DEC-16, AG-14, AG-13, K1a); Manual de Boas Práticas (pp. 3–14); Mapeamento MKT (etapas e seção 8)  
**Degrau da solução:** construção nova — uma Edge Function `ai-atendente` com o Gemini, sobre a fila, o envio, o estado da conversa (SPEC-4-002) e a configuração (SPEC-4-003).

## Contexto e decisões fechadas

- **Estado atual:** nenhuma resposta automática além da boas-vindas. Hoje, quem escreve de madrugada espera até o expediente (no exemplo ruim de Taguatinga 1, o lead escreve às 06:43 e a pessoa responde às 08:20).
- **Estado desejado:** o lead recebe resposta em até 1 minuto, a qualquer hora, de uma atendente virtual que conversa no jeito do Manual, preenche a ficha e passa para a equipe no momento certo, com o resumo.
- **Decisões já fechadas (Emenda 01):**
  - **Dois modos.** *Fora do horário:* atende até o transbordo. *Abertura no horário:* atende só até a ficha completa ou o primeiro motivo de transbordo; se uma pessoa assumir antes, para na hora.
  - **Nunca:** preço ou faixa de preço, desconto, data prometida, turma recomendada em definitivo, a outra unidade, modelo pago, mensagem fora da janela de 24h, resposta depois que uma pessoa falou no atendimento.
  - **Persona:** nome e assinatura da SPEC-4-003. A assinatura é colocada pela central, não pelo modelo. Se o lead perguntar se é robô, sai o texto fixo do admin (padrão: "Sou o atendimento virtual da Unidade {unidade}. Já chamo alguém da equipe para continuar com você, tudo bem?") e a conversa vai para humano (`perguntou_se_e_robo`).
  - **Modelo:** Gemini, pela API paga, chave `GEMINI_API_KEY` só no segredo da Edge Function; nome do modelo da versão publicada (padrão `gemini-2.5-flash`). Áudio vai direto para o modelo; imagem e documento vão para humano. Enquanto a SPEC-4-001 não estiver no ar, o áudio do WhatsApp não chega à IA: ela transfere para a equipe com o motivo `documento_ou_imagem`. O entendimento de áudio é provado no modo simulação.
  - **Saída estruturada:** o modelo devolve `{texto, intencao, transferir}`; `intencao` e o motivo de `transferir` vêm das listas fechadas da SPEC-4-002.
  - **Ferramentas do modelo:** `registrar_ficha` (só os campos da ficha, validados pela RPC da SPEC-3-002), `enviar_anexo_faq` (um anexo de `kb_attachments`), `transferir_para_humano` (motivo, intenção, resumo) e as de contato da SPEC-4-005. No máximo 3 rodadas de ferramenta por resposta.
  - **Etapas:** quando o lead responde, a IA move de `Novo contato` para `Em atendimento`; com a ficha completa, move para `Qualificado` pela mesma RPC das pessoas (que valida a ficha) e transfere com `ficha_completa`. Não move para nenhuma outra etapa.
  - **Guardião antes de cada envio** (regras fixas, sem modelo): recusa texto com valor em dinheiro (`R$`, "reais"), percentual, "desconto", "parcela", "mensalidade" com número, data com dia ou mês, nome da outra unidade, link fora da lista do FAQ, mais de 600 caracteres ou mais de 2 mensagens seguidas. Recusado: nada sai e a conversa vai para humano (`guardiao`).
  - **Desligamento automático:** com 5 falhas do Gemini em 15 minutos, ou com 30% das respostas barradas pelo Guardião em 15 minutos (mínimo de 10), a IA geral desliga e o admin é avisado.
- **Bloqueios:** nenhum para construir. Rodar com lead real depende da SPEC-4-007.

## Resultado observável

Às 23h, um lead `TESTE` escreve "quero saber do PAS". Em menos de 1 minuto recebe, assinado pela atendente virtual da unidade, uma resposta curta no estilo do Manual que termina com pergunta. Ele conta que está no 2º ano e prefere estudar à tarde; quando pede o preço, ela pede nome completo e e-mail para o orçamento, avisa o horário em que a equipe responde e passa a conversa. De manhã, o lead está no topo do filtro Transbordo, com a ficha preenchida e o resumo.

## Limites e dependências

- **Inclui:** `ai-atendente` (Edge Function); fila `ai_jobs` com espera de 25 s que recomeça a cada mensagem nova (máximo 60 s); RPC `ia_pode_responder(lead)`; montagem do contexto (últimas 30 mensagens, ficha, unidade, versão publicada, FAQ ativo da unidade, exemplos); chamada ao Gemini; ferramentas; Guardião; envio pelo `envio_enfileira` com `kind = 'ai'`; tabela `ai_runs`; desligamento automático; modo simulação para o "Simular" da SPEC-4-003; bateria de avaliação.
- **Fora de escopo:** sugestão de resposta para a atendente e classificador de perda (fase 5); IA mandando orçamento.
- **Entradas e pré-condições:** SPECs 4-002 e 4-003; versão publicada com nome nas duas unidades; `GEMINI_API_KEY` cadastrada.
- **Atores e permissões:** a função roda com `service_role`, chamada pelo agendador do banco com uma senha própria (`IA_CHAVE`), no mesmo padrão do `message-send`. Nenhum papel de usuário chama a função direto, exceto o "Simular" (`platform_admin`, modo simulação).
- **Risco e plano B:** Gemini fora do ar ou lento: 20 s de prazo, 1 nova tentativa; depois a conversa vai para humano (`falha_ia`). Resposta estranha: Guardião e desligamento automático. A equipe pode pausar tudo pelos três níveis da SPEC-4-002.
- **Rollback:** desligar a IA geral. Nada muda no resto da central.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Banco → `ai-atendente` | Central | `ai_jobs` (`lead_id`, `visible_at`, `attempts`) | Cabeçalho `x-ia-chave` | Um job por lead; mensagem nova adia o job | Senha errada: 401 |
| `ai-atendente` → Gemini `POST https://generativelanguage.googleapis.com/v1beta/models/{modelo}:generateContent` | Google | `systemInstruction` (montada pela SPEC-4-003), `contents` (texto e áudio `inline_data`), `tools.functionDeclarations`, `generationConfig` com `responseMimeType: application/json` e esquema | `x-goog-api-key` (segredo) | 20 s; 1 nova tentativa | `falha_ia` |
| `ai-atendente` → `envio_enfileira` | Central | mensagem `kind = 'ai'`, texto com assinatura | `service_role` | Uma resposta por job | Guardião recusa: não grava |
| `ai_runs` | Central | `lead_id`, `config_version`, `model`, `tokens_in`, `tokens_out`, `latency_ms`, `tools` (nomes), `result` (`enviado`, `transferido`, `barrado`, `falha`, `simulado`), `guard_reason`, `cost_estimate`, `is_test` | `service_role`; leitura `platform_admin` | — | Sem texto de mensagem (o texto fica em `messages`) |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-4-10 | `ia_pode_responder` | Verdadeiro só com: estado `ia`; IA ligada na unidade e geral; versão publicada com nome; janela aberta; sem opt-out; etapa `Novo contato`, `Em atendimento` ou `Sem resposta`; modo ativo pelo horário | — | DEC-04 |
| RN-4-11 | Modo abertura e ficha completa | Move para `Qualificado` e transfere com `ficha_completa` | — | DEC-04 |
| RN-4-12 | Lead pede preço ou orçamento | Pede nome completo e e-mail (Manual p. 9), registra, transfere com `pediu_preco` | Já tem os dois: transfere direto | Manual |
| RN-4-13 | Lead pede pessoa, manda imagem ou documento, quer fechar, faz objeção, pergunta fora do FAQ | Transfere com o motivo correspondente e o resumo | — | DEC-14 |
| RN-4-14 | Lead pergunta se é robô | Texto fixo do admin; transfere com `perguntou_se_e_robo` | — | DEC-13 |
| RN-4-15 | Fora do horário, ao transferir | Avisa quando a equipe responde (texto do admin com o horário de abertura) | — | Manual p. 8 ("seja transparente") |
| RN-4-16 | 6 mensagens da IA sem campo novo na ficha | Transfere com `sem_avanco` | Limite editável no admin | DEC-04 |
| RN-4-17 | Resposta sai | Grava a hora para o K1a | — | Emenda 01, §4 |

## Fluxo e regras

1. Mensagem do lead chega → o banco cria ou adia o `ai_job` (25 s, máximo 60 s).
2. O agendador chama `ai-atendente`; a função confere `ia_pode_responder` (RN-4-10). Falso: encerra o job.
3. Monta o contexto, chama o Gemini, executa as ferramentas (até 3 rodadas) e recebe `{texto, intencao, transferir}`.
4. Pergunta sobre robô: texto fixo + transferência (RN-4-14), sem chamar o modelo para isso.
5. Guardião confere o texto. Barrado: nada sai, transferência `guardiao`.
6. Sai a resposta pelo envio da fase 3 com a assinatura; grava `ai_runs`; aplica as transferências (RN-4-11 a RN-4-16).
7. O desligamento automático olha os últimos 15 minutos de `ai_runs` a cada execução.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Lead `TESTE` às 23h: "quero saber do PAS" | Resposta em até 1 min, curta, com pergunta, assinada | — |
| Abertura | Lead às 10h, ficha completa na 3ª mensagem | Vai para `Qualificado` e para o topo do transbordo | — |
| Limite | Lead manda três mensagens seguidas | Uma resposta só, depois da última | — |
| Falha | Modelo escreve "a mensalidade fica em R$ 500" | Guardião barra, nada sai, transbordo `guardiao` | — |
| Falha | Gemini fora | 1 nova tentativa, depois `falha_ia` | Desligamento automático se repetir |
| Humano | Atendente assumiu durante os 25 s | Job encerra sem resposta | — |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC, a Emenda 01, o anexo de carga inicial, as SPECs 4-002 e 4-003, `message-send` e `envio_enfileira`.
2. **Alterar somente:** o que está em "Inclui".
3. **Não alterar:** janela, opt-out, custo, SLA, cadência e as RPCs de etapa da fase 3 (só chamar).
4. **Executar nesta ordem:** `ai_jobs`, `ia_pode_responder` e modo simulação; chamada ao Gemini com saída estruturada; ferramentas e etapas; Guardião; transferências e textos fixos; áudio; desligamento automático; bateria de avaliação.
5. **Parar e pedir validação quando:** a bateria de avaliação não passar, ou alguém pedir para a IA falar preço ou fechar matrícula.
6. **Estado válido ao parar:** IA geral desligada.
7. **Ritmo:** tasks em sequência; teste do cliente no fim.

## Checklist de execução

- [ ] `ia_pode_responder` com todos os casos da RN-4-10.
- [ ] Espera de 25 s que agrupa mensagens.
- [ ] Gemini com saída estruturada e ferramentas; chave só no servidor.
- [ ] Guardião e textos fixos.
- [ ] Transferências com motivo, intenção e resumo.
- [ ] Áudio entendido.
- [ ] Desligamento automático.
- [ ] Bateria de avaliação verde.

## Critérios de aceite

- [ ] **CA-4-14:** `ia_pode_responder` é falso em cada condição da RN-4-10 (estado, interruptores, versão sem nome, janela fechada, opt-out, etapa, horário) e verdadeiro só no modo certo do horário.
- [ ] **CA-4-15:** mensagens seguidas recebem uma resposta só; a resposta sai pelo envio da fase 3 com a assinatura da unidade, `kind = 'ai'`, e grava a hora do K1a; nenhuma resposta da IA sai fora da janela nem como modelo.
- [ ] **CA-4-16:** `registrar_ficha` grava só campos da ficha; com a ficha completa o lead vai para `Qualificado` e para o transbordo `ficha_completa`; `enviar_anexo_faq` envia o anexo cadastrado.
- [ ] **CA-4-17:** o Guardião barra cada padrão proibido (prova com um caso por padrão); texto barrado não sai e vira transbordo `guardiao`.
- [ ] **CA-4-18:** pedido de pessoa, imagem, documento, "quero fechar", objeção, pergunta fora do FAQ, "é robô?" e 6 mensagens sem avanço geram o transbordo com o motivo certo e o resumo; fora do horário o lead recebe o aviso do horário da equipe.
- [ ] **CA-4-19:** áudio é entendido no modo simulação, com resposta coerente com o conteúdo (o áudio que chega pelo WhatsApp liga com a SPEC-4-001); 5 falhas do Gemini em 15 minutos desligam a IA geral e avisam o admin; `ai_runs` registra cada execução sem texto de mensagem; a chave não aparece no navegador nem em log.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P4.4-01: `ia_pode_responder` em cada condição | pgTAP | falso/verdadeiro conforme a RN-4-10 | saída do teste |
| RED | P4.4-02: Guardião com um texto por padrão proibido | teste unitário da função | todos barrados | saída do teste |
| GREEN | P4.4-03: três mensagens em 10 s | Gemini de mentira (`scripts/gemini-falsa.mjs`) + consulta | uma resposta | saída do teste |
| GREEN | P4.4-04: ficha completa pela IA | Gemini de mentira | `Qualificado` + transbordo | consulta |
| GREEN | P4.4-05: Gemini de mentira fora do ar 5 vezes | teste | IA geral desligada + aviso | consulta |
| AVALIAÇÃO | P4.4-06: bateria com 20 conversas tiradas do Manual (as perguntas das pp. 3–14, "Quanto custa?", "Vocês têm PAS?", "Quero fechar", "Achei caro", "É robô?", um áudio) contra o Gemini real em modo simulação | script `scripts/avaliar-ia.mjs` | **0** respostas com preço, desconto ou outra unidade; motivo de transbordo certo em ≥ 18 de 20; toda resposta com no máximo 600 caracteres | relatório da bateria |
| REGRESSÃO | P4.4-07: provas da fase 3 e das SPECs 4-002 e 4-003 | pgTAP + Playwright | verdes | saída dos testes |

**Dados/fixtures:** leads `TESTE`; um Gemini de mentira local para os testes determinísticos; o Gemini real só na bateria de avaliação e no "Simular".

## Teste humano do cliente

- **Quando:** quando todas as tasks vinculadas estiverem no ar e a bateria de avaliação estiver verde
- **Quem testa:** Luiz, com o Hikari e um atendente de cada unidade
- **Passos:** na tela Simular (ou com o número de teste, se a SPEC-4-001 já estiver no ar), fora do horário simulado, conversar como um pai interessado no ENEM e depois pedir o preço; repetir no horário até a ficha completar; mandar um áudio; perguntar "é robô?"; tentar arrancar um preço ("só me fala mais ou menos"); mandar uma foto; ver os transbordos na fila de manhã; desligar a IA da unidade no meio de uma conversa.
- **Resultado esperado:** respostas curtas e no jeito do Manual, nenhum preço, transbordo certo com resumo, IA parando quando desligada.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Handoff e operação

- **Como operar:** o gestor olha o filtro Transbordo na abertura do expediente; o admin ajusta a área Assistente e confere `ai_runs` no painel da SPEC-4-007.
- **Como monitorar:** respostas barradas, falhas, desligamentos automáticos e custo por conversa.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Decidir quando a IA pode responder e agrupar as mensagens do lead | Academia das Específicas | SPEC-4-004 | CA-4-14, CA-4-15 | P4.4-01, P4.4-03 | pgTAP + teste | SPECs 4-002 e 4-003 | ☐ |
| pendente | Responder com o Gemini, preencher a ficha e enviar anexos do FAQ | Academia das Específicas | SPEC-4-004 | CA-4-15, CA-4-16, CA-4-19 | P4.4-04 | consulta + teste | Task anterior; chave da Gemini | ☐ |
| pendente | Barrar no Guardião o que a IA não pode dizer | Academia das Específicas | SPEC-4-004 | CA-4-17 | P4.4-02 | teste unitário | Task anterior | ☐ |
| pendente | Transferir para a equipe com motivo e resumo, e avisar o horário | Academia das Específicas | SPEC-4-004 | CA-4-18 | P4.4-04 | consulta + captura | Task anterior | ☐ |
| pendente | Desligar a IA sozinha em caso de erro e passar na bateria de avaliação | Academia das Específicas | SPEC-4-004 | CA-4-19 | P4.4-05, P4.4-06, P4.4-07 | relatório da bateria | Tasks anteriores | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| 2026-11-10 | Consultora | Ordem da fase | Vem antes da SPEC-4-001: áudio provado na simulação; áudio do WhatsApp vai para a equipe até a 4-001 entrar |
