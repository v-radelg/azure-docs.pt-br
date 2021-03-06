---
title: "Máquinas virtuais de ciência de dados no Azure | Microsoft Docs"
description: "Configurar uma Máquina Virtual de ciência de dados"
services: machine-learning
documentationcenter: 
author: bradsev
manager: jhubbard
editor: cgronlun
ms.assetid: 95e1fa87-794a-4d03-80a4-af4f3f3ac31e
ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/19/2016
ms.author: xibingao;bradsev
translationtype: Human Translation
ms.sourcegitcommit: a6bc79b2cb5b73109cddd6cf57caeba754b52e2e
ms.openlocfilehash: 4de0683f2f58cc598a5c37e74a30b7e17aa7fd0d


---
# <a name="data-science-virtual-machines-in-azure"></a>Máquinas virtuais de ciência de dados no Azure
São fornecidas aqui instruções que descrevem como configurar uma VM do Azure e uma VM do Azure com o serviço do SQL como servidores IPython Notebook. A máquina virtual do Windows está configurada com ferramentas de suporte, como o IPython Notebook, o Gerenciador de Armazenamento do Azure e o AzCopy, bem como outros utilitários que são úteis para projetos de ciência de dados. O Azure Storage Explorer e o AzCopy, por exemplo, fornecem maneiras convenientes para carregar dados no armazenamento do Azure em seu computador local ou baixá-lo em seu computador local por meio do armazenamento. 

Esse menu leva a tópicos que descrevem como configurar os diversos ambientes de ciência de dados usados pelo [TDSP (Processo de Ciência de Dados de Equipe)](data-science-process-overview.md).

[!INCLUDE [data-science-environment-setup](../../includes/cap-setup-environments.md)]

Vários tipos de máquinas virtuais do Azure podem ser provisionados e configurados para serem usados como parte de um ambiente de ciência de dados baseado em nuvem. A decisão sobre qual tipo de máquina virtual usar depende do tipo e da quantidade de dados que serão modelados com o aprendizado de máquina e do destino de tais dados na nuvem. 

* Para obter diretrizes sobre as perguntas a serem consideradas ao tomar essa decisão, consulte [Planejar seu ambiente de ciência de dados do Aprendizado de Máquina do Azure](machine-learning-data-science-plan-your-environment.md). 
* Para um catálogo de alguns dos cenários que você pode encontrar ao fazer análises avançadas, consulte [Cenários para o processo de análise avançada e tecnologia no Aprendizado de Máquina do Azure](machine-learning-data-science-plan-sample-scenarios.md)

São fornecidos dois conjuntos de instruções:

* [Configurar uma máquina virtual do Azure como um servidor do IPython Notebook para análise avançada](machine-learning-data-science-setup-virtual-machine.md) mostra como provisionar uma máquina virtual do Azure com o IPython Notebook e outras ferramentas usadas para executar o processo de ciência de dados para casos em que uma forma de armazenamento do Azure, com a exceção de SQL, pode ser usada para armazenar os dados.
* [Configurar uma máquina virtual do Azure SQL Server como um servidor do IPython Notebook para análise avançada](machine-learning-data-science-setup-sql-server-virtual-machine.md) mostra como provisionar uma máquina virtual do Azure SQL Server com o IPython Notebook e outras ferramentas usadas para executar o processo de ciência de dados para casos em que um banco de dados SQL pode ser usado para armazenar os dados.

Uma vez provisionadas e configuradas, essas máquinas virtuais estão prontas para uso como servidores IPython Notebook para a exploração e o processamento de dados, bem como para outras tarefas necessárias em conjunto com o Aprendizado de Máquina do Azure e o TDSP (Processo de Ciência de Dados de Equipe). As próximas etapas no processo de ciência de dados são mapeadas no [Roteiro de aprendizagem do TDSP](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/) e podem incluir etapas que movem dados para o SQL Server ou HDInsight, que os processam e obtêm amostras deles como parte da preparação para o aprendizado com base nos dados com o Aprendizado de Máquina do Azure.

> [!NOTE]
> A cobrança das máquinas virtuais do Azure ocorre na forma **pague somente pelo que usa**. Para garantir que você não está sendo cobrado quando não estiver usando sua máquina virtual, ela deverá estar no estado **Parado (Desalocado)** do [Portal Clássico do Azure](http://manage.windowsazure.com/). Para obter instruções passo a passo ou como desalocar a sua máquina virtual, veja [Desligar e desalocar máquina virtual quando não estiver em uso](machine-learning-data-science-setup-virtual-machine.md#shutdown)
> 
> 




<!--HONumber=Dec16_HO3-->


