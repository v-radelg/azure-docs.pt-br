---
title: Usar VS Code com modelos do Resource Manager | Microsoft Docs
description: Mostra como definir o Visual Studio Code para criar modelos do Azure Resource Manager.
services: azure-resource-manager
documentationcenter: na
author: cmatskas
manager: timlt
editor: tysonn
ms.assetid: 78f2aa22-df1d-41bd-92ec-dabd1175db88
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/03/2017
ms.author: chmatsk;tomfitz
translationtype: Human Translation
ms.sourcegitcommit: 10c7051c9b1218081d95cb10403006bfd95126ba
ms.openlocfilehash: 2ac1c2cce7a9e045990894b0bbaa045df3d48954


---
# <a name="working-with-azure-resource-manager-templates-in-visual-studio-code"></a>Trabalhando com modelos do Azure Resource Manager no Visual Studio Code
Os modelos do Azure Resource Manager são arquivos JSON que descrevem um recurso e as dependências relacionadas. Esses arquivos às vezes podem ser grandes e complicados e, portanto, as ferramentas de suporte são importantes. O Visual Studio Code é um novo editor de código entre plataformas aberto e leve. Ele dá suporte à criação e edição de modelos do Resource Manager por meio de uma [nova extensão](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools). O Código VS é executado em todos os lugares e só requer acesso à Internet quando quiser implantar seus modelos do Resource Manager em sua assinatura do Azure.

Se você ainda não tiver o Código VS, poderá instalá-lo em [https://code.visualstudio.com/](https://code.visualstudio.com/).

## <a name="install-the-resource-manager-extension"></a>Instalar a extensão do Resource Manager
Para trabalhar com os modelos JSON no Código VS, você precisará instalar uma extensão. As etapas a seguir baixam e instalam o suporte de idiomas para modelos JSON do Resource Manager:

1. Iniciar o Código VS 
2. Abrir com a Abertura Rápida (Ctrl+P) 
3. Execute o comando a seguir: 
   
        ext install msazurermtools.azurerm-vscode-tools
4. Reinicie o Código VS quando solicitado para habilitar a extensão. 
   
   Pronto!

## <a name="set-up-resource-manager-snippets"></a>Configurar os trechos de código do Resource Manager
As etapas anteriores instalaram o suporte a ferramentas, mas agora precisamos configurar o Códio VS para usar trechos de código de modelos JSON.

1. Copie o conteúdo do arquivo do repositório [azure-xplat-arm-tooling](https://raw.githubusercontent.com/Azure/azure-xplat-arm-tooling/master/VSCode/armsnippets.json) na área de transferência.
2. Iniciar o Código VS 
3. No Código VS, você pode abrir o arquivo de trechos do JSON, navegando até **Arquivo** -> **Preferências** -> **Trechos de Usuário** -> **JSON**. Ou selecione **F1** e digite **preferências** até poder selecionar **Preferências: Trechos**.
   
    ![trechos de código de preferência](./media/resource-manager-vs-code/preferences-snippets.png)
   
    Em Opções, selecione **JSON**.
   
    ![selecionar json](./media/resource-manager-vs-code/select-json.png)
4. Cole o conteúdo do arquivo na etapa 1 em seu arquivo de trechos de código do usuário antes do final "}" 
5. Verifique se o JSON parece OK e se não há nenhum erro em algum lugar. 
6. Salve e feche o arquivo de trechos de código do usuário.

Isso é tudo o que precisa para começar a usar os trechos de código do Resource Manager. Em seguida, vamos testar essa configuração.

## <a name="work-with-template-in-vs-code"></a>Trabalhar com modelo no Código VS
A maneira mais fácil de começar a trabalhar com um modelo é usar um dos Modelos de Início Rápido no [Github](https://github.com/Azure/azure-quickstart-templates) ou usar um de seus próprios modelos. Você pode [exportar um modelo](resource-manager-export-template.md) facilmente para qualquer um de seus grupos de recursos por meio do portal. 

1. Se você tiver exportado um modelo de um grupo de recursos, abra os arquivos extraídos no Código VS.
   
    ![mostrar arquivos](./media/resource-manager-vs-code/show-files.png)
2. Abra o arquivo template.json para poder editá-lo e adicionar alguns recursos adicionais. Depois de `"resources": [`, pressione enter para iniciar uma nova linha. Se você digitar **arm**, verá uma lista de opções. Essas opções são os trechos de código do modelo que você instalou. 
   
    ![mostrar trechos de código](./media/resource-manager-vs-code/type-snippets.png)
3. Escolha o trecho de código desejado. Neste artigo, estou escolhendo **arm-ip** para criar um novo endereço IP público. Coloque uma vírgula após a chave de fechamento `}` do recurso recém-criado para verificar se o modelo de sintaxe é válido.
   
     ![adicionar vírgula](./media/resource-manager-vs-code/add-comma.png)
4. O Código VS tem IntelliSense interno. Conforme você vai editando os modelos, o Código VS sugere valores disponíveis. Por exemplo, para adicionar uma seção de variáveis ao modelo, adicione `""` (duas aspas) e selecione **Ctrl+Barra de Espaços** entre as aspas. Você vê as opções, incluindo **variáveis**.
   
    ![adicionar variáveis](./media/resource-manager-vs-code/add-variables.png)
5. O IntelliSense também pode sugerir valores ou funções disponíveis. Para definir uma propriedade de um valor de parâmetro, crie uma expressão com `"[]"` e **Ctrl+Barra de Espaços**. Você pode começar a digitar o nome de uma função. Selecione **Guia** quando encontrar a função desejada.
   
    ![adicionar parâmetro](./media/resource-manager-vs-code/select-parameters.png)
6. Para ver uma lista dos parâmetros disponíveis em seu modelo, selecione **Ctrl+Barra de Espaços** novamente dentro da função.
   
    ![adicionar parâmetro](./media/resource-manager-vs-code/select-avail-parameters.png)
7. Se você tiver qualquer problema de validação de esquema no modelo, verá rabiscos familiares no editor. Você pode exibir a lista de erros e avisos digitando **Ctrl+Shift+M** ou selecionando os glifos na barra de status no canto inferior esquerdo.
   
    ![erros](./media/resource-manager-vs-code/errors.png)
   
    A validação do modelo pode ajudar a detectar problemas de sintaxe. No entanto, você também poderá ver erros que pode ignorar. Em alguns casos, o editor compara o modelo com um esquema que não está atualizado e, portanto, relata um erro mesmo você sabendo que ele está correto. Por exemplo, suponha que uma função foi adicionada recentemente ao Resource Manager, mas o esquema não foi atualizado. O editor relata um erro apesar da função funcionar corretamente durante a implantação.
   
    ![mensagem de erro](./media/resource-manager-vs-code/unrecognized-function.png)

## <a name="deploy-your-new-resources"></a>Implantar novos recursos
Quando o modelo estiver pronto, você pode implantar os novos recursos seguindo as seguintes instruções: 

### <a name="windows"></a>Windows
1. Abra um prompt de comando do PowerShell 
2. Para fazer logon, digite: 
   
  ```powershell
  Login-AzureRmAccount
  ```

3. Se você tiver várias assinaturas, obtenha uma lista das assinaturas com:

  ```powershell 
  Get-AzureRmSubscription
  ```
   
    E selecione a assinatura a ser usada.

  ```powershell
  Select-AzureRmSubscription -SubscriptionId <Subscription Id>
  ```

4. Atualize os parâmetros no arquivo parameters.json
5. Execute o Deploy.ps1 para implantar o modelo no Azure

### <a name="osxlinux"></a>OSX/Linux
1. Abrir uma janela de terminal 
2. Para fazer logon, digite:

  ```azurecli
  azure login
  ```

3. Se você tiver várias assinaturas, selecione a assinatura correta com:

  ```azurecli
  azure account set <subscriptionNameOrId> 
  ```

4. Atualize os parâmetros no arquivo parameters.json.
5. Para implantar o modelo, execute:

  ```azurecli 
  azure group deployment create -f <PathToTemplate>
  ``` 

## <a name="next-steps"></a>Próximas etapas
* Para saber mais sobre os modelos, confira [Criando modelos do Azure Resource Manager](resource-group-authoring-templates.md).
* Para obter a lista completa das funções do modelo, confira [Funções de modelo do Azure Resource Manager](resource-group-template-functions.md).
* Para obter mais exemplos de como trabalhar com o Visual Studio Code, confira [Criar aplicativos de nuvem com o Visual Studio Code](https://github.com/Microsoft/HealthClinic.biz/wiki/Build-cloud-apps-with-Visual-Studio-Code) da [demonstração](https://github.com/Microsoft/HealthClinic.biz) [HealthClinic.biz](https://blogs.msdn.microsoft.com/visualstudio/2015/12/08/connectdemos-2015-healthclinic-biz/) 2015 Connect. Para ver mais guias de início rápido da demonstração do HealthClinic.biz, consulte [Azure Developer Tools Quickstarts (Guias de início rápido das ferramentas de desenvolvedor do Azure)](https://github.com/Microsoft/HealthClinic.biz/wiki/Azure-Developer-Tools-Quickstarts).




<!--HONumber=Jan17_HO1-->


