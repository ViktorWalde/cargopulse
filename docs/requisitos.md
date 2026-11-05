# CargoPulse — Requisitos e critérios de aceite

## 1. Convenções e escopo

O catálogo define os comportamentos e as qualidades exigidos para cada entrega. Todos os requisitos estão especificados, sem implementação ou validação de software concluída. Prioridades, regras RB, cenários T e marcos M são detalhados na [especificação funcional e técnica](especificacao.md), seções 2, 7, 17 e 20.

P0 compreende M0–M4 e resulta no fluxo local completo de telemetria. P1 compreende M5–M6 e depende do aceite de P0. P2 reúne alternativas técnicas sujeitas a necessidade demonstrada; não constitui uma entrega obrigatória. Redis, Functions, SOAP e outbox permanecem fora de P0/P1, conforme os cortes de escopo da especificação.

Os IDs são estáveis. A coluna de entrega indica quando o aceite integral é exigido; funcionalidades incrementais explicitam o mínimo anterior. Alterações de escopo atualizam o requisito e suas referências de teste, sem alterar o significado de um ID já utilizado.

## 2. Requisitos funcionais

### 2.1 P0 — Telemetria local

| ID | Entrega | Comportamento e critério de aceite | Regras e verificação |
|---|---|---|---|
| RF-01 | M1 | Cadastrar dispositivos ativos por padrão, listar, detalhar e ativar/desativar de forma idempotente. Código externo normalizado é único; entradas inválidas e conflitos não alteram registros existentes. | RB-11, RB-12; T10, T15, T33 |
| RF-02 | M1 | Cadastrar, listar e detalhar veículos fictícios. Código de frota normalizado é único, inclusive sob cadastros concorrentes. | T15 |
| RF-03 | M1 | Cadastrar, listar e detalhar entregas com veículo existente, referência externa única e faixa válida; alterar status apenas pelas transições de RB-10. Repetir o estado atual não cria efeitos adicionais. | RB-05, RB-10; T15, T18, T24 |
| RF-04 | M1 → M2 | Criar, consultar e encerrar associações conforme RB-04/RB-13, inclusive sob concorrência. M1 entrega intervalos e encerramento; M2 integra a proteção contra exclusão de leituras já vinculadas. Conflitos retornam `409`, sem atualização parcial. | RB-04, RB-13; T09, T16, T32 |
| RF-05 | M2 | Receber uma medição autenticada, validar contrato e dispositivo e publicar na fila. Somente a confirmação do broker produz `202`; dispositivo inexistente produz `404`, inativo `409`, contrato inválido `400` e falha/timeout do broker `503`. Rejeições anteriores à publicação não enviam mensagem. | RB-06, RB-07, RB-12; T01, T10, T13, T14, T18–T20 |
| RF-06 | M3 | Persistir e consultar histórico por dispositivo e entrega, com filtros e paginação limitada; obter a última leitura pela ordem de RB-03. M2 entrega a persistência e a consulta do fluxo simples; M3 verifica atrasos, empates, lacunas e limites. | RB-02–RB-04; T01, T04, T09, T17, T23 |
| RF-07 | M3 | Consumir mensagens com idempotência, detecção de conflito e confirmação após commit. M2 já exige chave única e commit antes de ACK; M3 completa tratamento concorrente, conflitos, redelivery e cancelamento. | RB-01, RB-08, RB-09; T02, T03, T06, T21, T22, T30 |
| RF-08 | M3 | Gerar um alerta por tipo/leitura fora da faixa da entrega, consultar alertas e reconhecer um alerta existente. Limites inclusivos não geram alerta; reconhecimento repetido preserva a primeira confirmação. | RB-05; T05, T25 |
| RF-09 | M3 | Receber lote e devolver resultado por posição do item conforme seção 8.2 da especificação. Aceites, invalidações e falhas de publicação são distinguíveis; repetição mantém a idempotência. | RB-01, RB-06; T07, T20, T34 |
| RF-10 | M3 | Consultar falhas permanentes registradas, filtradas por evento/correlação. Diagnóstico contém categoria, motivo e tentativa. Com SQL indisponível, a DLQ mantém motivo e correlação para inspeção; não há confirmação silenciosa de processamento. Reprocessamento operacional de telemetria não integra P0. | RB-09, RB-11; T03, T06, T22, T27, T30 |
| RF-11 | M4 | Exibir resumo, entregas, dispositivos, histórico, alertas e falhas disponíveis. Estados vazio, carregando e erro são explícitos; timezone e leituras atrasadas/fora da faixa são identificados. Entrega sem sensor ou leitura permanece consultável. | T29 |
| RF-12 | M4 | Gerar eventos por HTTP com seed e relógio de referência explícitos. M2 entrega `normal`; M4 entrega os cenários da seção 14, com relatório de envios e resultados. Mesmo cenário, seed e instante-base reproduzem os mesmos eventos. | T01–T09, T28 |
| RF-13 | M0 | Disponibilizar `/health/live` e `/health/ready`. Liveness avalia o processo; readiness consulta as dependências necessárias com timeout. Dependência indisponível produz readiness `503`, sem alterar liveness. | T26 |

### 2.2 P1 — Integrações simuladas

| ID | Entrega | Comportamento e critério de aceite | Regras e verificação |
|---|---|---|---|
| RF-14 | M5 | Importar pedidos do ERP simulado com mapeamento auditável, uma entrega por origem/ID e versão aplicada registrada. Reenvio não duplica entrega; versão antiga não substitui a atual; mesma versão com conteúdo conflitante gera diagnóstico. Falha de mapeamento produz `NeedsMapping`. | RB-01, RB-08; T11, T36 |
| RF-15 | M6 | Receber eventos de transportadora com assinatura HMAC, controle de replay e idempotência. Evento atrasado permanece no histórico, sem retroceder status; transição inválida recebe diagnóstico. | RB-01, RB-07, RB-10; T12, T35 |
| RF-16 | M5–M6 | Consultar tentativas de integração e solicitar reprocessamento seguro. Falhas permanentes exigem correção antes de nova execução; solicitações concorrentes não duplicam o efeito de negócio. M5 cobre ERP; M6 incorpora transportadoras. | RB-08, RB-11; T11, T12, T36 |

## 3. Requisitos não funcionais

Os requisitos abaixo integram P0 e continuam aplicáveis a P1. Intervalos de marcos indicam aplicação incremental: o primeiro exige as propriedades relacionadas ao comportamento entregue; o último exige o aceite integral. Extensões mantêm a mesma propriedade.

| ID | Entrega | Qualidade e critério de aceite | Verificação |
|---|---|---|---|
| RNF-01 | M0 → M4 | Execução local reproduzível, sem serviço pago, credencial corporativa ou hardware. SDK/imagens têm versões fixadas; restore, build, testes e infraestrutura possuem comandos e pré-requisitos documentados. Windows e Linux são alvos; cada plataforma só recebe indicação de validada com execução registrada. | Ambiente limpo, CI e T31 |
| RNF-02 | M0 → M4 | Domínio independente de infraestrutura; aplicação depende do domínio e de portas necessárias. Regras são compartilhadas pelos casos de uso, sem cópias em API, Worker, views ou simulador. Novos padrões dependem de problema concreto; SOLID e open/closed orientam as fronteiras. | Referências entre projetos, revisão de mudanças e testes de regras |
| RNF-03 | M2 → M3 | Uma chave `(deviceId, eventId)` produz no máximo uma leitura; leitura e alerta são atômicos. Redelivery após commit preserva efeitos existentes. Concorrência é protegida no SQL Server; a garantia não depende apenas de consulta prévia ou estado em memória. | T02, T03, T05, T06, T21, T22 |
| RNF-04 | M0 → M4 | Ingestão exige chave válida; administração/UI permanecem em loopback no perfil local. Segredos vêm do ambiente, não aparecem em respostas/logs/versionamento; erros públicos não expõem stack trace. Exposição remota depende de controle de acesso e TLS definidos previamente. | T14, T27 e detecção de segredos no CI |
| RNF-05 | M1 → M3 | Entradas, lotes e consultas respeitam os limites da seção 4. Consulta executa filtro/paginação no SQL; tamanho excessivo é rejeitado antes da publicação. Paralelismo do Worker possui limite positivo configurado, sem criação irrestrita de tarefas. | T18, T23, T34 e inspeção de configuração |
| RNF-06 | M2 → M3 | Operações respeitam budgets e limite de entregas contabilizadas pelo broker, conforme seção 4. Falha permanente não recebe retry de negócio; falha transitória possui backoff limitado. Retries do SDK compartilham o orçamento da operação. Cancelamento/perda de lock não autorizam ACK antecipado. Circuit breaker depende da revisão de arquitetura, sem obrigatoriedade de biblioteca. | T06, T20–T22, T30 |
| RNF-07 | M2 → M4 | Logs correlacionam requisição, mensagem e resultado com os campos da seção 11.3. Contagens distinguem tentativas HTTP, entregas do broker e eventos de negócio; duplicatas não representam novas leituras. Falhas terminais possuem motivo consultável no SQL ou na DLQ. | T08, T27, T30 |
| RNF-08 | M1 → M3 | Timestamps são normalizados em UTC, sem perda de precisão aceita. Ordenação por evento e desempate são iguais na aplicação e no SQL. Hash considera valores de negócio normalizados, sem horário de recepção/correlação; ordem de propriedades, zeros decimais finais e offset equivalente não geram conflito. | RB-02, RB-03, RB-07; T03, T17–T19 |
| RNF-09 | M0 → M4 | Regra alterada possui teste válido e cenários adversariais relevantes. Constraints/transações são verificadas no SQL Server. Testes de tempo/concorrência usam relógio, seed, barreiras e prazos finitos; suites com Docker são identificadas. Asserções detectam violação de invariantes, sem espelhar a implementação. | Matriz T01–T36, conforme o marco |
| RNF-10 | M4 | Cenário fechado de 1000 eventos possui reconciliação completa de identidades e resultados, conforme seção 17.3. O relatório informa ambiente, prazo do teste, duração, latências observadas e falhas. Não existe meta de throughput, p95 ou SLA de produção antes de um baseline medido. | T08 |
| RNF-11 | M0 → M4 | Documentação pública descreve comportamento e estado de entrega; contratos OpenAPI correspondem aos códigos e limites implementados. Funcionalidade planejada e funcionalidade validada são identificáveis. Procedimentos de execução não dependem de arquivos operacionais locais. | Revisão documental, OpenAPI, T27 e T31 |

## 4. Limites do MVP

Os valores são parâmetros iniciais do perfil local, não métricas de capacidade ou propriedades de um sensor real. Alterações preservam os IDs dos requisitos e atualizam testes/configuração. Ampliação de limites depende de novo aceite, sem arredondamento silencioso.

| Parâmetro | Definição |
|---|---|
| Códigos externos | `ExternalCode`, `FleetCode` e `ExternalReference`: trim, letras ASCII convertidas para maiúsculas, 1–64 caracteres em `[A-Z0-9._-]`; unicidade após normalização. |
| Campos textuais | `Name`: 1–128 unidades UTF-16 após trim; `Description`: opcional, até 256. Strings obrigatórias vazias ou compostas apenas por espaços são inválidas. |
| Temperatura | Perfil local de -100 a 200 °C, inclusive; valor representável sem arredondamento em escala decimal 3. Faixa da entrega respeita esse perfil e `MinTempC < MaxTempC`. |
| Umidade e coordenadas | Umidade: 0–100%, escala 2. Latitude: -90–90; longitude: -180–180, escala 6. Coordenadas aparecem juntas ou são ambas ausentes/null; números precisam ser finitos. |
| IDs e datas | UUID válido e diferente de zero. Timestamp com offset explícito, normalizado para UTC, com até 7 casas fracionárias de segundo. Ausência de offset e precisão superior são inválidas. |
| Evento futuro/atrasado | Futuro permitido até `receivedAtUtc + 5 minutos`, inclusive, com relógio da API. Não há limite de atraso no MVP; associação usa o tempo do evento e o estado de associação disponível no processamento. |
| Corpo HTTP e lotes | JSON de até 1 MiB (1.048.576 bytes), inclusive. Lote possui 1–1000 itens. Corpo acima do limite produz `413`; envelope inválido ou contagem fora do intervalo produz `400`, sem publicação. |
| Paginação e filtros | `page >= 1`; `pageSize` padrão 50, permitido 1–100. Intervalo de consulta `[fromUtc, toUtc)`; quando ambos existem, `fromUtc < toUtc`. Entrada inválida produz `400`; página vazia é válida. A ordenação é estável para o mesmo conjunto de dados; paginação por página não promete snapshot entre alterações concorrentes. |
| Paralelismo | Worker processa uma entrega por vez no perfil inicial; o máximo é o valor positivo configurado. Lotes e volume de entrada não criam paralelismo adicional sem limite. Cenários concorrentes configuram explicitamente mais de uma execução. |
| Budgets locais | Requisição de ingestão: até 30 s; readiness: até 5 s no conjunto das verificações; publicação individual e comando SQL: até 5 s, limitados pelo prazo restante; processamento de uma entrega de mensagem: até 30 s. O tempo inclui retries internos e persistência de diagnóstico quando aplicável. Valores são configuráveis, positivos e documentados. |
| Retentativas | `MaxDeliveryCount=5`, conforme contador do broker; backoff transitório entre 0,5 e 5 s, com variação aleatória limitada ao intervalo. Não há retry adicional de negócio dentro de uma entrega; tentativas do SDK permanecem limitadas pelo budget. Ao esgotar a contagem aplicável, a mensagem segue para DLQ com motivo/correlação. |

`MaxDeliveryCount` não constitui contador global de execuções: certas perdas de lock/conexão não incrementam a contagem. A recuperação mantém idempotência e budget por operação, conforme a [semântica de lock e confirmação do Service Bus](https://learn.microsoft.com/en-us/azure/service-bus-messaging/message-transfers-locks-settlement).

Os budgets limitam operações em execução; não estabelecem prazo de entrega durante indisponibilidade do host/broker nem durabilidade após reinício do emulador. P1 mantém limites próprios de payload, replay e autenticação externa, definidos antes de M5/M6; o aceite de P0 não depende dessas definições.
