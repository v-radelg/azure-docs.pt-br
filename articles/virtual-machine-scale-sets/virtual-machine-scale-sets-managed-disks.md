---
title: "Usar discos gerenciados com os conjuntos de dimensionamento de máquinas virtuais do Azure | Microsoft Docs"
description: "Saiba por que e como usar discos gerenciados com os conjuntos de dimensionamento de máquinas virtuais"
services: virtual-machine-scale-sets
documentationcenter: 
author: gatneil
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 76ac7fd7-2e05-4762-88ca-3b499e87906e
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 2/21/2017
ms.author: negat
translationtype: Human Translation
ms.sourcegitcommit: 9b2ef23effa32e9b7507ad6e6eb29e97609a2034
ms.openlocfilehash: e59e95c55beeb2af6c704bcaad11c8d622f4e853
ms.lasthandoff: 02/27/2017


---
# <a name="azure-vm-scale-sets-and-managed-disks"></a>Conjuntos de dimensionamento de VMs do Azure e discos gerenciados

Agora os [conjuntos de dimensionamento de máquinas virtuais](/azure/virtual-machine-scale-sets/) do Azure oferecem suporte a máquinas virtuais com discos gerenciados. O uso de discos gerenciados com conjuntos de dimensionamento tem vários benefícios, incluindo:

* Você não precisa mais pré-criar e gerenciar contas de armazenamento para armazenar os discos do sistema operacional para as VMs do conjunto de dimensionamento.

* Você pode anexar discos de dados gerenciados ao conjunto de dimensionamento.

* Com o disco gerenciado, um conjunto de dimensionamento poderá ter a capacidade de até 1.000 VMs se for baseado em uma imagem de plataforma, ou 100 VMs se for baseado em uma imagem personalizada.

## <a name="get-started"></a>Introdução

Uma maneira simples para começar com os conjuntos de dimensionamento de disco gerenciado é implantar um conjunto no Portal do Azure. Para obter mais informações, consulte [este artigo](./virtual-machine-scale-sets-portal-create.md). Outra maneira simples para começar é usar a [CLI do Azure 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2) para implantar um conjunto de dimensionamento. O exemplo a seguir mostra como criar um conjunto de dimensionamento com base em Ubuntu com 10 VMs, cada uma com um disco de dados de 50 GB e de 100 GB:

```azurecli
az group create -l southcentralus -n dsktest
az vmss create -g dsktest -n dskvmss --image ubuntults --instance-count 10 --data-disk-sizes-gb 50 100
```

Como alternativa, você pode procurar pastas no [repositório GitHub de modelos de início rápido do Azure](https://github.com/Azure/azure-quickstart-templates) que contenham `vmss` para ver exemplos pré-criados de modelos que implantam conjuntos de dimensionamento. Para saber quais modelos já estão usando discos gerenciados, consulte [essa lista](https://github.com/Azure/azure-quickstart-templates/blob/master/managed-disk-support-list.md).

## <a name="api-versions"></a>Versões de API

A versão atual disponível da API de conjuntos de dimensionamento com discos gerenciados é a `2016-04-30-preview`. Os conjuntos de dimensionamento com discos não gerenciados continuarão a funcionar como atualmente, mesmo nas novas versões de API que oferecem suporte para disco gerenciado. No entanto, os conjuntos de dimensionamento com discos não gerenciados não terão os benefícios dos discos gerenciados, mesmo nessas novas versões de API.

## <a name="next-steps"></a>Próximas etapas

Para obter mais informações sobre discos gerenciados em geral, consulte [este artigo](../storage/storage-managed-disks-overview.md).

Para ver como converter um modelo do Resource Manager para provisionar conjuntos de dimensionamento com discos gerenciados, consulte [este artigo](./virtual-machine-scale-sets-convert-template-to-md.md). As mesmas modificações nos modelos do Resource Manager também se aplicam à API REST do Azure.

Para saber mais sobre como usar discos de dados gerenciados com conjuntos de dimensionamento, veja [este artigo](./virtual-machine-scale-sets-attached-disks.md).

Para começar a trabalhar com grandes conjuntos de dimensionamento, consulte [este artigo](./virtual-machine-scale-sets-placement-groups.md).



