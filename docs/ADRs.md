# ADRs de Deploy Kubernetes

## ADR-001 — Um namespace por ambiente

- **Status:** Aceito
- **Decisão:** usar `mecanica-hermes-hml` e `mecanica-hermes-prd` como namespaces dedicados.
- **Justificativa:** isolamento de configuração e execução entre ambientes.

## ADR-002 — Segredos gerados no momento do deploy

- **Status:** Aceito
- **Decisão:** criar/atualizar `app-secret` via workflow, com dados descobertos do RDS e segredos do GitHub.
- **Justificativa:** evita segredos hardcoded em manifests e centraliza dados sensíveis no pipeline.

## ADR-003 — Observabilidade via New Relic em workflow separado

- **Status:** Aceito
- **Decisão:** manter deploy/destroy de New Relic em workflows dedicados.
- **Justificativa:** desacopla o ciclo de vida da observabilidade do ciclo da aplicação.

## ADR-004 — Workflow de force cleanup para namespace preso

- **Status:** Aceito
- **Decisão:** manter workflow específico para remover finalizers e recriar namespace quando necessário.
- **Justificativa:** acelera recuperação operacional em cenários de namespace em estado `Terminating`.
