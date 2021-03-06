---
title: "Balanceamento de carga em várias configurações de IP | Microsoft Docs"
description: "Balanceamento de carga entre as configurações de IP primárias e secundárias."
services: load-balancer
documentationcenter: na
author: anavinahar
manager: narayan
editor: na
ms.assetid: 244907cd-b275-4494-aaf7-dcfc4d93edfe
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/10/2017
ms.author: annahar
translationtype: Human Translation
ms.sourcegitcommit: 394315f81cf694cc2bb3a28b45694361b11e0670
ms.openlocfilehash: d7df8f3271c6c92df427a840d73fdb9a0c6b54a1
ms.lasthandoff: 02/14/2017

---

# <a name="load-balancing-on-multiple-ip-configurations"></a>Balanceamento de carga em várias configurações de IP

> [!div class="op_single_selector"]
> * [PowerShell](load-balancer-multiple-ip.md)
> * [CLI](load-balancer-multiple-ip-cli.md)
>

Este artigo descreve como usar o Azure Load Balancer com vários endereços IP por NIC (Interface de rede virtual). O suporte para vários endereços IP em uma NIC é um recurso que está na versão de Visualização no momento. Para saber mais, consulte a seção [Limitações](#limitations) deste artigo. O cenário a seguir ilustra como esse recurso funciona com o Load Balancer.

Para este cenário, temos duas VMs executando o Windows, cada um com uma única NIC. Cada NIC tem várias configurações de IP. Cada VM hospeda os sites de contoso.com e fabrikam.com. Cada site está associado a uma das configurações de IP na NIC. Usamos o balanceador de carga para expor dois endereços IP front-end, um para cada site, a fim de distribuir o tráfego para a respectiva configuração de IP do site. Esse cenário usa o mesmo número de porta entre os front-ends, bem como os dois endereços IP do pool de back-end.

![Imagem de cenário do LB](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

## <a name="limitations"></a>Limitações

Neste momento, a configuração de balanceamento de carga em configurações de IP secundárias só é possível usando o Azure PowerShell e a CLI do Azure. Essa limitação é temporária e pode mudar a qualquer momento. Examine esta página para verificar se há atualizações.

[!INCLUDE [virtual-network-preview](../../includes/virtual-network-preview.md)]

Registre-se para obter a prévia executando os seguintes comandos do PowerShell após o logon e selecione a assinatura apropriada:

```
Register-AzureRmProviderFeature -FeatureName AllowMultipleIpConfigurationsPerNic -ProviderNamespace Microsoft.Network

Register-AzureRmProviderFeature -FeatureName AllowLoadBalancingonSecondaryIpconfigs -ProviderNamespace Microsoft.Network

Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

Não tente concluir as etapas restantes até ver a saída a seguir ao executar o comando ```Get-AzureRmProviderFeature```:
        
```powershell
FeatureName                            ProviderName      RegistrationState
-----------                            ------------      -----------------      
AllowLoadBalancingOnSecondaryIpConfigs Microsoft.Network Registered       
AllowMultipleIpConfigurationsPerNic    Microsoft.Network Registered       
```
        
>[!NOTE] 
>Isso pode levar alguns minutos.

## <a name="steps-to-load-balance-on-multiple-ip-configurations"></a>Etapas para o balanceamento de carga em várias configurações de IP

Execute as etapas abaixo para obter o cenário descrito neste artigo:

1. Instale o PowerShell do Azure. Confira [Como instalar e configurar o Azure PowerShell](/powershell/azureps-cmdlets-docs) para saber mais sobre como instalar a versão mais recente do Azure PowerShell, selecionar a assinatura e entrar em sua conta.
2. Crie um grupo de recursos usando as seguinte configurações:

    ```powershell
    $location = "westcentralus".
    $myResourceGroup = "contosofabrikam"
    ```

    Para saber mais, consulte a Etapa 2 [Criar um grupo de recursos](../virtual-machines/virtual-machines-windows-ps-create.md?toc=%2fazure%2fload-balancer%2ftoc.json).

3. [Criar um conjunto de disponibilidade](../virtual-machines/virtual-machines-windows-create-availability-set.md?toc=%2fazure%2fload-balancer%2ftoc.json) para suas VMs. Para esse cenário, use o comando a seguir:

    ```powershell
    New-AzureRmAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset" -Location "West Central US"
    ```

4. Siga as instruções das etapas 3 a 5 no artigo [Criar uma VM do Windows](../virtual-machines/virtual-machines-windows-ps-create.md?toc=%2fazure%2fload-balancer%2ftoc.json) para preparar a criação de uma VM com uma única NIC. Execute a etapa 6.1 e use o seguinte, em vez da etapa 6.2:

    ```powershell
    $availset = Get-AzureRmAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset"
    New-AzureRmVMConfig -VMName "VM1" -VMSize "Standard_DS1_v2" -AvailabilitySetId $availset.Id
    ```

    Em seguida, conclua as etapas 6.3 a 6.8 [Criar uma VM do Windows](../virtual-machines/virtual-machines-windows-ps-create.md?toc=%2fazure%2fload-balancer%2ftoc.json).

5. Adicione uma segunda configuração de IP para cada uma das VMs. Siga as instruções no artigo [Atribuir vários endereços IP para máquinas virtuais](../virtual-network/virtual-network-multiple-ip-addresses-powershell.md#add). Use as seguintes configurações:

    ```powershell
    $NicName = "VM1-NIC"
    $RgName = "contosofabrikam"
    $NicLocation = "West Central US"
    $IPConfigName4 = "VM1-ipconfig2"
    $Subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $myVnet
    ```

    Não é necessário associar as configurações de IP secundárias aos IPs públicos para este tutorial. Edite o comando para remover a parte pública de associação de IP.

6. Conclua as etapas 4 a 6 deste artigo novamente para a VM2. Substitua o nome da VM para VM2 ao fazer isso. Observe que você não precisa criar uma rede virtual para a segunda VM. Você pode ou não criar uma nova sub-rede com base em seu caso de uso.

7. Crie dois endereços IP públicos e armazene-os nas variáveis apropriadas, conforme mostrado:

    ```powershell
    $publicIP1 = New-AzureRmPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel contoso
    $publicIP2 = New-AzureRmPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel fabrikam

    $publicIP1 = Get-AzureRmPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam
    $publicIP2 = Get-AzureRmPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam
    ```

8. Crie duas configurações de IP de front-end:

    ```powershell
    $frontendIP1 = New-AzureRmLoadBalancerFrontendIpConfig -Name contosofe -PublicIpAddress $publicIP1
    $frontendIP2 = New-AzureRmLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2
    ```

9. Crie os pools de endereços de back-end, uma investigação e regras de balanceamento de carga:

    ```powershell
    $beaddresspool1 = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name contosopool
    $beaddresspool2 = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name fabrikampool

    $healthProbe = New-AzureRmLoadBalancerProbeConfig -Name HTTP -RequestPath 'index.html' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

    $lbrule1 = New-AzureRmLoadBalancerRuleConfig -Name HTTPc -FrontendIpConfiguration $frontendIP1 -BackendAddressPool $beaddresspool1 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    $lbrule2 = New-AzureRmLoadBalancerRuleConfig -Name HTTPf -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    ```

10. Após a criação desses recursos, crie o balanceador de carga:

    ```powershell
    $mylb = New-AzureRmLoadBalancer -ResourceGroupName contosofabrikam -Name mylb -Location 'West Central US' -FrontendIpConfiguration $frontendIP1 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe
    ```

11. Adicione o segundo pool de endereços back-end e a configuração de IP de front-end ao seu balanceador de carga recém-criado:

    ```powershell
    $mylb = Get-AzureRmLoadBalancer -Name "mylb" -ResourceGroupName $myResourceGroup | Add-AzureRmLoadBalancerBackendAddressPoolConfig -Name fabrikampool | Set-AzureRmLoadBalancer

    $mylb | Add-AzureRmLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2 | Set-AzureRmLoadBalancer
    
    Add-AzureRmLoadBalancerRuleConfig -Name HTTP -LoadBalancer $mylb -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80 | Set-AzureRmLoadBalancer
    ```

12. Os comandos a seguir obtêm as NICs e adicione as duas configurações de IP de cada NIC ao pool de endereços de back-end do balanceador de carga:

    ```powershell
    $nic1 = Get-AzureRmNetworkInterface -Name "VM1-NIC" -ResourceGroupName "MyResourcegroup";
    $nic2 = Get-AzureRmNetworkInterface -Name "VM2-NIC" -ResourceGroupName "MyResourcegroup";

    $nic1.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic1.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);
    $nic2.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic2.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);

    $mylb = $mylb | Set-AzureRmLoadBalancer

    $nic1 | Set-AzureRmNetworkInterface
    $nic2 | Set-AzureRmNetworkInterface
    ```

13. Por fim, você deve configurar os registros de recurso DNS para apontar para o endereço IP do endereço IP front-end do Balanceador de Carga. Você pode hospedar seus domínios no DNS do Azure. Para saber mais sobre como usar o DNS do Azure com o Load Balancer, confira [Usar o DNS do Azure com outros serviços do Azure](../dns/dns-for-azure-services.md).

