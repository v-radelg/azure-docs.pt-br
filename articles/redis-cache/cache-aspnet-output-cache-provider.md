---
title: "Provedor de Cache de Saída ASP.NET do Cache"
description: "Saiba como armazenar em cache a saída de página ASP.NET usando o Cache Redis do Azure"
services: redis-cache
documentationcenter: na
author: steved0x
manager: douge
editor: tysonn
ms.assetid: 78469a66-0829-484f-8660-b2598ec60fbf
ms.service: cache
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: cache-redis
ms.workload: tbd
ms.date: 02/14/2017
ms.author: sdanie
translationtype: Human Translation
ms.sourcegitcommit: 70341f4a14ee807a085931c3480a19727683e958
ms.openlocfilehash: ce0f2ddb42e19ee33767878797188e924f5cd1e9
ms.lasthandoff: 02/17/2017


---
# <a name="aspnet-output-cache-provider-for-azure-redis-cache"></a>Provedor de Cache de Saída ASP.NET do Cache Redis do Azure
O Provedor de Cache de Saída Redis é um mecanismo de armazenamento fora do processo para dados do cache de saída. Esses dados são especificamente para respostas HTTP completas (cache de saída de página). O provedor conecta-se ao novo saída cache provedor ponto de extensibilidade que foi introduzido no ASP.NET 4.

Para usar o Provedor de Cache de Saída Redis, primeiro configure seu cache e, em seguida, configure seu aplicativo ASP.NET usando o pacote NuGet do Provedor de Cache de Saída Redis. Este tópico fornece orientação sobre como configurar seu aplicativo para usar o Provedor de Cache de Saída Redis. Para saber mais sobre como criar e configurar uma instância do Cache Redis do Azure, consulte [Criar um cache](cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).

## <a name="store-aspnet-page-output-in-the-cache"></a>Armazenar a saída da página ASP.NET no cache
Para configurar um aplicativo cliente no Visual Studio usando o pacote NuGet do Provedor de Cache de Saída Redis, clique com o botão direito no projeto em **Gerenciador de Soluções** e escolha **Gerenciar Pacotes NuGet**.

![Gerenciamento de pacotes NuGet pelo Cache Redis do Azure](./media/cache-aspnet-output-cache-provider/redis-cache-manage-nuget-menu.png)

Digite **RedisOutputCacheProvider** na caixa de texto de pesquisa, selecione-o nos resultados e clique em **Instalar**.

![Provedor de Cache de Saída do Cache Redis do Azure](./media/cache-aspnet-output-cache-provider/redis-cache-page-output-provider.png)

O pacote NuGet do Provedor de Cache de Saída Redis tem uma dependência do pacote StackExchange.Redis.StrongName. Se o pacote StackExchange.Redis.StrongName não estiver presente em seu projeto, ele será instalado.

>[!NOTE]
>Além do pacote StackExchange.Redis.StrongName de nome forte, também há a versão do StackExchange.Redis sem nome forte. Se o seu projeto estiver usando a versão StackExchange.Redis sem um nome forte, será necessário desinstalá-la, caso contrário você terá conflitos de nomenclatura em seu projeto. Para saber mais sobre esses pacotes, consulte [Configurar clientes de cache .NET](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients).
>
>

O pacote do NuGet baixa e adiciona as referências de assembly necessárias e adiciona a seção a seguir ao seu arquivo web.config. Esta seção contém a configuração necessária para seu aplicativo ASP.NET usar o Provedor de Cache de Saída Redis.

```xml
<caching>
  <outputCachedefault Provider="MyRedisOutputCache">
    <providers>
      <!--
      <add name="MyRedisOutputCache"
        host = "127.0.0.1" [String]
        port = "" [number]
        accessKey = "" [String]
        ssl = "false" [true|false]
        databaseId = "0" [number]
        applicationName = "" [String]
        connectionTimeoutInMilliseconds = "5000" [number]
        operationTimeoutInMilliseconds = "5000" [number]
      />
      -->
      <add name="MyRedisOutputCache" type="Microsoft.Web.Redis.RedisOutputCacheProvider" host="127.0.0.1" accessKey="" ssl="false"/>
    </providers>
  </outputCache>
</caching>
```

A seção comentada fornece um exemplo dos atributos e exemplos de configurações para cada atributo.

Configure os atributos com os valores de sua folha de cache no Portal do Microsoft Azure e configure os outros valores conforme o desejado. Para obter instruções sobre como acessar as propriedades do cache, consulte [Definir as configurações de cache Redis](cache-configure.md#configure-redis-cache-settings).

* **host** – especifique o ponto de extremidade do cache.
* **porta** – use sua porta SSL ou sua porta que não é do tipo SSL, dependendo das configurações de ssl.
* **accessKey** – use a chave primária ou secundária de seu cache.
* **ssl** – true se você quiser proteger as comunicações de cache/cliente com ssl; caso contrário, false. Especifique a porta correta.
  * A porta não SSL é desabilitada por padrão para novos caches. Especifique true para essa configuração a fim de usar a porta SSL. Para saber mais sobre como habilitar a porta que não é do tipo SSL, confira a seção [Portas de acesso](cache-configure.md#access-ports) no tópico [Configurar um cache](cache-configure.md).
* **databaseId** – especifica qual banco de dados usar para os dados de saída do cache. Se esse campo não for especificado, o valor padrão de 0 será usado.
* **applicationName** – As chaves são armazenadas em redis como `<AppName>_<SessionId>_Data`. Esse esquema de nomenclatura permite que vários aplicativos compartilhem a mesma chave. Esse parâmetro é opcional e, se você não fornecê-lo, um valor padrão será usado.
* **connectionTimeoutInMilliseconds** – essa configuração permite a substituição da configuração de connectTimeout no cliente StackExchange.Redis. Se ela não for especificada, a configuração padrão do connectTimeout, 5000, será usada. Para saber mais, consulte [Modelo de configuração StackExchange.Redis](http://go.microsoft.com/fwlink/?LinkId=398705).
* **operationTimeoutInMilliseconds** – essa configuração permite a substituição da configuração de syncTimeout no cliente StackExchange.Redis. Se ela não for especificada, a configuração padrão de syncTimeout, 1000, será usada. Para saber mais, consulte [Modelo de configuração StackExchange.Redis](http://go.microsoft.com/fwlink/?LinkId=398705).

Adicione uma diretiva OutputCache a cada página da qual você deseja armazenar a saída em cache.

```
<%@ OutputCache Duration="60" VaryByParam="*" %>
```

No exemplo anterior, os dados da página em cache permanecem no cache por 60 segundos, e outra versão da página será armazenada em cache para cada combinação de parâmetros. Para saber mais sobre a diretiva OutputCache, consulte [@OutputCache](http://go.microsoft.com/fwlink/?linkid=320837).

Após a execução dessas etapas, seu aplicativo será configurado para usar o Provedor de Cache de Saída Redis.

## <a name="next-steps"></a>Próximas etapas
Confira [Provedor de estado de sessão ASP.NET para Cache Redis do Azure](cache-aspnet-session-state-provider.md)


