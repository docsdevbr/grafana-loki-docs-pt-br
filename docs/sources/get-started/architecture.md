---
# SPDX-FileCopyrightText: 2026 Grafana Labs.
# Grafana and the Grafana logo are trademarks owned by Raintank, Inc. dba
# Grafana Labs.
#
# SPDX-License-Identifier: AGPL-3.0-only
# Documentation licensed under the GNU Affero General Public License Version 3.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/grafana-loki-docs-pt-br/blob/-/LICENSES/AGPL-3.0-only.txt

source_url: https://github.com/grafana/loki/blob/v3.7.8/docs/sources/get-started/architecture.md
revision: 9979490731774ce86b7204004e6a0889159cd844
status: ready

title: Arquitetura do Loki
menuTitle: Arquitetura
description: Descreve a arquitetura do Grafana Loki.
weight: 400
aliases:
  - ../architecture/
  - ../fundamentals/architecture/
---

# Arquitetura do Loki

O Grafana Loki possui uma arquitetura baseada em microsserviços e foi projetado
para operar como um sistema distribuído e horizontalmente escalável.
O sistema conta com múltiplos componentes que podem ser executados de forma
independente e em paralelo.
O projeto do Grafana Loki compila o código de todos os componentes em um único
binário ou imagem Docker.
O parâmetro de linha de comando `-target` define qual componente (ou
componentes) aquele binário irá executar.

Para começar de forma simples, execute o Grafana Loki no modo "binário único"
(com todos os componentes rodando simultaneamente em um único processo) ou no
modo "implantação escalável simples" (que agrupa os componentes em partes de
leitura, escrita e backend).

O Grafana Loki foi projetado para permitir a reimplementação fácil de um cluster
em um modo diferente conforme suas necessidades mudam, exigindo pouca ou nenhuma
alteração na configuração.

Para mais informações, consulte [Modos de implantação](../deployment-modes/) e
[Componentes](../components/).

![Componentes do Loki](../loki_architecture_components.svg "Componentes do Loki")

## Armazenamento

O Loki armazena todos os dados em um único backend de armazenamento de objetos,
como Amazon Simple Storage Service (S3), Google Cloud Storage (GCS), Azure Blob
Storage, entre outros.
Esse modo utiliza um adaptador chamado **index shipper** (ou simplesmente
**shipper**) para armazenar arquivos de índice (TSDB ou BoltDB) da mesma forma
que armazena arquivos de chunks no armazenamento de objetos.
Esse modo de operação tornou-se disponível para uso geral com o Loki 2.0 e é
rápido, econômico e simples.
É nele que se concentra todo o desenvolvimento atual e futuro.

Antes da versão 2.0, o Loki utilizava backends de armazenamento distintos para
índices e chunks.
Para mais informações, consulte
[Armazenamento legado](../../operations/storage/legacy-storage/).

### Formato de dados

O Grafana Loki possui dois tipos principais de arquivos: **índice** e
**chunks**.

- O [**índice**](#formato-do-index) é um índice que indica onde encontrar logs
  para um conjunto específico de rótulos.
- O [**chunk**](#formato-do-chunk) é um contêiner para entradas de log
  referentes a um conjunto específico de rótulos.

![Formato de dados do Loki: chunks e índices](../chunks_diagram.png)

O diagrama acima apresenta uma visão geral dos dados armazenados no chunk e dos
dados armazenados no índice.

#### Formato do índice

Atualmente, há dois formatos de índice suportados para uso com armazenamento
único com o *index shipper*:

- [TSDB](../../operations/storage/tsdb/) (recomendado)

  O Time Series Database (ou TSDB) é um
  [formato de índice](https://github.com/prometheus/prometheus/blob/main/tsdb/docs/format/index.md)
  desenvolvido originalmente pelos mantenedores do
  [Prometheus](https://github.com/prometheus/prometheus) para dados (métricos)
  de séries temporais.

  Ele é extensível e oferece muitas vantagens em relação ao índice BoltDB
  obsoleto.
  Novos recursos de armazenamento no Loki estão disponíveis exclusivamente ao
  utilizar o TSDB.

- [BoltDB](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/boltdb-shipper/)
  (obsoleto)

  O [Bolt](https://github.com/boltdb/bolt) é um armazenamento de chave-valor
  transacional de baixo nível, escrito em Go.

#### Formato do chunk

Um chunk é um contêiner para linhas de log de um fluxo (um conjunto único de
rótulos) referente a um intervalo de tempo específico.

O diagrama ASCII a seguir descreve detalhadamente o formato do chunk.

```ascii
----------------------------------------------------------------------------
|                        |                       |                         |
|     MagicNumber(4b)    |     version(1b)       |      encoding (1b)      |
|                        |                       |                         |
----------------------------------------------------------------------------
|                      #structuredMetadata (uvarint)                       |
----------------------------------------------------------------------------
|      len(label-1) (uvarint)      |          label-1 (bytes)              |
----------------------------------------------------------------------------
|      len(label-2) (uvarint)      |          label-2 (bytes)              |
----------------------------------------------------------------------------
|      len(label-n) (uvarint)      |          label-n (bytes)              |
----------------------------------------------------------------------------
|                      checksum(from #structuredMetadata)                  |
----------------------------------------------------------------------------
|           block-1 bytes          |           checksum (4b)               |
----------------------------------------------------------------------------
|           block-2 bytes          |           checksum (4b)               |
----------------------------------------------------------------------------
|           block-n bytes          |           checksum (4b)               |
----------------------------------------------------------------------------
|                           #blocks (uvarint)                              |
----------------------------------------------------------------------------
| #entries(uvarint) | mint, maxt (varint)  | offset, len (uvarint)         |
----------------------------------------------------------------------------
| #entries(uvarint) | mint, maxt (varint)  | offset, len (uvarint)         |
----------------------------------------------------------------------------
| #entries(uvarint) | mint, maxt (varint)  | offset, len (uvarint)         |
----------------------------------------------------------------------------
| #entries(uvarint) | mint, maxt (varint)  | offset, len (uvarint)         |
----------------------------------------------------------------------------
|                          checksum(from #blocks)                          |
----------------------------------------------------------------------------
| #structuredMetadata len (uvarint) | #structuredMetadata offset (uvarint) |
----------------------------------------------------------------------------
|     #blocks len (uvarint)         |       #blocks offset (uvarint)       |
----------------------------------------------------------------------------
```

`mint` e `maxt` descrevem, respectivamente, os timestamp Unix mínimo e máximo em
nanossegundos.

A seção `structuredMetadata` armazena strings não repetidas.
Ela é utilizada para armazenar nomes e valores de rótulos provenientes de
[metadados estruturados](../labels/structured-metadata/).
Observe que as strings e os comprimentos dos rótulos dentro da seção
`structuredMetadata` são armazenados de forma compactada.

#### Formato do bloco

Um bloco é composto por uma série de entradas, sendo cada uma delas uma linha de
log individual.
Observe que os bytes de um bloco são armazenados de forma compactada.
A seguir, apresenta-se o formato deles quando descompactados:

```ascii
-----------------------------------------------------------------------------------------------------------------------------------------------
|  ts (varint)  |  len (uvarint)  |  log-1 bytes  |  len(from #symbols)  |  #symbols (uvarint)  |  symbol-1 (uvarint)  | symbol-n*2 (uvarint) |
-----------------------------------------------------------------------------------------------------------------------------------------------
|  ts (varint)  |  len (uvarint)  |  log-2 bytes  |  len(from #symbols)  |  #symbols (uvarint)  |  symbol-1 (uvarint)  | symbol-n*2 (uvarint) |
-----------------------------------------------------------------------------------------------------------------------------------------------
|  ts (varint)  |  len (uvarint)  |  log-3 bytes  |  len(from #symbols)  |  #symbols (uvarint)  |  symbol-1 (uvarint)  | symbol-n*2 (uvarint) |
-----------------------------------------------------------------------------------------------------------------------------------------------
|  ts (varint)  |  len (uvarint)  |  log-n bytes  |  len(from #symbols)  |  #symbols (uvarint)  |  symbol-1 (uvarint)  | symbol-n*2 (uvarint) |
-----------------------------------------------------------------------------------------------------------------------------------------------
```

`ts` é o timestamp Unix em nanossegundos dos logs, enquanto `len` é o tamanho em
bytes da entrada de log.

Símbolos armazenam referências às strings reais que contêm nomes e valores de
rótulos na seção `structuredMetadata` do chunk.

## Caminho de escrita

Em linhas gerais, o caminho de escrita no Loki funciona da seguinte forma:

1. O distributor recebe uma requisição HTTP POST contendo fluxos e linhas de
   log.
1. O distributor aplica uma função de hash a cada fluxo contido na requisição
   para determinar a instância do ingester para a qual ele deve ser enviado, com
   base nas informações do anel de hash consistente.
1. O distributor envia cada fluxo para o ingester apropriado e suas réplicas
   (com base no fator de replicação configurado).
1. O ingester recebe o fluxo com as linhas de log e cria um chunk ou anexa dados
   a um chunk existente para aquele fluxo.
   Um chunk é único por tenant e por conjunto de rótulos.
1. O ingester confirma a escrita.
1. O distributor aguarda que a maioria (quórum) dos ingesters confirme suas
   escritas.
1. O distributor responde com sucesso (código de status 2xx) caso tenha
   recebido confirmações de pelo menos um quórum de escritas, ou com um erro
   (código de status 4xx ou 5xx) caso as operações de escrita tenham falhado.

Consulte [Componentes](../components/) para uma descrição mais detalhada dos
componentes envolvidos no caminho de escrita.

## Caminho de leitura

Em linhas gerais, o caminho de leitura no Loki funciona da seguinte forma:

1. O query frontend (frontend de consultas) recebe uma requisição HTTP GET
   contendo uma consulta LogQL.
1. O query frontend divide a consulta em subconsultas e as encaminha para o
   query scheduler (agendador de consultas).
1. O querier (consultor) obtém as subconsultas do scheduler.
1. O querier encaminha a consulta a todos os ingesters para buscar dados em
   memória.
1. Os ingesters retornam dados em memória que correspondam à consulta, caso
   existam.
1. O querier carrega dados do backing store (armazenamento de apoio) sob demanda
   e executa a consulta sobre eles, caso os ingesters não tenham retornado dados
   ou tenham retornado dados insuficientes.
1. O querier processa todos os dados recebidos, realiza a desduplicação e
   retorna o resultado da subconsulta ao query frontend.
1. O query frontend aguarda a conclusão e o retorno de todas as subconsultas por
   parte dos queriers.
1. O query frontend combina os resultados individuais em um único resultado e o
   retorna ao cliente.

Consulte [Componentes](../components/) para uma descrição mais detalhada dos
componentes envolvidos no fluxo de leitura.

## Multi-tenancy

Todos os dados, tanto em memória quanto no armazenamento de longo prazo, podem
ser particionados por um ID de tenant, obtido a partir do cabeçalho HTTP
`X-Scope-OrgID` na requisição quando o Grafana Loki está em execução no modo
multi-tenant.
Quando o Loki **não** está no modo multi-tenant, o cabeçalho é ignorado e o ID
de tenant é definido como `fake`, o qual aparecerá no índice e nos chunks
armazenados.
