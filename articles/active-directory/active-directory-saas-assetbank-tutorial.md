---
title: "Tutorial: Integração do Azure Active Directory com o Asset Bank | Microsoft Docs"
description: "Saiba como configurar o logon único entre o Azure Active Directory e o Asset Bank."
services: active-directory
documentationcenter: 
author: jeevansd
manager: femila
editor: 
ms.assetid: 3006ad6e-8831-41cd-94aa-7e7ae770ce7b
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/10/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 325d92e493f6e011367d2c85b52c92838327101e
ms.openlocfilehash: 3ef8a204144461092cdce2e797116ed51d4e7411
ms.lasthandoff: 02/17/2017


---
# <a name="tutorial-azure-active-directory-integration-with-asset-bank"></a>Tutorial: Integração do Azure Active Directory ao Asset Bank
O objetivo desse tutorial é mostrar como integrar o Asset Bank ao Azure AD (Azure Active Directory).

A integração do Asset Bank ao Azure AD oferece os seguintes benefícios:

* É possível controlar no Azure AD quem terá acesso ao Asset Bank
* Você pode permitir que os usuários façam logon automaticamente no Asset Bank (logon único) com suas contas do Azure AD
* Gerenciar suas contas em um único local: o Portal clássico do Azure

Para conhecer mais detalhadamente a integração de aplicativos de SaaS ao AD do Azure, consulte [O que é o acesso a aplicativos e logon único com o Active Directory do Azure](active-directory-appssoaccess-whatis.md).

## <a name="prerequisites"></a>Pré-requisitos
Para configurar a integração do Azure AD com o Asset Bank, você precisa dos seguintes itens:

* Uma assinatura do AD do Azure
* Uma assinatura habilitada para SSO (logon único) do Asset Bank

>[!NOTE]
>Para testar as etapas deste tutorial, nós não recomendamos o uso de um ambiente de produção.
>  

Para testar as etapas deste tutorial, você deve seguir estas recomendações:

* Não use o ambiente de produção, a menos que seja necessário.
* Se não tiver um ambiente de avaliação do AD do Azure, você pode obter uma versão de avaliação de um mês [aqui](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Descrição do cenário
O objetivo deste tutorial é permitir que você teste o logon único do Azure AD em um ambiente de teste. 

O cenário descrito neste tutorial consiste em dois blocos de construção principais:

* Adicionando o Asset Bank por meio da galeria
* Configurar e testar o logon único do AD do Azure

## <a name="add-asset-bank-from-the-gallery"></a>Adicionar o Asset Bank por meio da galeria
Para configurar a integração do Asset Bank ao Azure AD, você precisa adicionar o Asset Bank por meio da galeria à sua lista de aplicativos SaaS gerenciados.

**Para adicionar o Asset Bank por meio da galeria, realize as seguintes etapas:**

1. No **portal clássico do Azure**, no painel de navegação à esquerda, clique em **Active Directory**. 
   
    ![Active Directory][1]
2. Na lista **Diretório** , selecione o diretório para o qual você deseja habilitar a integração de diretórios.
3. Para abrir a visualização dos aplicativos, na exibição do diretório, clique em **Aplicativos** no menu principal.
   
    ![Aplicativos][2]
4. Clique em **Adicionar** na parte inferior da página.
   
    ![Aplicativos][3]
5. Na caixa de diálogo **O que você deseja fazer**, clique em **Adicionar um aplicativo da galeria**.
   
    ![Aplicativos][4]
6. Na caixa de pesquisa, digite **Asset Bank**.
   
    ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_01.png)
7. No painel de resultados, selecione **Banco de Ativos**, em seguida, clique em **Concluir** para adicionar o aplicativo.
   
    ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_02.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurar e testar logon único do Azure AD
O objetivo desta seção é mostrar como configurar e testar o logon único do Azure AD com o Asset Bank, com base em um usuário de teste chamado “Brenda Fernandes”.

Para que o logon único funcione, o Azure AD precisa saber qual usuário do Asset Bank é equivalente a um usuário do Azure AD. Em outras palavras, é necessário estabelecer uma relação de vínculo entre um usuário do Azure AD e o usuário relacionado no Asset Bank.

Essa relação de vínculo é estabelecida atribuindo o valor do **nome de usuário** no Azure AD como o valor do **Username** no Banco de Ativos.

Para configurar e testar o logon único do Azure AD com o Asset Bank, você precisa concluir os seguintes blocos de construção:

1. **[Configurar logon único do Azure AD](#configuring-azure-ad-single-sign-on)** – para habilitar seus usuários a usar esse recurso.
2. **[Criação de um usuário de teste do AD do Azure](#creating-an-azure-ad-test-user)** - para testar o logon único do AD do Azure com Brenda Fernandes.
3. **[Criando um usuário de teste do Asset Bank](#creating-a-asset-bank-test-user)** : para ter um equivalente de Brenda Fernandes no Asset Bank que esteja vinculado à representação dela no Azure AD.
4. **[Atribuição do usuário de teste do AD do Azure](#assigning-the-azure-ad-test-user)** - para habilitar Britta Simon a usar o logon único do Azure AD.
5. **[Teste do logon único](#testing-single-sign-on)** – para verificar se a configuração funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configurar o logon único do Azure AD
O objetivo desta seção é habilitar o logon único do Azure AD no portal clássico do Azure e configurar o logon único em seu aplicativo do Asset Bank.

**Para configurar o logon único do Azure AD com o Asset Bank, realize as seguintes etapas:**

1. No portal clássico do Azure, na página de integração de aplicativos **Banco de Ativos**, clique em **Configurar logon único** para abrir a caixa de diálogo **Configurar Logon Único**.

![Configurar o logon único][6] 

1. Na página **Como você gostaria que os usuários fizessem logon no Banco de Ativos**, selecione **Logon Único do Azure AD**, em seguida, clique em **Próximo**.
   
    ![Configurar o logon único](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_03.png) 
2. Na página de diálogo **Definir Configurações de Aplicativo** , execute as seguintes etapas:
   
    ![Configurar Logon Único](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_04.png) 
    1. Na caixa de texto URL de Logon, digite a URL usada pelos usuários para fazer logon no seu aplicativo Banco de Ativos usando o seguinte padrão: **“https://\<nome empresa\>.assetbank-server.com”**
    2. Clique em **Próximo**.
1. Na página **Configurar logon único no Asset Bank** , execute as seguintes etapas:
   
    ![Configurar Logon Único](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_05.png)    
    1. Clique em **Baixar metadados**e salve o arquivo no computador.
    2. Clique em **Próximo**.
2. Para configurar o SSO para seu aplicativo, entre em contato com a equipe de suporte do Asset Bank por meio de [support@assetbank.co.uk](mailto:support@assetbank.co.uk) e anexe o arquivo de metadados ao email.
3. No portal clássico do Azure, selecione a confirmação da configuração de logon único e clique em **Avançar**.
   
    ![Logon Único do AD do Azure][10]
4. Na página **Confirmação de logon único**, clique em **Concluir**.  
   
    ![Logon Único do AD do Azure][11]

### <a name="create-an-azure-ad-test-user"></a>Criar um usuário de teste do Azure AD
O objetivo desta seção é criar um usuário de teste no Portal Clássico do Azure chamado Brenda Fernandes.

Na lista de usuários, selecione **Brenda Fernandes**.

![Criar um usuário do AD do Azure][20]

**Para criar um usuário de teste no AD do Azure, execute as seguintes etapas:**

1. No **Portal clássico do Azure**, no painel de navegação à esquerda, clique em **Active Directory**.
   
    ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/create_aaduser_09.png) 
2. Na lista **Diretório** , selecione o diretório para o qual você deseja habilitar a integração de diretórios.
3. Para exibir a lista de usuários, no menu na parte superior, clique em **Usuários**.
   
    ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/create_aaduser_03.png) 
4. Para abrir a caixa de diálogo **Adicionar Usuário**, na barra de ferramentas na parte inferior, clique em **Adicionar Usuário**.
   
    ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/create_aaduser_04.png) 
5. Na página do diálogo **Conte-nos sobre este usuário** , realize as seguintes etapas:
   
    ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/create_aaduser_05.png) 
    1. Em Tipo de Usuário, selecione Novo usuário na organização.
    2. Na **caixa de texto** Nome do Usuário, digite **BrendaFernandes**.
    3. Clique em **Próximo**.
6. Na página do diálogo **Perfil do Usuário** , realize as seguintes etapas:
   
   ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/create_aaduser_06.png) 
   1. Na caixa de texto **Nome**, digite **Brenda**.  
   2. Na caixa de texto **Sobrenome**, digite **Fernandes**.
   3. Na caixa de texto **Nome de Exibição**, digite **Brenda Fernandes**.
   4. Na lista **Função**, selecione **Usuário**.
   5. Clique em **Próximo**.
7. Na página de diálogo **Obter senha temporária**, clique em **criar**.
    ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/create_aaduser_07.png) 
8. Na página de caixa de diálogo **Obter senha temporária** , execute as seguintes etapas:
   
    ![Criação de um usuário de teste do AD do Azure](./media/active-directory-saas-assetbank-tutorial/create_aaduser_08.png) 
    1. Anote o valor da **Nova Senha**.
    2. Clique em **Concluído**.   

### <a name="create-a-asset-bank-test-user"></a>Criar um usuário de teste do Asset Bank
O objetivo desta seção é criar um usuário chamado Brenda Fernandes no Asset Bank. O Asset Bank dá suporte ao provisionamento just-in-time, que é habilitado por padrão.

Não há itens de ação para você nesta seção. Um novo usuário será criado durante uma tentativa de acessar o Asset Bank, caso ainda não exista. 

>[!NOTE]
>Se precisar criar um usuário manualmente, entre em contato com a equipe de suporte do Asset Bank.
> 

### <a name="assign-the-azure-ad-test-user"></a>Atribuir o usuário de teste do Azure AD
O objetivo desta seção é permitir que Brenda Fernandes use o SSO do Azure, concedendo a ela acesso ao Asset Bank.

![Atribuir usuário][200] 

**Para atribuir Brenda Fernandes ao Asset Bank, realize as seguintes etapas:**

1. No portal clássico do Azure, para abrir o modo de exibição de aplicativos, na exibição de diretório, clique em **Aplicativos** no menu superior.
   
    ![Atribuir usuário][201] 
2. Na lista de aplicativos, selecione **Asset Bank**.
   
    ![Configurar Logon Único](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_50.png) 
3. No menu na parte superior, clique em **Usuários**.
   
    ![Atribuir usuário][203] 
4. Na lista de usuários, selecione **Brenda Fernandes**.
5. Na barra de ferramentas na parte inferior, clique em **Atribuir**.
   
    ![Atribuir usuário][205]

### <a name="test-single-sign-on"></a>Testar logon único
O objetivo desta seção é testar sua configuração de logon único do Azure AD usando o Painel de Acesso.

Ao clicar no bloco Asset Bank no Painel de Acesso, você deverá ser conectado automaticamente ao aplicativo do Asset Bank.

## <a name="additional-resources"></a>Recursos adicionais
* [Lista de tutoriais sobre como integrar aplicativos SaaS com o Active Directory do Azure](active-directory-saas-tutorial-list.md)
* [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](active-directory-appssoaccess-whatis.md)

<!--Image references-->

[1]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_205.png

