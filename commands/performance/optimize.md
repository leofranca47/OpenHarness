---
description: Investiga e otimiza performance com mudanças incrementais e benchmark before/after
agent: build
---

Você está executando `/performance optimize`. Execute o pipeline completo de
performance engineering com mudanças incrementais e medição before/after.

## Alvo da otimização

$ARGUMENTS

## Modo

`OPTIMIZE` — pipeline completo: `OBSERVE → MEASURE → LOCATE → HIPÓTESES → VALIDAR →
PLANEJAR → IMPLEMENTAR → TESTAR → BENCHMARK → COMPARAR → ACEITAR/REJEITAR`.

Mudanças incrementais (uma hipótese por vez), benchmark before/after, branch
`performance/<descricao-curta>`.

## Workflow

Carregue `@harness/workflows/performance.md` para o pipeline completo, taxonomia,
hierarquia de otimização e template de Performance Report.

## Módulos de apoio (sob demanda)

- `@harness/workflows/performance.md` — pipeline + template (sempre)
- `@harness/core/principles.md` — regras globais
- `@harness/profiles/generic.md` — guidance agnóstico
- `@harness/profiles/php.md` ou `@harness/profiles/laravel.md` — se stack exigir
- `@harness/workflows/tdd.md` — quando precisar de characterization tests antes de otimizar
- `@investigator` — para mapear "como o sistema faz X?"

## Delegação

Esta otimização é executada pelo agent `@performance`. O agent:

1. Detecta stack via manifestos.
2. Coleta observabilidade disponível. Quando Sentry MCP não está conectado, emite
   `SENTRY_UNAVAILABLE` e continua.
3. Estabelece **baseline numérica** antes de qualquer mudança (latência P50/P95/P99,
   throughput, CPU, memória, tempo de banco, número de queries, error rate).
4. Localiza gargalo dominante, classifica pela taxonomia, formula hipóteses.
5. Para cada hipótese:
   - Planeja mudança mínima
   - Implementa em branch `performance/<hipotese-curta>`
   - Roda suite de testes existente
   - Executa benchmark
   - Compara before/after
   - Classifica como `IMPROVED` / `NEUTRAL` / `REGRESSION` / `UNVALIDATED`
   - Se `NEUTRAL`/`REGRESSION` → reverter imediatamente
   - Se `IMPROVED` → manter e seguir para próxima hipótese
6. Consolida relatório final com mudanças que sobreviveram.

## Regra crítica

**Nunca alterar código sem baseline.** Se não for possível medir, parar em PLAN e
retornar ao usuário com sugestão de coletar baseline antes de prosseguir.

## Princípios

> **Evidence before optimization.** Uma alteração só é melhoria se houver métrica
> que comprove.

> **Don't do the work.** Eliminar trabalho > acelerar trabalho. Antes de otimizar
> uma operação, investigar se ela pode ser eliminada, cacheada, pré-calculada,
> executada depois, sob demanda, em lote.

> **Root cause.** Não parar no primeiro sintoma. Perguntar "por quê?" até causa
> suficiente para correção.

> **Mudanças incrementais.** Uma hipótese por vez. Empilhar mudanças torna
> impossível atribuir resultado à alteração correta.

## Hierarquia de otimização (ordem de prioridade)

```text
1.  Eliminar trabalho desnecessário
2.  Evitar chamadas desnecessárias
3.  Evitar queries desnecessárias
4.  Evitar processamento duplicado
5.  Reduzir quantidade de dados
6.  Corrigir algoritmos
7.  Melhorar acesso ao banco
8.  Melhorar cache
9.  Melhorar concorrência
10. Processar assincronamente
11. Melhorar utilização de recursos
12. Micro-otimizações
```

## Formato de saída

```markdown
## Stack detectada

Language: ...
Framework: ...
Database: ...
Infrastructure: ...

## Baseline (BEFORE)

P50: ...
P95: ...
P99: ...
Throughput: ...
DB time: ...
Queries: ...
CPU: ...
Memory: ...

## Mudança #1 (Hipótese H1)

Change: <diff resumido>
Tests: <PASS|FAIL>
Benchmark AFTER: ...
Improvement: ...
Result: IMPROVED | NEUTRAL | REGRESSION | UNVALIDATED
Action: manter | reverter

## Mudança #2 ...

## Resultado final

- Mudanças mantidas: ...
- Mudanças revertidas: ...
- Improvement total: ...

## Próximos passos sugeridos

- [ ] ...
```

## Restrições

- **Não inventar métricas, traces, planos de execução.**
- **Não recomendar mudanças arquiteturais sem evidência.**
- **Não paralelizar operações com dependências.**
- **Não adicionar cache sem verificar consistência.**
- **Não criar índice sem evidência de query lenta.**
- **Não mover trabalho essencial para background.**
- **Não alterar arquivos fora do escopo da otimização.**
- **Não empilhar múltiplas mudanças sem validar cada uma.**

## Próximo passo

Após `optimize`, o usuário pode:

- `/performance validate <alvo>` — re-executar benchmark em mudança já commitada
- `/performance report <alvo>` — gerar Performance Report completo
