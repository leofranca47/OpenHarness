---
description: Investiga gargalos de performance sem alterar código
agent: plan
---

Você está executando `/performance investigate`. Investigue gargalos de performance
em modo **read-only**. Não altere código.

## Alvo da investigação

$ARGUMENTS

## Modo

`INVESTIGATE` — pipeline aplicado: `OBSERVE → MEASURE → LOCATE → HIPÓTESES → VALIDAR → PLANEJAR`.

Sem implementação. Sem mudança de código. Sem commit.

## Workflow

Carregue `@harness/workflows/performance.md` para o pipeline completo, taxonomia e
hierarquia de otimização.

## Módulos de apoio (sob demanda)

- `@harness/workflows/performance.md` — pipeline + template (sempre)
- `@harness/core/principles.md` — regras globais
- `@harness/profiles/generic.md` — guidance agnóstico
- `@harness/profiles/php.md` ou `@harness/profiles/laravel.md` — se stack exigir
- `@investigator` — para mapear "como o sistema faz X?"

## Delegação

Esta investigação é executada pelo agent `@performance`. O agent:

1. Detecta stack via manifestos (`composer.json`, `package.json`, `pyproject.toml`,
   `requirements.txt`, `go.mod`, `Cargo.toml`, `pom.xml`, `*.csproj`, `Gemfile`).
2. Coleta observabilidade disponível (Sentry, OpenTelemetry, Prometheus, Grafana,
   Datadog, New Relic, logs, métricas, traces, profiling). Quando Sentry MCP não
   está conectado, emite `SENTRY_UNAVAILABLE` e continua com outras fontes.
3. Carrega `@harness/profiles/generic.md` por padrão; profiles específicos apenas
   quando a stack exigir.
4. Estabelece baseline numérica (latência P50/P95/P99, throughput, CPU, memória,
   tempo de banco, número de queries, error rate).
5. Localiza o gargalo dominante e classifica pela taxonomia.
6. Formula 2-5 hipóteses ranqueadas, cada uma com teste que **refute**.
7. Produz plano ordenado (sem implementação).

## Princípios

> **Evidence before optimization.** Nunca inventar métricas, traces, queries ou
> planos de execução. Usar `UNKNOWN` / `INSUFFICIENT_DATA` quando faltar evidência.

> **Don't do the work.** Antes de propor otimização, perguntar se o trabalho pode
> ser eliminado, cacheado, pré-calculado, executado depois, sob demanda, em lote.

> **Root cause.** Não parar no primeiro sintoma. Perguntar "por quê?" até causa
> suficiente para correção, sem análise infinita.

## Formato de saída

```markdown
## Stack detectada

Language:
Framework:
Database:
Cache:
Infrastructure:

## Fontes de evidência consultadas

- <fonte 1> — dado: ...
- <fonte 2> — `SENTRY_UNAVAILABLE` se Sentry MCP não conectado
- ...

## Baseline

P50: ...
P95: ...
P99: ...
Throughput: ...
DB time: ...
Queries: ...
CPU: ...
Memory: ...

## Bottlenecks (ranqueados)

1. **[CATEGORIA]** — descrição — evidência: <arquivo:linha | métrica | trace>
2. ...

## Hipóteses

1. H1 — plausibilidade: alta — teste que refuta: ...
2. H2 — plausibilidade: média — teste que refuta: ...
3. ...

## Plano (sem implementação)

- [ ] H1 — validar primeiro porque ...
- [ ] H2 — validar em segundo lugar

## UNKNOWN / INSUFFICIENT_DATA

- <o que faltou> — sugestão de como obter
```

## Restrições

- **Não alterar código do projeto.**
- **Não inventar métricas, traces, planos de execução.**
- **Não recomendar mudanças arquiteturais sem evidência.**
- **Não paralelizar operações sem análise de dependências.**
- **Não adicionar cache sem verificar consistência.**
- **Não criar índice sem evidência de query lenta.**

## Próximo passo

Após `investigate`, o usuário pode:

- `/performance optimize <alvo>` — executar o plano
- `/performance report <alvo>` — gerar Performance Report completo
- `@investigator <pergunta>` — mapear área específica
