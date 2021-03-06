---
title: "Atribuir um usuário ou grupo a um aplicativo empresarial na visualização do Azure Active Directory | Microsoft Docs"
description: "Como selecionar um aplicativo empresarial par atribuir um usuário ou um grupo a ele no Azure Active Directory"
services: active-directory
documentationcenter: 
author: curtand
manager: femila
editor: 
ms.assetid: 5817ad48-d916-492b-a8d0-2ade8c50a224
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/13/2017
ms.author: curtand
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 18330aa5e0fe8e3bbad6c266f823a4969b9b8b6c


---
# <a name="assign-a-user-or-group-to-an-enterprise-app-in-azure-active-directory-preview"></a>Atribuir um usuário ou um grupo a um aplicativo empresarial na visualização do Azure Active Directory
É fácil atribuir um usuário ou um grupo aos aplicativos empresariais na visualização do Azure Active Directory (Azure AD). [O que está na visualização?](active-directory-preview-explainer.md)  Você deve ter as permissões apropriadas para gerenciar o aplicativo empresarial. Na visualização atual, você deve ser um administrador global do diretório.

## <a name="how-do-i-assign-user-access-to-an-enterprise-app"></a>Como atribuir acesso de usuário a um aplicativo empresarial?
1. Entre no [Portal do Azure](https://portal.azure.com) com uma conta que seja um administrador global do diretório.
2. Selecione **Mais serviços**, insira Azure Active Directory na caixa de texto, em seguida, selecione **Enter**.
3. Na folha **Azure Active Directory – *nomedodiretório*** (ou seja, a folha do Azure AD para o diretório que você está gerenciando), escolha **Aplicativos empresariais**.

    ![Abrir aplicativos empresariais](./media/active-directory-coreapps-assign-user-azure-portal/open-enterprise-apps.png)
4. Na folha **Aplicativos empresariais**, escolha **Todos os aplicativos**. Você verá uma lista dos aplicativos que pode gerenciar.
5. Na folha **Aplicativos empresariais – Todos os aplicativos** , escolha um aplicativo.
6. Na folha ***appname*** (ou seja, a folha com o nome do aplicativo selecionado no título), selecione **Usuários e Grupos**.

    ![Seleção do comando todos os aplicativos](./media/active-directory-coreapps-assign-user-azure-portal/select-app-users.png)
7. Na folha ***appname*** **- Usuário e Atribuição de Grupos**, selecione o comando **Adicionar**.
8. Na folha **Adicionar Atribuição**, selecione **Usuários e grupos**.

    ![Atribuir um usuário ou um grupo ao aplicativo](./media/active-directory-coreapps-assign-user-azure-portal/assign-users.png)
9. Na folha **Usuários e grupos**, selecione um ou mais usuários ou grupos na lista, em seguida, pressione o botão **Selecionar** na parte inferior da folha.
10. Na folha **Adicionar Atribuição**, selecione **Função**. Então, na folha **Selecionar Função**, escolha uma função para aplicar nos usuários ou grupos selecionados, em seguida, selecione o botão **OK** na parte inferior da folha.
11. Na folha **Adicionar Atribuição**, selecione o botão **Atribuir** na parte inferior da folha. Os usuários ou os grupos atribuídos terão as permissões definidas pela função selecionada para esse aplicativo empresarial.

## <a name="next-steps"></a>Próximas etapas
* [Ver todos os meus grupos](active-directory-groups-view-azure-portal.md)
* [Remover uma atribuição de usuário ou de grupo de um aplicativo empresarial](active-directory-coreapps-remove-assignment-azure-portal.md)
* [Desabilitar as entradas de usuário em um aplicativo empresarial](active-directory-coreapps-disable-app-azure-portal.md)
* [Alterar o nome ou logotipo de um aplicativo empresarial](active-directory-coreapps-change-app-logo-user-azure-portal.md)



<!--HONumber=Nov16_HO3-->


