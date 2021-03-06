---
title: Como exibir a integridade agregada das entidades do Azure Service Fabric | Microsoft Docs
description: Descreve como consultar, exibir e avaliar a integridade agregada das entidades do Azure Service Fabric, por meio de consultas de integridade e consultas gerais.
services: service-fabric
documentationcenter: .net
author: oanapl
manager: timlt
editor: 
ms.assetid: fa34c52d-3a74-4b90-b045-ad67afa43fe5
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/12/2017
ms.author: oanapl
translationtype: Human Translation
ms.sourcegitcommit: 2573f300231ceb080343dad5c1c2aa65cc31c59e
ms.openlocfilehash: f62a7a8f594c67cac687159d72bfdb00a3355025


---
# <a name="view-service-fabric-health-reports"></a>Como exibir relatórios de integridade do Service Fabric
O Azure Service Fabric apresenta um [modelo de integridade](service-fabric-health-introduction.md) composto por entidades de integridade nas quais os componentes do sistema e watchdogs podem relatar condições locais que estão monitorando. O [repositório de integridade](service-fabric-health-introduction.md#health-store) agrega todos os dados de integridade para determinar se as entidades estão íntegras.

Pronto para uso, o cluster é populado com relatórios de integridade enviados pelos componentes do sistema. Leia mais em [Usar relatórios de integridade do sistema para solução de problemas](service-fabric-understand-and-troubleshoot-with-system-health-reports.md).

O Service Fabric fornece várias maneiras de obter a integridade agregada de entidades:

* [Gerenciador de Malha do Serviço](service-fabric-visualizing-your-cluster.md) ou outras ferramentas de visualização
* Consultas de integridade (por meio do PowerShell, API ou REST)
* Consultas gerais que retornam uma lista de entidades que têm a integridade como uma das propriedades (por meio do PowerShell, API ou REST)

Para demonstrar essas opções, vamos usar um cluster local com cinco nós. Ao lado do aplicativo **fabric:/System** (que existe originalmente), alguns outros aplicativos são implantados. Um deles é **fabric:/WordCount**. Esse aplicativo contém um serviço com estado configurado com sete réplicas. Como há apenas cinco nós, os componentes do sistema mostram um aviso de que a partição está abaixo da contagem de destino.

```xml
<Service Name="WordCountService">
    <StatefulService ServiceTypeName="WordCountServiceType" TargetReplicaSetSize="7" MinReplicaSetSize="2">
      <UniformInt64Partition PartitionCount="1" LowKey="1" HighKey="26" />
    </StatefulService>
</Service>
```

## <a name="health-in-service-fabric-explorer"></a>Integridade no Gerenciador da Malha do Serviço
O Gerenciador da Malha do Serviço fornece uma exibição visual do cluster. Na imagem abaixo, você pode ver que:

* O aplicativo **fabric:/WordCount** está vermelho (em erro) porque ele tem um evento de erro relatado por **MyWatchdog** para a propriedade **Disponibilidade**.
* Um de seus serviços, **fabric:/WordCount/WordCountService** está amarelo (em aviso). Uma vez que o serviço está configurado com sete réplicas e há apenas cinco nós, elas não podem ser todas colocadas. Embora não seja mostrado aqui, a partição de serviço está amarela devido ao relatório do sistema. A partição amarela dispara o serviço amarelo.
* O cluster está vermelho devido ao aplicativo vermelho.

A avaliação usa políticas padrão do manifesto do cluster e do manifesto do aplicativo. Elas são políticas rígidas e não toleram falhas.

Visualização do cluster com o Gerenciador do Service Fabric:

![Visualização do cluster com o Gerenciador do Service Fabric.][1]

[1]: ./media/service-fabric-view-entities-aggregated-health/servicefabric-explorer-cluster-health.png


> [!NOTE]
> Leia mais sobre o [Gerenciador da Malha do Serviço](service-fabric-visualizing-your-cluster.md).
>
>

## <a name="health-queries"></a>Consultas de integridade
A Malha do Serviço expõe consultas de integridade para cada um dos [tipos de entidade](service-fabric-health-introduction.md#health-entities-and-hierarchy)com suporte. Elas podem ser acessados por API (os métodos podem ser encontrados em **FabricClient.HealthManager**), por cmdlets do PowerShell e por REST. Essas consultas retornam informações de integridade completas sobre a entidade: o estado de integridade agregada, eventos de integridade da entidade, estados de integridade dos filhos (quando aplicável) e avaliações não íntegras quando a entidade não está íntegra.

> [!NOTE]
> Uma entidade de integridade retorna para o usuário quando ela é totalmente populada no repositório de integridade. A entidade deve estar ativa (não excluída) e ter um relatório do sistema. Suas entidades pai na cadeia hierárquica também devem ter relatórios do sistema. Se alguma dessas condições não for atendida, as consultas de integridade retornarão uma exceção que mostra por que a entidade não retornou.
>
>

As consultas de integridade devem passar pelo identificador da entidade, que depende do tipo de entidade. As consultas aceitam parâmetros opcionais da política de integridade. Se nenhuma política de integridade for especificada, as [políticas de integridade](service-fabric-health-introduction.md#health-policies) do manifesto do cluster ou do aplicativo serão usadas para avaliação. As consultas também aceitam filtros para retornar apenas eventos ou filhos parciais; aqueles que respeitarem os filtros especificados.

> [!NOTE]
> Os filtros de saída são aplicados no lado do servidor, de modo que o tamanho da resposta da mensagem é reduzido. Recomendamos o uso dos filtros de saída para limitar os dados retornados, em vez de aplicar filtros no lado do cliente.
>
>

A integridade da entidade contém:

* O estado da integridade agregada da entidade. Ele é calculado pelo repositório de integridade com base nos relatórios de integridade da entidade, pelos estados da integridade dos filhos (quando aplicável) e por políticas de integridade. Leia mais sobre [Avaliação de integridade da entidade](service-fabric-health-introduction.md#health-evaluation).  
* Os eventos de integridade da entidade.
* A coleção de estados de integridade de todos os filhos das entidades que podem ter filhos. Os estados da integridade contêm identificadores de entidade e o estado da integridade agregada. Para obter toda a integridade de um filho, chame a integridade de consulta do tipo de entidade filho e passe o identificador do filho.
* As avaliações não íntegras que apontam para o relatório que disparou o estado da entidade, se a entidade não estiver íntegra.

## <a name="get-cluster-health"></a>Obter integridade do cluster
Retorna a integridade da entidade do cluster e contém os estados da integridade de aplicativos e nós (filhos do cluster). Entrada:

* [Opcional] A política de integridade do cluster usada para avaliar os nós e os eventos de cluster.
* [Opcional] Mapa da política de integridade do aplicativo, com políticas de integridade usadas para substituir as políticas do manifesto do aplicativo.
* [Opcional] Filtros de eventos, nós e aplicativos que especificam quais entradas são interessantes e devem retornar nos resultados (por exemplo, apenas erros ou avisos e erros). Todos os eventos, nós e aplicativos são usados para avaliar a integridade agregada da entidade, independentemente do filtro.

### <a name="api"></a>API
Para obter a integridade do cluster, crie um `FabricClient` e chame o método [GetClusterHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getclusterhealthasync.aspx) em seu **HealthManager**.

A chamada a seguir obtém a integridade do cluster:

```csharp
ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync();
```

O código a seguir obtém a integridade do cluster usando política de integridade de cluster personalizada para nós e aplicativos. Ele cria [ClusterHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.clusterhealthquerydescription.aspx), que contém as informações de entrada.

```csharp
var policy = new ClusterHealthPolicy()
{
    MaxPercentUnhealthyNodes = 20
};
var nodesFilter = new NodeHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error | HealthStateFilter.Warning
};
var applicationsFilter = new ApplicationHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error
};
var queryDescription = new ClusterHealthQueryDescription()
{
    HealthPolicy = policy,
    ApplicationsFilter = applicationsFilter,
    NodesFilter = nodesFilter,
};

ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade do cluster é [Get-ServiceFabricClusterHealth](https://msdn.microsoft.com/library/mt125850.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) .

Estado do cluster é de cinco nós, o aplicativo do sistema e fabric:/WordCount configurados conforme descrito.

O cmdlet a seguir obtém a integridade do cluster usando políticas de integridade padrão. O estado de integridade agregada é de aviso, pois o aplicativo fabric:/WordCount está em aviso. Observe como as avaliações não íntegras mostram com detalhes a condição que disparou a integridade agregada.

```xml
PS C:\> Get-ServiceFabricClusterHealth

AggregatedHealthState   : Warning
UnhealthyEvaluations    :
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Warning'.

                              Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                              Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                  Unhealthy event: SourceId='System.PLB',
                          Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                          ConsiderWarningAsError=false.


NodeHealthStates        :
                          NodeName              : _Node_2
                          AggregatedHealthState : Ok

                          NodeName              : _Node_0
                          AggregatedHealthState : Ok

                          NodeName              : _Node_1
                          AggregatedHealthState : Ok

                          NodeName              : _Node_3
                          AggregatedHealthState : Ok

                          NodeName              : _Node_4
                          AggregatedHealthState : Ok

ApplicationHealthStates :
                          ApplicationName       : fabric:/System
                          AggregatedHealthState : Ok

                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Warning

HealthEvents            : None
```

O cmdlet do PowerShell a seguir obtém a integridade do cluster usando uma política de aplicativo personalizada. Ele filtra os resultados para obter apenas os aplicativos e nós com erro ou aviso. Como resultado nenhum nó é retornado, pois todos estão íntegros. Somente o aplicativo fabric:/WordCount respeita o filtro de aplicativos. Como a política personalizada especifica a consideração de avisos como erros para o aplicativo fabric:/WordCount, o aplicativo é avaliado como em estado de erro, assim como o cluster.

```powershell
PS c:\> $appHealthPolicy = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicy
$appHealthPolicy.ConsiderWarningAsError = $true
$appHealthPolicyMap = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicyMap
$appUri1 = New-Object -TypeName System.Uri -ArgumentList "fabric:/WordCount"
$appHealthPolicyMap.Add($appUri1, $appHealthPolicy)
Get-ServiceFabricClusterHealth -ApplicationHealthPolicyMap $appHealthPolicyMap -ApplicationsFilter "Warning,Error" -NodesFilter "Warning,Error"


AggregatedHealthState   : Error
UnhealthyEvaluations    :
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Error'.

                              Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                              Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                  Unhealthy event: SourceId='System.PLB',
                          Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                          ConsiderWarningAsError=true.


NodeHealthStates        : None
ApplicationHealthStates :
                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Error

HealthEvents            : None

```

### <a name="rest"></a>REST
Você pode obter a integridade do cluster com uma [solicitação GET](https://msdn.microsoft.com/library/azure/dn707669.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/dn707696.aspx), que inclui as políticas de integridade descritas no corpo.

## <a name="get-node-health"></a>Obter integridade do nó
Retorna a integridade de uma entidade de nó e contém os eventos de integridade reportados no nó. Entrada:

* [Obrigatório] O nome do nó que identifica o nó.
* [Opcional] As configurações da política de integridade do cluster usadas para avaliar a integridade.
* [Opcional] Filtros de eventos que especificam quais entradas são interessantes e devem retornar nos resultados (por exemplo, apenas erros ou avisos e erros). Todos os eventos são usados para avaliar a integridade agregada da entidade, independentemente do filtro.

### <a name="api"></a>API
Para obter a integridade do nó por meio da API, crie um `FabricClient` e chame o método [GetNodeHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getnodehealthasync.aspx) em seu HealthManager.

O código a seguir obtém a integridade de nó para o nome de nó especificado:

```csharp
NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(nodeName);
```

O código a seguir obtém a integridade de nó para o nome do nó especificado e passa um filtro de eventos e política personalizada por meio de [NodeHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.nodehealthquerydescription.aspx):

```csharp
var queryDescription = new NodeHealthQueryDescription(nodeName)
{
    HealthPolicy = new ClusterHealthPolicy() {  ConsiderWarningAsError = true },
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.Warning },
};

NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade do nó é [Get-ServiceFabricNodeHealth](https://msdn.microsoft.com/library/mt125937.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) .
O cmdlet a seguir obtém a integridade do nó usando políticas de integridade padrão:

```powershell
PS C:\> Get-ServiceFabricNodeHealth _Node_1


NodeName              : _Node_1
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 6
                        SentAt                : 3/22/2016 7:47:56 PM
                        ReceivedAt            : 3/22/2016 7:48:19 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:48:19 PM, LastWarning = 1/1/0001 12:00:00 AM
```

O cmdlet a seguir obtém a integridade de todos os nós no cluster.

```powershell
PS C:\> Get-ServiceFabricNode | Get-ServiceFabricNodeHealth | select NodeName, AggregatedHealthState | ft -AutoSize

NodeName AggregatedHealthState
-------- ---------------------
_Node_2                     Ok
_Node_0                     Ok
_Node_1                     Ok
_Node_3                     Ok
_Node_4                     Ok
```

### <a name="rest"></a>REST
Você pode obter a integridade do nó com uma [solicitação GET](https://msdn.microsoft.com/library/azure/dn707650.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/dn707665.aspx), que inclui as políticas de integridade descritas no corpo.

## <a name="get-application-health"></a>Obter integridade do aplicativo
Retorna a integridade de uma entidade de aplicativo. Contém os estados de integridade do aplicativo implantado e filhos de serviço. Entrada:

* [Obrigatório] O nome do aplicativo (URI) que identifica o aplicativo.
* [Opcional] A política de integridade do aplicativo usada para substituir as políticas do manifesto do aplicativo.
* [Opcional] Filtros de eventos, serviços e aplicativos implantados que especificam quais entradas são interessantes e devem retornar nos resultados (por exemplo, apenas erros ou avisos e erros). Todos os eventos, serviços e aplicativos implantados são usados para avaliar a integridade agregada da entidade, independentemente do filtro.

### <a name="api"></a>API
Para obter a integridade do aplicativo, crie um `FabricClient` e chame o método [GetApplicationHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getapplicationhealthasync.aspx) em seu HealthManager.

O código a seguir obtém a integridade do aplicativo para o nome do aplicativo especificado (URI):

```csharp
ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(applicationName);
```

O código a seguir obtém a integridade do aplicativo para o nome do aplicativo especificado (URI), com filtros e políticas personalizadas especificadas por meio de [ApplicationHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.applicationhealthquerydescription.aspx).

```csharp
HealthStateFilter warningAndErrors = HealthStateFilter.Error | HealthStateFilter.Warning;
var serviceTypePolicy = new ServiceTypeHealthPolicy()
{
    MaxPercentUnhealthyPartitionsPerService = 0,
    MaxPercentUnhealthyReplicasPerPartition = 5,
    MaxPercentUnhealthyServices = 0,
};
var policy = new ApplicationHealthPolicy()
{
    ConsiderWarningAsError = false,
    DefaultServiceTypeHealthPolicy = serviceTypePolicy,
    MaxPercentUnhealthyDeployedApplications = 0,
};

var queryDescription = new ApplicationHealthQueryDescription(applicationName)
{
    HealthPolicy = policy,
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = warningAndErrors },
    ServicesFilter = new ServiceHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
    DeployedApplicationsFilter = new DeployedApplicationHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
};

ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade do aplicativos é [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/library/mt125976.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) .

O cmdlet a seguir retorna a integridade do aplicativo **fabric:/WordCount** :

```powershell
PS c:\>
PS C:\WINDOWS\system32>  Get-ServiceFabricApplicationHealth fabric:/WordCount


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Warning
UnhealthyEvaluations            :
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                      Unhealthy event: SourceId='System.PLB',
                                  Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                                  ConsiderWarningAsError=false.

ServiceHealthStates             :
                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Warning

                                  ServiceName           : fabric:/WordCount/WordCountWebService
                                  AggregatedHealthState : Ok

DeployedApplicationHealthStates :
                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_0
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_2
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_3
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_4
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_1
                                  AggregatedHealthState : Ok

HealthEvents                    :
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 360
                                  SentAt                : 3/22/2016 7:56:53 PM
                                  ReceivedAt            : 3/22/2016 7:56:53 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 3/22/2016 7:56:53 PM, LastWarning = 1/1/0001 12:00:00 AM

                                  SourceId              : MyWatchdog
                                  Property              : Availability
                                  HealthState           : Ok
                                  SequenceNumber        : 131031545225930951
                                  SentAt                : 3/22/2016 9:08:42 PM
                                  ReceivedAt            : 3/22/2016 9:08:42 PM
                                  TTL                   : Infinite
                                  Description           : Availability checked successfully, latency ok
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 3/22/2016 8:55:39 PM, LastWarning = 1/1/0001 12:00:00 AM
```

O cmdlet do PowerShell a seguir passa políticas personalizadas. Ele também filtra filhos e eventos.

```powershell
PS C:\> Get-ServiceFabricApplicationHealth -ApplicationName fabric:/WordCount -ConsiderWarningAsError $true -ServicesFilter Error -EventsFilter Error -DeployedApplicationsFilter Error

ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            :
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      Unhealthy event: SourceId='System.PLB',
                                  Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                                  ConsiderWarningAsError=true.

ServiceHealthStates             :
                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Error

DeployedApplicationHealthStates : None
HealthEvents                    : None
```

### <a name="rest"></a>REST
Você pode obter a integridade do aplicativo com uma [solicitação GET](https://msdn.microsoft.com/library/azure/dn707681.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/dn707643.aspx), que inclui as políticas de integridade descritas no corpo.

## <a name="get-service-health"></a>Obter integridade do serviço
Retorna a integridade de uma entidade de serviço. Além disso, contém os estados de integridade da partição. Entrada:

* [Obrigatório] O nome de serviço (URI) que identifica o serviço.
* [Opcional] A política de integridade do aplicativo usada para substituir a política do manifesto do aplicativo.
* [Opcional] Filtros de eventos e partições que especificam quais entradas são interessantes e devem retornar nos resultados (por exemplo, apenas erros ou avisos e erros). Todos os eventos e partições são usados para avaliar a integridade agregada da entidade, independentemente do filtro.

### <a name="api"></a>API
Para obter a integridade do serviço por meio de API, crie um `FabricClient` e chame o método [GetServiceHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getservicehealthasync.aspx) em seu HealthManager.

O exemplo a seguir obtém a integridade de um serviço com o nome do serviço especificado (URI):

```charp
ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(serviceName);
```

O código a seguir obtém a integridade do serviço para o nome do serviço especificado (URI), especificando filtros e política personalizada por meio de [ServiceHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.servicehealthquerydescription.aspx):

```csharp
var queryDescription = new ServiceHealthQueryDescription(serviceName)
{
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.All },
    PartitionsFilter = new PartitionHealthStatesFilter() { HealthStateFilterValue = HealthStateFilter.Error },
};

ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade do serviço é [Get-ServiceFabricServiceHealth](https://msdn.microsoft.com/library/mt125984.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) .

O cmdlet a seguir obtém a integridade do serviço usando políticas de integridade padrão:

```powershell
PS C:\> Get-ServiceFabricServiceHealth -ServiceName fabric:/WordCount/WordCountService


ServiceName           : fabric:/WordCount/WordCountService
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.PLB',
                        Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                        ConsiderWarningAsError=false.

PartitionHealthStates :
                        PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
                        AggregatedHealthState : Warning

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 10
                        SentAt                : 3/22/2016 7:56:53 PM
                        ReceivedAt            : 3/22/2016 7:57:18 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b
                        HealthState           : Warning
                        SequenceNumber        : 131031547693687021
                        SentAt                : 3/22/2016 9:12:49 PM
                        ReceivedAt            : 3/22/2016 9:12:49 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        fabric:/WordCount/WordCountService Secondary Partition a1f83a35-d6bf-4d39-b90d-28d15f39599b could not be placed, possibly,
                        due to the following constraints and properties:  
                        Placement Constraint: N/A
                        Depended Service: N/A

                        Constraint Elimination Sequence:
                        ReplicaExclusionStatic eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        ReplicaExclusionDynamic eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.

                        Nodes Eliminated By Constraints:

                        ReplicaExclusionStatic:
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status:
                        None/None

                        ReplicaExclusionDynamic:
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status:
                        None/None


                        RemoveWhenExpired     : True
                        IsExpired             : False
```

### <a name="rest"></a>REST
Você pode obter a integridade do serviço com uma [solicitação GET](https://msdn.microsoft.com/library/azure/dn707609.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/dn707646.aspx), que inclui as políticas de integridade descritas no corpo.

## <a name="get-partition-health"></a>Obter integridade da partição
Retorna a integridade de uma entidade de partição. Além disso, contém os estados de integridade da réplica. Entrada:

* [Obrigatório] A ID de partição (GUID) que identifica a partição.
* [Opcional] A política de integridade do aplicativo usada para substituir a política do manifesto do aplicativo.
* [Opcional] Filtros para eventos e réplicas que especificam quais entradas são interessantes e devem retornar nos resultados (por exemplo, apenas erros ou avisos e erros). Todos os eventos e réplicas são usados para avaliar a integridade agregada da entidade, independentemente do filtro.

### <a name="api"></a>API
Para obter a integridade da partição por meio de API, crie um `FabricClient` e chame o método [GetPartitionHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getpartitionhealthasync.aspx) em seu HealthManager. Para especificar parâmetros opcionais, crie [PartitionHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.partitionhealthquerydescription.aspx).

```csharp
PartitionHealth partitionHealth = await fabricClient.HealthManager.GetPartitionHealthAsync(partitionId);
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade da partição é [Get-ServiceFabricPartitionHealth](https://msdn.microsoft.com/library/mt125869.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) .

O cmdlet a seguir obtém a integridade de todas as partições do serviço **fabric:/WordCount/WordCountService** :

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth


PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ReplicaHealthStates   :
                        ReplicaId             : 131031502143040223
                        AggregatedHealthState : Ok

                        ReplicaId             : 131031502346844060
                        AggregatedHealthState : Ok

                        ReplicaId             : 131031502346844059
                        AggregatedHealthState : Ok

                        ReplicaId             : 131031502346844061
                        AggregatedHealthState : Ok

                        ReplicaId             : 131031502346844058
                        AggregatedHealthState : Ok

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 76
                        SentAt                : 3/22/2016 7:57:26 PM
                        ReceivedAt            : 3/22/2016 7:57:48 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 3/22/2016 7:57:48 PM, LastOk = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Você pode obter a integridade da partição com uma [solicitação GET](https://msdn.microsoft.com/library/azure/dn707683.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/dn707680.aspx), que inclui as políticas de integridade descritas no corpo.

## <a name="get-replica-health"></a>Obter integridade da réplica
Retorna a integridade de uma réplica de serviço com estado ou uma instância de serviço sem estado. Entrada:

* [Obrigatório] A ID da partição (GUID) e a ID da réplica que identifica a réplica.
* [Opcional] Os parâmetros da política de integridade do aplicativo usados para substituir as políticas de manifesto do aplicativo.
* [Opcional] Filtros de eventos que especificam quais entradas são interessantes e devem retornar nos resultados (por exemplo, apenas erros ou avisos e erros). Todos os eventos são usados para avaliar a integridade agregada da entidade, independentemente do filtro.

### <a name="api"></a>API
Para obter a integridade da réplica por meio de API, crie um `FabricClient` e chame o método [GetReplicaHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getreplicahealthasync.aspx) em seu HealthManager. Para especificar parâmetros avançados, use [ReplicaHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.replicahealthquerydescription.aspx).

```csharp
ReplicaHealth replicaHealth = await fabricClient.HealthManager.GetReplicaHealthAsync(partitionId, replicaId);
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade da réplica é [Get-ServiceFabricReplicaHealth](https://msdn.microsoft.com/library/mt125808.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) .

O cmdlet a seguir obtém a integridade da réplica primária para todas as partições do serviço.

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth


PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
ReplicaId             : 131031502143040223
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131031502145556748
                        SentAt                : 3/22/2016 7:56:54 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Você pode obter a integridade da réplica com uma [solicitação GET](https://msdn.microsoft.com/library/azure/dn707673.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/dn707641.aspx), que inclui as políticas de integridade descritas no corpo.

## <a name="get-deployed-application-health"></a>Obter integridade do aplicativo implantado
Retorna a integridade de um aplicativo implantado em uma entidade de nó. Além disso, contém os estados de integridade do pacote de serviço implantado. Entrada:

* [Obrigatório] O nome do aplicativo (URI) e o nome do nó (cadeia de caracteres) que identificam o aplicativo implantado
* [Opcional] A política de integridade do aplicativo usada para substituir as políticas do manifesto do aplicativo.
* [Opcional] Filtros de eventos e pacotes de serviço implantados que especificam quais entradas são interessantes e devem retornar nos resultados (por exemplo, apenas erros ou avisos e erros). Todos os eventos e pacotes de serviço implantados são usados para avaliar a integridade agregada da entidade, independentemente do filtro.

### <a name="api"></a>API
Para obter a integridade de um aplicativo implantado em um nó por meio de API, crie um `FabricClient` e chame o método [GetDeployedApplicationHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getdeployedapplicationhealthasync.aspx) em seu HealthManager. Para especificar parâmetros opcionais, use [DeployedApplicationHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.deployedapplicationhealthquerydescription.aspx).

```csharp
DeployedApplicationHealth health = await fabricClient.HealthManager.GetDeployedApplicationHealthAsync(
    new DeployedApplicationHealthQueryDescription(applicationName, nodeName));
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade do aplicativo implantado é [Get-ServiceFabricDeployedApplicationHealth](https://msdn.microsoft.com/library/mt163523.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) . Para descobrir onde um aplicativo está implantado, execute [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/library/mt125976.aspx) e observe os filhos do aplicativo implantado.

O cmdlet a seguir obtém a integridade do aplicativo **fabric:/WordCount** implantado em **_Node_2**.

```powershell
PS C:\> Get-ServiceFabricDeployedApplicationHealth -ApplicationName fabric:/WordCount -NodeName _Node_2


ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_2
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates :
                                     ServiceManifestName   : WordCountServicePkg
                                     NodeName              : _Node_2
                                     AggregatedHealthState : Ok

                                     ServiceManifestName   : WordCountWebServicePkg
                                     NodeName              : _Node_2
                                     AggregatedHealthState : Ok

HealthEvents                       :
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131031502143710698
                                     SentAt                : 3/22/2016 7:56:54 PM
                                     ReceivedAt            : 3/22/2016 7:57:12 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Você pode obter a integridade do aplicativo implantado com uma [solicitação GET](https://msdn.microsoft.com/library/azure/dn707644.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/dn707688.aspx), que inclui as políticas de integridade descritas no corpo.

## <a name="get-deployed-service-package-health"></a>Obter integridade do pacote de serviço implantado
Retorna a integridade de uma entidade de pacote de serviço implantada. Entrada:

* [Obrigatório] O nome do aplicativo (URI), nome do nó (cadeia de caracteres) e nome do manifesto do serviço (cadeia de caracteres) que identificam o pacote de serviço implantado
* [Opcional] A política de integridade do aplicativo usada para substituir a política do manifesto do aplicativo.
* [Opcional] Filtros de eventos que especificam quais entradas são interessantes e devem retornar nos resultados (por exemplo, apenas erros ou avisos e erros). Todos os eventos são usados para avaliar a integridade agregada da entidade, independentemente do filtro.

### <a name="api"></a>API
Para obter a integridade de um pacote de serviço implantado por meio de API, crie um `FabricClient` e chame o método [GetDeployedServicePackageHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getdeployedservicepackagehealthasync.aspx) em seu HealthManager. Para especificar parâmetros opcionais, use [DeployedServicePackageHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.deployedservicepackagehealthquerydescription.aspx).

```csharp
DeployedServicePackageHealth health = await fabricClient.HealthManager.GetDeployedServicePackageHealthAsync(
    new DeployedServicePackageHealthQueryDescription(applicationName, nodeName, serviceManifestName));
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade do pacote de serviço implantado é [Get-ServiceFabricDeployedServicePackageHealth](https://msdn.microsoft.com/library/mt163525.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) . Para descobrir onde um aplicativo está implantado, execute [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/library/mt125976.aspx) e observe os aplicativos implantados. Para ver quais pacotes de serviço estão em um aplicativo, observe os filhos do pacote de serviço implantado na saída de [Get-ServiceFabricDeployedApplicationHealth](https://msdn.microsoft.com/library/mt163523.aspx) .

O cmdlet a seguir obtém a integridade do pacote de serviço **WordCountServicePkg** do aplicativo **fabric:/WordCount** implantado em **_Node_2**. A entidade tem relatórios **System.Hosting** para ativação bem-sucedida do pacote de serviço e do ponto de entrada e registro bem-sucedido do tipo de serviço.

```powershell
PS C:\> Get-ServiceFabricDeployedApplication -ApplicationName fabric:/WordCount -NodeName _Node_2 | Get-ServiceFabricDeployedServicePackageHealth -ServiceManifestName WordCountServicePkg


ApplicationName       : fabric:/WordCount
ServiceManifestName   : WordCountServicePkg
NodeName              : _Node_2
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.Hosting
                        Property              : Activation
                        HealthState           : Ok
                        SequenceNumber        : 131031502301306211
                        SentAt                : 3/22/2016 7:57:10 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The ServicePackage was activated successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.Hosting
                        Property              : CodePackageActivation:Code:EntryPoint
                        HealthState           : Ok
                        SequenceNumber        : 131031502301568982
                        SentAt                : 3/22/2016 7:57:10 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The CodePackage was activated successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.Hosting
                        Property              : ServiceTypeRegistration:WordCountServiceType
                        HealthState           : Ok
                        SequenceNumber        : 131031502314788519
                        SentAt                : 3/22/2016 7:57:11 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The ServiceType was registered successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Você pode obter a integridade do pacote de serviço implantado com uma [solicitação GET](https://msdn.microsoft.com/library/azure/dn707677.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/dn707689.aspx), que inclui as políticas de integridade descritas no corpo.

## <a name="health-chunk-queries"></a>Consultas de integridade em blocos
As consultas de integridade em bloco podem retornar os filhos de cluster em vários níveis (recursivamente), de acordo com os filtros de entrada. Elas dão suporte a filtros avançados que permitem uma grande flexibilidade para expressar quais filhos específicos devem retornar, identificados por seu identificador exclusivo ou outro identificador de grupo e/ou estado de integridade. Por padrão, nenhum filho é incluído, ao contrário dos comandos de integridade que incluem sempre o filho de primeiro nível.

As [consultas de integridade](service-fabric-view-entities-aggregated-health.md#health-queries) retornam apenas os filhos de primeiro nível da entidade especificada, de acordo com os filtros necessários. Para obter os filhos dos filhos, você deve chamar outras APIs de integridade para cada entidade de interesse. Da mesma forma, para obter a integridade de entidades específicas, você deve chamar uma API e integridade para cada entidade desejada. A filtragem avançada da consulta em bloco permite a você solicitar vários itens de interesse em uma consulta, reduzindo o tamanho da mensagem e o número de mensagens.

O valor da consulta em blocos é que você pode obter o estado da integridade para mais entidades de cluster (potencialmente todas as entidades de cluster começando na raiz exigida) em uma chamada. Você pode expressar uma consulta de integridade complexa da seguinte maneira:

* Retorne apenas os aplicativos com erro e, para esses aplicativos, inclua todos os serviços em warning|error. Para os serviços retornados, inclua todas as partições.
* Retorne apenas a integridade dos quatro aplicativos, especificados por seus nomes.
* Retorne apenas a integridade dos aplicativos de um tipo de aplicativo desejado.
* Retorne todas as entidades implantadas em um nó. Retorna todos os aplicativos, todos os aplicativos implantados no nó especificado e todos os pacotes de serviço implantados nesse nó.
* Retorna todas as réplicas com erro. Retorna todos os aplicativos, serviços, partições e somente réplicas com erro.
* Retorna todos os aplicativos. Para um serviço específico, inclua todas as partições.

Atualmente, a consulta de integridade em blocos é exposta somente para a entidade do cluster. Ela retorna uma parte da integridade do cluster, que contém:

* O estado de integridade agregado do cluster.
* A lista de nós da parte de estado de integridade que respeita os filtros de entrada.
* A lista de aplicativos da parte de estado de integridade que respeita os filtros de entrada. Cada parte do estado de integridade do aplicativo contém uma lista com todos os serviços que respeitam os filtros de entrada e uma lista com todos os aplicativos implantados que respeitam os filtros. O mesmo vale para os filhos dos serviços e aplicativos implantados. Dessa forma, todas as entidades no cluster podem retornar caso solicitado, de uma maneira hierárquica.

### <a name="cluster-health-chunk-query"></a>Consulta em bloco sobre a integridade do cluster
Retorna a integridade da entidade do cluster e contém as partes hierárquicas do estado de integridade dos filhos necessários. Entrada:

* [Opcional] A política de integridade do cluster usada para avaliar os nós e os eventos de cluster.
* [Opcional] Mapa da política de integridade do aplicativo, com políticas de integridade usadas para substituir as políticas do manifesto do aplicativo.
* [Opcional] Filtros de eventos, nós e aplicativos que especificam quais entradas são interessantes e devem retornar no resultado. Os filtros são específicos a uma entidade/grupo de entidades ou são aplicáveis a todas as entidades nesse nível. A lista de filtros pode conter um filtro geral e/ou filtros para identificadores específicos, a fim de refinar as entidades retornadas pela consulta. Se estiver vazia, os filhos não retornarão por padrão.
  Leia mais sobre os filtros em [NodeHealthStateFilter](https://msdn.microsoft.com/library/azure/system.fabric.health.nodehealthstatefilter.aspx) e [ApplicationHealthStateFilter](https://msdn.microsoft.com/library/azure/system.fabric.health.applicationhealthstatefilter.aspx). O filtros de aplicativo pode especificar de forma recursiva os filtros avançados para os filhos.

O resultado da parte inclui os filhos que respeitam os filtros.

Atualmente, a consulta em bloco não retorna avaliações não íntegras ou eventos de entidade. Essas informações extras podem ser obtidas com a consulta de integridade de cluster existente.

### <a name="api"></a>API
Para obter a parte da integridade do cluster, crie um `FabricClient` e chame o método [GetClusterHealthChunkAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getclusterhealthchunkasync.aspx) em seu **HealthManager**. Você pode passar [ClusterHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.clusterhealthchunkquerydescription.aspx) para descrever as políticas de integridade e os filtros avançados.

O código a seguir obtém parte da integridade do cluster com filtros avançados.

```csharp
var queryDescription = new ClusterHealthChunkQueryDescription();
queryDescription.ApplicationFilters.Add(new ApplicationHealthStateFilter()
    {
        // Return applications only if they are in error
        HealthStateFilter = HealthStateFilter.Error
    });

// Return all replicas
var wordCountServiceReplicaFilter = new ReplicaHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };

// Return all replicas and all partitions
var wordCountServicePartitionFilter = new PartitionHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };
wordCountServicePartitionFilter.ReplicaFilters.Add(wordCountServiceReplicaFilter);

// For specific service, return all partitions and all replicas
var wordCountServiceFilter = new ServiceHealthStateFilter()
{
    ServiceNameFilter = new Uri("fabric:/WordCount/WordCountService"),
};
wordCountServiceFilter.PartitionFilters.Add(wordCountServicePartitionFilter);

// Application filter: for specific application, return no services except the ones of interest
var wordCountApplicationFilter = new ApplicationHealthStateFilter()
    {
        // Always return fabric:/WordCount application
        ApplicationNameFilter = new Uri("fabric:/WordCount"),
    };
wordCountApplicationFilter.ServiceFilters.Add(wordCountServiceFilter);

queryDescription.ApplicationFilters.Add(wordCountApplicationFilter);

var result = await fabricClient.HealthManager.GetClusterHealthChunkAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
O cmdlet para obter a integridade do cluster é [Get-ServiceFabricClusterChunkHealth](https://msdn.microsoft.com/library/mt644772.aspx). Primeiro, conecte-se ao cluster usando o cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx) .

O código a seguir obterá nós apenas se eles apresentarem Erro, exceto para um nó específico, que sempre deverá ser retornado.

```xml
PS C:\> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$nodeFilter1 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{HealthStateFilter=$errorFilter}
$nodeFilter2 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{NodeNameFilter="_Node_1";HealthStateFilter=$allFilter}
# Create node filter list that will be passed in the cmdlet
$nodeFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.NodeHealthStateFilter]
$nodeFilters.Add($nodeFilter1)
$nodeFilters.Add($nodeFilter2)

Get-ServiceFabricClusterHealthChunk -NodeFilters $nodeFilters

HealthState                  : Error
NodeHealthStateChunks        :
                               TotalCount            : 1

                               NodeName              : _Node_1
                               HealthState           : Ok

ApplicationHealthStateChunks : None
```

O cmdlet a seguir obtém parte do cluster com os filtros de aplicativo.

```xml
$errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

# All replicas
$replicaFilter = New-Object System.Fabric.Health.ReplicaHealthStateFilter -Property @{HealthStateFilter=$allFilter}

# All partitions
$partitionFilter = New-Object System.Fabric.Health.PartitionHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$partitionFilter.ReplicaFilters.Add($replicaFilter)

# For WordCountService, return all partitions and all replicas
$svcFilter1 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{ServiceNameFilter="fabric:/WordCount/WordCountService"}
$svcFilter1.PartitionFilters.Add($partitionFilter)

$svcFilter2 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{HealthStateFilter=$errorFilter}

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{ApplicationNameFilter="fabric:/WordCount"}
$appFilter.ServiceFilters.Add($svcFilter1)
$appFilter.ServiceFilters.Add($svcFilter2)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)

Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters

HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks :
                               TotalCount            : 1

                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               ServiceHealthStateChunks :
                                   TotalCount            : 1

                                   ServiceName           : fabric:/WordCount/WordCountService
                                   HealthState           : Error
                                   PartitionHealthStateChunks :
                                       TotalCount            : 1

                                       PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
                                       HealthState           : Error
                                       ReplicaHealthStateChunks :
                                           TotalCount            : 5

                                           ReplicaOrInstanceId   : 131031502143040223
                                           HealthState           : Ok

                                           ReplicaOrInstanceId   : 131031502346844060
                                           HealthState           : Ok

                                           ReplicaOrInstanceId   : 131031502346844059
                                           HealthState           : Ok

                                           ReplicaOrInstanceId   : 131031502346844061
                                           HealthState           : Ok

                                           ReplicaOrInstanceId   : 131031502346844058
                                           HealthState           : Error
```

O cmdlet a seguir retorna todas as entidades implantadas em um nó.

```xml
$errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$dspFilter = New-Object System.Fabric.Health.DeployedServicePackageHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$daFilter =  New-Object System.Fabric.Health.DeployedApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter;NodeNameFilter="_Node_2"}
$daFilter.DeployedServicePackageFilters.Add($dspFilter)

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$appFilter.DeployedApplicationFilters.Add($daFilter)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)
Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks :
                               TotalCount            : 2

                               ApplicationName       : fabric:/System
                               HealthState           : Ok
                               DeployedApplicationHealthStateChunks :
                                   TotalCount            : 1

                                   NodeName              : _Node_2
                                   HealthState           : Ok
                                   DeployedServicePackageHealthStateChunks :
                                       TotalCount            : 1

                                       ServiceManifestName   : FAS
                                       HealthState           : Ok



                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               DeployedApplicationHealthStateChunks :
                                   TotalCount            : 1

                                   NodeName              : _Node_2
                                   HealthState           : Ok
                                   DeployedServicePackageHealthStateChunks :
                                       TotalCount            : 2

                                       ServiceManifestName   : WordCountServicePkg
                                       HealthState           : Ok

                                       ServiceManifestName   : WordCountWebServicePkg
                                       HealthState           : Ok
```

### <a name="rest"></a>REST
Você pode obter parte da integridade do cluster com uma [solicitação GET](https://msdn.microsoft.com/library/azure/mt656722.aspx) ou uma [solicitação POST](https://msdn.microsoft.com/library/azure/mt656721.aspx), que inclui os filtros avançados e políticas de integridade descritos no corpo.

## <a name="general-queries"></a>Consultas gerais
As consultas gerais retornam a lista de entidades do Service Fabric de um tipo especificado. Elas são expostas por meio da API (métodos em **FabricClient.QueryManager**), dos cmdlets do PowerShell e da REST. Essas consultas agregam subconsultas de vários componentes. Um deles é o [Repositório de Integridade](service-fabric-health-introduction.md#health-store), que preenche o estado de integridade agregada para cada resultado da consulta.  

> [!NOTE]
> As consultas gerais retornam o estado da integridade da entidade agregada e não contêm dados de integridade avançados. Se uma entidade não estiver íntegra, você poderá acompanhá-la com consultas de integridade a fim de obter todas as informações de integridade, incluindo eventos, estados da integridade dos filhos e avaliações não íntegras.
>
>

Se as consultas gerais retornarem um estado de integridade desconhecido para uma entidade, talvez o repositório de integridade não tenha dados completos sobre a entidade. Também é possível que uma subconsulta ao repositório de integridade não tenha êxito (por exemplo, ocorreu um erro de comunicação ou repositório de integridade apresentava alguma restrição). Acompanhe uma consulta de integridade da entidade. Se a subconsulta tiver encontrado erros transitórios, como problemas de rede, essa consulta de acompanhamento poderá ser bem-sucedida. Ela também pode fornecer mais detalhes, com base no repositório de integridade, sobre o motivo de a entidade não ser exposta.

As consultas que contêm **HealthState** para entidades são:

* Lista de nós: retorna os nós da lista no cluster (paginados).
  * API: [FabricClient.QueryClient.GetNodeListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getnodelistasync.aspx)
  * PowerShell: Get-ServiceFabricNode
* Lista de aplicativos: retorna a lista de aplicativos no cluster (paginados).
  * API: [FabricClient.QueryClient.GetApplicationListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getapplicationlistasync.aspx)
  * PowerShell: Get-ServiceFabricApplication
* Lista de serviços: retorna a lista de serviços em um aplicativo (paginados).
  * API: [FabricClient.QueryClient.GetServiceListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getservicelistasync.aspx)
  * PowerShell: Get-ServiceFabricService
* Lista de partições: retorna a lista de partições em um serviço (paginadas).
  * API: [FabricClient.QueryClient.GetPartitionListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getpartitionlistasync.aspx)
  * PowerShell: Get-ServiceFabricPartition
* Lista de réplicas: retorna a lista de réplicas em uma partição (paginadas).
  * API: [FabricClient.QueryClient.GetReplicaListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getreplicalistasync.aspx)
  * PowerShell: Get-ServiceFabricReplica
* Lista de aplicativos implantados: retorna a lista de aplicativos implantados em um nó.
  * API: [FabricClient.QueryClient.GetDeployedApplicationListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getdeployedapplicationlistasync.aspx)
  * PowerShell: Get-ServiceFabricDeployedApplication
* Lista de pacotes de serviço implantados: retorna a lista de pacotes de serviço em um aplicativo implantado.
  * API: [FabricClient.QueryClient.GetDeployedServicePackageListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync.aspx)
  * PowerShell: Get-ServiceFabricDeployedApplication

> [!NOTE]
> Algumas das consultas retornam resultados paginados. O retorno dessas consultas é uma lista derivada de [PagedList<T>](https://msdn.microsoft.com/library/azure/mt280056.aspx). Se os resultados não couberem em uma mensagem, serão retornados apenas uma página e um ContinuationToken que acompanha onde a enumeração parou. Você deve continuar a chamar a mesma consulta e passar o token de continuação da consulta anterior a fim de obter os próximos resultados.
>
>

### <a name="examples"></a>Exemplos
O código a seguir obtém os aplicativos não íntegros no cluster:

```csharp
var applications = fabricClient.QueryManager.GetApplicationListAsync().Result.Where(
  app => app.HealthState == HealthState.Error);
```

O cmdlet a seguir obtém detalhes de aplicativo para o aplicativo fabric:/WordCount. Observe que o estado de integridade é de aviso.

```powershell
PS C:\> Get-ServiceFabricApplication -ApplicationName fabric:/WordCount

ApplicationName        : fabric:/WordCount
ApplicationTypeName    : WordCount
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Warning
ApplicationParameters  : { "WordCountWebService_InstanceCount" = "1";
                         "_WFDebugParams_" = "[{"ServiceManifestName":"WordCountWebServicePkg","CodePackageName":"Code","EntryPointType":"Main","Debug
                         ExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {74f7e5d5-71a9-47e2-a8cd-1878ec4734f1} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"},{"ServiceManifestName":"WordCountServicePkg","CodeP
                         ackageName":"Code","EntryPointType":"Main","DebugExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {2ab462e6-e0d1-4fda-a844-972f561fe751} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"}]" }
```

O cmdlet a seguir obtém os serviços com o estado de integridade de aviso.

```powershell
PS C:\> Get-ServiceFabricApplication | Get-ServiceFabricService | where {$_.HealthState -eq "Warning"}


ServiceName            : fabric:/WordCount/WordCountService
ServiceKind            : Stateful
ServiceTypeName        : WordCountServiceType
IsServiceGroup         : False
ServiceManifestVersion : 1.0.0
HasPersistedState      : True
ServiceStatus          : Active
HealthState            : Warning
```

## <a name="cluster-and-application-upgrades"></a>Atualização de cluster e aplicativo
Durante a atualização monitorada do cluster e do aplicativo, o Service Fabric verifica a integridade para garantir que tudo esteja íntegro e permaneça assim. Se uma entidade não estiver íntegra, conforme avaliação por meio de políticas de integridade, a atualização aplica políticas específicas à atualização para determinar a próxima ação. A atualização pode ser pausada para permitir a interação do usuário (por exemplo, corrigir condições de erro ou alterar políticas), ou pode ser revertida automaticamente para a versão anterior em boa qualidade.

Durante uma atualização de *cluster* , você pode obter o status de atualização do cluster. O status de atualização inclui quaisquer avaliações não íntegras, que apontam para o que não está íntegro no cluster. Se a atualização for revertida devido a problemas de integridade, o status da atualização se lembrará dos últimos motivos não íntegros. Essas informações podem ajudar os administradores a investigar o que deu errado após a atualização ser revertida ou interrompida.

Da mesma forma, durante uma atualização do *aplicativo* , quaisquer avaliações não íntegras ficam contidas no status de atualização do aplicativo.

O exemplo a seguir mostra o status da atualização do aplicativo para um aplicativo fabric:/WordCount modificado. Um watchdog relatou um erro em uma de suas réplicas. A atualização está sendo revertida porque as verificações de integridade não são respeitadas.

```powershell
PS C:\> Get-ServiceFabricApplicationUpgrade fabric:/WordCount

ApplicationName               : fabric:/WordCount
ApplicationTypeName           : WordCount
TargetApplicationTypeVersion  : 1.0.0.0
ApplicationParameters         : {}
StartTimestampUtc             : 4/21/2015 5:23:26 PM
FailureTimestampUtc           : 4/21/2015 5:23:37 PM
FailureReason                 : HealthCheck
UpgradeState                  : RollingBackInProgress
UpgradeDuration               : 00:00:23
CurrentUpgradeDomainDuration  : 00:00:00
CurrentUpgradeDomainProgress  : UD1

                                NodeName            : _Node_1
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_2
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_3
                                UpgradePhase        : PreUpgradeSafetyCheck
                                PendingSafetyChecks :
                                EnsurePartitionQuorum - PartitionId: 30db5be6-4e20-4698-8185-4bd7ca744020
NextUpgradeDomain             : UD2
UpgradeDomainsStatus          : { "UD1" = "Completed";
                                "UD2" = "Pending";
                                "UD3" = "Pending";
                                "UD4" = "Pending" }
UnhealthyEvaluations          :
                                Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                      Unhealthy partition: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b', AggregatedHealthState='Error'.

                                          Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                          Unhealthy replica: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b',
                                  ReplicaOrInstanceId='131031502346844058', AggregatedHealthState='Error'.

                                              Error event: SourceId='DiskWatcher', Property='Disk'.

UpgradeKind                   : Rolling
RollingUpgradeMode            : UnmonitoredAuto
ForceRestart                  : False
UpgradeReplicaSetCheckTimeout : 00:15:00
```

Leia mais sobre a [Atualização de aplicativo do Service Fabric](service-fabric-application-upgrade.md).

## <a name="use-health-evaluations-to-troubleshoot"></a>Usar avaliações de integridade para solucionar problemas
Sempre que houver um problema no cluster ou aplicativo, observe a integridade do cluster ou do aplicativo para identificar o que está errado. As avaliações não íntegras mostram detalhes sobre o que disparou o estado não íntegro atual. Se for necessário, você poderá analisar entidades filho não íntegras para identificar a causa raiz.

> [!NOTE]
> As avaliações não íntegras mostram o primeiro motivo pelo qual a entidade é avaliada com o estado de integridade atual. Pode haver vários outros eventos que disparam esse estado, mas eles não são mostrados nas avaliações. Para obter mais informações, faça uma busca detalhada das entidades de integridade para descobrir todos os relatórios não íntegros no cluster.
>
>

## <a name="next-steps"></a>Próximas etapas
[Usar relatórios de integridade do sistema para solução de problemas](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)

[Adicionar relatórios de integridade personalizados do Service Fabric](service-fabric-report-health.md)

[Como relatar e verificar a integridade do serviço](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

[Monitorar e diagnosticar serviços localmente](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Atualização de aplicativos do Service Fabric](service-fabric-application-upgrade.md)



<!--HONumber=Jan17_HO2-->


