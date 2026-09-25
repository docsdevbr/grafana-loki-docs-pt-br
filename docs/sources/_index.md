---
# SPDX-FileCopyrightText: 2026 Grafana Labs.
# Grafana and the Grafana logo are trademarks owned by Raintank, Inc. dba
# Grafana Labs.
#
# SPDX-License-Identifier: AGPL-3.0-only
# Documentation licensed under the GNU Affero General Public License Version 3.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/grafana-loki-docs-pt-br/blob/-/LICENSES/AGPL-3.0-only.txt

source_url: https://github.com/grafana/loki/blob/v3.7.8/docs/sources/_index.md
source_revision: 3483f0f5e93ba0688b28c012dea035c3f020445a
translation_status: ready

title: Grafana Loki
description: >-
  O Grafana Loki é um conjunto de componentes de código aberto que podem ser
  combinados para formar uma solução completa de registro de logs.
aliases:
  - /docs/loki/
weight: 100
cascade:
  GRAFANA_VERSION: latest
hero:
  title: Grafana Loki
  level: 1
  image: /media/docs/loki/logo-grafana-loki.png
  width: 110
  height: 110
  description: >-
    O Grafana Loki é um conjunto de componentes de código aberto que podem ser
    combinados para formar uma solução completa de registro de logs.
    Um índice pequeno e chunks altamente compactados simplificam a operação e
    reduzem significativamente o custo do Loki.
cards:
  title_class: pt-0 lh-1
  items:
    - title: Aprenda sobre o Loki
      href: /docs/loki/latest/get-started/
      description: >-
        Aprenda sobre a arquitetura e os componentes do Loki, os vários modos de
        implantação e as melhores práticas para rótulos.
    - title: Configure o Loki
      href: /docs/loki/latest/setup/
      description: >-
        Consulte as instruções de como configurar e instalar o Loki, migrar de
        implantações anteriores e atualizar seu ambiente Loki.
    - title: Configure o Loki
      href: /docs/loki/latest/configure/
      description: >-
        Veja a referência de configuração do Loki e exemplos de configuração.
    - title: Envie logs para o Loki
      href: /docs/loki/latest/send-data/
      description: >-
        Selecione um ou mais clientes para usar para enviar seus logs para o
        Loki.
    - title: Gerencie o Loki
      href: /docs/loki/latest/operations/
      description: >-
        Saiba como gerenciar tenants, ingestão de logs, armazenamento, consultas
        e muito mais.
    - title: Consulte com LogQL
      href: /docs/loki/latest/query/
      description: >-
        Inspirado no PromQL, o LogQL é a linguagem de consulta do Grafana Loki.
        O LogQL utiliza rótulos e operadores para filtragem.
---

{{< docs/hero-simple key="hero" >}}

---

## Visão geral

Ao contrário de outros sistemas de registro, o Loki é construído em torno da
ideia de indexar apenas metadados sobre os rótulos dos seus logs (assim como os
rótulos do Prometheus).
Os dados de log em si são então compactados e armazenados em chunks em serviços
de armazenamento de objetos como o Amazon Simple Storage Service (S3) ou o
Google Cloud Storage (GCS), ou até mesmo localmente no sistema de arquivos.

## Explore

{{< card-grid key="cards" type="simple" >}}

## Grafana Loki: perguntas frequentes

Aqui estão as respostas para algumas perguntas frequentes sobre como configurar
e executar o Loki.

### Por onde começar se eu for iniciante no Loki?

Comece pelo
[Tutorial do Loki](https://grafana.com/docs/loki/latest/get-started/), que
explica o que é o Loki, como ele funciona e como os componentes se integram.

- **Experimente localmente.**
  A maneira mais rápida de testar é executar o Loki no
  [modo monolítico (binário único)](https://grafana.com/docs/loki/latest/setup/install/)
  usando o sistema de arquivos local como backend e enviar logs para ele com o
  [Grafana Alloy](https://grafana.com/docs/alloy/latest/).
- **Entenda os rótulos.**
  O modelo de indexação do Loki baseia-se em rótulos, o que difere da maioria
  dos sistemas de log.
  Leia sobre [Rótulos](https://grafana.com/docs/loki/latest/get-started/labels/)
  e as
  [melhores práticas para rótulos](https://grafana.com/docs/loki/latest/get-started/labels/bp-labels/)
  logo no início — como você rotula seus logs afeta diretamente o desempenho das
  consultas e os custos de armazenamento.
- **Aprenda LogQL.**
  [LogQL](https://grafana.com/docs/loki/latest/query/) é a linguagem de consulta
  do Loki.
  Comece com correspondências de rótulos e filtros de linha simples e, em
  seguida, explore os estágios da pipeline de log e as consultas de métricas
  conforme a necessidade.
- **Explore com o Grafana.**
  Conecte o Loki como uma
  [fonte de dados no Grafana](https://grafana.com/docs/grafana/latest/datasources/loki/)
  e use o recurso [Explore](https://grafana.com/docs/grafana/latest/explore/)
  para executar consultas ad-hoc em seus logs.

O [Grafana Labs Learning Hub](https://grafana.com/docs/learning-hub/) oferece
cursos práticos e autoguiados relevantes para o Loki:

- **[Envie logs para o Grafana Cloud usando o Alloy](https://grafana.com/docs/learning-hub/)**
  — Aprenda a configurar o Grafana Alloy para enviar logs para o Loki no Grafana
  Cloud.
- **[Envie dados de coletores externos](https://grafana.com/docs/learning-hub/)**
  — Aprenda a enviar logs para o Loki usando OpenTelemetry (OTLP) ou a API HTTP
  e escolha o método ideal para sua configuração.
- **[Explore os dados da sua infraestrutura com as aplicações de Drilldown do Grafana](https://grafana.com/docs/learning-hub/)**
  — Explore visualmente os logs do Loki sem escrever consultas LogQL, utilizando
  a aplicação Logs Drilldown para filtrar por rótulos, detectar padrões e
  investigar erros.
- **[Otimize o Adaptive Logs](https://grafana.com/docs/learning-hub/)** — Reduza
  o volume de logs e os custos de armazenamento no Grafana Cloud ajustando quais
  logs são ingeridos pelo Loki.

### Devo hospedar por conta própria ou usar o Grafana Cloud para logs?

A resposta certa depende da capacidade e das prioridades da sua equipe.
Escolha o Grafana Cloud se quiser começar a usar a ferramenta rapidamente,
preferir não gerenciar a infraestrutura ou fizer parte de uma equipe pequena sem
recursos dedicados de operações.
O plano gratuito é generoso o suficiente para a maioria das pessoas usuárias e
pequenos projetos.

Escolha a versão OSS ou Enterprise hospedada por conta própria se tiver
requisitos rigorosos de residência de dados ou conformidade, precisar manter
todos os dados em sua própria infraestrutura ou já tiver investimentos em
ferramentas autogerenciadas, como o Prometheus, e quiser expandi-los.

O conjunto de recursos do Cloud e de uma stack Enterprise autogerenciada é, na
maioria, equivalente.
A principal diferença reside na carga de trabalho operacional: hospedar por
conta própria significa que você é responsável pela disponibilidade, pelo
escalonamento e pela manutenção de cada componente da stack.

### Quais são as melhores práticas para o uso de rótulos vs. metadados estruturados no Loki?

O modelo de indexação do Loki difere significativamente de outros produtos de
observabilidade.
Escolher campos inadequados como rótulos é a causa mais comum de baixo
desempenho em consultas e alto consumo de recursos.

**Rótulos** definem um fluxo de logs.
Cada combinação única de valores de rótulo cria um novo fluxo, que é armazenado
e indexado separadamente.

- ✅ Use rótulos para dimensões de **baixa cardinalidade** pelas quais você
  sempre filtrará: `env`, `cluster`, `namespace`, `app`, `job`.
- ❌ Não use rótulos para valores de **alta cardinalidade**: pod, IDs de
  instância, IDs de requisição, IDs de usuário, IDs de trace, códigos de status
  HTTP, endereços IP.
  Cada valor único cria um novo fluxo, causando uma "explosão de cardinalidade"
  que prejudica o desempenho de ingestão e de consulta.

**Metadados estruturados** (introduzidos no Loki 3.0) permitem associar pares
chave-valor a entradas de log sem criar novos fluxos.
Utilize-os para valores que você deseja filtrar ou exibir, mas que possuem
cardinalidade muito alta para serem usados como rótulos:

```alloy
// Exemplo: anexar trace_id como metadado estruturado via Alloy
loki.process "default" {
  stage.json {
    expressions = { "trace_id" = "" }
  }
  stage.structured_metadata {
    values = { "trace_id" = "" }
  }
  forward_to = [loki.write.local.receiver]
}
loki.write "local" {
  endpoint {
    url = "http://loki:3100/loki/api/v1/push"
  }
}
```

**Campos analisados** (extraídos no momento da consulta usando `| json`,
`| logfmt`, `| pattern` ou `| regexp`) não exigem alterações de esquema nem
adicionam sobrecarga de indexação.
Dê preferência a eles para filtragem ad-hoc do conteúdo dos logs.

**Regra geral:** se você fosse filtrar por um campo em _todas_ as consultas, ele
deve ser um rótulo.
Se você só precisar dele ocasionalmente ou se ele tiver muitos valores
exclusivos, utilize metadados estruturados ou extraia-o da linha de log no
momento da consulta.

### Por que `service_name` aparece como `unknown_service` no Explore Logs?

O recurso Explore Logs do Loki utiliza um rótulo `service_name` para agrupar e
navegar pelos logs.
Se ele exibir `unknown_service`, significa que o Loki não conseguiu determinar
automaticamente um nome de serviço a partir do fluxo de logs recebido.

Se um rótulo `service_name` já estiver presente no fluxo, o Loki o utiliza
diretamente.
Caso contrário, o recurso `discover_service_name` do Loki busca o primeiro valor
não vazio entre as seguintes chaves de rótulo, nesta ordem:

- service
- app
- application
- app_name
- name
- app_kubernetes_io_name
- container
- container_name
- k8s_container_name
- component
- workload
- job
- k8s_job_name

**Causas comuns:**

- O log shipper (remetente de logs, por exemplo, Alloy, OTel Collector,
  kube-logging-operator) não está encaminhando metadados do Kubernetes como
  rótulos de fluxo.
- O nome do rótulo utiliza notação de ponto (`service.name`).
  O Loki normaliza os nomes dos rótulos no servidor, substituindo pontos por
  sublinhados (por exemplo, `service.name` torna-se `service_name`).
- A opção `discover_service_name` está desabilitada na configuração do Loki.

**Verifique os rótulos de fluxo** na seção Explore.
Se nenhuma das chaves mencionadas acima estiver presente como rótulo, configure
o log shipper para incluí-las.
No caso do Alloy, certifique-se de que `loki.source.kubernetes` ou
`discovery.kubernetes` esteja repassando os metadados do pod.

Você pode personalizar a lista de rótulos verificados pelo Loki definindo
`discover_service_name` em `limits_config`.

### Como corrijo erros de "ingestion rate limit exceeded" (HTTP 429)?

Um erro 429 do Loki indica que um limite de taxa de ingestão foi atingido.
O Loki aplica limites tanto globalmente quanto por fluxo.

**Limites globais de ingestão** (aplicados por tenant):

```yaml
limits_config:
  ingestion_rate_mb: 16        # padrão: 4 MB/s
  ingestion_burst_size_mb: 32  # padrão: 6 MB
```

**Limite de taxa por fluxo:**

```yaml
limits_config:
  per_stream_rate_limit: 5MB        # padrão: 3MB
  per_stream_rate_limit_burst: 15MB # padrão: 15MB (5x the rate limit)
```

**Para identificar quais fluxos são os responsáveis**, verifique a mensagem de
erro no seu log shipper.
Ela inclui os rótulos do fluxo causador do problema.
Você também pode consultar o endpoint de métricas do Loki para a métrica `loki_ingester_streams_created_total`, detalhada por tenant.

Se você hospeda sua própria instância e atinge limites consistentemente com um
volume legítimo de logs, aumente os valores mencionados acima na seção
`limits_config`.
Se você utiliza o Grafana Cloud, entre em contato com o suporte para ajustar os
limites do seu plano.

{{< admonition type="note" >}}
Aumentar os limites sem investigar a causa raiz pode mascarar produtores de logs
descontrolados.
Sempre verifique se uma carga de trabalho ou namespace específico está gerando
um pico inesperado antes de aumentar os limites.
{{< /admonition >}}

### Por que o Grafana não consegue se conectar ao Loki quando ambos estão sendo executados no Docker?

Quando o Grafana e o Loki são executados como contêineres Docker separados, o
`localhost` dentro do contêiner do Grafana refere-se ao próprio contêiner do
Grafana, e não à máquina host ou ao contêiner do Loki.
Essa é a configuração incorreta mais comum entre pessoas usuárias iniciantes.

**URL da fonte de dados correta no Docker Compose:**

```yaml
# docker-compose.yml
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_DATASOURCES_DEFAULT_URL=http://loki:3100
```

Nas configurações de fonte de dados do Grafana, use `http://loki:3100` (o nome
do serviço no Docker Compose) em vez de `http://localhost:3100`.

**Para implantações no Kubernetes**, use o nome DNS do serviço dentro do
cluster:

```
http://loki.monitoring.svc.cluster.local:3100
```

**Verifique a conectividade** acessando o contêiner do Grafana e executando:

```bash
curl http://loki:3100/ready
```

Se isso retornar `ready`, a configuração de rede está correta e o problema
reside na própria configuração da fonte de dados do Grafana.

### O que causa erros de "Too many outstanding requests" no meu dashboard?

Esse erro ocorre quando o número de consultas simultâneas ao Loki excede o
limite configurado para um tenant.
É mais comum quando um dashboard possui quatro ou mais painéis realizando
consultas em um longo intervalo de tempo contra uma implantação (monolítica) de
nó único.

**Principais parâmetros de configuração:**

```yaml
query_scheduler:
  max_outstanding_requests_per_tenant: 32000  # padrão: 32000

frontend:
  max_outstanding_per_tenant: 2048            # padrão: 2048

limits_config:
  split_queries_by_interval: 24h              # padrão: 1h
```

Dividir consultas por intervalo reduz a carga por requisição ao fragmentar
consultas que abrangem grandes períodos em chunks menores processados em
paralelo.
Para implantações em nó único sujeitas a uma carga constante de dashboards,
considere também aumentar o tamanho dos chunks para reduzir o número total de
leituras:

```yaml
ingester:
  chunk_target_size: 1572864
  max_chunk_age: 2h
  chunk_idle_period: 30m
```

Se o problema persistir em escala, considere migrar de uma implantação
monolítica para o modo de
[microsserviços](https://grafana.com/docs/loki/latest/get-started/deployment-modes/).

### Como fazer com que uma consulta LogQL retorne zero em vez de "no data"?

Por padrão, as consultas de métricas do LogQL não retornam pontos de dados para
intervalos em que nenhuma linha de log correspondeu aos critérios, em vez de
retornarem `0`.
Isso compromete cálculos de porcentagem e regras de alerta que esperam uma
baseline numérica.

**Solução alternativa usando `or on() vector(0)`** (segue o padrão do PromQL):

```logql
(
  sum(count_over_time({app="my-app"} |= "error" [5m]))
  or on() vector(0)
)
```

**Para consultas de proporção/porcentagem**, proteja o denominador com `> 0`
para evitar a divisão por zero (que resulta em `NaN`) e envolva toda a expressão
com `or on() vector(0)` para que períodos sem dados retornem `0` em vez de
nenhum resultado:

```logql
(
  (
    sum(count_over_time({app="my-app"} | json | status=~"5.." [5m]))
    or on() vector(0)
  )
  /
  (
    sum(count_over_time({app="my-app"} [5m]))
    > 0
  )
)
or on() vector(0)
```

**Para a geração de alertas**, este padrão é essencial.
Sem ele, uma regra de alerta que utiliza `count_over_time` não produz resultado
de avaliação (nem mesmo `0`) durante períodos de inatividade, o que pode fazer
com que os alertas oscilem ou nunca sejam resolvidos corretamente.

### Por que minha política de retenção de logs não está excluindo dados antigos?

A retenção no Loki é gerenciada pelo **Compactor**, e não diretamente pelo
ingester ou pelo backend de armazenamento.
Algumas razões comuns para a persistência de dados antigos, mesmo após a
configuração de um período de retenção, são:

- A opção `retention_enabled: true` deve ser definida no bloco `compactor`, e
  não apenas em `limits_config`.
- O Compactor deve estar em execução.
  Em implantações monolíticas, ele pode ser desativado inadvertidamente.
- A exclusão não é imediata.
  O Compactor é executado segundo um cronograma e marca os chunks para exclusão
  apenas após um período de carência configurável (`retention_delete_delay`).
- Ao utilizar armazenamento no sistema de arquivos, o tamanho do diretório de
  chunks pode não diminuir visivelmente de imediato.
  O Compactor marca os arquivos para exclusão primeiro e, em seguida, os remove
  em execuções subsequentes.

**Configuração mínima necessária:**

```yaml
compactor:
  working_directory: /data/loki/compactor
  retention_enabled: true
  delete_request_store: <your-object-store>
limits_config:
  retention_period: 360h
```

Para verificar se o Compactor está em execução e a retenção está ativa, examine
os logs do Loki em busca de linhas que mencionem `compactor` e procure pelas
métricas `loki_boltdb_shipper_compact_tables_operation_total` (execuções de
compactação) e `loki_compactor_apply_retention_operation_total` (execuções de
retenção).
