# SPEC-4-001 — Meta de verdade, mídia e MatriculaWeb de verdade

**Fase:** 4  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** Emenda 01 (DEC-04, DEC-18); pendências da fase 3 (teste com o número de teste da Meta e com a API real do MatriculaWeb); escopo §6.9 e §15  
**Degrau da solução:** reuso + construção mínima — usa `wa-webhook`, `message-send` e `quote-simulate` da fase 3 e acrescenta três chamadas da Meta (baixar mídia, enviar arquivo, marcar como lida).

## Contexto e decisões fechadas

- **Estado atual (v0.0.77):** a entrada da Meta confere a assinatura e não duplica; o `message-send` envia texto e modelo pela Cloud API, com tentativas e motivos de erro mapeados. A mídia recebida fica só com o `media_id` (a tela diz que o arquivo não abre). Nada foi testado ainda contra a Meta de verdade nem contra a API real do MatriculaWeb.
- **Estado desejado:** com o número de teste do app da Academia, mensagem, áudio, imagem e documento chegam e abrem na conversa; a central envia texto, modelo, imagem e PDF; a conversa respondida aparece como lida para o lead; unidades, turmas e simulação vêm do MatriculaWeb real.
- **Decisões já fechadas:**
  - Arquivos recebidos e enviados ficam num bucket **privado** do Storage (`midias`), por lead, com link assinado de curta duração para a tela. Prazo de guarda configurável no console (padrão 180 dias); depois disso o arquivo é apagado e a conversa mostra "arquivo expirado".
  - O token da Meta continua só nos segredos das Edge Functions. Nada de token, telefone ou texto em log.
  - O áudio recebido também fica disponível para a IA (SPEC-4-004), que o entende direto.
  - O MatriculaWeb público não pede login e limita requisições por IP. A central continua chamando só pelo servidor.
- **Ordem (emenda de 10/11):** esta SPEC é a **última da construção**; depois dela vem só a SPEC-4-007 (métricas e virada). Assim nada da fase espera o número de teste da Meta. O envio de documento (PDF) é construído na SPEC-4-006 e aqui é testado com o número de teste; o áudio que chega pelo WhatsApp passa a ir para a IA (SPEC-4-004) quando o `media-fetch` entrar.
- **Bloqueios:** nenhum para construir. A operação com o número real fica para a SPEC-4-007 (GATE-03).

## Resultado observável

Do celular da equipe para o número de teste: um áudio, uma foto e um PDF chegam e abrem na conversa; o atendente responde com texto e com um PDF; no celular, a mensagem do lead aparece com os dois tiques azuis. Na tela de orçamento, unidades, turmas e valores vêm do MatriculaWeb de verdade.

## Limites e dependências

- **Inclui:** Edge Function `media-fetch` (baixa a mídia recebida); envio de `image` pelo `message-send` (o `document` já vem da SPEC-4-006, aqui só é testado); marcação de lida; exibição de áudio, imagem e documento na conversa; teste ponta a ponta com o número de teste da Meta; teste da `quote-simulate` contra o MatriculaWeb real.
- **Fora de escopo:** gravar orçamento no MatriculaWeb (SPEC-4-006); IA (SPEC-4-004); número real (SPEC-4-007).
- **Entradas e pré-condições:** app da Academia no Meta for Developers com o número de teste e os celulares da equipe cadastrados como destinatários; consulta ao MatriculaWeb ligada no console.
- **Saídas/artefatos:** `media-fetch`; bucket `midias`; colunas de arquivo em `messages` (`storage_path`, `mime`, `bytes`, `sha256`); componente de mídia na conversa; relatório do teste com a Meta e com o MatriculaWeb.
- **Atores e permissões mínimas:** `attendant` vê e envia mídia só da própria unidade; `marketing` não vê.
- **Superfícies afetadas:** `wa-webhook` (dispara `media-fetch`), `message-send`, `quote-simulate` (só configuração), tela da conversa, console (prazo de guarda).
- **Risco e plano B:** Meta recusa o download (URL vencida): nova tentativa pedindo a URL de novo; falha esgotada mostra "arquivo indisponível" e registra no dead-letter. MatriculaWeb com 429: a tela oferece o orçamento manual (já existe).
- **Rollback:** desligar `media-fetch` no console; as mensagens continuam chegando como hoje.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Central → Meta `GET /{versão}/{media-id}` | Meta | devolve `url`, `mime_type`, `sha256`, `file_size` | Token da Meta (segredo) | 10 s; 3 tentativas; idempotente por `media_id` | URL vencida: pede de novo |
| Central → `url` da mídia | Meta | binário | Cabeçalho de autorização com o token da Meta (segredo) | 30 s; limite de tamanho no console (padrão 16 MB) | Acima do limite: não baixa, mostra o aviso |
| Central → Meta `POST /{versão}/{phone-number-id}/media` | Meta | `file`, `type`, `messaging_product=whatsapp` → `id` | Token | 30 s; 2 tentativas | Falha: mensagem `failed` com o motivo |
| Central → Meta `POST /{versão}/{phone-number-id}/messages` | Meta | `type: image` ou `document` com `id`, `filename`, `caption` | Token | Igual ao texto (fase 3) | Igual ao texto |
| Central → Meta `POST /{versão}/{phone-number-id}/messages` | Meta | `{"messaging_product":"whatsapp","status":"read","message_id":…}` | Token | 1 tentativa; não bloqueia nada | Falha só registra |
| Central → MatriculaWeb `GET /unidades`, `GET /unidades/{id}/turmas`, `POST /simular` | MatriculaWeb | os contratos da SPEC-3-002 | Público, limite por IP | Igual à SPEC-3-002 | Igual à SPEC-3-002 |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-4-01 | Chega mensagem com mídia | `media-fetch` baixa para `midias/{lead}/{id}` e grava o caminho | Tamanho acima do limite | Emenda 01 |
| RN-4-02 | Atendente ou IA respondem uma conversa | Marca como lidas as mensagens do lead até ali | Conversa pausada não marca | Emenda 01 |
| RN-4-03 | Arquivo passa do prazo de guarda | Apaga do Storage; a conversa mostra "arquivo expirado" | — | Emenda 01, §5 |
| RN-4-04 | Envio de arquivo fora da janela de 24h | Recusado, como o texto livre | — | Escopo §14.3 |

## Fluxo e regras

1. A mensagem com mídia chega pelo `wa-webhook`; depois do registro, `media-fetch` baixa o arquivo (RN-4-01).
2. A conversa mostra o player de áudio, a imagem ou o documento, com link assinado.
3. O atendente anexa um arquivo; `message-send` sobe na Meta e envia (RN-4-04).
4. Ao responder, a central marca como lidas as mensagens do lead (RN-4-02).
5. Um job diário apaga o que passou do prazo de guarda (RN-4-03).
6. A `quote-simulate` é ligada no console e testada contra o MatriculaWeb real.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Áudio do celular da equipe | Player na conversa, arquivo no bucket privado | — |
| Limite | Vídeo acima do limite | Mensagem registrada, aviso "arquivo grande demais" | — |
| Falha | URL da Meta vencida | Pede a URL de novo e baixa | 3 falhas: dead-letter |
| Falha | Atendente de outra unidade pede o link do arquivo | 403 | — |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC, a Emenda 01, `wa-webhook`, `message-send`, `quote-simulate` e a migração `caixa_entrada`.
2. **Alterar somente:** o que está em "Inclui".
3. **Não alterar:** regras de janela, custo, opt-out e tentativas da fase 3.
4. **Executar nesta ordem:** testes com o número de teste da Meta (texto e modelo, como estão); `media-fetch` e exibição; envio de arquivo; marcar como lida; MatriculaWeb real.
5. **Parar e pedir validação quando:** alguém pedir para usar o número real.
6. **Estado válido ao parar:** entrada e envio funcionando com o número de teste.
7. **Ritmo:** tasks em sequência; teste do cliente no fim, pelo roteiro abaixo.

## Checklist de execução

- [ ] Texto e modelo testados com o número de teste da Meta (envio, entrega, lida).
- [ ] Áudio, imagem e documento recebidos abrem na conversa; bucket privado; link assinado.
- [ ] Envio de imagem e PDF pela Cloud API.
- [ ] Marcação de lida.
- [ ] Unidades, turmas e simulação contra o MatriculaWeb real.

## Critérios de aceite

- [ ] **CA-4-01:** com o número de teste, texto e modelo enviados pela central chegam no celular; status `sent`, `delivered` e `read` voltam pelo webhook e aparecem na conversa.
- [ ] **CA-4-02:** áudio, imagem e documento recebidos são baixados para o bucket privado `midias` e abrem na conversa por link assinado; atendente de outra unidade e `marketing` recebem 403; arquivo acima do limite não é baixado e mostra o aviso.
- [ ] **CA-4-03:** o atendente envia imagem e PDF dentro da janela; fora dela o envio de arquivo é recusado; ao responder, as mensagens do lead ficam lidas no celular.
- [ ] **CA-4-04:** com a consulta ligada, unidades, turmas e simulação vêm do MatriculaWeb real e batem com a página pública de orçamento da Academia para a mesma seleção; com 429 a tela oferece o orçamento manual.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P4.1-01: pedido do link de um arquivo por atendente de outra unidade | teste de integração | 403 | saída do teste |
| GREEN | P4.1-02: áudio, imagem e PDF pelo número de teste | celular da equipe + consulta | três arquivos no bucket, abrindo na conversa | capturas + consulta |
| GREEN | P4.1-03: envio de PDF e imagem | tela | chegam no celular | captura do celular |
| RED | P4.1-04: envio de arquivo com a janela fechada | fixture | recusado | saída do teste |
| GREEN | P4.1-05: marcar como lida | resposta pela central | dois tiques azuis no celular | captura |
| GREEN | P4.1-06: MatriculaWeb real | mesma seleção na central e na página pública | mesmos valores | capturas lado a lado |

**Caminhos de erro obrigatórios:** outra unidade, arquivo grande, URL vencida, janela fechada, MatriculaWeb 429.

## Teste humano do cliente

- **Quando:** quando todas as tasks vinculadas estiverem no ar
- **Quem testa:** Luiz, com o Hikari
- **Passos:** do celular, mandar um áudio, uma foto e um PDF para o número de teste e abrir os três na central; responder com texto e com um PDF; ver os tiques azuis; montar um orçamento e comparar com a página pública de orçamento.
- **Resultado esperado:** tudo abre, tudo chega, valores iguais.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Testar texto e modelo com o número de teste da Meta | Academia das Específicas | SPEC-4-001 | CA-4-01 | P4.1-05 | capturas do celular + status | Número de teste cadastrado | ☐ |
| pendente | Receber e abrir áudio, imagem e documento na conversa | Academia das Específicas | SPEC-4-001 | CA-4-02 | P4.1-01, P4.1-02 | capturas + consulta | Task anterior | ☐ |
| pendente | Enviar imagem e PDF e marcar a conversa como lida | Academia das Específicas | SPEC-4-001 | CA-4-03 | P4.1-03, P4.1-04, P4.1-05 | capturas | Mídia no ar | ☐ |
| pendente | Testar o orçamento com a API real do MatriculaWeb | Academia das Específicas | SPEC-4-001 | CA-4-04 | P4.1-06 | capturas lado a lado | Consulta ligada no console | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| 2026-11-10 | Consultora | Ordem da fase | SPEC passa a ser a última da construção (antes só da SPEC-4-007), para não travar as outras; envio de documento vai para a SPEC-4-006 |
