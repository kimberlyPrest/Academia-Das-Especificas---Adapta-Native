# SPEC-4-002 — Assumir a conversa, transbordo e fila de prioridade

**Fase:** 4  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** Emenda 01 (DEC-14, K1b, indicadores por atendente); dívida de 12/10 (fila sem dono, CL-016); dívida de 19/10 (E2E da caixa)  
**Degrau da solução:** construção sobre a Caixa da fase 3 — acrescenta o estado da conversa, o dono, a etiqueta de transbordo e a ordenação. É a base que a IA (SPEC-4-004) usa para saber quando pode falar.

## Contexto e decisões fechadas

- **Estado atual (v0.0.77):** a Caixa lista as conversas da unidade por tempo de espera, sem dono. Duas atendentes podem responder o mesmo lead. A tela atualiza a cada 10 segundos, sem aviso de mensagem nova.
- **Estado desejado:** cada conversa tem um estado (`ia`, `humano`, `pausada`) e, quando está com humano, um dono. A atendente assume, devolve ou pausa. A conversa que a IA passa recebe a etiqueta `Transbordo`, com o motivo e o resumo, e entra numa fila ordenada pela janela de 24h e pela temperatura.
- **Decisões já fechadas:**
  - **Estados:** `ia` (a IA pode responder, nos modos da SPEC-4-004), `humano` (só pessoas; tem dono), `pausada` (ninguém automatiza nada; só pessoas).
  - **Depois que uma pessoa responde, a conversa fica `humano`** até alguém devolver para a IA. A IA nunca fala depois de um humano no mesmo atendimento.
  - **Assumir** grava o dono. Outra atendente vê "com Fulana" e só responde se assumir de novo (fica no histórico).
  - **Três níveis de parada:** na conversa (Assumir, Devolver, Pausar); na unidade (gestor liga e desliga a IA da unidade); geral (admin desliga a IA de todas as unidades). O desligamento automático por erro fica na SPEC-4-004.
  - **Etiqueta `Transbordo`** com um motivo de lista fechada: `ficha_completa`, `pediu_preco`, `quer_fechar`, `objecao_ou_desconto`, `pediu_humano`, `documento_ou_imagem`, `fora_do_faq`, `perguntou_se_e_robo`, `sem_avanco`, `guardiao`, `falha_ia`. A etiqueta sai quando uma pessoa responde.
  - **Temperatura:** a central calcula pela intenção registrada (lista fechada, pesos editáveis no admin). Padrão: quer fechar 5; pediu orçamento ou preço 4; objeção ou desconto 3; quer visita 3; ficha completa 2; dúvida, documento ou pediu humano 1.
  - **Ordem da fila de transbordo:** 1º janela de 24h com menos de 2 horas (crítica); 2º temperatura (maior primeiro); 3º tempo de espera (maior primeiro). Conversas com a janela fechada vão para um grupo separado no fim ("só modelo").
- **Bloqueios:** nenhum.

## Resultado observável

Na Caixa, o filtro **Transbordo** mostra os leads que a IA passou, do mais urgente para o menos, com 🔥 pela temperatura, o tempo que falta da janela, o motivo e o resumo. A atendente clica em **Assumir**, responde, e o cartão mostra o nome dela. O gestor desliga a IA da unidade com um botão.

## Limites e dependências

- **Inclui:** tabela `conversation_control`; botões Assumir, Devolver para a IA e Pausar; interruptor da IA por unidade (gestor) e geral (admin); etiqueta e motivo de transbordo; temperatura e fila de prioridade; filtros "Transbordo", "Com a IA", "Comigo"; aviso de mensagem nova (som e notificação do navegador, com permissão); respostas prontas por unidade (editáveis pelo gestor); teste E2E do fluxo da caixa.
- **Fora de escopo:** a IA em si (SPEC-4-004); configurações da IA (SPEC-4-003).
- **Saídas/artefatos:** migração com `conversation_control`, `handoff_reasons`, `intent_weights`, `quick_replies`; RPCs `conversa_assumir`, `conversa_devolver`, `conversa_pausar`, `ia_unidade_definir`, `ia_geral_definir`; telas da Caixa; Playwright com o fluxo da caixa.
- **Atores e permissões mínimas:** `attendant` assume, devolve e pausa conversas da própria unidade; `unit_manager` também liga e desliga a IA da unidade e edita respostas prontas; `platform_admin` desliga a IA geral e edita os pesos.
- **Risco e plano B:** duas atendentes clicam em Assumir ao mesmo tempo: a RPC trava a linha; a segunda recebe "já está com Fulana".
- **Rollback:** desligar a IA geral; a Caixa continua funcionando como na fase 3.

## Dados e integrações

| Tabela/RPC | Campos/contrato | Permissão | Idempotência/concorrência | Erro |
|---|---|---|---|---|
| `conversation_control` | `lead_id` (único), `state` (`ia`, `humano`, `pausada`), `owner_user_id`, `owner_since`, `handoff_reason`, `handoff_summary`, `intent`, `temperature`, `handoff_at`, `updated_at` | RLS por unidade | Uma linha por lead; histórico em auditoria append-only | — |
| `conversa_assumir(lead_id)` | estado vira `humano`, dono = quem chamou | `attendant` da unidade | `select … for update` | Outra pessoa já dona: devolve o nome, não troca sem confirmação |
| `conversa_devolver(lead_id)` | estado vira `ia`, dono vazio | dono ou `unit_manager` | — | IA desligada na unidade: avisa e deixa `humano` |
| `conversa_pausar(lead_id)` | estado vira `pausada` | `attendant` da unidade | — | — |
| `ia_unidade_definir(unit_id, ligada)` / `ia_geral_definir(ligada)` | grava em `ai_settings` | `unit_manager` / `platform_admin` | — | — |
| `intent_weights` | intenção → peso | `platform_admin` | — | — |
| `quick_replies` | unidade, atalho, texto | `unit_manager` | — | — |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-4-05 | Pessoa envia mensagem num lead | Estado `humano`, dono = quem enviou (se vazio), etiqueta de transbordo sai | — | DEC-14 |
| RN-4-06 | Conversa `humano` ou `pausada` | A IA não responde | — | DEC-04 |
| RN-4-07 | IA desligada na unidade ou geral | A IA não responde nenhuma conversa da unidade | — | DEC-04 |
| RN-4-08 | Transbordo registrado | Etiqueta + motivo + resumo + intenção; temperatura calculada pela central | — | DEC-14 |
| RN-4-09 | Ordem da fila | Janela < 2h → temperatura → espera; janela fechada no fim | — | DEC-14 |

## Fluxo e regras

1. Lead novo nasce com estado `ia` se a IA estiver ligada na unidade; senão, `humano` sem dono.
2. Quando a IA transfere (SPEC-4-004), grava motivo, resumo e intenção; a central calcula a temperatura (RN-4-08).
3. A fila de transbordo ordena pela RN-4-09 e avisa as atendentes da unidade com som e notificação.
4. A atendente assume e responde (RN-4-05); devolve para a IA ou pausa quando quiser.
5. O gestor liga e desliga a IA da unidade; o admin, a geral (RN-4-07).

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Transbordo `quer_fechar` com janela de 1h30 | Primeiro da fila, 🔥, motivo e resumo visíveis | — |
| Concorrência | Duas atendentes assumem juntas | Uma fica dona; a outra vê o nome | — |
| Limite | Janela fechada | Grupo "só modelo" no fim da fila | — |
| Falha | Atendente de outra unidade tenta assumir | 403 | — |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC, a Emenda 01, `src/pages/Caixa.tsx` e as migrações da caixa e do K1.
2. **Alterar somente:** o que está em "Inclui".
3. **Não alterar:** janela, custo, opt-out, SLA e K1 da fase 3 (o K1b é da SPEC-4-007).
4. **Executar nesta ordem:** estado e dono; botões e interruptores; transbordo, temperatura e fila; aviso e respostas prontas; E2E.
5. **Estado válido ao parar:** Caixa da fase 3 funcionando, com ou sem os botões novos.
6. **Ritmo:** tasks em sequência; teste do cliente no fim.

## Checklist de execução

- [ ] Estado e dono por conversa, com histórico.
- [ ] Assumir, Devolver e Pausar, com trava de concorrência.
- [ ] Interruptor por unidade e geral.
- [ ] Etiqueta, motivo, resumo, temperatura e fila ordenada.
- [ ] Aviso de mensagem nova e respostas prontas.
- [ ] E2E da caixa no Playwright.

## Critérios de aceite

- [ ] **CA-4-05:** a mensagem de uma pessoa deixa a conversa `humano`, com dono, e tira a etiqueta de transbordo; Assumir com outro dono avisa o nome; outra unidade recebe 403.
- [ ] **CA-4-06:** com a conversa `humano` ou `pausada`, ou com a IA desligada na unidade ou geral, nenhuma resposta automática sai (prova pela RPC usada pela IA).
- [ ] **CA-4-07:** o filtro Transbordo ordena por janela crítica, temperatura e espera, e põe a janela fechada no fim; a temperatura bate com os pesos do admin.
- [ ] **CA-4-08:** mensagem nova numa conversa da atendente toca o aviso; respostas prontas da unidade aparecem pelo atalho na caixa de texto.
- [ ] **CA-4-09:** o E2E (login, filtro Transbordo, assumir, responder, devolver) passa no Playwright, junto com as provas da fase 3.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P4.2-01: RPC "pode a IA responder?" com conversa `humano`, `pausada`, IA da unidade desligada e IA geral desligada | pgTAP | falso nos quatro casos | saída do teste |
| RED | P4.2-02: duas sessões chamam `conversa_assumir` juntas | pgTAP | uma dona, a outra recebe o nome | saída do teste |
| GREEN | P4.2-03: três transbordos `TESTE` (quer fechar com 1h30 de janela, pediu preço com 20h, dúvida com janela fechada) | consulta da fila | ordem: quer fechar, pediu preço, dúvida (no fim) | consulta + captura |
| GREEN | P4.2-04: atendente responde | tela + consulta | estado `humano`, dono, etiqueta removida | consulta |
| REFACTOR/REGRESSÃO | P4.2-05: E2E da caixa; `supabase test db` | Playwright + pgTAP | verdes | saída dos testes |

## Teste humano do cliente

- **Quando:** quando todas as tasks vinculadas estiverem no ar
- **Quem testa:** Luiz, com um atendente de cada unidade
- **Passos:** criar três transbordos `TESTE`; abrir o filtro e conferir a ordem; assumir um com uma atendente e tentar assumir o mesmo com a outra; responder e ver a etiqueta sair; devolver para a IA; desligar e ligar a IA da unidade; usar uma resposta pronta.
- **Resultado esperado:** fila na ordem certa, dono visível, IA parando nos três níveis.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Dar dono à conversa com Assumir, Devolver e Pausar | Academia das Específicas | SPEC-4-002 | CA-4-05 | P4.2-02, P4.2-04 | pgTAP + captura | — | ☐ |
| pendente | Ligar e desligar a IA na conversa, na unidade e em tudo | Academia das Específicas | SPEC-4-002 | CA-4-06 | P4.2-01 | pgTAP | Task anterior | ☐ |
| pendente | Mostrar o transbordo com etiqueta, motivo e fila por janela e temperatura | Academia das Específicas | SPEC-4-002 | CA-4-07 | P4.2-03 | consulta + captura | Estado da conversa | ☐ |
| pendente | Avisar mensagem nova, criar respostas prontas e cobrir a caixa com E2E | Academia das Específicas | SPEC-4-002 | CA-4-08, CA-4-09 | P4.2-05 | Playwright verde | Fila no ar | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
