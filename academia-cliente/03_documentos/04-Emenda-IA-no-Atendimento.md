# Emenda 01 ao Escopo Definitivo — IA no atendimento pelo WhatsApp

| Item | Informação |
|---|---|
| **Status** | Vigente |
| **Data** | 10/11/2026 |
| **Decisão** | Da Academia (uso de IA no atendimento), comunicada pela consultora (Kim) em 10/11/2026; desenho técnico da consultora, aprovado na mesma data |
| **Altera** | Escopo Definitivo v1.0 (`02-Escopo-Definitivo.md`): DEC-04, DEC-05, §1 (K1), §9, §18 (fases 4 e 5), §19, §20 (GATE-02, GATE-09, GATE-13) |
| **Substitui** | A minuta `03-Proposta-Revisao-Arco-Fases.md` (19/10), que nunca foi aprovada |

Onde este documento e o Escopo Definitivo divergem, vale este documento.

## 1. O que muda e por quê

A Academia decidiu usar IA para responder o lead em dois momentos: **fora do horário de atendimento** e **no começo da conversa, para qualificar**. Fora disso, quem atende são as pessoas. O objetivo do projeto não muda: todo lead atendido rápido, qualificado e acompanhado até a matrícula ou a perda registrada.

A IA ataca as duas perdas que mais pesam hoje (Mapeamento MKT, seção 8): o lead que espera (sobretudo à noite e de madrugada) e a qualificação incompleta antes do orçamento. Preço, desconto e fechamento continuam com as pessoas.

## 2. Decisões

| ID | Antes | Agora |
|---|---|---|
| **DEC-04** | Nenhum agente de IA envia mensagem, move etapa, promete condição ou recomenda curso autonomamente | **A IA responde o lead sozinha** em dois modos: fora do horário (até o transbordo) e na abertura e qualificação dentro do horário. Ela **não** fala preço nem faixa de preço, não dá desconto, não promete data, não recomenda turma em definitivo, não fala da outra unidade e não envia modelo pago. Toda ação escrita dela passa pelo Guardião (AG-13) |
| **DEC-05** | Atendimento humano consultivo preservado | Mantido para orçamento, objeção, negociação, visita e matrícula. A IA segue o Manual de Boas Práticas: uma pergunta por vez, termina com pergunta, não despeja informação, chama pelo nome |
| **DEC-13 (nova)** | — | **Persona com nome.** A IA se apresenta com o nome configurado por unidade no admin, assinando no padrão das atendentes (`*Nome - Unidade X:*`). Se o lead perguntar se está falando com um robô, ela não nega: diz que é o atendimento virtual da unidade e passa para uma pessoa |
| **DEC-14 (nova)** | — | **Transbordo como etiqueta.** Quando a IA passa a conversa, o lead recebe a etiqueta `Transbordo` com o motivo. A Caixa tem o filtro e uma fila de prioridade: janela de 24h perto de fechar, depois temperatura do lead, depois tempo de espera. A temperatura é calculada pela central a partir de uma lista fechada de intenções |
| **DEC-15 (nova)** | — | **Tudo da IA se edita no admin**, sem código: identidade, horários, instruções, FAQ, exemplos, textos do orçamento, liga e desliga. Cada mudança é testada em "Simular" e publicada como versão, com volta à anterior |
| **DEC-16 (nova)** | — | **Modelo de IA: Gemini 2.5 Flash, pela API paga do Google** (sem uso dos dados para treino). O nome do modelo fica na configuração para poder trocar sem mexer no código |
| **DEC-17 (nova)** | — | **A ficha do lead vive só na central.** O Kommo não recebe escrita; dele só se importa o histórico |
| **DEC-18 (nova)** | — | **Orçamento gravado no MatriculaWeb** (`POST /api/OrcamentoPublico/interesse` e `POST /api/OrcamentoPublico`), que devolve o número oficial. O PDF é gerado pela central com esses valores (modelo em `04_fase-atual/anexos/`) e enviado pelo WhatsApp com a mensagem padrão, a validade e uma pergunta. Isto substitui a decisão de 28/10 de só simular |
| **DEC-19 (nova)** | — | **Contatos da mesma compra.** Um lead pode ter vários números (aluno, mãe, pai, responsável financeiro). Um número novo só é ligado a um lead existente quando é a mesma compra: mesmo aluno e compra ainda aberta. Aluno diferente é indicação e vira lead novo |

## 3. Agentes (§9)

| ID | Agente | Autonomia |
|---|---|---|
| **AG-14 (novo)** | Atendente virtual (IA) | Responde nos dois modos da DEC-04. Ferramentas: registrar a ficha, responder pelo FAQ aprovado, enviar anexo do FAQ (por exemplo, a grade horária), pedir o número de quem falou antes, transferir para humano. Não move etapa além do que a ficha completa já move pela regra existente |
| AG-10, AG-11, AG-12 | — | O resumo do transbordo (AG-11) entra na fase 4 como parte do AG-14. O sugestor de resposta (AG-10) e o classificador de perda (AG-12) ficam para depois da Black Friday |
| AG-13 | Guardião | Agora valida também cada resposta do AG-14 antes do envio. Resposta barrada não sai: a conversa vai para humano |

## 4. Indicadores (§1 e §17)

| ID | Antes | Agora |
|---|---|---|
| **K1a** | — | ≥95% dos leads com **primeira resposta (IA ou humano) em até 1 minuto** |
| **K1b** | K1 (primeira resposta humana ≤ SLA) | ≥95% dos transbordos com **resposta humana dentro do SLA** (10 minutos no horário; até o fim do primeiro intervalo de SLA depois da abertura do expediente, para o que chegou fora do horário) |
| K2, K3 | — | Sem mudança |
| Novos | — | Por atendente: tempo do transbordo até a resposta, orçamentos enviados, conversão. IA: transbordos por motivo, fichas completas pela IA, respostas barradas pelo Guardião, custo por conversa |

## 5. Gates (§20)

| Gate | Antes | Agora |
|---|---|---|
| **GATE-09** | Política LGPD aprovada antes de qualquer dado real | **Dispensado pela consultora.** Ficam obrigatórias as proteções técnicas: RLS por unidade, segredos só no servidor, registros sem texto de mensagem, mídia em armazenamento privado com expiração, IA pela API paga |
| **GATE-13** | Aprovação formal da camada de IA | **Fechado** com DEC-16 (Gemini 2.5 Flash, API paga, sem treino com os dados) e com os limites da DEC-04 |
| GATE-03 | Conta Meta, número, modelos | Sem mudança: trava só a virada para a operação real (SPEC-4-007) |
| GATE-04 | Exportação do Kommo | Sem mudança: trava só a importação real (SPEC-4-007) |
| GATE-02 | Baseline de 30 dias de operação real | Passa a ser medido pelo histórico importado do Kommo (período comparável), na fase 5 |

## 6. Fases (§18)

- **Fase 4 — Atendimento com IA e virada para a operação real** (antes da Black Friday): SPECs 4-001 a 4-007 em `04_fase-atual/`.
- **Fase 5 — Validação, KPIs e retirada do Kommo** (ciclo jan–fev/2027): K1a, K1b, K2 e K3 medidos, sugestor de resposta e classificador de perda (AG-10 e AG-12), retirada do Kommo. Detalhada na liberação dela.

## 7. Fora do escopo (§19)

- Sai: "agente autônomo de atendimento" (agora existe, nos limites da DEC-04).
- Continua fora: IA falando preço, dando desconto ou fechando matrícula; escrita no Kommo; envio de modelo pago pela IA; disparo em massa.
