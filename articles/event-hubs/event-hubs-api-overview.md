---
title: "Visão geral da API dos Hubs de Eventos do Azure | Microsoft Docs"
description: "Visão geral das APIs de Hubs de Eventos do Azure disponíveis"
services: event-hubs
documentationcenter: na
author: jtaubensee
manager: timlt
editor: 
ms.assetid: 3f221a0c-182d-4e39-9f3d-3a3c16c5c6ed
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/31/2017
ms.author: jotaub
translationtype: Human Translation
ms.sourcegitcommit: aa7244849f6286e8ef9f9785c133b4c326193c12
ms.openlocfilehash: 5a360462288e5df6e0ede5f11adabba158a9dd57

---

# <a name="available-event-hubs-apis"></a>APIs de Hubs de Eventos disponíveis

## <a name="runtime-apis"></a>APIs de tempo de execução

A seguir, uma lista de todos os clientes de tempo de execução dos Hubs de Eventos disponíveis no momento. Embora algumas dessas bibliotecas também incluem a funcionalidade de gerenciamento limitado, também há [bibliotecas específicas](#management-apis) dedicada às operações de gerenciamento. O foco principal dessas bibliotecas é enviar e receber mensagens de um Hub de Eventos.

Veja [informações adicionais](#additional-information) para obter mais detalhes sobre o status atual de cada biblioteca de tempo de execução.

| Linguagem/plataforma | Pacote de cliente | Pacote EventProcessorHost | Repositório |
| --- | --- | --- | --- |
| .NET Standard | [NuGet](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) | [NuGet](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor/) | [GitHub](https://github.com/azure/azure-event-hubs-dotnet) |
| .NET Framework | [NuGet](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) | [NuGet](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/) | N/D |
| Java | [Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22) | [Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22) | [GitHub](https://github.com/Azure/azure-event-hubs-java) |
| Nó | [NPM](https://www.npmjs.com/package/azure-event-hubs) | N/D | [GitHub](https://github.com/Azure/azure-event-hubs-node) |
| C | N/D | N/D | [GitHub](https://github.com/Azure/azure-event-hubs-c) |

### <a name="additional-information"></a>Informações adicionais

#### <a name="net"></a>.NET
O ecossistema do .NET tem vários tempos de execução, portanto, há várias bibliotecas .NET de Hubs de Eventos. A biblioteca .NET Standard pode ser executada usando o .NET Core ou o .NET Framework, enquanto a biblioteca do .NET Framework só pode ser executada em um ambiente do .NET Framework. Para saber mais sobre o .NET Framework, veja [versões do framework.](https://docs.microsoft.com/dotnet/articles/standard/frameworks#framework-versions)

#### <a name="node"></a>Nó

A biblioteca do Node. js está atualmente em visualização e é mantida como um projeto de lado por funcionários da Microsoft e colaboradores externos. Todas as contribuições, incluindo código-fonte são bem-vindas e serão analisadas.

## <a name="management-apis"></a>APIs de gerenciamento

A seguir está uma lista de todas as bibliotecas específicas de gerenciamento disponíveis no momento. Nenhuma dessas bibliotecas contém operações de tempo de execução e elas têm o único propósito de gerenciar as entidades de Hubs de Eventos.

| Linguagem/plataforma | Pacote de gerenciamento | Repositório |
| --- | --- | --- | --- |
| .NET Standard | [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Management.EventHub) | [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/ResourceManagement/EventHub) |

## <a name="next-steps"></a>Próximas etapas
Você pode saber mais sobre Hubs de Eventos visitando os links abaixo:

* [Visão Geral dos Hubs de Eventos](event-hubs-what-is-event-hubs.md)
* [Criar um Hub de Eventos](event-hubs-create.md)
* [Perguntas frequentes sobre os Hubs de Eventos](event-hubs-faq.md)


<!--HONumber=Feb17_HO1-->


