---
title: "Erros comuns durante a migração do Clássico para o Azure Resource Manager | Microsoft Docs"
description: "Este artigo cataloga os erros e mitigações mais comuns durante a migração de recursos de IaaS do Gerenciamento de Serviços do Azure para a pilha do Azure Resource Manager."
services: virtual-machines-windows
documentationcenter: 
author: singhkays
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 5bc03a1b-eb1c-438c-83d9-f0e9d61f1b6a
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 10/13/2016
ms.author: kasing
translationtype: Human Translation
ms.sourcegitcommit: daa311fcfd1ef06cf36e9443150fc967f0f39708
ms.openlocfilehash: 052505260c0998c8528146c4985400126d094f0d


---
# <a name="common-errors-during-classic-to-azure-resource-manager-migration"></a>Erros comuns durante a migração do Clássico para o Azure Resource Manager
Este artigo cataloga os erros e mitigações mais comuns durante a migração de recursos de IaaS do modelo de implantação clássico do Azure para a pilha do Azure Resource Manager.

## <a name="list-of-errors"></a>Lista de erros
| Cadeia de caracteres de erro | Redução |
| --- | --- |
| Erro interno do servidor |Em alguns casos, isso é um erro transitório desaparece com uma nova tentativa. Se ele persistir, [entre em contato com o suporte do Azure](../azure-supportability/how-to-create-azure-support-request.md) pois ele precisará de investigação dos logs da plataforma. <br><br> **OBSERVAÇÃO:** não tente realizar nenhuma automitigação depois que o incidente for controlado pela equipe de suporte, pois isso poderá ter consequências indesejadas em seu ambiente. |
| Não há suporte para migração para implantação {nome_da_implantação} em {nome_do_serviço_hospedado} HostedService porque é uma implantação de PaaS (Web/Trabalho). |Isso ocorre quando uma implantação contém uma função de trabalho/Web. Uma vez que a migração tem suporte apenas para máquinas virtuais, remova a função Web/de trabalho da implantação e tente novamente a migração. |
| Falha na implantação do modelo {nome_do_ modelo}. CorrelationId={guid} |No back-end do serviço de migração, usamos modelos do Azure Resource Manager para criar recursos na pilha do Azure Resource Manager. Já que os modelos são idempotentes, geralmente você pode realizar com segurança novas tentativas da operação de migração para passar por esse erro. Se esse erro persistir, faça [entre em contato com o suporte do Azure](../azure-supportability/how-to-create-azure-support-request.md) e dê a eles a CorrelationId. <br><br> **OBSERVAÇÃO:** não tente realizar nenhuma automitigação depois que o incidente for controlado pela equipe de suporte, pois isso poderá ter consequências indesejadas em seu ambiente. |
| A rede virtual {nome_da_ rede_virtual} não existe. |Isso poderá acontecer se você tiver criado a rede virtual no novo Portal do Azure. O nome de rede virtual real segue o padrão "Group * <VNET name>" |
| A VM {nome_da_VM} no HostedService {nome_do_serviço_hospedado} contém a Extensão {nome_da_extensão} para a qual não há suporte no Azure Resource Manager. É recomendável desinstalá-la da VM antes de continuar com a migração. |Extensões XML como a BGInfo 1. * não têm suporte no Azure Resource Manager. Portanto, essas extensões não podem ser migradas. Se essas extensões forem deixadas instaladas na máquina virtual, elas serão automaticamente desinstaladas antes da conclusão da migração. |
| A VM {nome_da_VM} no serviço hospedado {nome_do_serviço_hospedado} contém a Extensão VMSnapshot/VMSnapshotLinux, que atualmente não tem suporte para Migração. Desinstale-a da VM e adicione-a usando o Azure Resource Manager após a conclusão da migração |Esse é o cenário em que a máquina virtual está configurada para o Backup do Azure. Como este é atualmente um cenário sem suporte, siga a solução https://aka.ms/vmbackupmigration |
| A VM {nome_da_VM} no serviço hospedado {nome_do_serviço_hospedado} contém a Extensão {nome_da_extensão}, cujo status não está sendo relatado da VM. Portanto, esta VM não pode ser migrada. Certifique-se de que o status da Extensão está sendo relatado ou desinstale a extensão da VM e tente novamente realizar a migração. <br><br> A VM {nome_da_VM} no serviço hospedado {nome_do_serviço_hospedado} contém a Extensão {nome_da_extensão} relatando o status do manipulador: {status_do_manipulador}. Portanto, a VM não pode ser migrada. Certifique-se de que o status do manipulador de Extensões sendo relatado é {status_do_manipulador} ou desinstale-o da VM e tente novamente realizar a migração. <br><br> Agente de VM para a VM {nome_da_VM} no serviço hospedado {nome_do_serviço_hospedado} está relatando o status geral do agente como Não Pronto. Portanto, se a VM tiver uma extensão migrável, ela não poderá ser migrada. Certifique-se de que o agente de VM esteja relatando o status do agente geral como Pronto. Consulte https://aka.ms/classiciaasmigrationfaqs. |O agente convidado do Azure e extensões de VM precisam de acesso de saída à Internet para que seus status sejam populados pela conta de armazenamento da VM. Causas comuns de falha de status incluem <li> um Grupo de Segurança de Rede que bloqueia o acesso de saída à Internet <li> Se a VNET tiver servidores DNS locais e a conectividade DNS for perdida <br><br> Se você continuar a ver um status sem suporte, você poderá desinstalar as extensões para ignorar essa verificação e prosseguir com a migração. |
| Não há suporte para migração para implantação {nome_da_implantação} no serviço hospedado {nome_do_serviço_hospedado} porque ele tem vários conjuntos de disponibilidade. |Atualmente, apenas os serviços hospedados com um ou nenhum conjunto de disponibilidade podem ser migrados. Para contornar esse problema, mova os conjuntos de disponibilidade e as máquinas virtuais adicionais nesses conjuntos de disponibilidade para um serviço hospedado diferente. |
| Não há suporte para migração para a implantação {nome_da_implantação} no serviço hospedado {nome_do_serviço_hospedado} porque ela tem VMs que não são parte do conjunto de disponibilidade, embora o serviço hospedado contenha uma. |A solução para esse cenário é para mover todas as máquinas virtuais para um único conjunto de disponibilidade ou remover todas as máquinas virtuais do conjunto de disponibilidade no serviço hospedado. |
| A conta de armazenamento/serviço hospedado/rede virtual {nome_da_rede_virtual} está no processo de migração e, portanto, não pode ser alterada |Esse erro ocorre quando a operação de migração "Prepare" foi concluída no recurso e uma operação que faça uma alteração no recurso é disparada. Por causa do bloqueio no plano de gerenciamento após a operação de "Prepare", as alterações ao recurso são bloqueadas. Para desbloquear o plano de gerenciamento, você pode executar a operação de migração de "Commit" para concluir a migração ou a operação de migração "Abort" para reverter a operação "Prepare". |
| A migração não é permitida para o serviço hospedado {nome_do_serviço_hospedado} porque ele tem a VM {nome_da_vm} no estado: RoleStateUnknown. A migração é permitida somente quando a VM está em um dos seguintes estados – Em execução, Parada, Interrompida, Desalocada. |A VM pode estar passando por uma transição de estado que geralmente acontece durante uma operação de atualização no serviço hospedado como uma reinicialização, instalação de extensão, etc. É recomendável que a operação de atualização seja concluída no HostedService antes de tentar a migração. |
| A implantação {deployment-name} em HostedService {hosted-service-name} contém uma VM {vm-name} com Disco de Dados {data-disk-name} cujo tamanho de blob físico de {size-of-the-vhd-blob-backing-the-data-disk} bytes não corresponde ao tamanho lógico do Disco de Dados da VM de {size-of-the-data-disk-specified-in-the-vm-api} bytes. A migração continuará sem especificar um tamanho para o disco de dados para a VM do Azure Resource Manager. Se você deseja corrigir o tamanho do disco de dados antes de continuar com a migração, visite https://aka.ms/vmdiskresize. | Esse erro ocorre se você redimensionar o blob VHD sem atualizar o tamanho do modelo da API da VM. As etapas de atenuação detalhadas estão descritas [abaixo](#vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes).|

## <a name="detailed-mitigations"></a>Atenuação detalhada

### <a name="vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes"></a>A VM com o Disco de Dados cujo tamanho de blob físico é de bytes não corresponde ao tamanho lógico do Disco de Dados da VM de bytes.

Isso acontece quando o tamanho lógico do Disco de Dados perde a sincronia com o tamanho real do blob VHD. Isso pode ser facilmente verificado usando os seguintes comandos:

#### <a name="verifying-the-issue"></a>Verificando o problema

```PowerShell
# Store the VM details in the VM object
$vm = Get-AzureVM -ServiceName $servicename -Name $vmname

# Display the data disk properties
# NOTE the data disk LogicalDiskSizeInGB below which is 11GB. Also note the MediaLink Uri of the VHD blob as we'll use this in the next step
$vm.VM.DataVirtualHardDisks


HostCaching         : None
DiskLabel           : 
DiskName            : coreosvm-coreosvm-0-201611230636240687
Lun                 : 0
LogicalDiskSizeInGB : 11
MediaLink           : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
SourceMediaLink     : 
IOType              : Standard
ExtensionData       : 

# Now get the properties of the blob backing the data disk above
# NOTE the size of the blob is about 15 GB which is different from LogicalDiskSizeInGB above
$blob = Get-AzureStorageblob -Blob "coreosvm-dd1.vhd" -Container vhds 

$blob

ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudPageBlob
BlobType          : PageBlob
Length            : 16106127872
ContentType       : application/octet-stream
LastModified      : 11/23/2016 7:16:22 AM +00:00
SnapshotTime      : 
ContinuationToken : 
Context           : Microsoft.WindowsAzure.Commands.Common.Storage.AzureStorageContext
Name              : coreosvm-dd1.vhd
```

#### <a name="mitigating-the-issue"></a>Atenuando o problema

```PowerShell
# Convert the blob size in bytes to GB into a variable which we'll use later
$newSize = [int]($blob.Length / 1GB)

# See the calculated size in GB
$newSize

15

# Store the disk name of the data disk as we'll use this to identify the disk to be updated
$diskName = $vm.VM.DataVirtualHardDisks[0].DiskName

# Identify the LUN of the data disk to remove
$lunToRemove = $vm.VM.DataVirtualHardDisks[0].Lun

# Now remove the data disk from the VM so that the disk isn't leased by the VM and it's size can be updated
Remove-AzureDataDisk -LUN $lunToRemove -VM $vm | Update-AzureVm -Name $vmname -ServiceName $servicename

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureVM       213xx1-b44b-1v6n-23gg-591f2a13cd16   Succeeded  

# Verify we have the right disk that's going to be updated
Get-AzureDisk -DiskName $diskName

AffinityGroup        : 
AttachedTo           : 
IsCorrupted          : False
Label                : 
Location             : East US
DiskSizeInGB         : 11
MediaLink            : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
DiskName             : coreosvm-coreosvm-0-201611230636240687
SourceImageName      : 
OS                   : 
IOType               : Standard
OperationDescription : Get-AzureDisk
OperationId          : 0c56a2b7-a325-123b-7043-74c27d5a61fd
OperationStatus      : Succeeded

# Now update the disk to the new size
Update-AzureDisk -DiskName $diskName -ResizedSizeInGB $newSize -Label $diskName

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureDisk     cv134b65-1b6n-8908-abuo-ce9e395ac3e7 Succeeded 

# Now verify that the "DiskSizeInGB" property of the disk matches the size of the blob 
Get-AzureDisk -DiskName $diskName


AffinityGroup        : 
AttachedTo           : 
IsCorrupted          : False
Label                : coreosvm-coreosvm-0-201611230636240687
Location             : East US
DiskSizeInGB         : 15
MediaLink            : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
DiskName             : coreosvm-coreosvm-0-201611230636240687
SourceImageName      : 
OS                   : 
IOType               : Standard
OperationDescription : Get-AzureDisk
OperationId          : 1v53bde5-cv56-5621-9078-16b9c8a0bad2
OperationStatus      : Succeeded

# Now we'll add the disk back to the VM as a data disk. First we need to get an updated VM object
$vm = Get-AzureVM -ServiceName $servicename -Name $vmname

Add-AzureDataDisk -Import -DiskName $diskName -LUN 0 -VM $vm -HostCaching ReadWrite | Update-AzureVm -Name $vmname -ServiceName $servicename

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureVM       b0ad3d4c-4v68-45vb-xxc1-134fd010d0f8 Succeeded      
```

## <a name="next-steps"></a>Próximas etapas
Aqui está uma lista de artigos de migração que explicam o processo.

* [Migração de recursos de IaaS com suporte da plataforma do clássico para o Azure Resource Manager](virtual-machines-windows-migration-classic-resource-manager.md)
* [Usar o PowerShell para migrar recursos de IaaS do clássico para o Azure Resource Manager](virtual-machines-windows-ps-migration-classic-resource-manager.md)
* [Usar a CLI para migrar recursos de IaaS do clássico para o Azure Resource Manager](virtual-machines-linux-cli-migration-classic-resource-manager.md)




<!--HONumber=Dec16_HO1-->


