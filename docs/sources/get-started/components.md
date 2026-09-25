---
# SPDX-FileCopyrightText: 2026 Grafana Labs.
# Grafana and the Grafana logo are trademarks owned by Raintank, Inc. dba
# Grafana Labs.
#
# SPDX-License-Identifier: AGPL-3.0-only
# Documentation licensed under the GNU Affero General Public License Version 3.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/grafana-loki-docs-pt-br/blob/-/LICENSES/AGPL-3.0-only.txt

source_url: https://github.com/grafana/loki/blob/v3.7.8/docs/sources/get-started/components.md
source_revision: 624a93dfa7ae0af059ef96bdd19f2334d8366368
translation_status: ready

title: Componentes do Loki
menuTitle: Componentes
description: Descreve os vários componentes que compõem o Grafana Loki.
weight: 500
aliases:
  - ../fundamentals/architecture/components
---

# Componentes do Loki

{{< youtube id="_hv4i84Z68s" >}}

O Loki é um sistema modular composto por vários componentes que podem ser
executados em conjunto (no modo "binário único", com o target `all`), em grupos
lógicos (no modo "implantação simples e escalável", com os targets `read`,
`write`, `backend`) ou individualmente (no modo "microsserviço").
Para mais informações, consulte [Modos de implantação](https://grafana.com/docs/loki/<LOKI_VERSION>/get-started/deployment-modes/).

| Componente                                     | _individual_ | `all` | `read` | `write` | `backend` |
|------------------------------------------------|--------------| - | - | - | - |
| [Distributor](#distributor)                   | x            | x |   | x |   |
| [Ingester](#ingester)                          | x            | x |   | x |   |
| [Query Frontend](#query-frontend)              | x            | x | x |   |   |
| [Query Scheduler](#query-scheduler)            | x            | x |   |   | x |
| [Querier](#querier)                            | x            | x | x |   |   |
| [Index Gateway](#index-gateway)                | x            |   |   |   | x |
| [Compactor](#compactor)                        | x            | x |   |   | x |
| [Ruler](#ruler)                                | x            | x |   |   | x |
| [Pattern ingester](#pattern-ingester)          | x            | x |   | x |   |
| [Bloom Planner (Experimental)](#bloom-planner) | x            |   |   |   | x |
| [Bloom Builder (Experimental)](#bloom-builder) | x            |   |   |   | x |
| [Bloom Gateway (Experimental)](#bloom-gateway) | x            |   |   |   | x |

Esta página descreve as responsabilidades de cada um desses componentes.

## Distributor

O serviço **distributor** é responsável por processar as requisições de push
recebidas de clientes.
Ele representa a primeira etapa no fluxo de gravação de dados de log.
Assim que o distributor recebe um conjunto de fluxos em uma requisição HTTP,
cada fluxo é validado quanto à sua integridade e para garantir que esteja dentro
dos limites configurados para o tenant (ou limites globais).
Cada fluxo válido é então enviado em paralelo para `n` [ingesters](#ingester),
onde `n` é o [fator de replicação](#fator-de-replicação) dos dados.
O distributor determina para quais ingesters um fluxo será enviado utilizando
[hashing consistente](#hashing).

Um balanceador de carga deve ser posicionado à frente do distributor para
equilibrar adequadamente o tráfego de entrada.
No Kubernetes, o balanceador de carga do serviço desempenha essa função.

O distributor é um componente stateless (sem estado).
Isso facilita o escalonamento e permite transferir a maior carga de trabalho
possível dos ingesters, que são o componente mais crítico no fluxo de gravação.
A capacidade de escalar essas operações de validação de forma independente
significa que o Loki também pode se proteger contra ataques de negação de
serviço que, de outra forma, poderiam sobrecarregar os ingesters.
Isso também nos permite realizar a distribuição das gravações conforme o
[fator de replicação](#fator-de-replicação).

### Validação

A primeira etapa realizada pelo distributor é garantir que todos os dados
recebidos estejam conforme as especificações.
Isso inclui verificar se os rótulos são rótulos Prometheus válidos, bem como
assegurar que os timestamps não sejam excessivamente antigos ou recentes e que
as linhas de log não sejam excessivamente longas.

### Pré-processamento

Atualmente, a única forma de o distributor modificar os dados recebidos é por
meio da normalização dos rótulos.
Isso significa tornar `{foo="bar", bazz="buzz"}` equivalente a
`{bazz="buzz", foo="bar"}`, ou seja, ordenar os rótulos.
Isso permite que o Loki realize o cache e a geração de hash de maneira
determinística.

### Limitação de taxa

O distributor também pode limitar a taxa de logs recebidos com base na taxa
máxima de ingestão de dados por tenant.
Ele faz isso verificando o limite definido para cada tenant e dividindo-o pelo
número atual de distributors.
Isso permite definir o limite de taxa por tenant em nível de cluster e
possibilita aumentar ou diminuir o número de distributors, ajustando
automaticamente o limite individual de cada distributor.
Por exemplo, suponha que tenhamos 10 distributors e o tenant A tenha um limite
de taxa de 10 MB.
Cada distributor permitirá até 1 MB/s antes de aplicar a limitação.
Agora, suponha que outro tenant grande entre no cluster e precisemos iniciar
mais 10 distributors.
Os 20 distributors ajustarão seus limites de taxa para o tenant A para
`(10 MB / 20 distributors) = 500 KB/s`.
É assim que os limites globais permitem uma operação muito mais simples e segura
do cluster Loki.

{{< admonition type="note" >}}
Internamente, o distributor utiliza o componente `ring` para registrar-se entre
seus pares e obter o número total de distributors ativos.
Essa "chave" é diferente daquela utilizada pelos ingesters no anel e provém da
própria
[configuração de anel](https://grafana.com/docs/loki/<LOKI_VERSION>/configure/#distributor)
do distributor.
{{< /admonition >}}

### Encaminhamento

Assim que o distributor conclui todas as suas tarefas de validação, ele
encaminha os dados para o componente ingester, que é o responsável final por
confirmar a operação de escrita.

#### Fator de replicação

Para mitigar a chance de _perda_ de dados em um único ingester, o distributor
encaminha as escritas para um número de ingesters definido pelo _fator de
replicação_.
Geralmente, o fator de replicação é `3`.
A replicação permite reinicializações e rollouts dos ingesters sem causar falhas
nas escritas e adiciona proteção extra contra perda de dados em alguns cenários.
De forma simplificada: para cada conjunto de rótulos (chamado de _fluxo_)
enviado a um distributor, ele gera um hash dos rótulos e usa o valor resultante
para localizar `replication_factor` ingesters no anel (um subcomponente que
expõe uma
[tabela de hash distribuído](https://en.wikipedia.org/wiki/Distributed_hash_table)).
Em seguida, ele tenta gravar os mesmos dados neles todos.
Isso gera um erro se menos do que um _quórum_ de escritas for bem-sucedido.
O quórum é definido como `floor( replication_factor / 2 ) + 1`.
Portanto, para nosso fator de replicação de `3`, exigimos que duas escritas
sejam bem-sucedidas.
Se menos de duas escritas tiverem sucesso, o distributor retorna um erro e a
operação de escrita é tentada novamente.

{{< admonition type="caution" >}}
Se uma escrita for confirmada por 2 de 3 ingesters, podemos tolerar a perda de
um ingester, mas não de dois, pois isso resultaria em perda de dados.
{{< /admonition >}}

No entanto, o fator de replicação não é o único mecanismo que previne a perda de
dados; seu objetivo principal é permitir que as escritas continuem
ininterruptamente durante rollouts e reinicializações.
O [componente ingester](#ingester) agora inclui um
[log de pré-gravação](https://en.wikipedia.org/wiki/Write-ahead_logging) (WAL),
que persiste as escritas recebidas em disco para garantir que não sejam
perdidas, desde que o disco não seja corrompido.
A natureza complementar do fator de replicação e do WAL garante que os dados não
sejam perdidos, a menos que ocorram falhas significativas em ambos os mecanismos
(ou seja, múltiplos ingesters falham e perdem ou corrompem seus discos).

### Hashing

Os distributors utilizam hashing consistente em conjunto com um fator de
replicação configurável para determinar quais instâncias do serviço ingester
devem receber um determinado fluxo.

Um fluxo é um conjunto de logs associado a um tenant e a um conjunto único de
rótulos.
O fluxo passa por uma função de hash utilizando tanto o ID do tenant quanto o
conjunto de rótulos; em seguida, o hash resultante é usado para localizar os
ingesters para os quais o fluxo será enviado.

Um anel de hash, mantido por meio de comunicação peer-to-peer utilizando o
protocolo [Memberlist](https://github.com/hashicorp/memberlist) ou armazenado em
um sistema de armazenamento chave-valor como o [Consul](https://www.consul.io),
é utilizado para implementar o hashing consistente; todos os ingesters
registram-se no anel de hash com um conjunto de tokens que lhes pertencem.
Cada token é um número aleatório de 32 bits sem sinal.
Juntamente com o conjunto de tokens, os ingesters registram seu estado no anel
de hash.
Ingesters nos estados `JOINING` e `ACTIVE` podem receber requisições de escrita,
enquanto ingesters nos estados `ACTIVE` e `LEAVING` podem receber requisições de
leitura.
Ao realizar a busca no hash, os distributors consideram apenas os tokens de
ingesters que estejam em um estado apropriado para a requisição.

Para realizar a busca do hash, os distributors encontram o menor token
apropriado cujo valor seja maior que o hash do fluxo.
Quando o fator de replicação é maior que 1, os tokens subsequentes (no sentido
horário do anel) que pertencem a ingesters diferentes também são incluídos no
resultado.

O efeito dessa configuração de hash é que cada token pertencente a um ingester é
responsável por uma faixa de valores de hash.
Se houver três tokens com valores 0, 25 e 50, um hash de valor 3 seria atribuído
ao ingester que possui o token 25; o ingester proprietário do token 25 é
responsável pela faixa de hash de 1 a 25.

### Consistência de quórum

Como todos os distributors compartilham o acesso ao mesmo anel de hash, as
requisições de escrita podem ser enviadas a qualquer distributor.

Para garantir resultados de consulta consistentes, o Loki utiliza consistência
de quórum no estilo
[Dynamo](https://www.cs.princeton.edu/courses/archive/fall15/cos518/studpres/dynamo.pdf)
para operações de leitura e escrita.
Isso significa que o distributor aguardará uma resposta positiva de pelo menos
metade mais um dos ingesters para os quais a amostra foi enviada, antes de
responder ao cliente que iniciou o envio.

## Ingester

O serviço ingester é responsável por persistir dados e enviá-los para
armazenamento de longo prazo (Amazon Simple Storage Service, Google Cloud
Storage, Azure Blob Storage, etc.) no caminho de escrita, e por retornar dados
de log recém-ingeridos mantidos em memória para consultas no caminho de leitura.

Os ingesters contêm um _lifecycler_ que gerencia o ciclo de vida do ingester no
anel de hash.
Cada ingester possui um estado que pode ser `PENDING`, `JOINING`, `ACTIVE`,
`LEAVING` ou `UNHEALTHY`:

1. `PENDING` é o estado de um ingester quando ele aguarda um [handoff](#handoff)
   (transferência de dados) de outro ingester que está no estado `LEAVING`.
   Isso se aplica apenas a modos de implantação legados.

   {{< admonition type="note" >}}
   O handoff é um comportamento obsoleto, utilizado principalmente em
   implantações stateless de ingesters, prática que não é recomendada.
   Em vez disso, recomenda-se utilizar um modelo de implantação stateful (com
   estado) em conjunto com o
   [write ahead log](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/wal/).
   {{< /admonition >}}

1. `JOINING` é o estado de um ingester quando ele está inserindo seus tokens no
   anel e inicializando a si mesmo.
   Ele pode receber requisições de escrita para tokens que possui.

1. `ACTIVE` é o estado de um ingester quando ele está totalmente inicializado.
   Ele pode receber requisições tanto de escrita quanto de leitura para tokens
   que possui.

1. `LEAVING` é o estado de um ingester quando ele está sendo encerrado.
   Ele pode receber requisições de leitura para dados que ainda mantém na
   memória.

1. `UNHEALTHY` é o estado de um ingester quando ele falha ao enviar o sinal de
   heartbeat.
   O estado `UNHEALTHY` é definido pelo distributor quando este verifica
   periodicamente o anel.

Cada fluxo de logs recebido por um ingester é organizado em memória como um
conjunto de vários "chunks" e gravado no backing storage (armazenamento de
apoio) em intervalos configuráveis.

Os chunks são compactados e marcados como somente leitura quando:

1. O chunk atual atinge sua capacidade máxima (um valor configurável).
1. Passa-se muito tempo sem que o chunk atual seja atualizado.
1. Ocorre um flush (operação de gravação).

Sempre que um chunk é compactado e marcado como somente leitura, um chunk que
permite escrita assume o seu lugar.

Se um processo ingester falhar ou encerrar abruptamente, todos os dados que
ainda não tiverem sido gravados no armazenamento serão perdidos.
O Loki é geralmente configurado para manter múltiplas réplicas (normalmente 3)
de cada log, a fim de mitigar esse risco.

Quando ocorre o flush em um provedor de armazenamento persistente, o chunk passa
por um processo de hashing baseado em seu tenant, rótulos e conteúdo.
Isso significa que múltiplos ingesters que possuam a mesma cópia de dados não
gravarão os mesmos dados duas vezes no backing store; no entanto, se a gravação
falhar em alguma das réplicas, múltiplos objetos de chunk distintos serão
criados no armazenamento.
Consulte a seção [Querier](#querier) para saber como os dados são desduplicados.

### Ordenação por Timestamp

O Loki é configurado por padrão para
[aceitar gravações fora de ordem](https://grafana.com/docs/loki/<LOKI_VERSION>/configure/#accept-out-of-order-writes).

Quando não está configurado para aceitar gravações fora de ordem, o ingester
valida se as linhas de log ingeridas estão ordenadas.
Quando um ingester recebe uma linha de log que não segue a ordem esperada, a
linha é rejeitada e um erro é retornado à pessoa usuária.

O ingester valida se as linhas de log são recebidas em ordem crescente de
timestamp.
Cada log possui um timestamp posterior ao do log anterior.
Quando o ingester recebe um log que não segue essa ordem, a linha de log é
rejeitada e um erro é retornado.

Os logs de cada conjunto único de rótulos são agrupados em "chunks" na memória
e, em seguida, gravados no backend do armazenamento de apoio.

Se um processo ingester falhar ou encerrar abruptamente, todos os dados que
ainda não tiveram a gravação concluída podem ser perdidos.
O Loki é geralmente configurado com um
[Write Ahead Log (WAL)](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/wal/)
que pode ser _reprocessado_ na reinicialização e com um `replication_factor`
(geralmente 3) para cada log, a fim de mitigar esse risco.

Quando não configurado para aceitar gravações fora de ordem, todas as linhas
enviadas ao Loki para um determinado fluxo (combinação única de rótulos) devem
ter um timestamp mais recente do que a linha recebida anteriormente.
Existem, no entanto, dois casos para o tratamento de logs do mesmo fluxo com
timestamps idênticos em nível de nanossegundo:

1. Se a linha recebida corresponder exatamente à linha recebida anteriormente
   (coincidindo tanto no timestamp quanto no texto do log), a linha recebida
   será tratada como uma duplicata exata e ignorada.

1. Se a linha recebida tiver o mesmo timestamp da linha anterior, mas conteúdo
   diferente, a linha de log será aceita.
   Isso significa que é possível ter duas linhas de log diferentes para o mesmo
   timestamp.

### Handoff (transferência de responsabilidade)

{{< admonition type="warning" >}}
O handoff é um comportamento obsoleto, utilizado principalmente em implantações
stateless de ingesters, prática que não é recomendada.
Em vez disso, recomenda-se utilizar um modelo de implantação stateful em
conjunto com o
[write-ahead log](https://grafana.com/docs/loki/latest<LOKI_VERSION>/operations/storage/wal/).
{{< /admonition >}}

Por padrão, quando um ingester está sendo encerrado e tenta sair do anel de
hash, ele aguarda para verificar se um novo ingester tenta ingressar antes de
realizar o flush e tenta iniciar um handoff.
O handoff transfere todos os tokens e chunks em memória pertencentes ao ingester
que está saindo para o novo ingester.

Antes de ingressar no anel de hash, os ingesters aguardam no estado `PENDING`
que ocorra um handoff.
Após um tempo limite configurável, os ingesters no estado `PENDING` que não
receberam uma transferência ingressam no anel normalmente, inserindo um novo
conjunto de tokens.

Esse processo é utilizado para evitar o flush de todos os chunks durante o
encerramento, o que é um processo lento.

### Suporte ao sistema de arquivos

Os ingesters podem gravar chunks no sistema de arquivos quando o Loki utiliza o
TSDB como armazenamento de índice.
O TSDB envia seus arquivos de índice para o armazenamento de objetos, incluindo
o armazenamento de objetos baseado em sistema de arquivos, permitindo que os
[queriers](#querier) (em execução em um processo separado) leiam os mesmos
dados, desde que todos os processos tenham acesso ao mesmo diretório.
Isso difere do armazenamento de índice BoltDB (já descontinuado e que não
utilizava esse mecanismo de envio), o qual permitia que apenas um processo
mantivesse um bloqueio no arquivo do banco de dados por vez.

## Query frontend

O query frontend (frontend de consultas) é um **serviço opcional** que
disponibiliza os endpoints de API do querier e pode ser utilizado para acelerar
o fluxo de leitura.
Quando o query frontend está em uso, as requisições de consulta recebidas devem
ser direcionadas a ele, em vez de aos queriers.
O serviço de querier continua sendo necessário no cluster para a execução
efetiva das consultas.

Internamente, o query frontend realiza ajustes nas consultas e as mantém em uma
fila interna.
Nessa configuração, os queriers atuam como workers que buscam tarefas na fila,
executam-nas e as devolvem ao query frontend para agregação.
É necessário configurar os queriers com o endereço do query frontend (por meio
da flag de CLI `-querier.frontend-address`) para permitir a conexão entre eles.

Os query frontends são **stateless**.
No entanto, devido ao funcionamento da fila interna, recomenda-se executar
algumas réplicas do query frontend para aproveitar os benefícios de um
escalonamento justo.
Geralmente, duas réplicas devem ser suficientes.

### Enfileiramento

Caso não seja utilizado um componente [query scheduler](#query-scheduler)
separado, o query frontend também realizará o enfileiramento básico de
consultas.

- Garante que consultas grandes, capazes de provocar um erro de falta de memória
  (OOM) no querier, sejam reexecutadas em caso de falha.
  Isso permite às pessoas administradoras subdimensionar a memória destinada às
  consultas ou executar, de forma otimista, mais consultas pequenas em paralelo,
  o que ajuda a reduzir o custo total de propriedade (TCO).
- Evita que múltiplas requisições grandes fiquem acumuladas em um único querier,
  distribuindo-as entre todos os queriers por meio de uma fila do tipo FIFO
  (primeiro a entrar, primeiro a sair).
- Impede que um único tenant cause uma negação de serviço (DoS) para outros
  tenants, realizando o agendamento equitativo de consultas entre eles.

### Splitting

O frontend de consultas divide consultas maiores em várias consultas menores,
executando-as em paralelo nos queriers subsequentes e reunindo os resultados
posteriormente.
Isso evita que consultas extensas (que abrangem vários dias, por exemplo) causem
problemas de falta de memória em um único querier e ajuda a acelerar sua
execução.

### Cache

#### Consultas de métricas

O query frontend oferece suporte ao cache de resultados de consultas de métricas
e os reutiliza em consultas subsequentes.
Se os resultados em cache estiverem incompletos, o query frontend calcula as
subconsultas necessárias e as executa em paralelo nos queriers subsequentes.
O query frontend pode, opcionalmente, alinhar as consultas ao seu parâmetro de
step para melhorar a capacidade de cache dos resultados.
O cache de resultados é compatível com qualquer backend de cache do Loki
(atualmente Memcached, Redis e cache em memória).

#### Consultas de logs

O query frontend também oferece suporte ao cache de consultas de logs na forma
de um cache negativo.
Isso significa que, em vez de armazenar em cache os resultados de logs para
intervalos de tempo quantizados, o Loki armazena apenas resultados vazios para
esses intervalos.
Essa abordagem é mais eficiente do que armazenar os resultados reais, pois as
consultas de logs são limitadas (geralmente a 1000 resultados); assim, se você
tiver uma consulta abrangendo um longo intervalo de tempo que corresponda a
apenas algumas linhas, e armazenasse apenas os resultados reais, ainda
precisaria processar uma grande quantidade de dados (além daqueles vindos do
cache de resultados) para verificar se não há outras correspondências.

#### Consultas de estatísticas de índice

O query frontend armazena em cache os resultados de consultas de estatísticas de
índice de forma semelhante aos resultados de
[consultas de métricas](#consultas-de-métricas).
Esse cache é aplicável apenas ao utilizar o TSDB de armazenamento único.

#### Consultas de volume de logs

O query frontend armazena em cache os resultados de consultas de volume de logs
de forma semelhante aos resultados de
[consultas de métricas](#consultas-de-métricas).
Esse cache é aplicável apenas ao utilizar o TSDB de armazenamento único.

## Query scheduler

O **query scheduler** é um **serviço opcional** que oferece
[funcionalidades de enfileiramento mais avançadas](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/query-fairness/)
do que o [query frontend](#query-frontend).
Ao utilizar esse componente na implantação do Loki, o query frontend envia
consultas fragmentadas para o query scheduler, que as coloca em uma fila interna
na memória.
Existe uma fila para cada tenant para garantir a equidade na execução de
consultas entre todos eles.
Os queriers que se conectam ao query scheduler atuam como workers que buscam
suas tarefas na fila, executam-nas e as devolvem ao query frontend para
agregação.
Portanto, os queriers precisam ser configurados com o endereço do query
scheduler (via flag de CLI `-querier.scheduler-address` ou pelo campo
`scheduler_address` no bloco YAML
[`frontend_worker`](https://grafana.com/docs/loki/<LOKI_VERSION>/configure/#frontend_worker))
para permitir a conexão.
Como alternativa a um endereço estático, os agendadores de consultas podem se
registrar em um anel de hash, permitindo que queriers e query frontends os
descubram automaticamente.
Para mais informações, consulte
[Escale o Loki](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/scalability/#scheduler-discovery-using-a-ring).

Os query schedulers são **stateless**.
No entanto, devido à fila em memória, recomenda-se executar mais de uma réplica
para manter o benefício da alta disponibilidade.
Geralmente, duas réplicas devem ser suficientes.

## Querier

O serviço **querier** é responsável por executar consultas na linguagem
[Log Query Language (LogQL)](https://grafana.com/docs/loki/<LOKI_VERSION>/query/).
O querier pode processar requisições HTTP diretamente do cliente (no modo
"binário único" ou como parte do caminho de leitura na "implantação simples e
escalável") ou buscar subconsultas do query frontend ou do query scheduler (no
modo "microsserviços").

Ele recupera dados de log tanto dos ingesters quanto do armazenamento de longo
prazo.
Os queriers consultam todos os ingesters em busca de dados armazenados na
memória antes de recorrerem à execução da mesma consulta no armazenamento de
backend.
Devido ao fator de replicação, é possível que o querier receba dados duplicados.
Para resolver isso, o querier realiza internamente a **desduplicação** de dados
que possuam o mesmo timestamp em nanossegundos, conjunto de rótulos e mensagem
de log.

## Index Gateway

O serviço **index gateway** é responsável por processar e atender a consultas de
metadados.
Consultas de metadados são consultas que buscam dados no índice.
O index gateway é utilizado apenas por "shipper stores", como o
[TSDB de instância única](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/tsdb/)
ou o
[BoltDB de instância única](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/boltdb-shipper/).

O query frontend consulta o index gateway para obter o volume de logs das
consultas, permitindo decidir como realizar o particionamento das mesmas.
Os queriers consultam o index gateway em busca de referências de chunks para uma
determinada consulta, a fim de saber quais chunks devem ser buscados e
processados.

O index gateway pode operar nos modos `simple` ou `ring`.
No modo `simple`, cada instância do index gateway atende a todos os índices de
todos os tenants.
No modo `ring`, os index gateways utilizam um anel de hash consistente para
distribuir e particionar os índices por tenant entre as instâncias disponíveis.

## Compactor

O serviço **compactor** é utilizado por "shipper stores", como o
[TSDB de instância única](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/tsdb/)
ou o
[BoltDB de instância única](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/boltdb-shipper/)
para compactar os múltiplos arquivos de índice produzidos pelos ingesters e
enviados para o armazenamento de objetos, transformando-os em arquivos de índice
únicos por dia e por tenant.
Isso torna as consultas aos índices mais eficientes.

Para isso, o compactor baixa os arquivos do armazenamento de objetos em
intervalos regulares, mescla-os em um único arquivo, faz o upload do índice
recém-criado e remove os arquivos antigos.

Além disso, o compactor também é responsável pela
[retenção de logs](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/retention/)
e pela
[exclusão de logs](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/logs-deletion/).

Em uma implantação do Loki, o serviço compactor é geralmente executado como uma
instância única.

## Ruler

O serviço **ruler** gerencia e avalia expressões de regras e/ou alertas
fornecidas em uma configuração de regras.
A configuração de regras é armazenada em armazenamento de objetos (ou,
alternativamente, no sistema de arquivos local) e pode ser gerenciada via API do
ruler ou diretamente pelo upload dos arquivos para o armazenamento de objetos.

Alternativamente, o ruler também pode delegar a avaliação de regras para o query
frontend.
Esse modo é chamado de avaliação remota de regras e é utilizado para obter as
vantagens de divisão de consultas, fragmentação de consultas e cache oferecidas
pelo query frontend.

Ao executar múltiplos rulers, eles utilizam um anel de hash consistente para
distribuir grupos de regras entre as instâncias de ruler disponíveis.

## Pattern ingester

O componente opcional **pattern ingester** recebe dados de log dos ingesters e
analisa os logs para detectar e agregar padrões.
Isso pode ser útil para entender a estrutura dos seus logs em larga escala.
O pattern ingester é utilizado pelo recurso de padrões no Logs Drilldown, que
permite detectar linhas de log semelhantes e incluí-las ou excluí-las da sua
pesquisa.

O ingester utiliza um algoritmo drain para identificar logs relacionados que
compartilham o mesmo padrão e manter suas contagens ao longo do tempo.
Os padrões consistem em um número, uma string e um identificador de série do
Loki.

O pattern ingester expõe uma API de consulta para que você possa recuperar os
padrões detectados.
Essa API é utilizada pela aba Patterns no plugin Grafana Logs Drilldown.

Este componente está desabilitado por padrão e deve ser habilitado no seu
[arquivo de configuração do Loki](https://grafana.com/docs/loki/latest/configure/#supported-contents-and-default-values-of-lokiyaml).

## Bloom Planner

{{< admonition type="warning" >}}
Este recurso é um
[recurso experimental](https://grafana.com/docs/release-life-cycle/).
Não há suporte de engenharia ou de plantão disponível.
Não é fornecido SLA.
{{< /admonition >}}

O serviço Bloom Planner é responsável por planejar as tarefas para a criação de
blooms.
Ele é executado como uma instância única e fornece uma fila da qual as tarefas
são retiradas pelos Bloom Builders.
O planejamento é executado periodicamente e leva em consideração quais blooms já
foram criados para um determinado dia e tenant, bem como quais séries precisam
ser adicionadas.

Este serviço também é utilizado para aplicar a retenção de blooms.

## Bloom Builder

{{< admonition type="warning" >}}
Este recurso é um
[recurso experimental](https://grafana.com/docs/release-life-cycle/).
Não há suporte de engenharia ou de plantão disponível.
Não é fornecido SLA.
{{< /admonition >}}

O serviço Bloom Builder é responsável por processar as tarefas criadas pelo
Bloom Planner.
O Bloom Builder cria blocos de bloom a partir de metadados estruturados de
entradas de log.
Os blooms resultantes são agrupados em blocos de bloom que abrangem múltiplas
séries e chunks de um determinado dia.
Este componente também gera arquivos de metadados para rastrear quais blocos
estão disponíveis para cada série e arquivo de índice do TSDB.

O serviço é stateless e escalável horizontalmente.

## Bloom Gateway

{{< admonition type="warning" >}}
Este recurso é um
[recurso experimental](https://grafana.com/docs/release-life-cycle/).
Não há suporte de engenharia ou de plantão disponível.
Não é fornecido SLA.
{{< /admonition >}}

O serviço Bloom Gateway é responsável por lidar com e atender a requisições de
filtragem de chunks.
O index gateway consulta o Bloom Gateway ao calcular referências de chunks ou ao
calcular fragmentos para uma determinada consulta.
O serviço de gateway recebe uma lista de chunks e uma expressão de filtragem e
os compara com os blooms, filtrando e descartando quaisquer chunks que não
correspondam à expressão de filtro de rótulos fornecida.

O serviço é escalável horizontalmente.
Ao executar múltiplas instâncias, o cliente (Index Gateway) distribui as
requisições entre as instâncias com base no hash dos blocos de bloom
referenciados.
