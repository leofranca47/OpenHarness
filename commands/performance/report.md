---
description: Produz Performance Report estruturado sem alterar o projeto
agent: plan
---

Você está executando `/performance report`. Produza um **Performance Report**
estruturado em 13 seções, sem alterar o projeto.

## Alvo do relatório

$ARGUMENTS

## Modo

`REPORT` — pipeline aplicado: `OBSERVE → MEASURE → LOCATE → HIPÓTESES → VALIDAR →
PLANEJAR` + render do Performance Report.

Read-only. Sem mudanças de código. Sem commit. Saída é o relatório em si.

## Workflow

Carregue `@harness/workflows/performance.md` para o pipeline completo, taxonomia,
hierarquia de otimização e **template exato** do Performance Report.

## Módulos de apoio (sob demanda)

- `@harness/workflows/performance.md` — pipeline + template (sempre)
- `@harness/core/principles.md` — regras globais
- `@harness/profiles/generic.md` — guidance agnóstico
- `@harness/profiles/php.md` ou `@harness/profiles/laravel.md` — se stack exigir
- `@investigator` — para mapear "como o sistema faz X?"

## Delegação

Este relatório é executado pelo agent `@performance`. O agent:

1. Detecta stack via manifestos.
2. Coleta observabilidade disponível. Quando Sentry MCP não está conectado, emite
   `SENTRY_UNAVAILABLE` e continua.
3. Estabelece baseline (quando possível) ou marca `INSUFFICIENT_DATA`.
4. Localiza gargalos pela taxonomia.
5. Formula hipóteses.
6. Renderiza o Performance Report exatamente nas **13 seções** do template abaixo.

## Formato de saída — Performance Report (13 seções)

O output deve seguir **exatamente** este template, nesta ordem, com estes nomes
de seção (case-sensitive):

```markdown
# Performance Report

## Stack

Language:
Framework:
Database:
Infrastructure:

## Scope

...

## Baseline

...

## Bottlenecks

1.
2.
3.

## Root Cause

...

## Hypotheses

...

## Changes

...

## Tests

...

## Benchmark

BEFORE:
...

AFTER:
...

## Result

IMPROVED / NEUTRAL / REGRESSION / UNVALIDATED

## Risks

...

## Recommendations

...

## Next Steps

...
```

## Regras de preenchimento

- **Nunca inventar valores.** Quando não houver dado, escrever `UNKNOWN` ou
  `INSUFFICIENT_DATA` e explicar o que falta.
- **Cada seção deve ter conteúdo substantivo.** Se uma seção não se aplicar ao
  escopo, escrever `N/A — <motivo>` em vez de omitir.
- **Bottlenecks** deve ser lista numerada ranqueada (do mais impactante ao menos),
  usando a taxonomia (`CPU`, `MEMORY`, `DATABASE`, `NETWORK`, `IO`, `CACHE`,
  `QUEUE`, `HTTP`, `EXTERNAL_SERVICE`, `APPLICATION`, `ALGORITHM`, `CONCURRENCY`,
  `LOCK`, `SERIALIZATION`, `DESERIALIZATION`, `GC`, `RUNTIME`, `INFRASTRUCTURE`,
  `ARCHITECTURE`, `UNKNOWN`).
- **Benchmark** deve ter blocos `BEFORE:` e `AFTER:` explícitos. Se não houver
  after (escopo read-only), escrever `AFTER: N/A — read-only report`.
- **Result** deve ser exatamente um dos 4 valores literais.
- **Risks** deve listar riscos reais (probabilidade, impacto, mitigação).
- **Recommendations** deve distinguir "fazer agora" de "considerar futuramente",
  apenas quando há evidência.
- **Next Steps** deve listar ações concretas que o usuário pode tomar.

## Princípios

> **Evidence before optimization.** O relatório é uma fotografia baseada em
> evidência. Não é uma opinião.

> **Não inventar.** Métricas, traces, queries, planos de execução e resultados
> de produção só aparecem quando foram observados. Quando faltar, marcar
> `UNKNOWN` ou `INSUFFICIENT_DATA`.

> **Don't do the work.** Mudanças que **eliminam** trabalho são listadas antes
> de mudanças que **aceleram** trabalho.

## Restrições

- **Não alterar código do projeto.**
- **Não inventar métricas, traces, queries, planos de execução.**
- **Não usar nomes de seção diferentes do template.**
- **Não omitir seções (usar `N/A — <motivo>` quando não aplicável).**
- **Não classificar como `IMPROVED` sem benchmark before/after.**

## Próximo passo

Após `report`, o usuário pode:

- `/performance optimize <alvo>` — executar plano recomendado
- `/performance validate <mudança>` — validar otimização existente
- Compartilhar o relatório com a equipe
