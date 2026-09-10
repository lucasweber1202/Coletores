# Coletores Macroeconômicos

Repositório central para organizar, especificar e acompanhar a criação de coletores de dados macroeconômicos.

> Este repositório é o hub de governança da frota. Cada coletor de fonte/dataset deve continuar em um repositório independente chamado `collector_<source>_<dataset>`.

## Comece por aqui

1. Leia [MASTER_MACRO_COLLECTOR_GUIDELINES.md](MASTER_MACRO_COLLECTOR_GUIDELINES.md).
2. Registre cada demanda em [intake/collector_demands.csv](intake/collector_demands.csv).
3. Para uma demanda individual, preencha [templates/COLLECTOR_REQUEST.md](templates/COLLECTOR_REQUEST.md).
4. Siga o fluxo descrito em [docs/WORKFLOW.md](docs/WORKFLOW.md).
5. Só marque um coletor como `ready` depois da auditoria da fonte e da verificação completa de idempotência.

## Estrutura

- `MASTER_MACRO_COLLECTOR_GUIDELINES.md`: fonte única de verdade.
- `AGENTS.md`, `CLAUDE.md` e `.github/copilot-instructions.md`: pontos de entrada para agentes.
- `.github/skills/`: procedimentos especializados.
- `intake/`: backlog dos países, fontes e datasets.
- `templates/`: formulário reutilizável para iniciar um coletor.
- `docs/WORKFLOW.md`: estados, critérios de passagem e handoff.

## Regra central

Não crie uma biblioteca compartilhada entre os coletores. A padronização é mantida pela guideline e pela cópia do piloto; cada coletor permanece autocontido.

## Próximo passo

Envie ou cole a lista de países e demandas. Ela será normalizada no backlog sem iniciar código enquanto faltarem fonte oficial, dataset, frequência pretendida, séries-alvo ou autenticação.
