---
title: "Serviços de Domínio do Azure AD: habilitar a sincronização de senha | Microsoft Docs"
description: "Introdução aos Serviços de Domínio do Active Directory do Azure"
services: active-directory-ds
documentationcenter: 
author: mahesh-unnikrishnan
manager: stevenpo
editor: curtand
ms.assetid: 5a32a0df-a3ca-4ebe-b980-91f58f8030fc
ms.service: active-directory-ds
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 03/06/2017
ms.author: maheshu
translationtype: Human Translation
ms.sourcegitcommit: 8a531f70f0d9e173d6ea9fb72b9c997f73c23244
ms.openlocfilehash: e4b1907b95576468654703c843a5f6e06846814b
ms.lasthandoff: 03/10/2017


---
# <a name="enable-password-synchronization-to-azure-ad-domain-services"></a>Habilitar a sincronização de senhas para os Serviços de Domínio do Azure AD
Nas tarefas anteriores, você habilitou os serviços de domínio do Azure AD para seu locatário do Azure AD. A próxima tarefa é habilitar os hashes de credenciais necessários para a autenticação Kerberos e NTLM para sincronização com os serviços de domínio do Azure AD. Depois que a sincronização de credenciais é configurada, os usuários podem entrar no domínio gerenciado usando suas credenciais corporativas.

As etapas envolvidas são diferentes dependendo de sua organização ser um locatário do Azure AD somente na nuvem ou estar configurada para sincronização com seu diretório local usando o Azure AD Connect.

<br>

> [!div class="op_single_selector"]
> * [Locatário do Azure AD somente na nuvem](active-directory-ds-getting-started-password-sync.md)
> * [Locatário do Azure AD sincronizado](active-directory-ds-getting-started-password-sync-synced-tenant.md)
>
>

<br>

## <a name="task-5-enable-password-synchronization-to-aad-domain-services-for-a-cloud-only-azure-ad-tenant"></a>Tarefa 5: habilitar a sincronização de senhas para os Serviços de Domínio do AAD para um locatário do Azure AD somente na nuvem
Os Serviços de domínio do Azure AD precisam de hashes de credenciais em um formato adequado para a autenticação NTLM e Kerberos, para autenticar os usuários no domínio gerenciado. A menos que você habilite os serviços de domínio AAD para seu locatário, o Azure AD não gera nem armazena hashes de credenciais no formato necessário para a autenticação NTLM ou Kerberos. Por motivos de segurança óbvios, o Azure AD também não armazena credenciais no formato de texto não criptografado. Portanto, o Azure AD não tem uma maneira de gerar esses hashes de credenciais NTLM ou Kerberos com base em credenciais de usuários existentes.

> [!NOTE]
> Se a sua organização tem um locatário do Azure AD somente na nuvem, os usuários que precisam usar os Serviços de Domínio do Azure AD devem alterar suas senhas.
>
>

Esse processo de alteração de senhas faz com que os hashes de credencial requeridos pelos Serviços de Domínio do AD do Azure para a autenticação Kerberos e NTLM sejam gerados no AD do Azure. Você também pode expirar senhas para todos os usuários no locatário que precisam usar os Serviços de Domínio do AD do Azure ou instruir esses usuários para alterar suas senhas.

### <a name="enable-ntlm-and-kerberos-credential-hash-generation-for-a-cloud-only-azure-ad-tenant"></a>Habilitar a geração de hashes de credencial Kerberos e NTLM para um locatário do Azure AD somente na nuvem
Aqui estão as instruções que você precisa fornecer aos usuários finais para que eles possam alterar suas senhas:

1. Navegue até a página do Painel de Acesso do Azure AD para sua organização em [http://myapps.microsoft.com](http://myapps.microsoft.com).
2. Selecione a guia **perfil** nesta página.
3. Clique no bloco **Alterar senha** nesta página.

    ![Criar uma rede virtual para os Serviços de Domínio do AD do Azure.](./media/active-directory-domain-services-getting-started/user-change-password.png)

   > [!NOTE]
   > Se você não vir a opção **Alterar senha** na página do Painel de Acesso, verifique se sua organização configurou [gerenciamento de senhas no Azure AD](../active-directory/active-directory-passwords-getting-started.md).
   >
   >
4. Na página **Alterar senha** , digite sua senha (antiga) existente e, em seguida, digite uma nova senha e confirme-a. Clique em **Enviar**.

    ![Criar uma rede virtual para os Serviços de Domínio do AD do Azure.](./media/active-directory-domain-services-getting-started/user-change-password2.png)

Depois que você alterar sua senha, a nova senha poderá ser usada nos serviços de domínio do Azure AD em breve. Depois de alguns minutos (normalmente, cerca de 20 minutos), você pode entrar computadores que ingressaram no domínio gerenciado usando a senha recém-alterada.

<br>

## <a name="related-content"></a>Conteúdo relacionado
* [Como atualizar sua própria senha](../active-directory/active-directory-passwords-update-your-own-password.md#how-to-reset-your-password).)
* [Introdução ao Gerenciamento de Senhas no Azure AD](../active-directory/active-directory-passwords-getting-started.md).
* [Habilitar a sincronização de senhas para os Serviços de Domínio do AAD para um locatário do Azure AD sincronizado](active-directory-ds-getting-started-password-sync-synced-tenant.md)
* [Administrar um domínio gerenciado dos Serviços de Domínio do Azure AD](active-directory-ds-admin-guide-administer-domain.md)
* [Ingressar em uma máquina virtual do Windows para um domínio gerenciado dos Serviços de Domínio do AD do Azure](active-directory-ds-admin-guide-join-windows-vm.md)
* [Ingressar em uma máquina virtual do Red Hat Enterprise Linux para um domínio gerenciado dos Serviços de Domínio do AD do Azure](active-directory-ds-admin-guide-join-rhel-linux-vm.md)

