# Fase 3 — o que a Academia precisa testar, enviar ou decidir (28/10/2026)

Itens da fase 3, liberada pelo consultor em 28/10/2026. Nenhum item trava a construção: o executor segue com massa de teste e usa cada dado quando chegar. Os ids `CL-` desta lista são os únicos válidos; item novo vai para o consultor.

Só dois itens travam um teste, e são o próprio teste: CL-001 (um teste por SPEC, no fim dela) e CL-002 (aceite da fase 1, junto do teste da SPEC-3-001).

## Para homologar a fase 3

| ID | Item | Para que serve | Trava o quê |
|---|---|---|---|
| CL-001 | Testar cada entrega quando o executor avisar que todas as tasks da SPEC estão no ar, seguindo o roteiro de `## Teste humano do cliente` da SPEC. O Luiz testa, com o Hikari conduzindo; um atendente de cada unidade ajuda quando o roteiro pedir | Aceite de cada SPEC (3-001 a 3-007) | **O teste de cada SPEC.** A construção da SPEC seguinte não espera |
| CL-002 | No teste da SPEC-3-001, a direção percorre também a base da fase 1 (funil de 14 etapas, motivos, console e separação por unidade) e registra o aceite no grupo | Fechar o aceite da fase 1, que não tinha sido registrado | **O teste da SPEC-3-001** |

## Decisões e envios que não travam

| ID | Item | Para que serve | Trava o quê |
|---|---|---|---|
| CL-003 | Definir o número do SLA da primeira resposta e a régua de alerta (hoje vale a referência de 10 minutos do Manual) | SLA e K1 (SPEC-3-001) | Nada; o console usa 10 minutos |
| CL-004 | Definir a contagem dos prazos de follow-up (horas corridas, úteis ou horário comercial), confirmar 20h e 24h e o prazo e o destino depois do terceiro retorno | Follow-up (SPEC-3-003) | Nada; vale horas corridas e 24h para o lembrete |
| CL-005 | Contar como a equipe marca hoje a próxima ação no Kommo (que tipos de ação existem e como dão prazo) | Catálogo de próximas ações (SPEC-3-002) | Nada; vale o catálogo inicial |
| CL-006 | Confirmar os campos obrigatórios da ficha de qualificação: objetivo, etapa do PAS, turno e origem; dizer se ano escolar ou maior dificuldade também devem ser obrigatórios | Ficha (SPEC-3-002) | Nada; vale a lista da SPEC |
| CL-007 | Definir as alçadas de desconto: faixas e quem aprova cada uma | Negociação (SPEC-3-004) | Nada; o teste usa faixas `TESTE` |
| CL-008 | Indicar quem faz reunião com o lead (papel diretor) e quem negocia condição (papel negociador), e criar os usuários de teste desses papéis e de marketing e diretoria | Reunião, negociação e painel (SPECs 3-004 e 3-006) | Nada; o teste usa usuários `TESTE` |
| CL-009 | Confirmar os itens do checklist de matrícula e o marco oficial de conversão (checklist concluído, pagamento ou início das aulas) | Matrícula e K3 (SPEC-3-005) | Nada; vale "checklist concluído" |
| CL-010 | Confirmar a data de retomada do Interesse 2027 (hoje: primeiro dia útil de novembro do ano anterior à turma) e cadastrar os feriados no console | Retomada (SPEC-3-005) | Nada |
| CL-011 | Confirmar que a central pode usar a API pública de orçamento do MatriculaWeb (unidades, turmas e simulação), e o limite de requisições. Dizer se querem, no futuro, que o orçamento também seja gravado no MatriculaWeb | Orçamento (SPEC-3-002) | Nada; nesta fase a central só consulta e simula |
| CL-012 | Para a importação do Kommo: listar os campos personalizados e os funis (sem dados de aluno) e a data de corte. Quando a importação real for liberada, gerar uma chave nova da integração do Kommo e cadastrá-la direto no segredo do servidor, sem passar pelo grupo | Importação (SPEC-3-007) | Só a importação real |
| CL-013 | Aprovar a política de LGPD provisória: retenção, consentimento, dados de menor e as duas unidades como empresas distintas | Qualquer dado real na central | Só lead real, envio real e importação real |
| CL-014 | Conta Meta Business, número que vai operar (a API oficial já foi escolhida em 19/10), modelos aprovados e orçamento de disparos, considerando as duas empresas | Canal com lead real | Só a operação real |
| CL-015 | Padronizar os formulários de captação (máscara de telefone e campos mínimos) | Qualidade da entrada | Nada |
| CL-016 | Dizer se a operação quer um responsável fixo por lead ou segue com a fila única da unidade | Distribuição (dívida registrada) | Nada; segue a fila única |

Antes de pedir qualquer entrada, conferir o que já foi enviado no grupo e no repositório. Arquivo com dado de aluno ou responsável não vai para o repositório; registrar só a referência e o estado.
