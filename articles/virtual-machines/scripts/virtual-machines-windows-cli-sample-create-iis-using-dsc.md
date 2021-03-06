---
title: Exemplo de script da CLI do Azure - Criar uma VM do Windows Server 2016 com o IIS usando a DSC | Microsoft Docs
description: Exemplo de script da CLI do Azure - Criar uma VM do Windows Server 2016 com o IIS usando a DSC
services: virtual-machines-windows
documentationcenter: virtual-machines
author: rickstercdn
manager: timlt
editor: tysonn
tags: 
ms.assetid: 
ms.service: virtual-machines-Windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
translationtype: Human Translation
ms.sourcegitcommit: 82d40c30c92f5da090e7ec4e2f25ead3908cc603
ms.openlocfilehash: ea0598251e979094c5841c56c85f7297aaf3d21d
ms.lasthandoff: 03/02/2017

---

# <a name="create-a-vm-with-iis-using-dsc"></a>Criar uma máquina virtual com o IIS usando a DSC

Este script cria uma máquina virtual e, em seguida, usa a extensão de script personalizado da DSC da Máquina Virtual do Azure para instalar e configurar o IIS. 

Antes de executar esse script, certifique-se de que uma conexão com o Azure foi criada usando o comando `az login`. Além disso, você deve alterar a variável $AdminPassword no início do script para atender aos requisitos exclusivos de complexidade da senha.

Este exemplo funciona em um shell Bash. Para opções sobre como executar scripts da CLI do Azure no cliente Windows, veja [Execução da CLI do Azure no Windows](../virtual-machines-windows-cli-options.md).

## <a name="sample-script"></a>Script de exemplo

[!code-azurecli[principal](../../../cli_scripts/virtual-machine/create-windows-iis-using-dsc/create-windows-iis-using-dsc.sh "Criação rápida de VM")]

## <a name="clean-up-deployment"></a>Limpar implantação 

Após a execução do exemplo de script, o comando a seguir pode ser usado para remover o Grupo de Recursos, a VM e todos os recursos relacionados.

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Explicação sobre o script

Este script usa os comandos a seguir para criar um grupo de recursos, uma máquina virtual e todos os recursos relacionados. Cada comando na tabela redireciona para a documentação específica do comando.

| Comando | Observações |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | Cria um grupo de recursos no qual todos os recursos são armazenados. |
| [az vm create](https://docs.microsoft.com/cli/azure/vm#create) | Cria a máquina virtual e a conecta a placa de rede, a rede virtual, a sub-rede e o NSG. Este comando também especifica a imagem de máquina virtual a ser usada e as credenciais administrativas.  |
| [az vm extension set](https://docs.microsoft.com/cli/azure/vm#create) | Adicione a Extensão de Script Personalizado à máquina virtual que invoca um script para instalar o IIS. |
| [az vm open-port](https://docs.microsoft.com/cli/azure/vm#open-port) | Cria uma regra de grupo de segurança de rede para permitir o tráfego de entrada. Neste exemplo, a porta 80 está aberta para tráfego HTTP. |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#set) | Exclui um grupo de recursos, incluindo todos os recursos aninhados. |

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre a CLI do Azure, veja a [documentação da CLI do Azure](https://docs.microsoft.com/cli/azure/overview).

Exemplos de script CLI máquinas virtuais adicionais podem ser encontrados na [documentação da VM do Windows do Azure](../virtual-machines-windows-cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
