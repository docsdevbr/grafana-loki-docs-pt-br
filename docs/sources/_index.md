---
# SPDX-FileCopyrightText: 2026 Grafana Labs.
# Grafana and the Grafana logo are trademarks owned by Raintank, Inc. dba
# Grafana Labs.
#
# SPDX-License-Identifier: AGPL-3.0-only
# Documentation licensed under the GNU Affero General Public License Version 3.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/grafana-loki-docs-pt-br/blob/-/LICENSES/AGPL-3.0-only.txt

source_url: https://github.com/grafana/loki/blob/main/docs/sources/_index.md
revision: 5a1d9162517463e824742902ea97b8c7e548c729
status: ready

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
    Um índice pequeno e blocos altamente compactados simplificam a operação e
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
Os dados de log em si são então compactados e armazenados em blocos em serviços
de armazenamento de objetos como o Amazon Simple Storage Service (S3) ou o
Google Cloud Storage (GCS), ou até mesmo localmente no sistema de arquivos.

## Explore

{{< card-grid key="cards" type="simple" >}}
