# SPEC-4-005 — Contatos da mesma compra (aluno, mãe, pai, responsável)

**Fase:** 4  
**Status:** planejada  
**Dono:** Academia (Hikari executa com o agente); aceite do Luiz  
**Origem no escopo:** Emenda 01 (DEC-19); deduplicação da SPEC-2-001 (RN-14: só caso exato, histórico preservado); dor relatada pelo Luiz em 11/09 (a mãe é atendida e o pai fecha; a conversão não aparece)  
**Degrau da solução:** construção sobre a deduplicação da fase 2 — um lead passa a ter vários números, com o papel de cada um.

## Contexto e decisões fechadas

- **Estado atual:** um lead tem um telefone. Se o pai escreve de outro número, nasce outro lead; a matrícula fecha num e o atendimento fica no outro. O Mapeamento MKT (7.1) mostra o mesmo problema: responsável que vem presencialmente, manda documentos pelo WhatsApp e o contato fica "sem histórico nenhum".
- **Estado desejado:** quando outra pessoa da mesma compra entra em contato, o número dela é ligado ao lead que já existe, com o papel (aluno, mãe, pai, responsável financeiro, outro). A conversa e a contagem ficam num lead só.
- **Decisões já fechadas:**
  - **Só liga quando é a mesma compra:** o número informado pertence a um lead **da mesma unidade**, o **nome do aluno confere** e a compra **está aberta** (etapa diferente de `Novo aluno`, `Desqualificado` e `Interesse 2027`).
  - **Não liga, e explica por quê, quando:**
    - o aluno é outro: é **indicação**, vira lead novo com origem "Indicação" e o campo "indicado por" apontando para o lead de quem indicou;
    - a compra anterior já fechou (`Novo aluno`) e a pessoa quer outro produto ou outro aluno: lead novo, com o campo "mesma família" apontando para o anterior (sem juntar);
    - o lead encontrado é da outra unidade: a atendente decide.
  - **Na dúvida, não liga sozinho:** quando falta um dos três sinais, a central mostra a sugestão no cartão e a atendente confirma com um clique.
  - **Ligar é juntar sem perder nada:** o lead que nasceu do número novo fica marcado como "juntado em" e todo o histórico e a origem dele continuam visíveis no lead principal (RN-14 da fase 2). Nada se apaga.
  - **De onde vêm os sinais:** a IA percebe ("sou a mãe do…", "meu filho falou com vocês", "quero fechar a matrícula dele") e pergunta o número ou o nome de quem falou antes; a atendente pode ligar pelo botão "Ligar a outro lead"; o checklist de matrícula (SPEC-3-005) já pede o telefone do aluno e do responsável financeiro, e esses números entram como contatos do lead com o papel.
  - **A IA nunca diz ao lead dados de outra pessoa.** A ferramenta devolve só "encontrei a mesma compra" ou "não encontrei".
- **Bloqueios:** nenhum.

## Resultado observável

A mãe (`TESTE`) conversa pela manhã e recebe o orçamento. À noite, o pai escreve de outro número: "minha esposa falou com vocês sobre a Ana". A IA pergunta o número dela, confere e liga. No cartão aparecem os dois números com os papéis, a conversa dos dois e um orçamento só.

## Limites e dependências

- **Inclui:** tabela `lead_contacts`; roteamento da mensagem que chega pelo número ligado; RPCs `compra_buscar` e `contato_vincular`; ferramentas da IA (`buscar_mesma_compra`, `ligar_contato`); botão "Ligar a outro lead" com a sugestão; campos "indicado por" e "mesma família"; contatos vindos do checklist de matrícula; ajuste do K3 e do painel gerencial para contar o lead principal uma vez.
- **Fora de escopo:** juntar leads de unidades diferentes sem a atendente.
- **Atores e permissões mínimas:** `attendant` liga contatos da própria unidade; a IA usa as RPCs com `service_role`, limitadas à unidade do lead.
- **Superfícies afetadas:** roteamento da entrada (fase 2), cartão do lead, checklist de matrícula, métricas gerenciais (SPEC-3-006).
- **Risco e plano B:** ligação errada: botão "Desfazer ligação" (gestor), que separa o contato e devolve as mensagens dele ao lead de origem; fica na auditoria.

## Dados e integrações

| Tabela/RPC | Campos/contrato | Permissão | Regras |
|---|---|---|---|
| `lead_contacts` | `lead_id`, `phone_e164`, `name`, `role` (`aluno`, `mae`, `pai`, `responsavel_financeiro`, `outro`), `source` (`entrada`, `ia`, `atendente`, `matricula`), `linked_at`, `linked_by` | RLS por unidade | Telefone único por lead ativo na unidade |
| `compra_buscar(lead_atual, telefone, nome_aluno)` | devolve `mesma_compra` (sim/não/sugestão) e o motivo; nunca dados pessoais para a IA | `attendant` ou `service_role` | Mesma unidade; nome do aluno por semelhança (sem acento, primeiro nome igual e semelhança ≥ 0,6); compra aberta |
| `contato_vincular(lead_principal, lead_origem, papel)` | junta mantendo histórico | `attendant` da unidade; `service_role` só quando `compra_buscar` = sim | Auditoria append-only |
| `contato_desvincular(...)` | desfaz | `unit_manager` | Auditoria |
| `leads.referred_by_lead_id`, `leads.family_lead_id` | "indicado por" e "mesma família" | RLS | Não juntam leads |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-4-18 | Número, aluno e compra aberta conferem, mesma unidade | Liga automaticamente, com o papel | — | DEC-19 |
| RN-4-19 | Falta um sinal | Sugestão no cartão; liga só com o clique | — | DEC-19 |
| RN-4-20 | Aluno diferente | Lead novo, origem "Indicação", "indicado por" | — | DEC-19 |
| RN-4-21 | Compra anterior fechada | Lead novo, "mesma família" | — | DEC-19 |
| RN-4-22 | Mensagem chega de um número ligado | Entra na conversa do lead principal | Número ligado a lead fechado: segue a RN-4-21 | DEC-19 |
| RN-4-23 | K3 e painel | Contam o lead principal uma vez | — | Escopo §1 |

## Critérios de aceite

- [ ] **CA-4-20:** um lead guarda vários números com papel; a mensagem que chega por qualquer um deles entra na conversa do lead principal; os contatos do checklist de matrícula aparecem com o papel.
- [ ] **CA-4-21:** a ligação automática só acontece com os três sinais na mesma unidade; nos quatro casos de "não liga" (aluno diferente, compra fechada, outra unidade, sinal faltando) nada é juntado sozinho e o motivo aparece.
- [ ] **CA-4-22:** aluno diferente gera lead novo com "indicado por"; compra fechada gera lead novo com "mesma família"; a ferramenta da IA nunca devolve nome ou telefone de outra pessoa; "Desfazer ligação" separa sem perder histórico; K3 e o painel contam o lead uma vez.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | P4.5-01: os quatro casos de "não liga" | pgTAP | nada juntado; motivo certo | saída do teste |
| GREEN | P4.5-02: pai `TESTE` informa o número da mãe `TESTE`, mesmo aluno, compra aberta | pgTAP + consulta | ligado com papel `pai` | consulta |
| GREEN | P4.5-03: mensagem nova do pai | webhook de teste | entra na conversa do lead principal | consulta |
| GREEN | P4.5-04: amigo indica outro aluno | pgTAP | lead novo com "indicado por" | consulta |
| RED | P4.5-05: resposta da ferramenta da IA | teste da função | sem nome nem telefone | saída do teste |
| REGRESSÃO | P4.5-06: deduplicação da fase 2, K3 e painel gerencial | pgTAP | verdes; contagem única | saída do teste |

## Instruções de execução para o Maestro

1. **Ler antes de alterar:** esta SPEC, a SPEC-2-001 (deduplicação), a SPEC-3-005 (checklist) e a SPEC-3-006 (métricas).
2. **Não alterar:** a regra de deduplicação exata da fase 2.
3. **Executar nesta ordem:** `lead_contacts` e roteamento; `compra_buscar` e `contato_vincular`; indicação e família; ferramentas da IA; botão e desfazer; métricas.
4. **Ritmo:** tasks em sequência; teste do cliente no fim.

## Checklist de execução

- [ ] Vários números por lead, com papel e roteamento.
- [ ] Regras de ligação e de "não liga".
- [ ] Indicação e mesma família.
- [ ] Ferramentas da IA sem dado pessoal.
- [ ] Desfazer ligação.
- [ ] Métricas com contagem única.

## Teste humano do cliente

- **Quando:** quando todas as tasks vinculadas estiverem no ar
- **Quem testa:** Luiz, com um atendente
- **Passos:** mãe `TESTE` conversa e recebe orçamento; pai `TESTE` escreve de outro número falando do mesmo aluno; um terceiro número diz que foi indicado por esse aluno e quer matricular outra pessoa; abrir o cartão e conferir; desfazer uma ligação.
- **Resultado esperado:** pai ligado ao mesmo lead; indicação como lead novo; nada perdido ao desfazer.
- **Evidência do aceite:** "testei, pode seguir" do Luiz no grupo em resposta a este roteiro.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| pendente | Guardar vários números por lead, com o papel de cada um | Academia das Específicas | SPEC-4-005 | CA-4-20 | P4.5-03 | consulta | — | ☐ |
| pendente | Ligar só quando for a mesma compra, e separar indicação e família | Academia das Específicas | SPEC-4-005 | CA-4-21, CA-4-22 | P4.5-01, P4.5-02, P4.5-04 | pgTAP | Task anterior | ☐ |
| pendente | Dar à IA e à atendente o jeito de ligar e desfazer, com a contagem única | Academia das Específicas | SPEC-4-005 | CA-4-22 | P4.5-05, P4.5-06 | teste + pgTAP | Task anterior; SPEC-4-004 | ☐ |

## Emendas

<!-- Append-only: mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
