# SPEC-4-007 — Métricas do atendimento e virada para a operação real

**Fase:** 4  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); a virada é feita junto com a consultora; aceite do Luiz  
**Origem no escopo:** Emenda 01 (K1a, K1b, indicadores por atendente e da IA, GATE-09 dispensado); GATE-03 (Meta) e GATE-04 (Kommo); dívida de 28/10 (limpeza da massa de teste antes do primeiro dado real); SPEC-3-007 (importação)  
**Degrau da solução:** reuso — painel gerencial da SPEC-3-006 com abas novas; importação da SPEC-3-007 com o arquivo real; roteiro de virada.

## Contexto e decisões fechadas

- **Estado atual:** tudo roda em teste. Os números de WhatsApp das duas unidades estão ligados ao Kommo (API oficial, com SalesBot de boas-vindas). Cada unidade tem o próprio número, sem ligação entre eles (Mapeamento MKT, seção 4).
- **Estado desejado:** antes da Black Friday, os dois números respondem na central; a IA atende fora do horário e na abertura; a equipe atende o transbordo; o gestor vê K1a, K1b, o desempenho de cada atendente e o da IA.
- **Decisões já fechadas:**
  - **K1a:** leads com primeira resposta (IA ou humano) em até 1 minuto ÷ leads recebidos. **K1b:** transbordos com resposta humana dentro do SLA ÷ transbordos; para o que chegou fora do horário, o prazo conta a partir da abertura do expediente. O K1 da fase 3 continua calculado.
  - **Por atendente** (gestor vê a unidade; atendente vê só o próprio): conversas assumidas, tempo do transbordo até a primeira resposta (mediana e p90), orçamentos enviados, conversão para `Novo aluno`.
  - **Da IA:** conversas atendidas, fichas completas pela IA, transbordos por motivo, respostas barradas pelo Guardião, falhas, desligamentos automáticos, custo estimado por conversa (tokens × preço por milhão cadastrado no admin).
  - **Sem política LGPD formal** (Emenda 01). Ficam as proteções técnicas já construídas.
  - **Virada única por unidade, numa data marcada:** importar o histórico, mudar o número do Kommo para o app da Academia, desligar o SalesBot, ligar o envio real. Sem período com os dois sistemas respondendo o mesmo número.
  - **Primeiro dia:** IA só fora do horário. A abertura no horário é ligada no dia seguinte, se o primeiro dia estiver bem.
- **Bloqueios:** GATE-03 (conta Meta verificada, números no app da Academia, modelos aprovados: boas-vindas de formulário, FUP 3 e retomada) e GATE-04 (exportação do Kommo com chave nova) travam só a virada, não a construção.

## Resultado observável

No painel gerencial, as abas "Atendimento" (K1a, K1b), "Equipe" (por atendente) e "Assistente" (IA). No dia da virada, uma mensagem real para o número da Asa Sul cai na central, a IA responde (fora do horário) e o histórico do Kommo aparece no lead que já existia.

## Limites e dependências

- **Inclui:** K1a, K1b e as abas novas do painel; preço por milhão de tokens no admin; limpeza da massa `TESTE`; ensaio com volume; importação real do Kommo; roteiro da virada e plano de volta; treino rápido das atendentes.
- **Fora de escopo:** K3 contra a linha de base e retirada do Kommo (fase 5).
- **Entradas e pré-condições:** SPECs 4-001 a 4-006 aceitas; GATE-03 e GATE-04 para a virada.
- **Atores e permissões mínimas:** `unit_manager` vê a unidade; `attendant` vê os próprios números; `platform_admin` vê tudo; `marketing` vê K1a e K1b sem nomes de atendentes.
- **Risco e plano B:** algo errado no dia: desligar a IA e o envio (níveis da SPEC-4-002), devolver o webhook do número ao Kommo e religar o SalesBot. O roteiro é ensaiado antes.

## Dados e integrações

| Item | Contrato | Observação |
|---|---|---|
| K1a | primeira mensagem `kind in ('ai','text','template','media')` enviada ao lead − chegada do lead ≤ 60 s | Boas-vindas por modelo de formulário conta como resposta só para lead de formulário |
| K1b | primeira mensagem humana depois de `handoff_at` − max(`handoff_at`, abertura do expediente) ≤ SLA | Expediente da área Assistente |
| `ai_runs` | tokens e resultado | Custo = tokens × preço do admin |
| Kommo (exportação real) | arquivo da exportação; data de corte | Chave nova cadastrada direto no segredo, nunca no grupo |
| Meta | números das duas unidades no app da Academia; `WA_APP_SECRETS`, `WA_ACCESS_TOKENS`, `WA_VERIFY_TOKEN` | Cadastro feito por uma pessoa da Academia com a consultora |

## Roteiro da virada (por unidade)

1. **Véspera:** limpeza da massa `TESTE` (registros operacionais com `is_test` saem; a auditoria append-only fica, marcada); ensaio do plano de volta.
2. **Importação real do Kommo** com a data de corte (SPEC-3-007, agora com o arquivo real) e o relatório de conferência.
3. **Publicar** a versão no Skip; publicar as Edge Functions (`wa-webhook`, `lead-webhook`, `message-send`, `media-fetch`, `ai-atendente`, `quote-simulate`, `quote-pdf`, `reports-export`, `kommo-import`); conferir segredos e agendadores.
4. **No horário combinado:** desligar o SalesBot no Kommo; mudar o webhook do número para o app da Academia; ligar o envio real; ligar a IA só no modo fora do horário.
5. **Primeira hora:** mensagem de teste de um celular da equipe; acompanhar dead-letter, falhas e a fila de transbordo.
6. **48 horas:** relatório curto (K1a, K1b, transbordos, barradas, custo) e decisão de ligar a abertura no horário.

## Critérios de aceite

- [ ] **CA-4-27:** K1a e K1b do painel batem com a consulta direta no mesmo período, por unidade, com e sem o expediente.
- [ ] **CA-4-28:** a aba Equipe mostra os números de cada atendente (gestor vê a unidade, atendente só o próprio, marketing não vê nomes); a aba Assistente mostra conversas, fichas, transbordos por motivo, barradas, falhas e custo por conversa.
- [ ] **CA-4-29:** depois da limpeza, nenhum registro operacional `is_test` sobra fora da auditoria, e os painéis não mudam (já excluíam teste).
- [ ] **CA-4-30:** no ensaio com 300 leads de mentira em 1 hora pelas duas unidades (o volume da última campanha inteira foi 334 contatos), nenhuma mensagem se perde, o p90 da resposta da IA fica em até 60 s e o plano de volta é executado e desfeito.
- [ ] **CA-4-31:** na virada, a mensagem real cai na central, a IA responde no modo do horário, o SalesBot não responde mais e o histórico importado aparece no lead.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P4.7-01: fixture com respostas dentro e fora de 60 s e transbordos noturnos | pgTAP | K1a e K1b exatos | saída do teste |
| GREEN | P4.7-02: painel × consulta direta | tela + consulta | iguais | capturas |
| RED | P4.7-03: atendente pede os números de outro atendente | pgTAP | recusado | saída do teste |
| GREEN | P4.7-04: limpeza em cópia do banco | script | zero `is_test` operacional; painéis iguais | consulta |
| GREEN | P4.7-05: ensaio com `meta-falsa.mjs` e Gemini de mentira | script de carga | nada perdido; p90 ≤ 60 s; plano de volta ok | relatório |
| GREEN | P4.7-06: virada | roteiro | itens 4 e 5 do roteiro conferidos | relatório de 48 h |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC, a Emenda 01, as SPECs 3-006 e 3-007.
2. **Executar nesta ordem:** K1a e K1b; abas Equipe e Assistente; limpeza (em cópia, depois para valer na véspera); ensaio; importação real e virada (com a consultora).
3. **Parar e pedir validação quando:** antes de cada passo do roteiro da virada que mexa no número, no Kommo ou no envio real.
4. **Estado válido ao parar:** envio real desligado até o passo 4 do roteiro.

## Checklist de execução

- [ ] K1a e K1b.
- [ ] Abas Equipe e Assistente.
- [ ] Limpeza da massa de teste.
- [ ] Ensaio com volume e plano de volta.
- [ ] Importação real do Kommo.
- [ ] Virada das duas unidades e relatório de 48 horas.

## Teste humano do cliente

- **Quando:** primeiro no ensaio; depois na virada
- **Quem testa:** Luiz, com o Hikari, os gestores e um atendente de cada unidade
- **Passos:** ver o painel durante o ensaio; executar o plano de volta; no dia da virada, mandar mensagem real de um celular da equipe e acompanhar a primeira hora; ler o relatório de 48 horas.
- **Resultado esperado:** números batendo, volta ensaiada, número real respondendo na central.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro, depois do relatório de 48 horas.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Medir o K1a e o K1b no painel | Academia das Específicas | SPEC-4-007 | CA-4-27 | P4.7-01, P4.7-02 | pgTAP + capturas | SPEC-4-004 | ☐ |
| pendente | Mostrar os números de cada atendente e da IA | Academia das Específicas | SPEC-4-007 | CA-4-28 | P4.7-03 | pgTAP + captura | Task anterior | ☐ |
| pendente | Limpar a massa de teste e ensaiar a virada com volume | Academia das Específicas | SPEC-4-007 | CA-4-29, CA-4-30 | P4.7-04, P4.7-05 | relatório do ensaio | SPECs 4-001 a 4-006 | ☐ |
| pendente | Importar o histórico real do Kommo e virar os números para a central | Academia das Específicas | SPEC-4-007 | CA-4-31 | P4.7-06 | relatório de 48 h | GATE-03, GATE-04; ensaio aprovado | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
