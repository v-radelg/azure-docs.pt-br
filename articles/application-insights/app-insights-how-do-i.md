---
title: Como... no Application Insights | Microsoft Docs
description: Perguntas Frequentes no Application Insights.
services: application-insights
documentationcenter: 
author: alancameronwills
manager: douge
ms.assetid: 48b2b644-92e4-44c3-bc14-068f1bbedd22
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: article
ms.date: 02/05/2016
ms.author: awills
translationtype: Human Translation
ms.sourcegitcommit: 9a3df0ad2483471023ebb954d613bc5cad8fb7bf
ms.openlocfilehash: 9e54ee2d67a8dfb5b480db01219e128607e26f51
ms.lasthandoff: 02/02/2017


---
# <a name="how-do-i--in-application-insights"></a>Como ... no Application Insights?
## <a name="get-an-email-when-"></a>Receber um email quando...
### <a name="email-if-my-site-goes-down"></a>Enviar emails se meu site ficar inativo
Definir um [teste da Web de disponibilidade](app-insights-monitor-web-app-availability.md).

### <a name="email-if-my-site-is-overloaded"></a>Enviar emails de meu site estiver sobrecarregado
Definir um [alerta](app-insights-alerts.md) para **Tempo de resposta do servidor**. Um limite entre 1 e 2 segundos deve funcionar.

![](./media/app-insights-how-do-i/030-server.png)

Seu aplicativo também pode mostrar sinais de sobrecarga retornando códigos de falha. Definir um alerta para **Solicitações com falha**.

Se definir um alerta para **Exceções do servidor**, talvez você precise fazer [algumas configurações adicionais](app-insights-asp-net-exceptions.md) para ver os dados.

### <a name="email-on-exceptions"></a>Exceções de email
1. [Configurar monitoramento de exceção](app-insights-asp-net-exceptions.md)
2. [Definir um alerta](app-insights-alerts.md) na métrica de contagem de exceção

### <a name="email-on-an-event-in-my-app"></a>Enviar emails sobre um evento em meu aplicativo
Vamos supor que você gostaria de receber um email quando ocorrer um evento específico. O Application Insights não oferece esse recurso diretamente, mas pode [enviar um alerta quando uma métrica ultrapassar um limite](app-insights-alerts.md).

Alertas podem ser definidos com base em [métricas personalizadas](app-insights-api-custom-events-metrics.md#trackmetric), mas não em eventos personalizados. Escreva algum código para aumentar uma métrica quando o evento ocorre:

    telemetry.TrackMetric("Alarm", 10);

ou:

    var measurements = new Dictionary<string,double>();
    measurements ["Alarm"] = 10;
    telemetry.TrackEvent("status", null, measurements);

Como os alertas têm dois estados, você precisa enviar um valor baixo quando considerar o alerta concluído:

    telemetry.TrackMetric("Alarm", 0.5);

Crie um gráfico no [Metrics Explorer](app-insights-metrics-explorer.md) para ver o alarme:

![](./media/app-insights-how-do-i/010-alarm.png)

Agora defina um alerta para ser disparado quando a métrica ultrapassar um valor médio por um curto período:

![](./media/app-insights-how-do-i/020-threshold.png)

Defina o período de média como o mínimo.

Quando a métrica ficar acima e abaixo do limite, você receberá emails.

Considere o seguinte:

* Um alerta tem dois estados ("alerta" e "íntegro"). O estado é avaliado somente quando uma métrica é recebida.
* Um email é enviado apenas quando o estado é alterado. Por isso você precisa enviar métricas de valor alto e baixo.
* Para avaliar o alerta, a média é calculada com base dos valores recebidos no período anterior. Isso ocorre sempre que uma métrica é recebida, para que emails podem ser enviados com mais frequência do que o período definido.
* Uma vez que os emails são enviados tanto para "alerta" quanto para "íntegro", convém passar a pensar em seu único como uma condição de dois estados. Por exemplo, em vez de um evento de "trabalho concluído", tenha uma condição de "trabalho em andamento", na qual você recebe emails no início e no final de um trabalho.

### <a name="set-up-alerts-automatically"></a>Configurar alertas automaticamente
[Usar o PowerShell para criar novos alertas](app-insights-alerts.md#automation)

## <a name="use-powershell-to-manage-application-insights"></a>Usar o PowerShell para gerenciar o Application Insights
* [Criar novos recursos](app-insights-powershell-script-create-resource.md)
* [Criar novos alertas](app-insights-alerts.md#automation)

## <a name="application-versions-and-stamps"></a>Carimbos e versões de aplicativos
### <a name="separate-the-results-from-dev-test-and-prod"></a>Separar os resultados de desenvolvimento, teste e produção
* Para ambientes diferentes, configurar diferentes ikeys
* Para diferentes carimbos (desenvolvimento, teste, produção) marcar a telemetria com diferentes valores de propriedade

[Saiba mais](app-insights-separate-resources.md)

### <a name="filter-on-build-number"></a>Filtrar por número de compilação
Quando publicar uma nova versão do seu aplicativo, você desejará ser capaz de separar a telemetria das compilações diferentes.

Você pode definir a propriedade de versão do aplicativo para que possa filtrar resultados da [pesquisa](app-insights-diagnostic-search.md) e do [Metrics Explorer](app-insights-metrics-explorer.md).

![](./media/app-insights-how-do-i/050-filter.png)

Há vários métodos diferentes de definir a propriedade de Versão do aplicativo.

* Definir diretamente:

    `telemetryClient.Context.Component.Version = typeof(MyProject.MyClass).Assembly.GetName().Version;`
* Defina a quebra automática de linha em um [inicializador de telemetria](app-insights-api-custom-events-metrics.md#defaults) para garantir que todas as instâncias de TelemetryClient sejam configuradas de forma consistente.
* [ASP.NET] Definir a versão em `BuildInfo.config`. O módulo da Web selecionará a versão do nó BuildLabel. Inclua esse arquivo no seu projeto e não se esqueça de definir a propriedade Copy Always no Gerenciador de Soluções.

    ```XML

    <?xml version="1.0" encoding="utf-8"?>
    <DeploymentEvent xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/VisualStudio/DeploymentEvent/2013/06">
      <ProjectName>AppVersionExpt</ProjectName>
      <Build type="MSBuild">
        <MSBuild>
          <BuildLabel kind="label">1.0.0.2</BuildLabel>
        </MSBuild>
      </Build>
    </DeploymentEvent>

    ```
* [ASP.NET] Gerar automaticamente BuildInfo.config no MSBuild. Para fazer isso, adicione algumas linhas ao arquivo .csproj:

    ```XML

    <PropertyGroup>
      <GenerateBuildInfoConfigFile>true</GenerateBuildInfoConfigFile>    <IncludeServerNameInBuildInfo>true</IncludeServerNameInBuildInfo>
    </PropertyGroup>
    ```

    Isso gera um arquivo chamado *nomedoSeuProjeto*.BuildInfo.config. O processo de Publicação renomeia o arquivo como BuildInfo.config.

    O rótulo da compilação contém um espaço reservado (AutoGen_...) quando você cria com o Visual Studio. Mas quando compilado com o MSBuild, ele é preenchido com o número de versão correta.

    Para permitir que o MSBuild gere números de versão, defina a versão como `1.0.*` em AssemblyReference.cs

## <a name="monitor-backend-servers-and-desktop-apps"></a>Monitorar servidores de back-end e aplicativos de desktop
[Use o módulo do SDK do Windows Server](app-insights-windows-desktop.md).

## <a name="visualize-data"></a>Visualizar dados
#### <a name="dashboard-with-metrics-from-multiple-apps"></a>Painel com métricas de vários aplicativos
* No [Metrics Explorer](app-insights-metrics-explorer.md), personalize o gráfico e salve-o como um favorito. Fixe-o no painel do Azure.

#### <a name="dashboard-with-data-from-other-sources-and-application-insights"></a>Painel com dados de outras fontes e Application Insights
* [Exportar telemetria para o Power BI](app-insights-export-power-bi.md).

Ou

* Use o SharePoint como seu painel e exiba dados em web parts do SharePoint. [Usar exportação contínua e Stream Analytics para exportar para o SQL](app-insights-code-sample-export-sql-stream-analytics.md).  Use o PowerView para examinar o banco de dados e criar uma web part do SharePoint para o PowerView.

<a name="search-specific-users"></a>

### <a name="filter-out-anonymous-or-authenticated-users"></a>Filtrar usuários anônimos ou autenticados
Se os seus usuários se conectarem, você poderá definir a [ID de usuário autenticado](app-insights-api-custom-events-metrics.md#authenticated-users). (Isso não ocorre automaticamente.)

Você pode:

* Pesquisar IDs de usuário específicos

![](./media/app-insights-how-do-i/110-search.png)

* Filtrar métricas para usuários anônimos ou autenticados

![](./media/app-insights-how-do-i/115-metrics.png)

## <a name="modify-property-names-or-values"></a>Modificar valores ou nomes de propriedade
Crie um [filtro](app-insights-api-filtering-sampling.md#filtering). Isso permite modificar ou filtrar a telemetria antes que ela seja enviada do seu aplicativo para o Application Insights.

## <a name="list-specific-users-and-their-usage"></a>Listar usuários específicos e seu uso
Se desejar apenas [pesquisar usuários específicos](#search-specific-users), poderá definir a [ID de usuário autenticado](app-insights-api-custom-events-metrics.md#authenticated-users).

Se você quiser uma lista de usuários com os dados como, por exemplo, quais páginas eles exibem e com qual frequência eles fazem logon, você terá duas opções:

* [Defina a ID de usuário autenticado](app-insights-api-custom-events-metrics.md#authenticated-users), [exporte para um banco de dados](app-insights-code-sample-export-sql-stream-analytics.md) e use ferramentas adequadas para analisar seus dados de usuário.
* Se você tiver apenas um pequeno número de usuários, envie métricas ou eventos personalizados usando os dados de interesse, como o valor da métrica ou o nome do evento, definindo a ID de usuário como uma propriedade. Para analisar os modos de exibição de página, substitua a chamada trackPageView JavaScript padrão. Para analisar a telemetria do lado do servidor, use um inicializador de telemetria para adicionar a ID de usuário a todas as telemetria do servidor. Em seguida, filtre e segmente as métricas e pesquisas na ID de usuário.

## <a name="reduce-traffic-from-my-app-to-application-insights"></a>Reduzir o tráfego do meu aplicativo no Application Insights
* Em [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md), desabilite todos os módulos dos quais você não precisa, como o coletor do contador de desempenho.
* Use a [Amostragem e filtragem](app-insights-api-filtering-sampling.md) no SDK.
* Nas páginas da Web, limite o número de chamadas do Ajax relatadas para cada exibição de página. No trecho de script após `instrumentationKey:...`, insira: `,maxAjaxCallsPerView:3` (ou um número adequado).
* Se estiver usando o [TrackMetric](app-insights-api-custom-events-metrics.md#trackmetric), calcule a agregação de lotes de valores de métrica antes de enviar o resultado. Há uma sobrecarga de TrackMetric() que possibilita isso.

Saiba mais sobre [preços e cotas](app-insights-pricing.md).

## <a name="disable-telemetry"></a>Desabilitar telemetria
Para **parar e iniciar dinamicamente** a coleta e a transmissão de telemetria do servidor:

```

    using  Microsoft.ApplicationInsights.Extensibility;

    TelemetryConfiguration.Active.DisableTelemetry = true;
```



Para **desabilitar os coletores padrão selecionados** - por exemplo, contadores de desempenho, solicitações HTTP ou dependências - exclua ou comente as linhas relevantes em [ApplicationInsights.config](app-insights-api-custom-events-metrics.md). Você poderá fazer isso, por exemplo, se quiser enviar seus próprios dados de TrackRequest.

## <a name="view-system-performance-counters"></a>Exibir contadores de desempenho do sistema
Entre as métricas que você pode exibir no Metrics Explorer, existe um conjunto de contadores de desempenho do sistema. Há uma folha predefinida intitulada **Servidores** que exibe vários deles.

![Abra o recurso Application Insights e clique em Servidores](./media/app-insights-how-do-i/121-servers.png)

### <a name="if-you-see-no-performance-counter-data"></a>Se você não vir dados do contador de desempenho
* **Servidor IIS** em seu próprio computador ou em uma VM. [Instalar Monitor de Status](app-insights-monitor-performance-live-website-now.md).
* **Site do Azure** - ainda não há suporte para contadores de desempenho. Existem várias métricas que você pode obter como parte padrão do painel de controle do site do Azure.
* **Servidor Unix** - [Instalar collectd](app-insights-java-collectd.md)

### <a name="to-display-more-performance-counters"></a>Para exibir mais contadores de desempenho
* Primeiro, [adicione um novo gráfico](app-insights-metrics-explorer.md) e veja se o contador está no conjunto básico que oferecemos.
* Caso contrário, [adicione o contador ao conjunto coletado pelo módulo do contador de desempenho](app-insights-performance-counters.md).

