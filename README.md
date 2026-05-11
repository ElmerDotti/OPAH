# Credit AI Hybrid Decision Platform

**Autor:** Elmer Dotti  
**Objetivo:** materializar em código-fonte a solução técnica proposta para o desafio de arquitetura de IA aplicada à análise de crédito PME, combinando motor de risco tabular, RAG contextual, DLP, política de decisão, fallback, observabilidade e blueprint de deploy em GCP.

## Visão geral

Este repositório implementa uma API funcional para análise de crédito PME. A proposta segue a arquitetura descrita no relatório técnico e na apresentação: **modelos supervisionados explicáveis** são usados para estimar risco em dados estruturados, enquanto **RAG com documentos de negócio** é usado para contextualizar evidências textuais e apoiar a tomada de decisão. A decisão final passa por uma política explícita de crédito, com zona cinzenta, revisão humana e fallback operacional.

A implementação local é executável sem serviços pagos. Em produção, os adaptadores locais foram desenhados para serem substituídos por serviços gerenciados da Google Cloud, como Cloud Run, BigQuery, Vertex AI, Feature Store, Model Registry, Vector Search, Sensitive Data Protection, Cloud Logging e Cloud Monitoring.

## Árvore de diretórios

```text
credit-ai-hybrid-decision-platform/
├── app/
│   ├── api/                 # Rotas FastAPI
│   ├── core/                # Configurações
│   ├── domain/              # Schemas de entrada e saída
│   ├── models/              # Motor de risco tabular
│   ├── orchestration/       # Orquestração score + RAG + política
│   ├── policies/            # Regras de decisão e fallback
│   ├── services/            # DLP e RAG locais
│   └── observability/       # Logs e métricas
├── configs/                 # Thresholds e política de crédito
├── docs/                    # Arquitetura, deploy, governança e model card
├── examples/                # Exemplos de request/response
├── infra/                   # Docker, Cloud Build e Terraform blueprint
├── scripts/                 # Utilitários de execução e publicação
├── tests/                   # Testes automatizados
└── README.md
```

## Como executar em 5 minutos

```bash
cp .env.example .env
make setup
source .venv/bin/activate
make run
```

Em outro terminal, execute:

```bash
curl -s -X POST http://localhost:8000/v1/credit/decide \
  -H 'Content-Type: application/json' \
  -d @examples/requests/credit_application_approve.json | python -m json.tool
```

A documentação interativa estará disponível em `http://localhost:8000/docs`.

## Decisão retornada pela API

A resposta contém decisão, fallback, PD, score, rating, reason codes, evidências recuperadas, parecer fundamentado e auditoria. Um exemplo simplificado é:

```json
{
  "request_id": "REQ-2026-0001",
  "decision": "APPROVE",
  "risk": {
    "probability_of_default": 0.12,
    "score": 880,
    "rating": "A"
  },
  "grounded_memo": "Parecer fundamentado...",
  "audit": {
    "model_family": "local_proxy_for_xgboost_calibrated_pd",
    "rag_backend": "local_tfidf_proxy_for_vertex_ai_vector_search_gemini"
  }
}
```

## Testes e qualidade

```bash
make test
make lint
```

Os testes cobrem a monotonicidade esperada do motor de risco, mascaramento de PII e contrato HTTP da API.

## Docker

```bash
make docker-build
make docker-run
```

## Como publicar no GitHub

```bash
cd credit-ai-hybrid-decision-platform
git init
git add .
git commit -m "feat: implement hybrid AI credit decision platform"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/credit-ai-hybrid-decision-platform.git
git push -u origin main
```

Um guia direto também está disponível em `scripts/publish_to_github.md` e em `docs/QUICKSTART_GITHUB.md`.

## Alinhamento com a solução técnica

| Tema do relatório | Materialização no código |
|---|---|
| XGBoost para dados estruturados | `app/models/risk_model.py`, com interface substituível por endpoint real. |
| RAG para documentos | `app/services/rag.py`, com recuperação local e contrato pronto para Vertex AI. |
| DLP e privacidade | `app/services/dlp.py`, com mascaramento de CNPJ, CPF, e-mail e telefone. |
| Política e fallback | `app/policies/decision_policy.py`, com hard rules, thresholds e revisão humana. |
| Orquestração multi-etapas | `app/orchestration/decision_orchestrator.py`. |
| Observabilidade | `app/observability/metrics.py` e `app/observability/logging.py`. |
| Deploy | `infra/docker`, `infra/cloudbuild` e `infra/terraform`. |
| Governança | `docs/GOVERNANCE.md` e `docs/MODEL_CARD.md`. |

## Documentação complementar

| Documento | Finalidade |
|---|---|
| `docs/QUICKSTART_GITHUB.md` | Passo a passo direto para executar, validar e publicar o projeto no GitHub. |
| `docs/ARCHITECTURE.md` | Explicação técnica da arquitetura, fluxo e formulação do score. |
| `docs/DEPLOYMENT.md` | Execução local, Docker e caminho recomendado para GCP. |
| `docs/GOVERNANCE.md` | Governança, segurança, fallback e revisão humana. |
| `docs/MODEL_CARD.md` | Model card do motor de risco demonstrativo. |
| `docs/REFERENCES.md` | Referências acadêmicas, técnicas e de documentação oficial. |
| `SECURITY.md` | Recomendações de segurança e privacidade. |

## Nota de uso responsável

Este projeto é uma implementação demonstrativa para processo seletivo. Para uso real em crédito, é indispensável treinar e validar modelos com dados históricos autorizados, aprovar políticas com áreas de risco, jurídico e compliance, monitorar vieses e drift, garantir revisão humana quando aplicável e manter trilhas auditáveis de decisão.
