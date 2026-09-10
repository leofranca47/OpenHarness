# Workflow: PERFORMANCE

Pipeline **evidence-first** do Performance Engineer. Agnóstico de linguagem, framework,
banco e infraestrutura. A tecnologia é **descoberta dinamicamente** a partir do projeto.

---

## Filosofia

> **Evidence before optimization.**

Uma alteração só é melhoria se houver métrica que comprove. Preferir eliminar trabalho
a acelerar trabalho. Preferir causa raiz a sintoma. Preferir evidência a opinião.

### Princípios inegociáveis

```text
EVIDENCE       >  HYPOTHESIS
MEASUREMENT    >  OPINION
ROOT CAUSE     >  SYMPTOM
ELIMINATE WORK >  OPTIMIZE WORK
MEASURABLE     >  "LOOKS BETTER"
```

### Princípio "Don't do the work"

Antes de tentar acelerar uma operação, perguntar:

> "Precisamos realmente executar esse trabalho?"

Investigar se a operação pode ser: eliminada, cacheada, pré-calculada, materializada,
executada depois, executada sob demanda, executada em lote. Operação eliminada é
preferível a operação mais rápida.

### Princípio "root cause"

Não parar no primeiro sintoma. Perguntar "por que isso está demorando?" até chegar à
causa mais profunda razoavelmente suportada por evidência. Parar quando houver causa
suficientemente comprovada para uma correção.

**Não fazer análise infinita.** Aceitar causa "boa o suficiente" quando há evidência
que justifique a correção.

---

## Pipeline

```text
OBSERVE
    ↓
MEASURE
    ↓
LOCATE
    ↓
FORMULAR HIPÓTESES
    ↓
VALIDAR HIPÓTESES
    ↓
PLANEJAR
    ↓
IMPLEMENTAR
    ↓
TESTAR
    ↓
BENCHMARK
    ↓
COMPARAR
    ↓
ACEITAR / REJEITAR
```

### Descrição das fases

| # | Fase | O que fazer | Saída |
|---|---|---|---|
| 1 | **OBSERVE** | Coletar observabilidade existente (logs, métricas, traces, profiling, sentries) | Lista de fontes consultadas |
| 2 | **MEASURE** | Estabelecer baseline antes de qualquer mudança (latência, throughput, CPU, memória, queries) | Baseline numérica |
| 3 | **LOCATE** | Identificar o gargalo dominante pela taxonomia | Gargalo classificado |
| 4 | **HIPÓTESES** | Formar 2-5 hipóteses plausíveis com evidência de suporte | Lista ranqueada |
| 5 | **VALIDAR HIPÓTESES** | Projetar teste que refute cada hipótese (não que confirme) | Resultado por hipótese |
| 6 | **PLANEJAR** | Definir mudança mínima para validar a hipótese mais provável | Plano incremental |
| 7 | **IMPLEMENTAR** | Executar UMA mudança por vez (nunca empilhar) | Diff isolado |
| 8 | **TESTAR** | Rodar suite existente + teste novo se aplicável | Resultado de testes |
| 9 | **BENCHMARK** | Medir novamente com mesma metodologia | Nova métrica |
| 10 | **COMPARAR** | Before vs after; classificar resultado | Classificação |
| 11 | **ACEITAR/REJEITAR** | `IMPROVED` → manter; `NEUTRAL`/`REGRESSION` → reverter | Decisão final |

### Aplicação por modo

| Modo | Passos aplicados |
|---|---|
| `investigate` | 1 → 2 → 3 → 4 → 5 → 6 (read-only) |
| `optimize` | 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10 → 11 |
| `validate` | 2 → 9 → 10 → 11 (sem novas otimizações) |
| `report` | 1 → 2 → 3 → 4 → 5 → 6 + render do Performance Report |

---

## Descoberta automática de stack

Antes de investigar, detectar a stack via manifestos:

| Categoria | Arquivos a procurar |
|---|---|
| **Language** | `composer.json`, `package.json`, `requirements.txt`, `pyproject.toml`, `pom.xml`, `build.gradle`, `*.csproj`, `go.mod`, `Cargo.toml`, `Gemfile` |
| **Framework** | nomes dentro dos manifestos (`laravel/framework`, `django`, `@nestjs/core`, `spring-boot`, `rails`, etc.) |
| **Database** | strings de conexão em `.env*`, `config/*.php`, `application.yml`, drivers nos manifestos |
| **Cache** | Redis/Memcached/Varnish nos manifestos + configs |
| **Infrastructure** | `Dockerfile`, `docker-compose.yml`, `.kubernetes/`, `k8s/`, `helm/`, `terraform/`, `serverless.yml` |

### Regra

Não assumir. Primeiro detectar. Quando não for possível determinar:

```text
UNKNOWN
```

Nunca inventar.

### Exemplo de saída

```text
Detected stack

Language:
PHP 8.x

Framework:
Laravel 10

Database:
MariaDB

Cache:
Redis

Infrastructure:
Docker + OpenShift
```

---

## Observabilidade — fontes de evidência

Quando disponível, usar como fonte primária de evidência:

- **APM:** Sentry, OpenTelemetry, Prometheus, Grafana, Datadog, New Relic, Elastic
- **Logs:** structured logs, application logs, access logs
- **Métricas:** CPU, memória, GC, IO, network
- **Traces:** distributed tracing, spans
- **Profiling:** CPU profile, memory profile, flame graphs

### Sentry MCP — quando disponível

Sentry é tratado como **uma** fonte de evidência, não a única. Quando conectado, usar:

```text
transactions, traces, spans, errors, performance,
releases, regressions, database spans, HTTP spans, cache spans, queue spans
```

Quando NÃO está conectado:

```text
SENTRY_UNAVAILABLE
```

e continuar com outras fontes. **Nunca inventar dados do Sentry.**

### Prioridade de MCPs

```text
observability  >  database  >  source control  >  infrastructure
```

Não assumir MCPs instalados. Descobrir dinamicamente.

---

## Investigação orientada por evidência

Sintomas devem ser quebrados em fatias com tempo atribuído:

```text
Request: 4.8s

Database:
3.9s

HTTP:
300ms

Application:
400ms

Cache:
200ms
```

```text
Primary suspect:
DATABASE
```

O agente então aprofunda no banco, **não** começa otimizando código PHP/Java/Go porque
"achou algo melhorável".

---

## Taxonomia de gargalos

Todo gargalo deve ser classificado em **uma** das categorias:

```text
CPU
MEMORY
DATABASE
NETWORK
IO
CACHE
QUEUE
HTTP
EXTERNAL_SERVICE
APPLICATION
ALGORITHM
CONCURRENCY
LOCK
SERIALIZATION
DESERIALIZATION
GC
RUNTIME
INFRASTRUCTURE
ARCHITECTURE
UNKNOWN
```

---

## Hierarquia de otimização

Procurar otimizações de **maior impacto primeiro**:

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

---

## Mudanças incrementais

Nunca modificar dezenas de coisas simultaneamente. Preferir:

```text
Hipótese 1
    ↓
Change
    ↓
Test
    ↓
Benchmark
    ↓
Result

Hipótese 2
    ↓
Change
    ↓
Test
    ↓
Benchmark
    ↓
Result
```

Isso permite atribuir o resultado à alteração correta.

---

## Investigação por categoria

### Banco de dados

Investigar:

- queries lentas
- planos de execução
- índices
- cardinalidade
- joins
- scans (table scan, index scan)
- sorting
- aggregation
- locks
- deadlocks
- queries repetidas
- grandes volumes
- paginação
- acesso aleatório
- excesso de dados

**Não criar índices automaticamente sem evidência.**

### Algoritmos

Detectar:

- O(n²), O(n³)
- loops aninhados
- busca linear repetitiva
- processamento duplicado
- ordenações desnecessárias
- conversões repetitivas
- serialização excessiva

Quando possível, **estimar complexidade antes/depois**.

### Network

Investigar:

- chamadas HTTP sequenciais
- chamadas HTTP duplicadas
- payloads grandes
- falta de compressão
- timeouts
- retries
- chamadas externas lentas
- falta de cache
- falta de paralelização

Avaliar quando chamadas independentes podem ser executadas concorrentemente.

### Cache

Investigar:

- ausência de cache
- cache inadequado
- TTL
- invalidação
- cache stampede
- dados estáveis
- consultas repetitivas

**Não adicionar cache sem verificar consistência.**

### Concorrência

Investigar quando apropriado:

- operações independentes executadas sequencialmente
- workers insuficientes
- locks
- contenção
- filas
- threads
- async/await
- processos
- paralelização

**Não paralelizar operações com dependências ou que possam gerar race conditions.**

### CPU

Investigar:

- loops caros
- algoritmos ineficientes
- processamento duplicado
- serialização
- compressão
- criptografia
- parsing
- regex custosas
- processamento síncrono

### Memória

Investigar:

- datasets gigantes em memória
- leaks
- objetos mantidos desnecessariamente
- cache local excessivo
- leitura integral de arquivos grandes
- ausência de streaming

Avaliar: `streaming`, `pagination`, `batch processing`, `iterators`, `generators`, `chunking`.

### I/O

Investigar:

- filesystem
- uploads
- downloads
- logs
- acesso a disco
- chamadas externas
- leitura/escrita desnecessária

### Filas

Detectar trabalho que não precisa bloquear a operação principal:

```text
request
    ↓
trabalho essencial
    ↓
response
    ↓
background processing
```

**Não mover automaticamente operações necessárias para background.**

### Arquitetura

Quando o gargalo não pode ser resolvido com alteração local, **avaliar** mudanças:

- cache distribuído
- materialized views
- read replicas
- filas
- processamento assíncrono
- particionamento
- agregações pré-calculadas
- bancos especializados
- CDN
- edge caching
- event-driven architecture
- separação de workloads

**Não recomendar mudanças arquiteturais apenas por preferência tecnológica.** Deve
existir evidência de que a arquitetura atual é responsável pelo problema.

---

## Benchmark

Sempre que possível, estabelecer baseline:

```text
Latency:
P50
P95
P99

Throughput:

CPU:

Memory:

Database time:

Query count:

Error rate:
```

Executar a alteração. Executar novamente. Comparar.

### Formato de comparação

```text
BEFORE

P95: 3.8s
Queries: 74
DB: 3.1s


AFTER

P95: 820ms
Queries: 12
DB: 410ms


RESULT

P95 improvement:        78.4%
Query reduction:        83.8%
Database reduction:     86.8%
```

---

## Classificação de resultado

Cada mudança avaliada recebe **uma** classificação:

```text
IMPROVED       → métrica melhora de forma mensurável
NEUTRAL        → métrica não muda significativamente
REGRESSION     → métrica piora
UNVALIDATED    → sem evidência suficiente para classificar
```

Apenas `IMPROVED` conta como otimização comprovada. Os outros três valores exigem
reversão ou investigação adicional.

---

## Testes

Executar a suite existente do projeto antes/depois. Detectar a ferramenta automaticamente:

```text
PHPUnit, Pest, Jest, Vitest, Pytest, JUnit, MSTest, xUnit, Go test, RSpec
```

(ou equivalente). Não assumir nenhuma ferramenta. Para meta-tooling (este próprio
harness), os "comportamentos" testáveis são formato de frontmatter e referências cruzadas.

---

## Git

Quando a mudança é de código:

```text
performance/<descricao-curta>
```

Antes de qualquer operação: `git status`. Depois: `git diff`. Revisar o diff.

**Não alterar arquivos fora do escopo.**

---

## Perfis de tecnologia

Carregar **sempre**:

- `@harness/profiles/generic.md`

Carregar **quando a stack exigir**:

- `@harness/profiles/php.md` (PHP puro)
- `@harness/profiles/laravel.md` (Laravel, requer PHP já carregado)

Profiles específicos adicionam guidance; **nunca substituem** o profile genérico.

---

## Módulos sob demanda

Carregar conforme necessário:

- `@harness/core/principles.md` — regras globais (evidência > suposição)
- `@harness/core/context-strategy.md` — quando precisar mapear código desconhecido
- `@harness/workflows/tdd.md` — para mudanças que precisam de characterization tests
- `@investigator` — quando precisar mapear "como o sistema faz X?"

---

## Regras contra alucinação

Nunca inventar:

- métricas
- traces
- spans
- queries
- planos de execução
- benchmarks
- infraestrutura
- comportamento do sistema
- resultados de produção

Se uma informação não está disponível:

```text
UNKNOWN
```

Se não há dados suficientes para conclusão:

```text
INSUFFICIENT_DATA
```

Explicar exatamente o que falta.

---

## Observabilidade do agent

Registrar eventos do ciclo de vida da execução:

```text
investigation_started
stack_detected
sources_consulted
bottleneck_detected
hypothesis_created
change_started
tests_started
benchmark_started
benchmark_finished
optimization_result
```

---

## Performance Report (template de 13 seções)

Usado pelo modo `report`. Produzir exatamente estas seções, nesta ordem:

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

**Não inventar valores.** Onde não houver evidência, escrever `UNKNOWN` ou
`INSUFFICIENT_DATA` e explicar o que falta.

---

## Detecção de escopo

Identificar em qual nível o problema vive:

```text
single function
single class
single endpoint
database query
service
worker
application
microservice
infrastructure
system architecture
```

Começar pelo **menor escopo possível**. Expandir apenas quando necessário.

---

## Performance regression

Quando houver histórico de métricas, comparar releases:

```text
release anterior  vs  release atual
```

Procurar:

- aumento de latência
- aumento de queries
- aumento de erros
- aumento de consumo
- alteração de infraestrutura
- alteração de dependências
- alteração de queries
- alteração de algoritmo

---

## Anti-padrões

- ❌ Otimizar sem baseline
- ❌ Empilhar múltiplas mudanças sem validar cada uma
- ❌ Recomendar mudança arquitetural sem evidência
- ❌ Adicionar cache sem verificar consistência
- ❌ Criar índice sem evidência de query lenta
- ❌ Paralelizar operações com dependências
- ❌ Inventar métricas/traces/plans
- ❌ Apresentar opinião como medição
- ❌ Parar no primeiro sintoma sem buscar causa raiz
- ❌ Mover trabalho essencial para background
