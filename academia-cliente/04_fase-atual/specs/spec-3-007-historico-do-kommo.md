# SPEC-3-007 — Importação do histórico do Kommo, pronta e travada para dado real

**Fase:** 3  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** fase 3 (migração do Kommo com snapshot imutável `migrated=true`); DEC-02, DEC-09; §3 (Kommo somente leitura); §12 (API v4 do Kommo, uso único); GATE-04 e GATE-09; CL-001, CL-012, CL-013  
**Degrau da solução:** construção mínima — importação única por arquivo para tabelas da central, com trava de imutabilidade no banco; nenhuma dependência do Kommo em tempo de execução.

## Contexto e decisões fechadas

- **Estado atual:** o histórico do funil está só no Kommo, que segue operando em paralelo até a fase 5. A central tem as 14 etapas e os 7 motivos decididos em 08/10, e a deduplicação da fase 2 já liga um lead novo a um antigo quando ele volta.
- **Estado desejado:** a central sabe ler uma exportação do Kommo, mapear etapas, motivos e campos, guardar o histórico sem permitir alteração e entregar um relatório que bate com o arquivo. Com dado de teste agora; com dado real só quando os gates abrirem.
- **Decisões já fechadas:**
  - Entrada: um arquivo JSON no formato da API v4 do Kommo (leads com `id`, `name`, `price`, `status_id`, `pipeline_id`, `loss_reason_id`, `created_at`, `updated_at`, `closed_at`, `custom_fields_values`, contatos e tags; contatos com telefone e e-mail; funis com etapas; motivos de perda). Nesta fase o arquivo é de teste, com essa estrutura e conteúdo inventado marcado `TESTE`.
  - Mapas editáveis pelo admin: etapa do Kommo → etapa da central; motivo de perda → motivo do catálogo; campo personalizado → campo da central; funil ou campo → unidade. O que não tiver mapa vai para o relatório de lacunas e não entra.
  - Registros importados vão para `leads` e `stage_transitions` com `migrated = true` e `kommo_id`. Um gatilho no banco recusa UPDATE e DELETE de registro com `migrated = true`.
  - Lead em andamento no Kommo também entra só como histórico. Se ele voltar a falar com a Academia, a deduplicação da fase 2 cria um lead novo ligado ao antigo.
  - Reimportar não duplica: `kommo_id` é único. Registro que mudou no Kommo depois aparece no relatório como "divergente, não alterado".
  - Painéis só mostram o histórico com o filtro "histórico do Kommo" (SPEC-3-006).
  - Importação real exige as três coisas: política de LGPD aprovada, exportação real autorizada pela Academia e a chave da integração do Kommo gerada de novo e cadastrada só no segredo do servidor. A chave nunca vai para o grupo, para o repositório ou para log.
- **Bloqueios:** nenhum para construir. Dado real depende de CL-012 e CL-013.

## Resultado observável

O admin carrega o arquivo de teste e vê o relatório: quantos leads por etapa, unidade e ano, o que ficou de fora e por quê, e as possíveis duplicatas com leads da central. Os leads importados aparecem como "histórico do Kommo" e não podem ser editados.

## Limites e dependências

- **Inclui:** leitura do arquivo; mapas; importação com `migrated = true`; trava de imutabilidade; relatório de conferência; filtro de histórico.
- **Fora de escopo:** qualquer escrita no Kommo; sincronização contínua; importação real (operação liberada depois dos gates); desligamento do Kommo (fase 5); cálculo do baseline do K3 (fase 5).
- **Entradas e pré-condições:** SPECs 3-001 a 3-006 no ar; arquivo de teste com a estrutura da API v4.
- **Saídas/artefatos:** Edge Function `kommo-import` (arquivo); tabelas `kommo_stage_map`, `kommo_reason_map`, `kommo_field_map`, `kommo_import_runs`; gatilho de imutabilidade; relatório por execução.
- **Dependências e responsáveis:** admin do Kommo na Academia confere os mapas; Luiz testa.
- **Atores e permissões mínimas:** só `platform_admin` carrega arquivo e edita mapas; `unit_manager` e `viewer` leem o histórico da própria visão; ninguém edita registro migrado.
- **Superfícies/arquivos/configurações afetadas:** console (nova aba Importação), `leads`, `stage_transitions`, painéis.
- **Risco e plano B:** campo do Kommo sem correspondência: importação parcial com lacunas listadas; complemento manual só em campo novo, nunca no registro migrado.
- **Rollback ou reversão:** cada execução tem `import_run_id`; desfazer uma execução de teste apaga só os registros dela, por rotina do `service_role` com registro em `audit_log`. Execução real não se desfaz sem decisão do consultor.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Arquivo → `kommo-import` | Kommo (somente leitura, via exportação) | JSON da API v4, como nas decisões | `platform_admin`; arquivo de teste nesta fase | `kommo_id` único; execução registrada | arquivo fora do formato: recusado com a linha do erro |
| `kommo-import` → `leads`, `stage_transitions` | Central | campos mapeados + `migrated = true`, `kommo_id`, `import_run_id`, `is_test` | `service_role` | upsert só para inserir; existente fica igual | linha sem mapa: lacuna |
| `kommo_import_runs` | Central | arquivo (nome e hash), início, fim, totais, lacunas, duplicatas, divergentes, executado por | `platform_admin` | uma linha por execução | — |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | Etapa, motivo ou unidade sem mapa | Lacuna no relatório; registro não entra | — | DEC-09 |
| RN-02 | Telefone fora do padrão | Entra como veio, marcado "contato a revisar"; nunca corrigido | — | Emenda de 12/10 |
| RN-03 | UPDATE ou DELETE em `migrated = true` | Recusado pelo banco | Rotina de desfazer execução de teste | DEC-09 |
| RN-04 | Mesmo telefone ou e-mail de lead já existente na mesma unidade | Listado como possível duplicata; nada é juntado | — | Emenda de 12/10 |
| RN-05 | Arquivo real ou chamada à API do Kommo | Parar e pedir autorização do Luiz e do consultor | — | GATE-04 e GATE-09 |

## Fluxo e regras

1. O admin cadastra ou confere os mapas (14 etapas e 7 motivos já vêm sugeridos pelo nome).
2. Carrega o arquivo; `kommo-import` valida o formato e grava a execução.
3. Importa o que tem mapa (RN-01, RN-02), marca `migrated = true` e lista duplicatas (RN-04).
4. O relatório mostra totais por etapa, unidade e ano, lacunas, duplicatas e divergentes.
5. Reimportar o mesmo arquivo não cria nada novo.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Arquivo de teste com 200 leads nas 14 etapas | Importação, relatório e totais batendo | — |
| Limite | Arquivo com etapa sem mapa | Leads dela no relatório de lacunas | Mapear e reimportar |
| Falha | Tentativa de editar lead migrado | Recusada pelo banco | — |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC; a SPEC-2-001 em `05_entregas/fase-2/specs/` (deduplicação); a documentação pública da API v4 do Kommo para o formato dos objetos.
2. **Alterar somente:** importação, mapas, execuções, gatilho de imutabilidade, filtro de histórico e a aba do console.
3. **Não alterar:** deduplicação da fase 2; nada no Kommo.
4. **Executar nesta ordem:** leitura e mapas; importação imutável; relatório.
5. **Parar e pedir validação quando:** for usar arquivo real ou chamar a API do Kommo. Nesta fase não se cadastra a chave do Kommo.
6. **Estado válido ao parar:** só dado de teste importado; Kommo sem nenhuma chamada.
7. **Ritmo:** construir as tasks vinculadas em sequência, sem pedir autorização nem teste a cada task. O teste do cliente é o roteiro único de `## Teste humano do cliente`, quando todas as tasks vinculadas estiverem no ar. Ponto de aprovação antes disso: nenhum na construção; a primeira importação real é uma operação separada, liberada pelo consultor.

## Checklist de execução

- [ ] Arquivo de teste no formato da API v4.
- [ ] Mapas de etapa, motivo, campo e unidade.
- [ ] Importação com `migrated = true`, sem duplicar.
- [ ] UPDATE e DELETE recusados no banco.
- [ ] Relatório batendo com o arquivo.

## Critérios de aceite

- [ ] **CA-3-25:** a central lê o arquivo no formato da API v4, recusa arquivo fora do formato citando a linha, aplica os mapas de etapa, motivo, campo e unidade e manda para o relatório de lacunas o que não tem mapa.
- [ ] **CA-3-26:** o importado fica com `migrated = true` e `kommo_id`; reimportar o mesmo arquivo não cria registro; UPDATE e DELETE em registro migrado são recusados pelo banco; o histórico só aparece no filtro próprio.
- [ ] **CA-3-27:** o relatório traz totais por etapa, unidade e ano, lacunas, possíveis duplicatas e divergentes, e bate com o arquivo; nenhuma chamada ao Kommo nem chave cadastrada nesta fase.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P3.7-01: arquivo com campo obrigatório faltando | `kommo-import` | recusado com a linha | saída |
| GREEN | P3.7-02: arquivo de teste com 200 leads e uma etapa sem mapa | `kommo-import` | importados com mapa; lacunas listadas | saída + relatório |
| RED | P3.7-03: UPDATE e DELETE em lead migrado | SQL com `service_role` e com usuário | recusado nos dois | saída pgTAP |
| GREEN | P3.7-04: reimportar o mesmo arquivo | `kommo-import` | zero novos; execução registrada | saída |
| GREEN | P3.7-05: relatório × contagem do arquivo | script de conferência | totais iguais | saída |
| REFACTOR/REGRESSÃO | P3.7-06: provas P3.1 a P3.6; busca por chave ou token no repositório e nos logs | suíte + varredura de segredos | verdes; nada encontrado | saída |

**Dados/fixtures:** arquivo `TESTE` com 200 leads, dois funis (um por unidade), as 14 etapas, os 7 motivos, campos personalizados, telefones fora do padrão e duplicatas com leads `TESTE` da central.  
**Caminhos de erro obrigatórios:** formato inválido, etapa sem mapa, unidade sem mapa, reimportação, tentativa de edição.  
**Evidência exigida:** saídas das provas e o relatório da execução de teste na linha `> ` da task.

## Teste humano do cliente

- **Origem:** CL-001
- **Quando:** quando todas as tasks de `## Tasks vinculadas` estiverem no ar
- **Quem testa:** Luiz, com quem administra o Kommo
- **Passos:** conferir os mapas de etapas e motivos com os nomes do Kommo; carregar o arquivo de teste; ler o relatório e conferir os totais; tentar editar um lead importado; carregar o mesmo arquivo de novo.
- **Resultado esperado:** nomes reconhecidos, relatório batendo, edição recusada e nenhuma duplicação.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Handoff e operação

- **Como demonstrar:** relatório da execução de teste e um lead migrado travado.
- **Como operar depois:** a importação real roda uma vez, depois dos gates, pelo admin, com o consultor acompanhando.
- **Como monitorar:** execuções e lacunas na aba Importação.
- **Pendência conhecida:** importação real e uso do histórico para o baseline, quando os gates abrirem.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Ler o arquivo de exportação do Kommo e mapear etapas, motivos e campos | Academia das Específicas | SPEC-3-007 | CA-3-25 | P3.7-01, P3.7-02 | saída + relatório | SPEC-3-006 no ar | ☐ |
| pendente | Guardar o histórico importado sem permitir alteração | Academia das Específicas | SPEC-3-007 | CA-3-26 | P3.7-03, P3.7-04 | saída pgTAP | Leitura no ar | ☐ |
| pendente | Gerar o relatório de conferência da importação | Academia das Específicas | SPEC-3-007 | CA-3-27 | P3.7-05, P3.7-06 | relatório + saída | — | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
