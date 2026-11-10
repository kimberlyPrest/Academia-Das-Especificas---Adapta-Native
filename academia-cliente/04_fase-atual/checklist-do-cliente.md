# Fase 4 — o que a Academia precisa testar, enviar ou decidir (10/11/2026)

Itens da fase 4, liberada pela consultora em 10/11/2026. A construção não espera nenhum deles: o executor segue com massa `TESTE` e usa cada dado quando chegar. Os ids `CL-` desta lista são os únicos válidos na fase 4; item novo vai para a consultora.

## Para homologar e virar

| ID | Item | Para que serve | Trava o quê |
|---|---|---|---|
| CL-017 | Testar cada SPEC quando o executor avisar, pelo roteiro de `## Teste humano do cliente` dela. O Luiz testa, com o Hikari; atendentes e gestores quando o roteiro pedir | Aceite de cada SPEC (4-001 a 4-007) | **O teste de cada SPEC** |
| CL-018 | Escolher o **nome da atendente virtual** de cada unidade | Identidade da IA (SPEC-4-003) | Ligar a IA (sem nome ela não liga) |
| CL-019 | Informar o **horário de atendimento** de cada unidade (dias e horas) e o endereço e os horários de visita | Quando a IA atende sozinha; FAQ de visita | Ligar a IA |
| CL-020 | Completar o **FAQ** marcado **[completar]** em `anexos/carga-inicial-assistente.md`: datas e dias do PAS semestral e do Treinamento UnB, Matemática Básica, grades horárias (imagem), conferir Orientação de Estudos e Cabine VIP | O que a IA sabe responder | Nada; o que faltar, a IA transfere |
| CL-021 | Definir os textos do orçamento da Black Week: número de parcelas e validade da condição | PDF e mensagem do orçamento (SPEC-4-006) | Nada; o PDF sai sem a condição |
| CL-022 | Criar a **chave da API do Gemini** numa conta do Google Cloud da Academia **com faturamento ativo** (a gratuita usa os dados para treino) e cadastrá-la direto no segredo `GEMINI_API_KEY`, sem passar pelo grupo | IA (SPEC-4-004) | A bateria de avaliação e a IA de verdade |
| CL-023 | Deixar o **número de teste** do app da Meta funcionando e cadastrar os celulares da equipe como destinatários | Testes reais da SPEC-4-001 | Os testes com a Meta |
| CL-024 | **GATE-03:** conta Meta Business verificada; os números das duas unidades levados do Kommo para o app da Academia; modelos aprovados (boas-vindas para lead de formulário, FUP 3 e retomada), todos como Utilidade | Operação real | **Só a virada** |
| CL-025 | MatriculaWeb: confirmar a gravação do orçamento pela API pública; pedir ao responsável pelo sistema uma liberação de limite para o servidor da central; avisar a secretaria do orçamento `TESTE` | SPEC-4-006 | Nada; com o limite público, a atendente tenta de novo |
| CL-026 | **GATE-04:** exportação do Kommo com a chave nova da integração (cadastrada direto no segredo, nunca no grupo) e a data de corte | Histórico na central | **Só a virada** |
| CL-027 | Marcar a **data e a hora da virada** de cada unidade e quem desliga o SalesBot no Kommo | Roteiro da SPEC-4-007 | **Só a virada** |
| CL-028 | Criar os **usuários reais** (atendentes, gestores, diretores, negociadores, marketing) com o papel e a unidade de cada um | Operação real e números por atendente | **Só a virada** |

## Já valem com o padrão (vindos da fase 3)

Continuam abertos e não travam nada: número do SLA (10 minutos), contagem dos prazos de follow-up (horas corridas), alçadas de desconto (faixas `TESTE` até cadastrar as reais), marco de conversão (checklist concluído), data de retomada do Interesse 2027. Os detalhes estão em `05_entregas/fase-3/checklist-do-cliente.md` (CL-003, CL-004, CL-007, CL-009, CL-010).

## Segurança

Nenhuma chave, token ou senha no grupo do WhatsApp. As chaves do Kommo e a senha do Maestro que já passaram pelo grupo devem ser trocadas antes da virada.
