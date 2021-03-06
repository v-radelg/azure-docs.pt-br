---
title: "Configurar e usar a API de recomendações de Machine Learning | Microsoft Docs"
description: "API de RECOMENDAÇÕES da Microsoft criada com as perguntas Frequentes do aprendizado de máquina do Azure"
services: machine-learning
documentationcenter: 
author: LuisCabrer
manager: jhubbard
editor: cgronlun
ms.assetid: 1ffc3c16-e040-4225-9d72-105129938dfa
ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/08/2016
ms.author: luisca
translationtype: Human Translation
ms.sourcegitcommit: 9ac2a1bf5987fc6fc30e20a1b4a10340eeed3825
ms.openlocfilehash: 15cf891b9c31e1f2e2b108e631f32883ce2f36a7


---
# <a name="setting-up-and-using-machine-learning-recommendations-api-faq"></a>Perguntas frequentes sobre configuração e uso da API de Recomendações do Aprendizado de Máquina
**O que são as RECOMENDAÇÕES?**

> [!NOTE]
> Você deve começar a usar o Serviço Cognitivo da API de Recomendações em vez desta versão. O Serviço Cognitivo de Recomendações substituirá esse serviço, e todos os recursos novos serão desenvolvidos lá. Ele possui novos recursos como suporte ao processamento em lotes, um Gerenciador de API aprimorado, uma superfície de API mais limpa, uma experiência de inscrição/cobrança mais consistente etc.
> Saiba mais sobre [Como migrar para o novo Serviço Cognitivo](http://aka.ms/recomigrate)
> 
> 

Para organizações e empresas que utilizam recomendações para venda cruzada e venda de produtos e serviços aos clientes, as RECOMENDAÇÕES no Aprendizado de Máquina do Azure fornecem um mecanismo de recomendações de autoatendimento. É uma implementação de filtragem  colaborativa que usa a fatoração de matriz como seu algoritmo central. Os desenvolvedores de aplicativos podem acessar RECOMENDAÇÕES usando APIs REST. 

[!INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]

**O que posso fazer com as RECOMENDAÇÕES?**

RECOMENDAÇÕES utiliza como entrada um item ou um conjunto de itens e retorna uma lista de recomendações relevantes. Por exemplo: um cliente de uma loja online clica em um produto. A loja online envia esse produto como uma entrada para RECOMENDAÇÕES, obtém uma lista de produtos em troca e decide quais desses produtos serão mostrados para o cliente. Você talvez queira usar RECOMENDAÇÕES para otimizar sua loja online ou até mesmo informar seu departamento de vendas interno ou central de atendimento.

**Há limitações de uso?**

As Recomendações apresentam as seguintes limitações de uso:

* Número máximo de modelos por assinatura: 10
* Número máximo de itens que um catálogo pode conter: 100.000
* O número máximo de pontos de uso mantidos é cerca de&5;.000.000. Os mais antigos serão excluídos se novos forem carregados ou relatados.
* O volume máximo de dados que podem ser enviados no email (por exemplo, importar dados de catálogo e importar dados de uso) é de 200 MB.
* O número de TPS (transações por segundo) para um build de modelo da Recomendação que não está ativo é de cerca de&2; TPS. Uma compilação de modelo de Recomendação ativa pode conter até 20 TPS.

## <a name="purchase-and-billing"></a>Compra e cobrança
**Qual é o custo de Recomendações durante o período de lançamento?**

As Recomendações são um serviço por assinatura. A cobrança é feita com base no volume de transações por mês. Você pode consultar a [página de ofertas](https://datamarket.azure.com/dataset/amla/recommendations) no Microsoft Azure Marketplace para obter informações sobre preços.

**Existem custos associados a fazer as Recomendações acompanharem e armazenarem a atividade do usuário para mim?**

Não no momento.

**As Recomendações têm uma avaliação gratuita?**

Há uma versão de avaliação gratuita que é restrita a 10.000 transações por mês.

**Quando serei cobrado por Recomendações?**

Uma assinatura paga é qualquer assinatura para a qual há uma taxa mensal. Ao comprar uma assinatura paga, você é cobrado imediatamente pelo uso do primeiro mês. Você é cobrado pelo valor associado à oferta na página de assinatura (além de impostos aplicáveis). A cobrança mensal é feita todos os meses na mesma data de calendário que a da primeira compra até você cancelar a assinatura. 

**Como atualizar para um nível superior de serviço?**

Você pode comprar ou atualizar sua assinatura na [página de ofertas](https://datamarket.azure.com/dataset/amla/recommendations) no Microsoft Azure Marketplace.

Quando você atualiza uma assinatura:

* As transações restantes em sua assinatura antiga não são adicionadas à sua nova assinatura. 
* Você paga o preço total da nova assinatura, embora haja transações não utilizadas em sua antiga assinatura.

Processo para atualizar uma assinatura:

* Navegue até a [página de ofertas](https://datamarket.azure.com/dataset/amla/recommendations).
* Entre no Marketplace, se você ainda não tiver feito isso.
* No painel à direita, são listados todos os planos disponíveis. Clique no botão de opção do plano para o qual você deseja atualizar.
* Se você desejar atualizar, clique em **OK**. Se você não desejar atualizar, clique em **Cancelar**.

**Importante** Leia atentamente a caixa de diálogo antes de atualizar, pois há implicações de cobrança e uso.

**Quando minha assinatura de Recomendações será encerrada?**

Sua assinatura será encerrada quando você a cancelar. Se quiser cancelar suas assinaturas, consulte as instruções a seguir.

**Como cancelo minha assinatura de Recomendações?**

Para cancelar sua assinatura, use as etapas a seguir. Se sua assinatura atual for uma assinatura paga, ela continuará em vigor até o final do período de cobrança atual. Se for necessário que o cancelamento entre em vigor imediatamente, entre em contato conosco por meio do [Suporte da Microsoft](https://support.microsoft.com/oas/default.aspx?gprid=17024&st=1&wfxredirect=1&sd=gn).

**Observação** Nenhum reembolso será fornecido se você cancelar antes do fim de um período de cobrança ou para transações não utilizadas em um período de cobrança.

* Navegue até a [página de ofertas](https://datamarket.azure.com/dataset/amla/recommendations).
* Entre no Marketplace, se você ainda não tiver feito isso.
* Clique em **Cancelar** à direita do nome do conjunto de dados e status. Você pode usar essa assinatura até o fim do período de cobrança atual ou até o limite de transação ser atingido (o que ocorrer primeiro).

Se quiser cancelar sua assinatura imediatamente para poder comprar uma nova assinatura, crie um tíquete no [Suporte da Microsoft](https://support.microsoft.com/oas/default.aspx?gprid=17024&st=1&wfxredirect=1&sd=gn).

## <a name="getting-started-with-recommendations"></a>Introdução às Recomendações
**As Recomendações são para mim?** 

As Recomendações no Aprendizado de Máquina são para organizações e empresas que utilizam recomendações para venda cruzada e venda de produtos ou serviços a seus clientes. Se você tiver um site voltado para o cliente, uma equipe de vendas, uma equipe de vendas interna ou um call center e se oferecer um catálogo de mais de algumas dezenas de produtos ou serviços, seu resultado final poderá se beneficiar do uso de Recomendações. 

Experimentar as Recomendações é projetado para ser um processo bastante simples. A versão baseada em API atual requer habilidades de programação básicas. Caso você precise de assistência, contate o fornecedor que desenvolveu seu site. Se você tiver um departamento de TI interno ou desenvolvedor interno, eles poderão fazer com que as Recomendações funcionem para você. 

**Quais são os pré-requisitos para a configuração de recomendações?**

As Recomendações exigem que você tenha um log das opções de usuário no que diz respeito ao catálogo. Se você não tiver um log e tiver um site voltado para o cliente, as Recomendações poderão coletar a atividade de usuário para você. 

As Recomendações também exigem um catálogo de produtos ou serviços. Se você não tiver o catálogo, as Recomendações poderão utilizar os dados de uso de clientes reais e extrair um catálogo. Um catálogo implícito não incluirá itens que não tenham sido relatados como parte das transações de usuário.

**Como configurar as Recomendações pela primeira vez?**

Depois de [assinar](https://datamarket.azure.com/dataset/amla/recommendations) as Recomendações, você deve usar a documentação da API em [Recomendações do Azure Machine Learning – Guia de Início Rápido](machine-learning-recommendation-api-quick-start-guide.md) para configurar o serviço.

**Onde posso encontrar a documentação da API?** 

A documentação da API está em [Recomendações do Azure Machine Learning – Guia de Início Rápido](machine-learning-recommendation-api-quick-start-guide.md).

**Quais são as opções que tenho para carregar dados de uso e catálogo para Recomendações?**

Você tem duas opções para carregar os dados do catálogo e de uso: pode exportar os dados do seu sistema CRM ou outros logs e carregá-los nas Recomendações ou pode adicionar marcas a seu site para controlar as atividades do usuário. Se você usar o segundo método, os dados serão armazenados no Azure.

## <a name="maintenance-and-support"></a>Manutenção e suporte
**Que tamanho meu conjunto de dados pode ter?**

Cada conjunto de dados pode conter até 100.000 itens de catálogo e até 2048 MB de dados de uso.
Além disso, uma assinatura pode conter até 10 conjuntos de dados (modelos).

**Onde posso obter suporte técnico para Recomendações?**

O suporte técnico está disponível no site de [Suporte do Microsoft Azure](https://social.msdn.microsoft.com/forums/azure/home?forum=MachineLearning) .

**Onde posso encontrar os termos de uso?**

[Termos de Serviço da API de Recomendações do Aprendizado de Máquina do Microsoft Azure](https://datamarket.azure.com/dataset/amla/recommendations#terms).




<!--HONumber=Jan17_HO1-->


