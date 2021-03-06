---
title: "Computação do Azure Governamental | Microsoft Docs"
description: "Este guia fornece uma comparação dos recursos e orientações sobre como desenvolver aplicativos para o Azure Government"
services: azure-government
cloud: gov
documentationcenter: 
author: ryansoc
manager: zakramer
ms.assetid: fb11f60c-5a70-46a9-82a0-abb2a4f4239b
ms.service: azure-government
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: azure-government
ms.date: 11/18/2016
ms.author: ryansoc
translationtype: Human Translation
ms.sourcegitcommit: ca01baebd755e92aff1ef080af1fcd4606b9a2ec
ms.openlocfilehash: 200caac4b480495b0bbb8f10cc89fdd73c42dbe2


---
# <a name="azure-government-compute"></a>Computação do Azure Governamental
## <a name="virtual-machines"></a>Máquinas Virtuais
Para obter detalhes sobre esse serviço e como usá-lo, consulte [Tamanhos de máquinas virtuais do Azure](../virtual-machines/virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

### <a name="variations"></a>Variações
Os produtos (VMs) estão disponíveis no Azure Governamental:

| Produtos (VMs) | Gov do VA nos EUA | Gov do IA nos EUA |
| --- | --- | --- |
| A0-A7 |S |S |
| Série D |S |N |
| Série Dv2 |S |S |
| Série DS |S |N |
| Série F |S |S |
| Série FS |N |N |
| Série G |S |N |
| Série GS |S |N |

### <a name="data-considerations"></a>Considerações de dados
As informações a seguir identificam o limite do Azure Governamental para o máquinas virtuais do Azure:

| Dados regulamentados/controlados permitidos | Dados regulamentados/controlados não permitidos |
| --- | --- |
| Dados inseridos, armazenados e processados em uma VM podem conter dados de exportação controlados. Binários em execução nas máquinas virtuais do Azure. Autenticadores estáticos, como senhas e PINs de cartão inteligente para acesso aos componentes da plataforma Azure. Chaves privadas de certificados usados para gerenciar os componentes da plataforma Azure. Cadeias de conexão SQL.  Outros informações/segredos de segurança, como certificados, chaves de criptografia, chaves mestras e chaves de armazenamento armazenadas nos serviços do Azure. |Metadados não podem conter dados de exportação controlados. Esses metadados incluem todos os dados de configuração inseridos durante a criação e manutenção de sua máquina virtual do Azure.  Não insira dados Regulamentados/controlados nos seguintes campos: Nomes de funções de locatários, Grupos de recursos, Nomes de implantação, Nomes de recursos, Marcas de recurso |

## <a name="next-steps"></a>Próximas etapas
Para obter informações complementares e atualizações, assine o <a href="https://blogs.msdn.microsoft.com/azuregov/">Blog do Microsoft Azure Governamental. </a>




<!--HONumber=Dec16_HO1-->


