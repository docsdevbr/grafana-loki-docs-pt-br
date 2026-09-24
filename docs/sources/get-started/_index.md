---
# SPDX-FileCopyrightText: 2026 Grafana Labs.
# Grafana and the Grafana logo are trademarks owned by Raintank, Inc. dba
# Grafana Labs.
#
# SPDX-License-Identifier: AGPL-3.0-only
# Documentation licensed under the GNU Affero General Public License Version 3.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/grafana-loki-docs-pt-br/blob/-/LICENSES/AGPL-3.0-only.txt

source_url: https://github.com/grafana/loki/blob/v3.7.8/docs/sources/get-started/_index.md
revision: 94a416ec7ef6492ef7f4b82ec2ef81ea0eb64b58
status: ready

title: Primeiros passos com o Grafana Loki
menuTitle: Primeiros passos
weight: 200
description: >-
  Fornece uma visão geral das etapas para implementar o Grafana Loki para
  coletar e visualizar logs.
aliases:
  - /getting-started/
---

# Primeiros passos com o Grafana Loki

{{< youtube id="1uk8LtQqsZQ" >}}

O Loki é um sistema de agregação de logs multi-tenant, de alta disponibilidade e
escalável horizontalmente, inspirado no Prometheus.
Ele foi projetado para ser econômico e fácil de operar.
Ele não indexa o conteúdo dos logs, mas sim um conjunto de rótulos para cada
fluxo de log.
Vale ressaltar que todo o conteúdo da linha de log pode ser pesquisado; o uso de
rótulos apenas torna a pesquisa mais eficiente, reduzindo a quantidade de logs
recuperados durante a consulta.

Como cada implementação do Loki é única, o processo de instalação varia para
cada cliente.
No entanto, existem algumas etapas comuns a todas as instalações.

A coleta e a visualização dos seus dados de log envolvem geralmente as seguintes
etapas:

![Etapas de implementação do Loki](loki-install.png)

1. Instale o Loki no Kubernetes no modo monolítico (binário único), utilizando o
   [Helm chart](https://grafana.com/docs/loki/<LOKI_VERSION>/setup/install/helm/install-monolithic/)
   recomendado.
   Forneça ao Helm chart os detalhes de autenticação do seu armazenamento de
   objetos.
   - [Opções de armazenamento](https://grafana.com/docs/loki/<LOKI_VERSION>/operations/storage/)
   - [Referência de configuração](https://grafana.com/docs/loki/<LOKI_VERSION>/configure/)
   - Existem
     [exemplos](https://grafana.com/docs/loki/<LOKI_VERSION>/configure/examples/)
     para provedores específicos de armazenamento de objetos que você pode
     modificar.
1. Implante o [Grafana Alloy](https://grafana.com/docs/alloy/latest/) para
   coletar logs de suas aplicações.
   1. No Kubernetes, implante o Grafana Alloy utilizando o Helm chart.
      Configure o Grafana Alloy para coletar logs do seu cluster Kubernetes e
      adicione os detalhes do endpoint do Loki.
      Consulte a seção a seguir para ver um exemplo de arquivo de configuração
      do Grafana Alloy.
   1. Adicione
      [rótulos](https://grafana.com/docs/loki/<LOKI_VERSION>/get-started/labels/)
      aos seus logs seguindo nossas
      [melhores práticas](https://grafana.com/docs/loki/<LOKI_VERSION>/get-started/labels/bp-labels/).
      A maioria das pessoas usuárias do Loki começa adicionando rótulos que
      descrevem a origem dos logs, como região, cluster ou ambiente.
1. Implante o [Grafana](https://grafana.com/docs/grafana/latest/setup-grafana/)
   ou o [Grafana Cloud](https://grafana.com/docs/grafana-cloud/quickstart/) e
   configure uma
   [fonte de dados do Loki](https://grafana.com/docs/grafana/latest/datasources/loki/configure-loki-data-source/).
1. Selecione o recurso
   [Explore](https://grafana.com/docs/grafana/latest/explore/) no menu principal
   do Grafana.
   Para
   [visualizar logs no Explore](https://grafana.com/docs/grafana/latest/explore/logs-integration/):
   1. Escolha um intervalo de tempo.
   1. Selecione a fonte de dados do Loki.
   1. Use [LogQL](https://grafana.com/docs/loki/<LOKI_VERSION>/query/) no
      [editor de consultas](https://grafana.com/docs/grafana/latest/datasources/loki/query-editor/),
      utilize a visualização Builder para explorar seus rótulos ou selecione
      consultas de exemplo pré-configuradas usando o botão **Kick start your
      query**.

**Próximos passos:** Saiba mais sobre a linguagem de consulta do Loki, o
[LogQL](https://grafana.com/docs/loki/<LOKI_VERSION>/query/).

## Exemplo de arquivo de configuração do Grafana Alloy para enviar logs de Pods do Kubernetes para o Loki

Para implantar o Grafana Alloy a fim de coletar logs de Pods do seu cluster
Kubernetes e enviá-los para o Loki, você pode usar um Helm chart e um arquivo
`values.yaml`.

Este exemplo de arquivo `values.yaml` está configurado para:

- Instalar o Grafana Alloy para descobrir logs de Pods.
- Adicionar rótulos de `container` e `pod` aos logs.
- Enviar os logs para o seu cluster Loki usando o ID de tenant `local`.

1. Instale o Loki usando o
   [Helm chart](https://grafana.com/docs/loki/<LOKI_VERSION>/setup/install/helm/install-scalable/).

1. Implante o Grafana Alloy usando o Helm chart.
   Consulte [Instalar o Grafana Alloy no Kubernetes](https://grafana.com/docs/alloy/latest/get-started/install/kubernetes/)
   para obter mais informações.

1. Crie um arquivo `values.yaml` com base no exemplo a seguir, certificando-se
   de atualizar o valor de `forward_to = [loki.write.endpoint.receiver]`:

   ```yaml
   alloy:
     mounts:
       varlog: true
     configMap:
       content: |
         logging {
           level  = "info"
           format = "logfmt"
         }

         discovery.kubernetes "pods" {
           role = "pod"
         }

         loki.source.kubernetes "pods" {
           targets    = discovery.kubernetes.pods.targets
           forward_to = [loki.write.endpoint.receiver]
         }

         loki.write "endpoint" {
           endpoint {
               url = "http://loki-gateway.default.svc.cluster.local:80/loki/api/v1/push"
               tenant_id = "local"
           }
         }
   ```

1. Em seguida, instale o Alloy no seu cluster Kubernetes usando:

   ```bash
   helm install alloy grafana/alloy -f ./values.yaml
   ```
