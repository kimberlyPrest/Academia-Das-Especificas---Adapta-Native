# SPEC-4-003 — Área "Assistente" no admin

**Fase:** 4  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** Emenda 01 (DEC-13, DEC-15); console de configuração das fases 1 e 3 (DEC-07, DEC-11)  
**Degrau da solução:** construção nova dentro do console que já existe — só configuração e base de conhecimento; quem usa é a SPEC-4-004.

## Contexto e decisões fechadas

- **Estado atual:** o console já edita etapas, SLA, motivos, modelos, feriados, campos da ficha e preços por categoria. Não existe nada de IA.
- **Estado desejado:** o Luiz (ou quem ele indicar) ajusta tudo da atendente virtual sem pedir código: nome, horários, jeito de atender, o que ela sabe, exemplos e textos do orçamento. Testa numa conversa simulada e publica.
- **Decisões já fechadas:**
  - **Só `platform_admin`** edita a área Assistente. O `unit_manager` vê (leitura) a configuração da própria unidade.
  - **Rascunho e versão publicada.** Toda edição vai para o rascunho; "Publicar" cria uma versão numerada; a IA usa sempre a última publicada; "Voltar para esta versão" republica uma anterior. Nada se apaga.
  - **Instruções em blocos guiados, não um texto gigante.** A central monta as instruções da IA juntando os blocos, nesta ordem, com as regras fixas da Emenda 01 sempre por cima (o admin não consegue desligar "não fale preço" nem "não negue que é atendimento virtual se perguntarem").
  - **Carga inicial** em `04_fase-atual/anexos/carga-inicial-assistente.md`, tirada do Manual de Boas Práticas e do Mapeamento MKT. Exemplos de conversas reais entram pelo admin, sem nome, telefone ou e-mail (o repositório não guarda dado de aluno).
  - **A chave da Gemini fica no segredo do servidor.** O admin só vê "chave configurada: sim/não" e o nome do modelo (padrão `gemini-2.5-flash`).
- **Bloqueios:** nenhum. O nome da atendente virtual de cada unidade é escolha da Academia (checklist); até lá vale o rascunho com o nome em branco, e a IA não liga sem nome.

## Resultado observável

Em Configuração → Assistente, o Luiz troca o nome da atendente da Asa Sul, acrescenta uma pergunta no FAQ com a grade horária anexada, testa em "Simular" perguntando "vocês têm PAS?", vê a resposta no jeito do Manual e publica a versão 2.

## Limites e dependências

- **Inclui:** as telas abaixo; tabelas `ai_config_versions`, `kb_items`, `kb_attachments`, `ai_examples`; montagem das instruções; "Simular" chamando a mesma função da SPEC-4-004 em modo simulação (sem WhatsApp, sem gravar lead); carga inicial.
- **Fora de escopo:** o comportamento da IA na conversa (SPEC-4-004); textos do orçamento sendo usados (SPEC-4-006 usa).
- **Atores e permissões mínimas:** `platform_admin` edita e publica; `unit_manager` lê a da própria unidade; os demais não veem.
- **Superfícies afetadas:** `src/pages/Configuracao.tsx` (nova aba), componentes em `src/components/configuracao/`, migração nova, bucket `midias` (anexos do FAQ em `midias/faq/`).
- **Risco e plano B:** publicação errada: "Voltar para esta versão" em um clique.

## As telas

| Tela | O que se edita | Observação |
|---|---|---|
| **Identidade** | Nome da atendente por unidade; assinatura (padrão `*{nome} - Unidade {unidade}:*`); emoji (sim, pouco, não); mensagem de abertura | Prévia da primeira mensagem ao lado |
| **Horários** | Expediente por unidade e dia da semana; feriados (os que já existem no console); modos ligados: "fora do horário" e "abertura no horário" | Mostra "agora a IA está: atendendo / só abertura / desligada" |
| **Como atender** | Blocos: como abrir; o que perguntar (puxa os campos da ficha, em ordem arrastável); tom e estilo; o que nunca fazer (lista + texto livre); quando passar para humano (caixas com os motivos da SPEC-4-002 + texto livre); limite de mensagens sem avanço (padrão 6) | As regras fixas aparecem travadas no topo, com o cadeado |
| **FAQ e turmas** | Pergunta, resposta, palavras-chave, unidade (as duas ou uma), anexos (imagem ou PDF), ativo | Busca e filtro por unidade |
| **Exemplos** | Conversas boas e ruins, coladas como texto, com o porquê | Aviso para tirar nome, telefone e e-mail antes de salvar; a tela recusa texto com telefone ou e-mail |
| **Orçamento** | Mensagem que acompanha o PDF; condição de pagamento; validade; chamada do próximo passo | Usados pela SPEC-4-006 |
| **Simular** | Conversa de teste com o rascunho ou com a versão publicada, escolhendo unidade e horário simulado | Mostra a resposta, as ferramentas chamadas e o que o Guardião barraria |
| **Versões** | Lista com quem publicou e quando; comparar; voltar | — |
| **Modelo** | Nome do modelo; chave configurada sim/não | Só leitura da chave |

## Dados e integrações

| Tabela | Campos | Permissão | Observação |
|---|---|---|---|
| `ai_config_versions` | `id`, `version` (null no rascunho), `status` (`rascunho`, `publicada`, `anterior`), `config` (jsonb: identidade, horários, blocos, orçamento, modelo), `published_by`, `published_at` | escrita só `platform_admin` por RPC | Um rascunho; publicadas imutáveis |
| `kb_items` | `id`, `unit_id` (null = as duas), `question`, `answer`, `keywords`, `active`, `version_from` | idem | Entram na versão ao publicar |
| `kb_attachments` | `kb_item_id`, `storage_path`, `mime`, `filename` | idem | Bucket privado |
| `ai_examples` | `id`, `kind` (`bom`, `ruim`), `text`, `why`, `active` | idem | Recusa texto com padrão de telefone ou e-mail |

## Critérios de aceite

- [ ] **CA-4-10:** só `platform_admin` edita; `unit_manager` lê a da própria unidade; os outros papéis recebem 403 nas RPCs e não veem a aba.
- [ ] **CA-4-11:** editar não muda o que a IA usa até publicar; publicar cria a versão N+1; voltar republica a versão escolhida; versões publicadas não mudam.
- [ ] **CA-4-12:** as instruções montadas trazem as regras fixas da Emenda 01 antes dos blocos do admin, e o admin não consegue removê-las; exemplo com telefone ou e-mail é recusado.
- [ ] **CA-4-13:** "Simular" responde com o rascunho ou a versão publicada, mostra as ferramentas chamadas e não grava lead, mensagem nem envio; a carga inicial do anexo aparece na primeira abertura.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P4.3-01: `attendant` e `unit_manager` chamam a RPC de publicar | pgTAP | 403 | saída do teste |
| GREEN | P4.3-02: editar rascunho, publicar, editar de novo, voltar à versão 1 | pgTAP + consulta | versão publicada correta em cada passo | saída do teste |
| RED | P4.3-03: salvar exemplo com um telefone | pgTAP | recusado | saída do teste |
| GREEN | P4.3-04: montagem das instruções | teste da função | regras fixas primeiro, blocos depois | saída do teste |
| GREEN | P4.3-05: "Simular" com "vocês têm PAS?" | tela + consulta | resposta no estilo do Manual; nenhuma linha nova em `leads`/`messages` | captura + consulta |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC, a Emenda 01, o anexo de carga inicial e a tela de Configuração atual.
2. **Alterar somente:** a aba Assistente, as tabelas listadas e as RPCs delas.
3. **Executar nesta ordem:** tabelas e versões; Identidade, Horários e Como atender; FAQ e Exemplos; Orçamento; Simular (depois que a função da SPEC-4-004 existir em modo simulação, pode ficar para o fim da SPEC-4-004).
4. **Ritmo:** tasks em sequência; teste do cliente no fim.

## Checklist de execução

- [ ] Rascunho, publicar e voltar.
- [ ] Telas Identidade, Horários, Como atender, FAQ, Exemplos, Orçamento, Versões e Modelo.
- [ ] Regras fixas travadas.
- [ ] Simular sem gravar nada.
- [ ] Carga inicial.

## Teste humano do cliente

- **Quando:** quando todas as tasks vinculadas estiverem no ar (o Simular pode fechar junto com a SPEC-4-004)
- **Quem testa:** Luiz
- **Passos:** dar nome às atendentes das duas unidades; ajustar o horário de uma unidade; acrescentar uma pergunta no FAQ com a grade anexada; tentar salvar um exemplo com telefone; simular "Quanto custa?" e "Vocês têm PAS?"; publicar; voltar para a versão anterior.
- **Resultado esperado:** tudo editável sem ajuda técnica; a IA nunca fala preço no simulador; volta de versão funcionando.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Criar a área Assistente com rascunho, versões e permissões | Academia das Específicas | SPEC-4-003 | CA-4-10, CA-4-11 | P4.3-01, P4.3-02 | pgTAP | — | ☐ |
| pendente | Editar identidade, horários e o jeito de atender em blocos | Academia das Específicas | SPEC-4-003 | CA-4-12 | P4.3-04 | teste da montagem + captura | Task anterior | ☐ |
| pendente | Montar o FAQ com anexos, os exemplos e os textos do orçamento | Academia das Específicas | SPEC-4-003 | CA-4-12, CA-4-13 | P4.3-03 | pgTAP + captura | Task anterior | ☐ |
| pendente | Simular uma conversa antes de publicar | Academia das Específicas | SPEC-4-003 | CA-4-13 | P4.3-05 | captura + consulta | Função da SPEC-4-004 em modo simulação | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
