---
title: Requisitos de imagem do Azure RemoteApp | Microsoft Docs
description: "Saiba mais sobre os requisitos para a criação de imagens a serem usadas com o Azure RemoteApp"
services: remoteapp
documentationcenter: 
author: msmbaldwin
manager: mbaldwin
ms.assetid: 7cbb90f4-6dc9-462c-a429-088cdb57414e
ms.service: remoteapp
ms.workload: compute
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/23/2016
ms.author: mbaldwin
translationtype: Human Translation
ms.sourcegitcommit: 0af5a4e2139a202c7f62f48c7a7e8552457ae76d
ms.openlocfilehash: a37804025cbd298ef2b98c84b5cc59c0cba07dd9


---
# <a name="requirements-for-azure-remoteapp-images"></a>Requisitos para as imagens do Azure RemoteApp
> [!IMPORTANT]
> O Azure RemoteApp está sendo descontinuado. Leia o [comunicado](https://go.microsoft.com/fwlink/?linkid=821148) para obter detalhes.
> 
> 

O Azure RemoteApp usa uma imagem do Windows Server 2012 R2 para hospedar todos os programas que deseja compartilhar com os seus usuários. Para criar uma imagem personalizada, você pode iniciar com uma imagem existente ou [criar uma nova](remoteapp-create-custom-image.md).

> [!TIP]
> Você sabia que a sua assinatura do Azure RemoteApp fornece acesso a uma imagem do Windows Server 2012 R2 na galeria de VM do Azure que você pode usar para criar a sua própria imagem de modelo? [Confira](remoteapp-image-on-azurevm.md).  
> 
> 

Os requisitos para a imagem passiva de upload para o uso com o RemoteApp do Azure são:

* Aplicativos personalizados não armazenam dados localmente na imagem. Essas imagens são sem monitoração de estado e devem conter apenas aplicativos.
* A imagem não contém dados que podem ser perdidos.
* O tamanho da imagem dever ser um múltiplo de MBs. Se você tentar carregar uma imagem que não é um múltiplo exato, o carregamento falhará.
* O tamanho da imagem deve ser de 127 GB ou menor.
* Deve estar em um arquivo VHD (arquivos VHDX atualmente não têm suporte).
* O VHD não deve ser uma máquina virtual de geração 2.
* O VHD podem ter tamanho fixo ou expandir dinamicamente. O VHD de expansão dinâmica é recomendado, pois demora menos para fazer o upload do Azure que o arquivo VHD de tamanho fixo.
* O disco deve ser inicializado usando o estilo de particionamento do Registro mestre de inicialização (MBR). O estilo de particionamento da tabela de partição GUID (GPT) não tem suporte.
* O VHD deve conter uma instalação única do Windows Server 2012 R2. Ele deve conter vários volumes, mas apenas um que contenha uma instalação do Windows.
* A função Host da Sessão da Área de Trabalho Remota (RDSH) e o recurso Desktop Experience devem estar instalados.
* A função do Agente de Conexão de Área de Trabalho Remota *não* deve ser instalada.
* O Encrypting File System (EFS) deve estar desabilitado.
* A imagem deve ser SYSPREPed usando os parâmetros **/oobe /generalize /shutdown** (NÃO use o parâmetro **/mode:vm**).
* Não há suporte para carregar o VHD de uma cadeia de instantâneo.

Consulte [Criar uma imagem do RemoteApp do Azure](remoteapp-imageoptions.md) para obter mais informações sobre a criação de imagens para o RemoteApp do Azure.




<!--HONumber=Dec16_HO2-->


