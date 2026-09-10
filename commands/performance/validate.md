---
description: Valida se uma otimização existente realmente melhorou a performance
agent: plan
---

Você está executando `/performance validate`. Re-execute benchmark em uma
otimização já existente e classifique o resultado. **Não realize novas otimizações.**

## Alvo da validação

$ARGUMENTS

## Modo

`VALIDATE` — pipeline aplicado: `MEASURE → BENCHMARK → COMPARAR → ACEITAR/REJEITAR`.

Sem novas otimizações. Sem mudanças de código. Re-medição apenas.

## Workflow

Carregue `@harness/workflows/performance.md` para o pipeline completo, taxonomia e
template de relatório.

## Módulos de apoio (sob demanda)

- `@harness/workflows/performance.md` — pipeline + template (sempre)
- `@harness/core/principles.md` — regras globais
- `@harness/profiles/generic.md` — guidance agnóstico
- `@harness/profiles/php.md` ou `@harness/profiles/laravel.md` — se stack exigir

## Delegação

Esta validação é executada pelo agent `@performance`. O agent:

1. Identifica a mudança alvo (commit, branch, arquivo, ou PR).
2. Detecta stack via manifestos (mesma metodologia de `investigate`).
3. Coleta observabilidade disponível. Quando Sentry MCP não está conectado, emite
   `SENTRY_UNAVAILABLE` e continua.
4. Mede **APÓS** a mudança (mesma metodologia que produziu o `BEFORE` da otimização).
5. Compara BEFORE vs AFTER.
6. Classifica:
   - `IMPROVED` — métrica melhora de forma mensurável
   - `NEUTRAL` — métrica não muda significativamente
   - `REGRESSION` — métrica piora
   - `UNVALIDATED` — sem evidência suficiente para classificar

## Princípios

> **Evidence before optimization.** Validação exige a **mesma** metodologia usada
> na baseline original. Comparar P95 com P50, ou medir em horário diferente, é
> inválido.

> **Regressão silenciosa é falha.** Se a métrica piorou, a otimização não
> comprovou seu benefício e deve ser revisada.

## Formato de saída

```markdown
## Mudança alvo

- Commit / branch / PR: ...
- Descrição: ...
- Data da mudança: ...

## Metodologia de medição

- Mesmo cenário da baseline: <sim | não — explicar>
- Mesmo horário/condição: <sim | não — explicar>
- Mesma ferramenta de benchmark: <sim | não — explicar>

## Comparação

| Métrica | BEFORE | AFTER | Δ |
|---|---|---|---|
| P50 | ... | ... | ...% |
| P95 | ... | ... | ...% |
| P99 | ... | ... | ...% |
| Throughput | ... | ... | ...% |
| DB time | ... | ... | ...% |
| Queries | ... | ... | ...% |
| CPU | ... | ... | ...% |
| Memory | ... | ... | ...% |
| Error rate | ... | ... | ...% |

## Resultado

`IMPROVED` | `NEUTRAL` | `REGRESSION` | `UNVALIDATED`

## Recomendações

- Se `IMPROVED`: manter.
- Se `NEUTRAL`: investigar se a mudança teve efeito real; considerar reversão se
  adicionar complexidade sem ganho.
- Se `REGRESSION`: reverter e investigar por que a baseline não previu.
- Se `UNVALIDATED`: refazer medição com mesma metodologia ou coletar mais dados.
```

## Restrições

- **Não realizar novas otimizações durante `validate`.**
- **Não usar metodologia diferente da baseline.**
- **Não inventar números.** Se a baseline original não existir ou não estiver
  disponível, classificar como `UNVALIDATED` e explicar.
- **Não aprovar mudança sem evidência.**

## Próximo passo

Após `validate`:

- Se `REGRESSION`: considerar reversão via `/refactor` ou reverter manualmente.
- Se `IMPROVED`: considerar `/performance report <alvo>` para consolidar evidência.
