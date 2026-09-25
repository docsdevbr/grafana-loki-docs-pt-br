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

menuTitle: Anéis de hash
title: Anéis de hash consistente
description: >-
  Descreve como a arquitetura do Loki utiliza anéis de hash consistente.
weight: 800
aliases:
  - ../fundamentals/architecture/rings
---

# Anéis de hash consistente

[Anéis de hash consistente](https://en.wikipedia.org/wiki/Consistent_hashing)
são incorporados às arquiteturas de cluster do Loki para:

- Auxiliar no particionamento de linhas de log.
- Implementar alta disponibilidade.
- Facilitar o escalonamento horizontal dos clusters.
  Há um menor impacto no desempenho para operações que precisam rebalancear
  dados.

Os anéis de hash conectam instâncias de um mesmo tipo de componente quando:

- Existe um conjunto de instâncias do Loki em modo de implantação monolítica.
- Existem múltiplos componentes de leitura ou de escrita em modo de implantação
  simple scalable.
- Existem múltiplas instâncias de um tipo de componente em modo de
  microsserviços.

Nem todos os componentes do Loki são conectados por anéis de hash.
Estes componentes precisam estar conectados em um anel de hash:

- distributors
- ingesters
- query schedulers
- compactors
- rulers

Estes componentes podem, opcionalmente, ser conectados em um anel de hash:

- index gateway

Em uma arquitetura que possui três distributors e três ingesters definidos, os
anéis de hash para esses componentes conectam as instâncias de componentes do
mesmo tipo.

![Anéis de distributors e ingesters](../ring-overview.png "Anéis de distributors e ingesters")

Cada nó no anel representa uma instância de um componente.
Cada nó possui um armazenamento chave-valor que mantém informações de
comunicação para cada um dos nós naquele anel.
Os nós atualizam o armazenamento chave-valor periodicamente para manter o
conteúdo consistente em todos os nós.
Para cada nó, o armazenamento chave-valor mantém:

- Um ID do nó do componente.
- Endereço do componente, usado por outros nós como canal de comunicação.
- Uma indicação da saúde do nó do componente.

## Configurando anéis

Defina a
[configuração do anel](https://grafana.com/docs/loki/<LOKI_VERSION>/configure/#common)
dentro do bloco `common.ring`.

Utilize o tipo de armazenamento chave-valor `memberlist`, a menos que haja um
motivo convincente para utilizar um tipo diferente.
O `memberlist` utiliza um
[protocolo gossip)](https://en.wikipedia.org/wiki/Gossip_protocol) para propagar
informações a todos os nós, garantindo a consistência eventual do conteúdo do
armazenamento chave-valor.

Existem opções de configuração adicionais para os anéis de distributors,
ingesters e rulers.
Essas opções destinam-se apenas a usos avançados e especializados.
Elas são definidas nos blocos `distributor.ring` para distributors,
`ingester.lifecycler.ring` para ingesters e `ruler.ring` para rulers.

## Sobre o anel de distributors

Os distributors utilizam as informações em seu armazenamento chave-valor para
manter uma contagem da quantidade de distributors no anel.
Essa contagem também orienta a definição de limites do cluster.

## Sobre o anel de ingesters

As informações do anel de ingesters presentes nos armazenamentos chave-valor são
utilizadas pelos distributors.
Essas informações permitem que os distributors façam o particionamento das
linhas de log, determinando para qual ingester ou conjunto de ingesters cada
distributor enviará as linhas de log.

## Sobre o anel de query schedulers

Os query schedulers utilizam as informações em seu armazenamento chave-valor
para a descoberta de serviços dos próprios agendadores.
Isso permite que os queriers se conectem a todos os schedulers disponíveis e que
os schedulers se conectem a todos os query frontends disponíveis, criando
efetivamente uma fila única que auxilia no balanceamento da carga de consultas.

## Sobre o anel de compactors

Os compactors utilizam as informações no armazenamento chave-valor para
identificar uma única instância de compactor que será responsável pela
compactação.
O compactor é habilitado apenas na instância responsável, embora o target
compactor esteja presente em múltiplas instâncias.

## Sobre o anel de rulers

O anel de rulers é utilizado para determinar quais rulers avaliam quais grupos
de regras.

## Sobre o anel de index gateways

O anel de index gateways é utilizado para determinar qual gateway é responsável
pelos índices de determinado tenant quando consultado por rulers ou queriers.
