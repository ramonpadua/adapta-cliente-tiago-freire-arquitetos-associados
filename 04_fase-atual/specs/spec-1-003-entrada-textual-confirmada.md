# SPEC-1-003 — Entrada conversacional textual com prévia e confirmação

**Fase:** 1
**Status:** preparação autorizada; implementação e aceite sujeitos às dependências e ao teste humano
**Dono:** coordenador de projetos
**Origem no escopo:** RQ-002, RQ-006, RQ-007; DEC-001, DEC-003, DEC-006; Fase 1 do `02-Escopo-Definitivo.md`
**Degrau da solução:** construção mínima — implementa somente a jornada textual complementar, com confirmação humana e sem escrita presumida no DoIt.

## Atualização vigente — 2ª consultoria de 29/10/2026

Este bloco prevalece sobre o texto anterior. Nenhuma tarefa foi concluída por ele.

- **Projeto-piloto:** 224 — Rooftop PRIO.
- **Equipe do piloto:** Bianca Martinelli (bianca@tiagofreire.arq.br), Akira Narimatsu (akira@tiagofreire.arq.br), Thales de Morais (thales@tiagofreire.arq.br) e Felipe Pinto (felipe@tiagofreire.arq.br). Confirmar o endereço da Akira antes de qualquer convite; não criar contas automaticamente.
- **Frequência de registro:** uma vez por dia (definido na consultoria de 29/10).
- **O que se espera do piloto:** que a equipe consiga registrar diariamente e que o gestor passe a ver andamento por projeto, etapa e responsável, com horas por pessoa e por projeto como evolução.
- **Aplicação:** usar este projeto Skip existente. Não criar outro projeto nem descartar a tela atual — chat e painel começam do que já foi criado.
- **Como trabalhar:** uma tarefa por vez. O agente pergunta ao Tiago o que faltar, registra a resposta e só segue para a próxima tarefa depois de confirmação ("testei, pode seguir").
- **DoIt:** a consulta está documentada, mas o acesso ainda não foi provado. Nesta fase não há escrita no DoIt e nenhum dado financeiro é usado. **18/11 — suporte oficial confirmou por escrito: API somente leitura, sem previsão de escrita, sem limite de requisições (Adendo E-F1-002 do Escopo Definitivo). Escrita no DoIt descartada no ciclo; registro operacional mora na camada complementar.**
- **Perguntas em aberto, a fazer durante as tarefas:** etapas reais do Rooftop PRIO; responsável pelas pendências; dias e horário-limite do registro; critério verificável de sucesso. Não criar número nem prazo por conta própria.


## Resultado observável

Em uma conversa textual de teste, o usuário autorizado envia uma atualização, recebe prévia com todos os campos, corrige ou esclarece o que faltar, confirma uma única vez e obtém um registro aceito na camada complementar. A visão posterior consegue localizar o registro e mostrar que a escrita no DoIt continua `pendente` ou fora do recorte.

## Limites e dependências

- **Inclui:** entrada de texto; extração/proposta de projeto, etapa, responsável, status, momento e observação; pedidos de esclarecimento; prévia; confirmar, editar/esclarecer e cancelar; event_id/idempotência; autoria e origem; estados de erro e pendência.
- **Fora de escopo:** áudio/transcrição; timer; escrita automática no DoIt; leitura financeira; notificação a cliente; classificação autônoma sem revisão; alteração de preço, contrato ou proposta.
- **Dependências e responsáveis:** coordenador define campos/estados e valida ambiguidade; usuário confirma; administrador técnico disponibiliza canal; responsável pelo DoIt confirma somente a referência de leitura, sem autorizar escrita por inferência.
- **Rollback ou reversão:** cancelar rascunho antes da confirmação; para correção posterior, criar evento de correção preservando o valor anterior; não apagar auditoria nem alterar DoIt.

## Fluxo e regras

1. Usuário autorizado envia atualização textual no canal definido.
2. Sistema extrai e propõe projeto, etapa, responsável, status, momento e observação; pede esclarecimento quando houver ambiguidade.
3. Prévia com todos os campos é apresentada para revisão.
4. Usuário confirma uma única vez; sistema grava na camada complementar com event_id, autoria, timestamp e origem.
5. Estados: `rascunho`, `aceito`, `pendente`, `erro`, `rejeitado`.
6. Escrita no DoIt não ocorre; estado de integração fica `pendente` quando aplicável.

## Caminhos de erro obrigatórios

- Entrada incompleta → pedir esclarecimento; não inventar campo.
- Entrada ambígua → oferecer opções; não escolher sozinho.
- Cancelamento → descartar rascunho; nenhum registro residual.
- Papel não autorizado → negar e auditar.
- Timeout/indisponibilidade → manter pendência com dono e próximo passo.

## Dados e integrações

| Transição | Destino | Campos | Papel | Idempotência | Erro |
|---|---|---|---|---|---|
| Prévia confirmada → registro complementar | camada complementar | event_id, valores confirmados, autoria, timestamp, origem, `registro_status=aceito`, `estado_integracao=pendente` quando não houver escrita DoIt | somente papel autorizado | confirmação repetida com mesmo request_id não cria segundo registro | mostrar resultado idempotente e auditar repetição |
| Registro complementar → DoIt | DoIt permanece referência existente; escrita fora desta fase | nenhuma escrita autorizada por esta SPEC | sem credencial de escrita | não tentar retry de escrita | estado fica `pendente` com dono/próximo passo |

## Regras de negócio

1. Nenhuma gravação sem prévia confirmada.
2. Nenhum campo inventado; esclarecimento sempre que faltar informação.
3. Registro aceito é imutável; correções criam novo evento referenciando o anterior.
4. Auditoria registra actor, papel, momento, ação, resultado e event_id.
5. Falha, duplicidade ou DoIt não autorizado mantém o evento pendente e encaminhável.

## Caminhos de erro

| Cenário | Gatilho | Comportamento | Recuperação |
|---|---|---|---|
| Ambiguidade de etapa | texto sem etapa ou com duas etapas possíveis | pergunta de esclarecimento; nenhuma escrita | após resposta, regenerar prévia; após abandono, pendência |
| Confirmação duplicada | mesmo request_id reenviado | não cria segundo registro; auditar repetição | exibir resultado idempotente |
| Indisponibilidade da camada | falha ao gravar | registro fica pendente, não descartado | retry manual com dono e próximo passo |
| Papel negado | usuário sem papel autorizado | negar e auditar | sem recuperação automática |

## Instruções de execução

1. **Escopo:** implementar somente a jornada textual descrita; nada além do aceite.
2. **Não alterar:** DoIt, dados financeiros, áudio, timer, credenciais, prompts/skills fora do recorte ou permissões.
3. **Cobertura mínima de idioma:** português.
4. **Segurança:** autenticação obrigatória; sessão com expiração; auditoria de todas as ações.
5. **Parar e pedir validação quando:** faltar uma resposta operacional do Tiago; pergunte somente o que falta, uma de cada vez, registre e confirme antes de usar dado real. Dúvida técnica vira pendência para o consultor. Não é necessária nova liberação do consultor a cada tarefa.
6. **Estado válido ao parar:** rascunhos não confirmados não aparecem como aceitos; nenhum retry gera duplicata; DoIt permanece sem escrita.

## Critérios de aceite

- [ ] **CA-1-011:** conversa completa com prévia e esclarecimento demonstrados.
- [ ] **CA-1-012:** rascunho cancelado não gera registro.
- [ ] **CA-1-013:** confirmação única gera exatamente um registro com event_id.
- [ ] **CA-1-014:** reenvio do mesmo request_id não cria segundo registro.
- [ ] **CA-1-015:** estados de erro e pendência demonstrados com dono e próximo passo.
- [ ] **CA-1-016:** a demonstração comprova que o DoIt não recebeu escrita nesta fase.

## TDD da SPEC

| Cenário | Fixture | Passos | Resultado esperado | Evidência |
|---|---|---|---|---|
| GREEN | registro completo | enviar fixture completo, responder esclarecimentos, confirmar uma vez | prévia aprovada, um event_id, registro aceito e estado DoIt pendente | conversa, registro, auditoria |
| AMBIGUIDADE | entrada com duas etapas possíveis | enviar entrada ambígua, responder esclarecimento, confirmar | prévia regenerada e registro aceito | conversa, prévia, registro |
| INCOMPLETO | entrada sem campo obrigatório | enviar entrada incompleta, não responder esclarecimento, cancelar | rascunho descartado, nenhum registro | conversa, ausência de registro |
| IDEMPOTÊNCIA | confirmação repetida | confirmar e reenviar o mesmo request_id | um único registro; repetição auditada | registro, auditoria |
| NÃO-ESCRITA | consulta posterior ao DoIt | confirmar registro e consultar o DoIt | DoIt sem alteração; estado pendente | captura do DoIt, registro |

**Evidência exigida:** conversa completa, prévia, decisão, event_id, registro, auditoria, estados de erro e confirmação de ausência de escrita no DoIt.

## Handoff e operação

- **Como demonstrar:** enviar o fixture, resolver uma ambiguidade, confirmar, repetir a confirmação e mostrar o estado pendente do DoIt.
- **Responsável pela demonstração:** coordenador de projetos.
- **Pendência conhecida:** parâmetros não respondidos são fechados durante a tarefa correspondente; não há aceite por silêncio. Integração de leitura ainda sem prova; escrita no DoIt fora desta fase.

## Tasks desta SPEC

| ID | Task | Dono | Critério | Recorte da prova | Evidência | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T007 | Fechar canal textual e contrato da prévia | Coordenador de projetos | SPEC-1-003 | CA-1-011, CA-1-012 | enviar entrada completa, ambígua e incompleta | conversa, prévia e rascunhos | canal definido; validar a prévia nesta tarefa | aguarda dependências e teste humano |
| F1-T008 | Demonstrar confirmação única e idempotência | Coordenador de projetos | SPEC-1-003 | CA-1-013, CA-1-014 | confirmar fixture e reenviar request_id | event_id, registro único e retry | F1-T007 concluída; sessão válida | aguarda dependências e teste humano |
| F1-T009 | Exercitar erros, cancelamento e ausência de escrita no DoIt | Coordenador de projetos | SPEC-1-003 | CA-1-015, CA-1-016 | testar timeout, cancelamento, papel negado e consulta DoIt | estados de erro e prova de não escrita | F1-T007 concluída; acesso somente leitura | aguarda dependências e teste humano |

## Emendas

<!-- Append-only (D19): mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| 18/11/2026 | Resposta por escrito do suporte oficial do DoIt (Carolina — DOit) ao time do cliente | Adendo E-F1-002 no `02-Escopo-Definitivo.md`; nenhuma task nova — T007–T009 seguem como estão | API confirmada **somente leitura, sem previsão de escrita, sem limite de requisições**: escrita no DoIt descartada no ciclo; a F2 de atualizar tarefas no DoIt morre; registro operacional mora na camada complementar. P1-003 documental fechado; prova de leitura continua na F1-T009. |
