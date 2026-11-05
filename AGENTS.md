# CargoPulse — orientação do agente

Telemetria e integração logística em C#/.NET. A especificação descreve o alvo; código, testes e comandos executados demonstram o que existe.

## Entrada e leitura sob demanda

1. Leia este arquivo e, se existir, `.agents/progress.md`. Esse registro é local e não versionado; quando ausente, consulte o histórico Git e os arquivos reais. A solicitação atual define a tarefa; o próximo passo registrado serve para continuidade.
2. Consulte apenas as seções abaixo relacionadas à tarefa, incluindo os IDs RF/RNF, regras e testes associados. Os requisitos e seus limites ficam em `docs/requisitos.md`; as demais seções da tabela referem-se a `docs/especificacao.md`. Não carregue toda a documentação por padrão.
3. Inspecione os arquivos envolvidos e o estado do Git, se houver. Amplie a leitura quando aparecer uma dependência ou divergência.

| Tarefa | Referências |
|---|---|
| Entender produto, prioridade e limites | 1–3 |
| Definir requisitos funcionais/não funcionais | [docs/requisitos.md](docs/requisitos.md); regras 7.3, testes 17, etapas 20 e decisões 23 |
| Alterar arquitetura ou dependências | 5–6, 18 e 23 |
| Domínio, cadastro, persistência | 7, 10; testes 17 |
| API, contratos, autenticação | 8–9, 12; regras aplicáveis em 7.3; testes 17 |
| Worker, idempotência, falhas | 6.4, 7.3, 10–11; testes 17 |
| Dashboard ou simulador | 13 ou 14, 12 e contratos utilizados em 8–9; testes 17 |
| ERP ou webhook, quando P1 estiver liberado | 15 ou 16, 7.3, 12; testes 17 |
| Ambiente, M0 ou outro marco | 5, 18–20; ler o marco e os IDs de requisitos associados |
| Testar ou concluir funcionalidade | 17 e 19.1, além da regra/contrato alterado |
| Publicar ou demonstrar | 21–22 |

Localize títulos com `rg -n '^#{2,3} ' docs/especificacao.md`. Leia o intervalo necessário com `sed`; por exemplo, `sed -n '/^## 7\. /,/^## 8\. /p' docs/especificacao.md`. Não registre números de linha como referência permanente.

Localize requisitos por ID, por exemplo `rg -n 'RF-04|RNF-03' docs/requisitos.md`, incluindo limites referenciados na seção 4 do catálogo. IDs RF/RNF identificam o aceite; regras RB e cenários T detalham comportamento e verificação.

## Fontes e manutenção

- **Este arquivo:** navegação e regras de trabalho.
- **`docs/especificacao.md`:** documentação técnica pública: escopo, regras, contratos, arquitetura proposta, testes e critérios dos marcos. Atualize a seção responsável; referencie em vez de copiar.
- **`docs/requisitos.md`:** fonte dos RF/RNF, critérios de aceite e valores do perfil local. A especificação referencia esse catálogo sem duplicar seus limites.
- **`.agents/progress.md`:** estado verificado, pendências e próximo incremento. Substitua o resumo ao encerrar a tarefa; evite diário de sessões ou saídas extensas.
- **`.agents/handoff.md`:** apenas encaminha para esta entrada; não mantém instruções próprias.

A documentação técnica de referência fica em `docs/`. A redação deve ser informativa, descritiva e técnica: comportamento, contexto, restrições, decisões e critérios verificáveis. Requisitos podem usar linguagem normativa; procedimentos podem usar passos objetivos. Não incluir diálogo com o usuário, relatos de sessão, explicações do agente sobre o próprio trabalho, instruções para IA, objetivos de candidatura ou dados pessoais. A documentação pública não referencia os arquivos operacionais de agentes. Instruções e continuidade ficam na raiz ou em `.agents/`; informações pessoais não pertencem a `docs/`.

Em divergências, siga a solicitação atual e registre o ajuste na fonte responsável. Não transforme o comportamento do código em requisito sem avaliá-lo. Se faltar informação essencial, explicite a lacuna e avance nas partes independentes.

Crie outro documento somente quando houver conteúdo real com uso independente que estas fontes não acomodem bem. Ao extrair conteúdo, mova-o e deixe um link. ADR curto só para decisão necessária com alternativas e consequência; não criar pastas vazias, templates ou cópias por marco. O README resume o projeto e aponta para as referências técnicas; instruções de execução serão incluídas quando verificáveis.

## Execução

- Trabalhe em um incremento verificável. Em implementação, relacione regra/requisito, comportamento esperado e testes; siga M0 → M4 antes de P1/P2. A documentação não autoriza iniciar M0 automaticamente.
- Preserve trabalho existente. Avalie impacto antes de commits, push, remoções ou instalações globais; explique ações destrutivas.
- Ao concluir cada incremento validado, revise o diff, selecione apenas os arquivos da tarefa e crie um commit com mensagem descritiva, por exemplo `docs: define requisitos funcionais e nao funcionais`. Não criar commits vazios nem incluir arquivos locais ignorados. Esta regra registra a autorização de continuidade para commits das próximas etapas.
- Valide no ambiente real apenas o necessário para a mudança. Não afirme build, testes, infraestrutura ou métricas sem execução; registre falhas e comandos não executados.
- Ao encerrar, informe alterações, validação e limitações; atualize o estado e o próximo passo em `.agents/progress.md`.
