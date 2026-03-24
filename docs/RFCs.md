# RFCs de Deploy Kubernetes

## RFC-001 — Kustomize para gestão multiambiente

- **Status:** Aceito
- **Contexto:** necessidade de manter base única com diferenças mínimas entre HML e PRD.
- **Decisão:** adotar `base/` compartilhado e `overlays/` por ambiente.
- **Impactos:** reduz duplicação e facilita manutenção das diferenças de configuração.

## RFC-002 — Deploy via GitHub Actions

- **Status:** Aceito
- **Contexto:** padronizar publicação sem dependência de execução local.
- **Decisão:** usar workflows manuais para deploy/destroy da API e observabilidade.
- **Impactos:** aumenta rastreabilidade e reduz variação operacional.

## RFC-003 — Exposição por NLB interno

- **Status:** Aceito
- **Contexto:** API deve ser consumida por API Gateway com VPC Link, sem exposição pública direta do cluster.
- **Decisão:** manter `Service` tipo `LoadBalancer` com anotações para NLB interno.
- **Impactos:** tráfego de entrada centralizado no API Gateway e melhor isolamento de rede.
