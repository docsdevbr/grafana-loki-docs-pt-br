---
# SPDX-FileCopyrightText: 2026 Grafana Labs.
# Grafana and the Grafana logo are trademarks owned by Raintank, Inc. dba
# Grafana Labs.
#
# SPDX-License-Identifier: AGPL-3.0-only
# Documentation licensed under the GNU Affero General Public License Version 3.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/grafana-loki-docs-pt-br/blob/-/LICENSES/AGPL-3.0-only.txt

source_url: https://github.com/grafana/loki/blob/v3.7.8/docs/sources/get-started/deployment-modes.md
source_revision: c8cfd0aca3505ed5513cdee9668ca6d2f9905f56
translation_status: ready

menuTitle: Modos de implantação
title: Modos de implantação do Loki
description: Descreve os três diferentes modelos de implantação do Loki.
weight: 600
aliases:
  - ../fundamentals/architecture/deployment-modes
---

# Modos de implantação do Loki

O Loki é um sistema distribuído composto por diversos microsserviços.
Ele também possui um modelo de construção único, no qual todos esses
microsserviços residem em um único binário.

É possível configurar o comportamento desse binário único utilizando a flag de
linha de comando `-target` para especificar quais microsserviços serão
executados na inicialização.
Além disso, você pode configurar cada um dos componentes no arquivo `loki.yaml`.

Como o Loki desacopla os dados armazenados do software responsável pela ingestão
e consulta, você pode reimplantar facilmente um cluster em um modo diferente
conforme suas necessidades mudam, exigindo poucas ou nenhuma alteração na
configuração.

## Modo monolítico

O modo de operação mais simples é o modo de implantação monolítica.
Você ativa o modo monolítico definindo o parâmetro de linha de comando
`-target=all`.
Esse modo executa todos os componentes de microsserviços do Loki em um único
processo, como um binário único ou uma imagem Docker.

![Diagrama do modo monolítico](../monolithic-mode.png "Modo monolítico")

O modo monolítico é útil para começar rapidamente a experimentar o Loki, bem
como para volumes reduzidos de leitura/escrita, de até aproximadamente 20 GB por
dia.

É possível escalar horizontalmente uma implantação em modo monolítico para mais
instâncias utilizando um armazenamento de objetos compartilhado e configurando a
seção [`ring`](https://grafana.com/docs/loki/<LOKI_VERSION>/configure/#common)
do arquivo `loki.yaml` para compartilhar o estado entre todas as instâncias; no
entanto, a recomendação é utilizar o modo de implantação com microsserviços caso
precise escalar sua implantação.

Você pode configurar a alta disponibilidade executando duas instâncias do Loki
com a configuração `memberlist_config` e um armazenamento de objetos
compartilhado, definindo o `replication_factor` como `3`.
O tráfego é direcionado para todas as instâncias do Loki utilizando uma
estratégia round-robin.

A paralelização de consultas é limitada pelo número de instâncias e pela
configuração `max_query_parallelism`, definida no arquivo `loki.yaml`.

## Simple scalable

{{< admonition type="caution" >}}
O modo de implantação simple scalable (simples e escalável, ou SSD) está sendo
descontinuado e será removido no lançamento do Loki 4.0.
Você deve planejar a migração do SSD para microsserviços ou para uma implantação
monolítica de alta disponibilidade (HA).
Não será possível executar o Loki 4.0 no modo SSD.
{{< /admonition >}}

A implantação simple scalable é a configuração padrão instalada pelo
[Chart do Helm do Loki](../../setup/install/helm/).
Esse modo de implantação é a maneira mais fácil de implantar o Loki em escala.
Ele equilibra a implantação no [modo monolítico](#modo-monolítico) e a
implantação de cada componente como um
[microsserviço separado](#microservices-mode).
A implantação simple scalable também é chamada de SSD.

{{< admonition type="note" >}}
Esse modo de implantação é às vezes referido pela sigla SSD (simple scalable
deployment); não o confunda com solid state drives (unidades de estado sólido).
O Loki utiliza armazenamento de objetos.
{{< /admonition >}}

O modo de implantação simple scalable do Loki separa os caminhos de execução
em alvos de leitura, escrita e backend.
Esses alvos podem ser escalados de forma independente, permitindo personalizar a
implantação do Loki para atender às necessidades do seu negócio em relação à
ingestão e consulta de logs, de modo que os custos de infraestrutura estejam
mais alinhados ao seu padrão de uso da ferramenta.

O modo de implantação simple scalable consegue escalar para volumes próximos a 1
TB de logs por dia.
Embora seja possível escalar além desse limite, nessa escala, o modo de
microsserviços passa a ser uma escolha melhor em termos de escalabilidade e
facilidade operacional.

![Diagrama do modo simple scalable](../scalable-monolithic-mode.png "Modo simple scalable")

Cada um dos três caminhos de execução no modo simple scalable é ativado
adicionando-se os seguintes argumentos ao Loki durante a inicialização:

- `-target=write` - O alvo de escrita possui estado e é controlado por um
  StatefulSet do Kubernetes.
  Ele contém os seguintes componentes:
  * Distributor
  * Ingester
- `-target=read` - O alvo de leitura não possui estado e pode ser executado como
  um Deployment do Kubernetes com escalonamento automático (observe que, no
  chart oficial do Helm, ele é implantado atualmente como um StatefulSet).
  Ele contém os seguintes componentes:
  * Query Frontend
  * Querier
- `-target=backend` - O alvo de backend possui estado e é controlado por um
  StatefulSet do Kubernetes.
  Contém os seguintes componentes:
  - Compactor
  - Index Gateway
  - Query Scheduler
  - Ruler
  - Bloom Planner (experimental)
  - Bloom Builder (experimental)
  - Bloom Gateway (experimental)

O modo de implantação simple scalable requer a implantação de um proxy reverso à
frente do Loki para direcionar as requisições de API do cliente para os nós de
leitura ou de escrita.
O chart do Helm do Loki inclui uma configuração padrão de proxy reverso
utilizando o NGINX.

## Modo de microsserviços

O modo de implantação de microsserviços executa os componentes do Loki como
processos distintos.
A implantação de microsserviços também é chamada de implantação distribuída.
Cada processo é invocado especificando seu `target`.
Para a versão 3.3, os componentes são:

- Bloom Builder (experimental)
- Bloom Gateway (experimental)
- Bloom Planner (experimental)
- Compactor
- Distributor
- Index Gateway
- Ingester
- Overrides Exporter
- Querier
- Query Frontend
- Query Scheduler
- Ruler
- Table Manager (obsoleto)

{{< admonition type="tip" >}}
Você pode visualizar a lista completa de alvos para a sua versão do Loki
executando o Loki com a flag `-list-targets`; por exemplo:

```bash
docker run docker.io/grafana/loki:3.7.0 -config.file=/etc/loki/local-config.yaml -list-targets
```
{{< /admonition >}}

![Diagrama do modo de microsserviços](../microservices-mode.png "Modo de microsserviços")

Executar componentes como microsserviços individuais proporciona maior
granularidade, permitindo escalar cada componente separadamente para melhor
atender ao seu caso de uso específico.

Implantações no modo de microsserviços podem resultar em instalações do Loki
mais eficientes.
No entanto, são também as mais complexas de configurar e manter.

O modo de microsserviços é recomendado apenas para clusters do Loki de grande
porte ou para operadores que necessitam de controle mais preciso sobre o
escalonamento e as operações do cluster.

O modo de microsserviços foi projetado para implantações no Kubernetes.
Um
[chart do Helm mantido pela comunidade](https://github.com/grafana/helm-charts/tree/main/charts/loki-distributed)
está disponível para implantar o Loki no modo de microsserviços.
