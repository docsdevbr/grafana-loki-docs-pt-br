---
# SPDX-FileCopyrightText: 2026 Grafana Labs.
# Grafana and the Grafana logo are trademarks owned by Raintank, Inc. dba
# Grafana Labs.
#
# SPDX-License-Identifier: AGPL-3.0-only
# Documentation licensed under the GNU Affero General Public License Version 3.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/grafana-loki-docs-pt-br/blob/-/LICENSES/AGPL-3.0-only.txt

source_url: https://github.com/grafana/loki/blob/v3.7.8/docs/sources/get-started/quick-start/quick-start.md
source_revision: 1fe5dbf2935a334c642bd6777c38fbbde6cbf85b
translation_status: ready

title: Início rápido para executar o Loki localmente
menuTitle: Início rápido do Loki
weight: 200
description: >-
  Como criar e usar um cluster Loki local para fins de teste e avaliação.
killercoda:
  comment: |
    The killercoda front matter and the HTML comments that start '<!-- INTERACTIVE ' are used by a transformation tool that converts this Markdown source into a Killercoda tutorial.

    You can find the tutorial in https://github.com/grafana/killercoda/tree/staging/loki/loki-quickstart.

    Changes to this source file affect the Killercoda tutorial.

    For more information about the transformation tool, refer to https://github.com/grafana/killercoda/blob/staging/docs/transformer.md.
  preprocessing:
    substitutions:
      - regexp: evaluate-loki-([^-]+)-
        replacement: evaluate-loki_${1}_
  title: Loki Quickstart Demo
  description: This sandbox provides an online enviroment for testing the Loki quickstart demo.
  details:
    intro:
      foreground: setup.sh
  backend:
    imageid: ubuntu
---

<!-- INTERACTIVE page intro.md START -->

# Guia rápido para executar o Loki localmente

Se você quiser experimentar o Loki, pode executá-lo localmente usando o arquivo
Docker Compose que acompanha o Loki.
Ele executa o Loki no modo de implantação
[simple scalable](https://grafana.com/docs/loki/<LOKI_VERSION>/get-started/deployment-modes/#simple-scalable)
e inclui uma aplicação de exemplo para gerar logs.
O modo simple scalable deployment (SSD) está obsoleto e sua remoção está
prevista para o Loki 4.0; para implantações em produção, consulte os
[modos de implantação](https://grafana.com/docs/loki/<LOKI_VERSION>/get-started/deployment-modes/)
para ver as opções recomendadas atualmente.

A configuração do Docker Compose executa os seguintes componentes, cada um em
seu próprio contêiner:

- **flog**: que gera linhas de log.
  O [flog](https://github.com/mingrammer/flog) é um gerador de logs para
  formatos de log comuns.

- **Grafana Alloy**: que coleta as linhas de log do flog e as envia para o Loki
  através do gateway.
- **Gateway** (NGINX): que recebe requisições e as redireciona para o contêiner
  apropriado com base na URL da requisição.
- **Componente de leitura do Loki**: que executa um *Query Frontend* e um
  *Querier*.
- **Componente de escrita do Loki**: que executa um *Distributor* e um
  *Ingester*.
- **Componente de backend do Loki**: que executa um *Index Gateway*,
  *Compactor*, *Ruler*, *Bloom Planner* (experimental), *Bloom Builder*
  (experimental) e *Bloom Gateway* (experimental).
- **Minio**: que o Loki utiliza para armazenar seu índice e *chunks*.
- **Grafana**: que fornece visualização das linhas de log capturadas no Loki.

{{< figure max-width="75%" src="/media/docs/loki/get-started-flog-v3.png" caption="Aplicação de exemplo para os primeiros passos" alt="Aplicação de exemplo para os primeiros passos" >}}

<!-- INTERACTIVE page intro.md END -->

<!-- INTERACTIVE ignore START -->

## Antes de começar

Antes de iniciar, você precisa ter o seguinte instalado em seu sistema local:
- Instale o [Docker](https://docs.docker.com/install).
- Instale o [Docker Compose](https://docs.docker.com/compose/install).

{{< admonition type="tip" >}}
Alternativamente, você pode experimentar este exemplo em nosso ambiente de
aprendizado interativo:
[Loki Quickstart Sandbox](https://killercoda.com/grafana-labs/course/loki/loki-quickstart).

É um ambiente totalmente configurado, com todas as dependências já instaladas.

![Interativo](/media/docs/loki/loki-ile.svg)

Envie feedback, relate erros e abra issues no
[repositório Grafana Killercoda](https://github.com/grafana/killercoda).
{{< /admonition >}}

<!-- INTERACTIVE ignore END -->

<!-- INTERACTIVE page step1.md START -->

## Instale o Loki e colete logs de exemplo

<!-- INTERACTIVE ignore START -->

{{< admonition type="note" >}}
Este guia de início rápido pressupõe que você esteja utilizando Linux.
{{< /admonition >}}

<!-- INTERACTIVE ignore END -->

**Para instalar o Loki localmente, siga estes passos:**

1. Crie um diretório chamado `evaluate-loki` para o ambiente de demonstração.
   Defina `evaluate-loki` como seu diretório de trabalho atual:

   ```bash
   mkdir evaluate-loki
   cd evaluate-loki
   ```

2. Baixe `loki-config.yaml`, `alloy-local-config.yaml` e `docker-compose.yaml`:

   <!-- INTERACTIVE ignore START -->
   {{< tabs >}}
   {{< tab-content name="wget" >}}
   ```bash
   wget https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/loki-config.yaml -O loki-config.yaml
   wget https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/alloy-local-config.yaml -O alloy-local-config.yaml
   wget https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/docker-compose.yaml -O docker-compose.yaml
   ```
   {{< /tab-content >}}
   {{< tab-content name="curl" >}}
   ```bash
   curl https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/loki-config.yaml --output loki-config.yaml
   curl https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/alloy-local-config.yaml --output alloy-local-config.yaml
   curl https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/docker-compose.yaml --output docker-compose.yaml
   ```
   {{< /tab-content >}}
   {{< /tabs >}}
   <!-- INTERACTIVE ignore END -->

   {{< docs/ignore >}}
   ```bash
   wget https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/loki-config.yaml -O loki-config.yaml
   wget https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/alloy-local-config.yaml -O alloy-local-config.yaml
   wget https://raw.githubusercontent.com/grafana/loki/main/examples/getting-started/docker-compose.yaml -O docker-compose.yaml
   ```
   {{< /docs/ignore >}}

3. Implante a imagem Docker de exemplo.

   Com `evaluate-loki` como o diretório de trabalho atual, inicie o ambiente de
   demonstração usando o `docker compose`:

   ```bash
   docker compose up -d
   ```

   Ao final do comando, você deverá ver algo semelhante ao seguinte:

   ```console
   ✔ Network evaluate-loki_loki          Created      0.1s
   ✔ Container evaluate-loki-minio-1     Started      0.6s
   ✔ Container evaluate-loki-flog-1      Started      0.6s
   ✔ Container evaluate-loki-backend-1   Started      0.8s
   ✔ Container evaluate-loki-write-1     Started      0.8s
   ✔ Container evaluate-loki-read-1      Started      0.8s
   ✔ Container evaluate-loki-gateway-1   Started      1.1s
   ✔ Container evaluate-loki-grafana-1   Started      1.4s
   ✔ Container evaluate-loki-alloy-1     Started      1.4s
   ```

4. (Opcional) Verifique se o cluster do Loki está em execução.

   - O componente de leitura retorna `ready` ao acessar
     [http://localhost:3101/ready](http://localhost:3101/ready).
     A mensagem `Query Frontend not ready: not ready: number of schedulers this
     worker is connected to is 0` é exibida até que o componente de leitura
     esteja pronto.
   - O componente de escrita retorna `ready` ao acessar
     [http://localhost:3102/ready](http://localhost:3102/ready).
     A mensagem `Ingester not ready: waiting for 15s after being ready` é
     exibida até que o componente de escrita esteja pronto.

5. (Opcional) Verifique se o Grafana Alloy está em execução.
   - Você pode acessar a interface do Grafana Alloy em
     [http://localhost:12345](http://localhost:12345).

6. (Opcional) Você pode verificar se todos os contêineres estão em execução
   executando o seguinte comando:

   ```bash
   docker ps -a
   ```

<!-- INTERACTIVE page step1.md END -->

<!-- INTERACTIVE page step2.md START -->

## Visualize seus logs no Grafana

Após coletar os logs, você provavelmente vai querer visualizá-los.
Você pode visualizar seus logs usando a interface de linha de comando,
[LogCLI](/docs/loki/<LOKI_VERSION>/query/logcli/), mas a maneira mais fácil de
visualizá-los é com o Grafana.

1. Use o Grafana para consultar a fonte de dados do Loki.

   O ambiente de teste inclui o
   [Grafana](https://grafana.com/docs/grafana/latest/), que você pode usar para
   consultar e observar os logs de exemplo gerados pela aplicação `flog`.

   Você pode acessar o Grafana acessando o endereço
   [http://localhost:3000](http://localhost:3000).

   A instância do Grafana nesta demonstração já possui uma
   [fonte de dados](https://grafana.com/docs/grafana/latest/datasources/loki/)
   do Loki configurada.

   {{< figure src="/media/docs/loki/grafana-query-builder-v2.png" caption="Grafana Explore" alt="Grafana Explore" >}}

2. No menu principal do Grafana, clique no ícone **Explore** (1) para abrir a
   aba Explore.

   Para saber mais sobre o Explore, consulte a documentação do
   [Explore](https://grafana.com/docs/grafana/latest/explore/).

3. No menu do cabeçalho do dashboard, selecione a fonte de dados Loki (2).

   Isso exibe o editor de consultas do Loki.

   No editor de consultas, você utiliza a linguagem de consulta do Loki, o
   [LogQL](https://grafana.com/docs/loki/<LOKI_VERSION>/query/), para consultar
   seus logs.
   Para saber mais sobre o editor de consultas, consulte a
   [documentação do editor de consultas](https://grafana.com/docs/grafana/latest/datasources/loki/query-editor/).

4. O editor de consultas do Loki possui dois modos (3):

   - [Modo Builder](https://grafana.com/docs/grafana/latest/datasources/loki/query-editor/#builder-mode),
     que oferece um designer visual de consultas.
   - [Modo Code](https://grafana.com/docs/grafana/latest/datasources/loki/query-editor/#code-mode),
     que oferece um editor repleto de recursos para escrever consultas LogQL.

   A seguir, veremos alguns exemplos de consultas simples utilizando as
   visualizações Builder e Code.

5. Clique em **Code** (3) para trabalhar no modo de código no editor de
   consultas.

   Aqui estão alguns exemplos de consultas para você começar a usar o LogQL.
   Essas consultas pressupõem que você seguiu as instruções para criar um
   diretório chamado `evaluate-loki`.

   Se você realizou a instalação em um diretório diferente, precisará modificar
   essas consultas para corresponder ao seu diretório de instalação.

   Após copiar qualquer uma dessas consultas para o editor, clique em **Run
   Query** (4) para executá-la.

   1. Visualize todas as linhas de log que possuem o rótulo de contêiner
      `evaluate-loki-flog-1`:

      <!-- INTERACTIVE copy START -->
      ```bash
      {container="evaluate-loki-flog-1"}
      ```
      <!-- INTERACTIVE copy END -->

      No Loki, isso é um fluxo de log.

      O Loki utiliza
      [rótulos](https://grafana.com/docs/loki/<LOKI_VERSION>/get-started/labels/)
      como metadados para descrever fluxos de log.

      As consultas do Loki sempre começam com um seletor de rótulos.
      Na consulta anterior, o seletor de rótulos é `{container="evaluate-loki-flog-1"}`.

   2. Para visualizar todas as linhas de log que possuem o rótulo de contêiner
      `evaluate-loki-grafana-1`:

      <!-- INTERACTIVE copy START -->
      ```bash
      {container="evaluate-loki-grafana-1"}
      ```
      <!-- INTERACTIVE copy END -->

   3. Encontre todas as linhas de log no fluxo
      `{container="evaluate-loki-flog-1"}` que contenham a string `status`:

      <!-- INTERACTIVE copy START -->
      ```bash
      {container="evaluate-loki-flog-1"} |= `status`
      ```
      <!-- INTERACTIVE copy END -->

   4. Encontre todas as linhas de log no fluxo
      `{container="evaluate-loki-flog-1"}` onde o campo JSON `status` tenha o
      valor `404`:

      <!-- INTERACTIVE copy START -->
      ```bash
      {container="evaluate-loki-flog-1"} | json | status=`404`
      ```
      <!-- INTERACTIVE copy END -->

   5. Calcule o número de logs por segundo em que o campo JSON `status` tem o
      valor `404`:

      <!-- INTERACTIVE copy START -->
      ```bash
      sum by(container) (rate({container="evaluate-loki-flog-1"} | json | status=`404` [$__auto]))
      ```
      <!-- INTERACTIVE copy END -->

   A consulta final é uma consulta de métrica que retorna uma série temporal.
   Isso faz com que o Grafana gere um gráfico dos resultados.

   Você pode alterar o tipo de gráfico para obter uma visualização diferente dos
   dados.
   Clique em **Bars** para visualizar um gráfico de barras dos dados.

6. Clique na aba **Builder** (3) para retornar ao modo de construção no editor
   de consultas.
   8. No modo de construção, clique em **Kick start your query** (5).
   9. Expanda a seção **Log query starters**.
   10. Selecione a primeira opção, **Parse log lines with logfmt parser**,
      clicando em **Use this query**.
   11. Na aba Explore, clique em **Label browser**; na caixa de diálogo,
      selecione um contêiner e clique em **Show logs**.

Para uma introdução detalhada ao LogQL, consulte a
[referência do LogQL](https://grafana.com/docs/loki/<LOKI_VERSION>/query/).

## Exemplos de consultas (visualização de código)

Aqui estão mais alguns exemplos de consultas que você pode executar usando os
dados de exemplo do Flog.

Para ver todas as linhas de log geradas pelo Flog, insira a consulta LogQL:

<!-- INTERACTIVE copy START -->
```bash
{container="evaluate-loki-flog-1"}
```
<!-- INTERACTIVE copy END -->

A aplicação Flog gera linhas de log para requisições HTTP simuladas.

Para ver todas as linhas de log do tipo `GET`, insira a consulta LogQL:

<!-- INTERACTIVE copy START -->
```bash
{container="evaluate-loki-flog-1"} |= "GET"
```
<!-- INTERACTIVE copy END -->

Para ver todos os métodos `POST`, insira a consulta LogQL:

<!-- INTERACTIVE copy START -->
```bash
{container="evaluate-loki-flog-1"} |= "POST"
```
<!-- INTERACTIVE copy END -->

Para ver todas as linhas de log com status 401 (erro de não autorizado), insira
a consulta LogQL:

<!-- INTERACTIVE copy START -->
```bash
{container="evaluate-loki-flog-1"} | json | status="401"
```
<!-- INTERACTIVE copy END -->

Para ver todas as linhas de log que não contêm o texto `401`:

<!-- INTERACTIVE copy START -->
```bash
{container="evaluate-loki-flog-1"} != "401"
```
<!-- INTERACTIVE copy END -->

Para mais exemplos, consulte a
[documentação de consultas](https://grafana.com/docs/loki/<LOKI_VERSION>/query/query_examples/).

## Fonte de dados Loki no Grafana

Neste exemplo, a fonte de dados Loki já está configurada no Grafana.
Isso pode ser verificado no arquivo `docker-compose.yaml`:

```yaml
  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    depends_on:
      - gateway
    entrypoint:
      - sh
      - -euc
      - |
        mkdir -p /etc/grafana/provisioning/datasources
        cat <<EOF > /etc/grafana/provisioning/datasources/ds.yaml
        apiVersion: 1
        datasources:
          - name: Loki
            type: loki
            access: proxy
            url: http://gateway:3100
            jsonData:
              httpHeaderName1: "X-Scope-OrgID"
            secureJsonData:
              httpHeaderValue1: "tenant1"
        EOF
        /run.sh
```

Na seção `entrypoint`, a fonte de dados Loki é configurada com os seguintes
detalhes:

- `Name: Loki` (nome da fonte de dados).
- `Type: loki` (tipo de fonte de dados).
- `Access: proxy` (tipo de acesso).
- `URL: http://gateway:3100` (URL da fonte de dados Loki; o Loki utiliza um
  gateway Nginx para direcionar o tráfego ao componente apropriado).
- `jsonData.httpHeaderName1: "X-Scope-OrgID"` (nome do cabeçalho para o ID da
  organização).
- `secureJsonData.httpHeaderValue1: "tenant1"` (valor do cabeçalho para o ID da
  organização).

É importante observar que, quando o Loki é configurado em qualquer modo que não
seja a implantação monolítica, é necessário fornecer um ID de tenant no
cabeçalho.
Caso contrário, as consultas retornarão um erro de autorização.

<!-- INTERACTIVE page step2.md END -->

<!-- INTERACTIVE page finish.md START -->

## Exemplo completo de métricas, logs, rastros e profiling

Você concluiu a demonstração de início rápido do Loki.
E agora, qual o próximo passo?

{{< docs/ignore >}}

## Volte à documentação

Retorne ao ponto de partida para continuar com a documentação do Loki:
[Documentação do Loki](https://grafana.com/docs/loki/latest/get-started/quick-start/).

{{< /docs/ignore >}}

Se você quiser executar um ambiente de demonstração que inclua Mimir, Loki,
Tempo e Grafana, pode utilizar o projeto
[Introduction to Metrics, Logs, Traces, and Profiling in Grafana](https://github.com/grafana/intro-to-mlt).
Trata-se de um ambiente autossuficiente para aprender sobre Mimir, Loki, Tempo e
Grafana.

O projeto inclui explicações detalhadas de cada componente e configurações
comentadas para uma implantação de instância única.
Você também pode enviar os dados desse ambiente para o
[Grafana Cloud](https://grafana.com/cloud/).

<!-- INTERACTIVE page finish.md END -->
