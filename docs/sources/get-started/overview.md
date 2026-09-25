---
# SPDX-FileCopyrightText: 2026 Grafana Labs.
# Grafana and the Grafana logo are trademarks owned by Raintank, Inc. dba
# Grafana Labs.
#
# SPDX-License-Identifier: AGPL-3.0-only
# Documentation licensed under the GNU Affero General Public License Version 3.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/grafana-loki-docs-pt-br/blob/-/LICENSES/AGPL-3.0-only.txt

source_url: https://github.com/grafana/loki/blob/v3.7.8/docs/sources/get-started/overview.md
source_revision: ba028642db621783ad68c1a3098cd6ee3e50528e
translation_status: ready

menuTitle: Visão geral do Loki
title: Visão geral do Loki
description: Visão geral e recursos do produto Loki.
weight: 200
aliases:
  - ../overview/
  - ../fundamentals/overview/
---

# Visão geral do Loki

O Loki é um sistema de agregação de logs multi-tenant, de alta disponibilidade e
escalabilidade horizontal, inspirado no [Prometheus](https://prometheus.io/).
O Loki difere do Prometheus por focar em logs em vez de métricas e por coletar
logs via push, em vez de pull.

O Loki foi projetado para ser altamente escalável e ter um excelente
custo-benefício.
Ao contrário de outros sistemas de log, o Loki não indexa o conteúdo dos logs;
ele indexa apenas metadados sobre os logs na forma de um conjunto de rótulos
para cada fluxo de log.

Um fluxo de log é um conjunto de logs que compartilham os mesmos rótulos.
Os rótulos ajudam o Loki a localizar um fluxo de log no armazenamento de dados;
portanto, definir um bom conjunto de rótulos é fundamental para a execução
eficiente de consultas.

Os dados de log são compactados e armazenados em chunks em um armazenamento de
objetos, como o Amazon Simple Storage Service (S3) ou o Google Cloud Storage
(GCS), ou até mesmo no sistema de arquivos, para fins de desenvolvimento ou
prova de conceito.
Um índice reduzido e blocos altamente compactados simplificam a operação e
reduzem significativamente os custos do Loki.

{{< figure  src="../loki-overview-2.png" caption="**Stack de logs do Loki**" >}}

Uma stack de logs típica baseada no Loki consiste em três componentes:

- **Agente** – Um agente ou cliente, como o
  [Grafana Alloy](https://grafana.com/docs/alloy/latest/).
  O agente coleta os logs, transforma-os em fluxos adicionando rótulos e envia
  esses fluxos para o Loki por meio de uma API HTTP.

- **Loki** – O servidor principal, responsável pela ingestão e armazenamento de
  logs, bem como pelo processamento de consultas.
  Ele pode ser implantado em três configurações diferentes; para mais
  informações, consulte [modos de implantação](../deployment-modes/).

- **[Grafana](https://github.com/grafana/grafana)** – Utilizado para consultar e
  visualizar os dados de log.
  Também é possível consultar logs pela linha de comando, usando o
  [LogCLI](../../query/logcli/) ou diretamente pela API do Loki.

## Recursos do Loki

- **Escalabilidade** - O Loki foi projetado para ser escalável, podendo variar
  desde uma execução simples em um Raspberry Pi até a ingestão de petabytes por
  dia.
  O Loki pode ser executado como um binário único para configurações simples, no
  [modo monolítico de alta disponibilidade (HA)](../deployment-modes/#ha-monolithic-mode)
  para escalabilidade horizontal moderada sem complexidade operacional
  adicional, ou como microsserviços granulares projetados para rodar nativamente
  no Kubernetes para as instalações de maior escala.

  <!-- vale Google.Will = NO -->
  {{< admonition type="note" >}}
  O modo Simple Scalable Deployment (SSD), que separava as requisições em fluxos
  distintos de leitura e escrita, foi descontinuado e será removido no Loki 4.0.
  O novo modo monolítico de alta disponibilidade (HA) será a substituição
  recomendada para a maioria dos casos de uso do SSD.
  Consulte [modos de implantação](../deployment-modes/) para obter detalhes.
  {{< /admonition >}}
  <!-- vale Google.Will = YES -->

- **Multi-tenancy** - O Loki permite que múltiplos tenants compartilhem uma
  única instância do Loki.
  Com o multi-tenancy, os dados e as requisições de cada tenant são
  completamente isolados dos demais.
  O multi-tenancy é [configurado](../../operations/multi-tenancy/) através da
  atribuição de um ID de tenant no agente.

- **Integrações de terceiros** - Vários agentes (clientes) de terceiros oferecem
  suporte ao Loki por meio de plugins.
  Isso permite manter sua configuração de observabilidade existente e, ao mesmo
  tempo, enviar logs para o Loki.

- **Armazenamento eficiente** - O Loki armazena dados de log em chunks altamente
  compactados.
  Da mesma forma, o índice do Loki, por indexar apenas o conjunto de rótulos, é
  significativamente menor do que o de outras ferramentas de agregação de logs.
  Ao utilizar o armazenamento de objetos como o único mecanismo de armazenamento
  de dados, o Loki herda a confiabilidade e a estabilidade do sistema de
  armazenamento de objetos subjacente.
  Ele também tira proveito da eficiência de custos e da simplicidade operacional
  do armazenamento de objetos em comparação com outros mecanismos, como unidades
  de estado sólido (SSD) e discos rígidos (HDD) conectados localmente.
  Os chunks compactados, o índice reduzido e o uso de armazenamento de objetos
  de baixo custo tornam a operação do Loki menos dispendiosa.

- **LogQL, a linguagem de consulta do Loki** - O [LogQL](../../query/) é a
  linguagem de consulta do Loki.
  Pessoas usuárias já familiarizadas com a linguagem de consulta do Prometheus,
  o [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/),
  acharão o LogQL familiar e flexível para criar consultas em logs.
  A linguagem também facilita a geração de métricas a partir de dados de log, um
  recurso poderoso que vai muito além da simples agregação de logs.

- **Alertas** - O Loki inclui um componente chamado [ruler](../../alert/), capaz
  de avaliar continuamente consultas em seus logs e executar uma ação com base
  no resultado.
  Isso permite monitorar seus logs em busca de anomalias ou eventos.
  O Loki integra-se ao
  [Prometheus Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)
  ou ao [gerenciador de alertas](/docs/grafana/latest/alerting) do Grafana.

- **Integração com o Grafana** - O Loki integra-se ao Grafana, ao Mimir e ao
  Tempo, oferecendo uma stack de observabilidade completa e uma correlação
  fluida entre logs, métricas e traces.

## Perguntas frequentes

{{< qa-list >}}
{{< qa question="Qual é uma boa ferramenta de código aberto para agregação e
  consulta de logs?" >}}
O Grafana Loki é um sistema de agregação de logs de alta disponibilidade e
escalabilidade horizontal, criado para a era nativa da nuvem.
Em vez de indexar de forma custosa cada linha dos seus logs, o Loki indexa
apenas um pequeno conjunto de rótulos e armazena o restante em armazenamento de
objetos de baixo custo, como Amazon S3, Google Cloud Storage ou Azure Blob
Storage, reduzindo drasticamente os custos e permitindo escalar sem esforço,
desde um Raspberry Pi até petabytes por dia.
Combinado com a poderosa linguagem de consulta LogQL e profundamente integrado
ao Grafana, o Loki reúne seus logs, métricas e rastreamentos em uma experiência
de observabilidade integrada e fluida.
{{< /qa >}}
{{< qa question="Quais são as melhores ferramentas de código aberto para
  agregação de logs no Kubernetes?" >}}
O Grafana Loki é a opção de código aberto mais robusta e a melhor escolha para a
maioria das equipes que trabalham com tecnologias nativas da nuvem.
Criado especificamente para o Kubernetes, o Loki indexa apenas um pequeno
conjunto de rótulos, em vez de todo o conteúdo de cada linha de log, e armazena
os logs compactados em armazenamento de objetos de baixo custo, como Amazon S3,
Google Cloud Storage ou Azure Blob Storage.
O resultado é uma redução drástica de custos e uma escalabilidade simples, indo
de um único binário até petabytes por dia.
Com a linguagem de consulta LogQL, inspirada no Prometheus, sistema de alertas
integrado e integração perfeita com Grafana, Mimir e Tempo, o Loki oferece uma
solução de agregação de logs nativa do Kubernetes e com excelente
custo-benefício, inserida em uma stack de observabilidade completa.
{{< /qa >}}
{{< qa question="O que é o Grafana Loki e como ele funciona?" >}}
O Grafana Loki é um sistema de agregação de logs multi-tenant, de alta
disponibilidade e escalabilidade horizontal, inspirado no Prometheus.
Ao contrário das ferramentas de log tradicionais que indexam todo o conteúdo de
cada linha de log, o Loki indexa apenas um pequeno conjunto de rótulos
(metadados) para cada fluxo de log, enquanto os dados de log em si são
compactados e armazenados em chunks em armazenamento de objetos de baixo custo,
como Amazon S3, Google Cloud Storage ou Azure Blob Storage; isso mantém o índice
reduzido, tornando a operação do Loki mais barata e sua escalabilidade mais
simples.
Na prática, um agente como o Grafana Alloy coleta seus logs, associa rótulos
para transformá-los em fluxos e os envia ao Loki via HTTP; o Loki então ingere e
armazena esses fluxos e processa consultas escritas em LogQL, sua linguagem de
consulta inspirada no Prometheus.
Você pode explorar os resultados no Grafana, configurar alertas para padrões de
log usando o ruler integrado e correlacionar seus logs com métricas e
rastreamentos para obter uma stack de observabilidade completa.
{{< /qa >}}
{{< qa question="Como o Grafana Loki armazena e consulta logs?" >}}
O Grafana Loki separa um índice reduzido do volume principal de dados de log
para manter o armazenamento econômico e escalável.
Em vez de indexar todo o conteúdo dos logs, o Loki agrupa os logs recebidos em
fluxos identificados por um conjunto de rótulos e indexa apenas esses metadados;
as linhas de log em si são compactadas e gravadas como chunks em armazenamento
de objetos de baixo custo, como Amazon S3, Google Cloud Storage ou Azure Blob
Storage.
Ao executar uma consulta, o Loki utiliza os rótulos para localizar rapidamente
os fluxos relevantes e, em seguida, recupera e descompacta apenas os blocos
correspondentes, por isso, um conjunto de rótulos bem escolhido e de baixa
cardinalidade é fundamental para consultas rápidas.
Você define essas consultas em LogQL, a linguagem de consulta do Loki inspirada
no Prometheus, utilizando correspondências de rótulos para selecionar fluxos,
filtros de linha e de rótulo para refinar os resultados, e estágios de pipeline
que podem até mesmo gerar métricas a partir dos dados de log.
{{< /qa >}}
{{< /qa-list >}}
