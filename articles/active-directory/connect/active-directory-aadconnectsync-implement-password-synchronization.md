---
title: "Implementação de sincronização de senha com a sincronização do Azure AD Connect | Microsoft Docs"
description: "Fornece informações sobre como funciona a sincronização de senha e como habilitá-la."
services: active-directory
documentationcenter: 
author: MarkusVi
manager: femila
editor: 
ms.assetid: 05f16c3e-9d23-45dc-afca-3d0fa9dbf501
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/13/2017
ms.author: markvi
translationtype: Human Translation
ms.sourcegitcommit: 64b6447608ecdd9bdd2b307f4bff2cae43a4b13f
ms.openlocfilehash: cff066ff2943443749ee8eb2ef71c7ca93bb829c
ms.lasthandoff: 03/01/2017


---
# <a name="implementing-password-synchronization-with-azure-ad-connect-sync"></a>Implementação de sincronização de senha com a sincronização do Azure AD Connect
Este tópico fornece as informações necessárias para sincronizar suas senhas de usuário do AD (Active Directory) local para um Azure AD (Azure Active Directory) baseado na nuvem.

## <a name="what-is-password-synchronization"></a>O que é sincronização de senha
A probabilidade de você ser impedido de concluir seu trabalho devido a uma senha esquecida está relacionada ao número de senhas diferentes que você precisa se lembrar. Quanto mais senhas você precisa lembrar, maior a probabilidade de se esquecer uma delas. Perguntas e chamadas sobre redefinições de senha e outros problemas relacionados à senha demandam a maior parte dos recursos de assistência técnica.

A sincronização de senha é um recurso para sincronizar senhas de usuário de um Active Directory local para um Azure Active Directory (AD do Azure) local.
Esse recurso permite que você entre nos serviços do Azure Active Directory (como Office 365, Microsoft Intune, CRM Online e Serviços de Domínio do Azure AD) usando a mesma senha usada para entrar no Active Directory local.

![O que é o Azure AD Connect](./media/active-directory-aadconnectsync-implement-password-synchronization/arch1.png)

Reduzindo o número de senhas que os usuários precisam manter para apenas uma, a sincronização de senha ajuda a:

* Aumentar a produtividade dos seus usuários
* Reduzir os custos de assistência técnica  

Se você optar por usar a [**Federação com o AD FS**](https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Configuring-AD-FS-for-user-sign-in-with-Azure-AD-Connect), outra opção será habilitar a sincronização de senha como um backup em caso de falha na infraestrutura do AD FS.

A sincronização de senha é uma extensão para o recurso de sincronização de diretório, implementado pelo Azure AD Connect Sync. Para usar a sincronização de senha no seu ambiente, você precisa:

* Instalar o Azure AD Connect  
* Configure a sincronização de diretórios entre seu AD local e o Azure Active Directory
* Habilitar a sincronização de senha

Para saber mais, consulte [Integrar suas identidades locais ao Azure Active Directory](active-directory-aadconnect.md)

> [!NOTE]
> Para obter mais detalhes sobre os Serviços de Domínio do Active Directory, que são configurados para sincronização de senha e FIPS, consulte [Sincronização de senha e FIPS](#password-synchronization-and-fips).
>
>

## <a name="how-password-synchronization-works"></a>Como a sincronização de senha funciona
O serviço de domínio do Active Directory armazena as senhas na forma de uma representação do valor de hash da senha de usuário real. Um valor de hash é um resultado de uma função matemática unidirecional (o "*algoritmo de hash*"). Não há um método para reverter o resultado de uma função unidirecional para a versão de texto sem formatação de uma senha. Não é possível usar o hash de senha para entrar na sua rede local.

Para sincronizar sua senha, a sincronização do Azure AD Connect extrai o hash da senha de usuário do Active Directory local. O processamento de segurança adicional é aplicado ao hash da senha antes que ele seja sincronizado com o serviço de autenticação do Azure Active Directory. As senhas são sincronizadas por usuário e em ordem cronológica.

O fluxo de dados real do processo de sincronização de senha é semelhante à sincronização de dados de usuário como DisplayName ou Endereços de Email. Contudo, as senhas são sincronizadas com mais frequência do que a janela de sincronização de diretório padrão para outros atributos. O processo de sincronização de senha é executado a cada 2 minutos. Não é possível modificar a frequência desse processo. Ao sincronizar uma senha, ela substituirá a senha de nuvem existente.

Na primeira vez que você habilitar o recurso de sincronização de senha, ele executará uma sincronização inicial das senhas de todos os usuários no escopo. Você não pode definir explicitamente um subconjunto de senhas de usuário que deseja sincronizar.

Quando você altera uma senha local, a senha atualizada é sincronizada, geralmente em questão de minutos.
O recurso de sincronização de senha repete automaticamente tentativas de sincronização com falha. Se ocorrer um erro durante uma tentativa de sincronização de senha, um erro é registrado no seu visualizador de eventos.

A sincronização de uma senha não afeta um usuário conectado atualmente.
Sua sessão de serviço de nuvem atual não será afetada imediatamente por uma alteração de senha sincronizada que ocorre enquanto você está logado em um serviço de nuvem. No entanto, quando o serviço de nuvem exigir que você se autentique novamente, será necessário fornecer a nova senha.

> [!NOTE]
> Somente há suporte para a sincronização de senha para o usuário do tipo de objeto no Active Directory. Não há suporte para o tipo de objeto iNetOrgPerson.
>
>

### <a name="how-password-synchronization-works-with-azure-ad-domain-services"></a>Como a sincronização de senha funciona com os Serviços de Domínio do AD do Azure
Você também pode usar o recurso de sincronização de senha para sincronizar suas senhas locais para os [Serviços de Domínio do Azure AD](../../active-directory-domain-services/active-directory-ds-overview.md). Esse cenário permite que os Serviços de Domínio do Azure AD autentiquem os usuários na nuvem com todos os métodos disponíveis no AD local. A experiência desse cenário é semelhante a usar a ADMT (Ferramenta de Migração do Active Directory) em um ambiente local.

### <a name="security-considerations"></a>Considerações de segurança
Ao sincronizar senhas, a versão da sua senha em texto sem formatação não é exposta ao recurso de sincronização de senha, nem ao Azure AD ou qualquer um dos serviços associados.

Além disso, não há nenhum requisito no Active Directory local para armazenar a senha em um formato criptografado de modo reversível. Um resumo do hash da senha do Active Directory é usado para a transmissão entre o AD local e o Active Directory do Azure. O resumo do hash da senha não pode ser usado para acessar recursos no seu ambiente do local.

### <a name="password-policy-considerations"></a>Considerações sobre política de senha
Há dois tipos de políticas de senha que são afetadas ao habilitar a sincronização de senha:

1. Política de complexidade de senha
2. Política de expiração de senha

**Password complexity policy**  
Ao habilitar a sincronização de senha, as políticas de complexidade de senha no Active Directory local substituem as políticas de complexidade na nuvem para usuários sincronizados. Você pode usar todas as senhas válidas do seu Active Directory local para acessar os serviços do Azure AD.

> [!NOTE]
> Senhas de usuários criadas diretamente na nuvem ainda estão sujeitas a políticas de senha, conforme definido na nuvem.
>
>

**Password expiration policy**  
Se um usuário estiver no escopo de sincronização de senha, a senha da conta de nuvem será definida como "*Nunca Expirar*".
Você pode continuar entrando nos serviços de nuvem usando uma senha sincronizada que expirou no seu ambiente local. A senha de nuvem será atualizada na próxima vez que você alterar a senha no ambiente local.

### <a name="overwriting-synchronized-passwords"></a>Substituindo senhas sincronizadas
Um administrador pode redefinir sua senha manualmente usando o Windows PowerShell.

Nesse caso, a nova senha substitui a senha sincronizada e todas as políticas de senha definidas na nuvem se aplicam à nova senha.

Se você alterar a senha local novamente, a nova senha será sincronizada com a nuvem e substituirá a senha atualizada manualmente.

## <a name="enabling-password-synchronization"></a>Habilitando a sincronização de senha
A sincronização de senha é habilitada automaticamente quando você instala o Azure AD Connect usando as **Configurações Expressas**. Para obter mais detalhes, confira [Introdução ao Azure AD Connect usando configurações expressas](active-directory-aadconnect-get-started-express.md).

Se você usar configurações personalizadas quando instalar o Azure AD Connect, poderá habilitar a sincronização de senha na página de entrada do usuário. Para obter mais detalhes, confira [Instalação personalizada do Azure AD Connect](active-directory-aadconnect-get-started-custom.md).

![Habilitando a sincronização de senha](./media/active-directory-aadconnectsync-implement-password-synchronization/usersignin.png)

### <a name="password-synchronization-and-fips"></a>Sincronização de senha e FIPS
Se o servidor foi bloqueado de acordo com o FIPS (Federal Information Processing Standard), o MD5 foi desabilitado.

**Para habilitar MD5 para a sincronização de senha, execute as seguintes etapas:**

1. Vá para **%programfiles%\Azure AD Sync\Bin**.
2. Abra **miiserver.exe.config**.
3. Acesse o nó **configuração/tempo de execução** (no fim do arquivo).
4. Adicione o seguinte nó: `<enforceFIPSPolicy enabled="false"/>`
5. Salve suas alterações.

Para referência, a captura mostra como deve ser a aparência:

```
    <configuration>
        <runtime>
            <enforceFIPSPolicy enabled="false"/>
        </runtime>
    </configuration>
```

Para obter informações sobre segurança e FIPS, veja [Sincronização de senha do AAD, criptografia e conformidade FIPS](https://blogs.technet.microsoft.com/enterprisemobility/2014/06/28/aad-password-sync-encryption-and-fips-compliance/)

## <a name="troubleshooting-password-synchronization"></a>Solução de problemas de sincronização de senha
Caso tenha problemas com a sincronização de senha, consulte [Solucionar problemas de sincronização de senha](active-directory-aadconnectsync-troubleshoot-password-synchronization.md).

## <a name="next-steps"></a>Próximas etapas
* [Azure AD Connect Sync: personalizando opções de sincronização](active-directory-aadconnectsync-whatis.md)
* [Integração de suas identidades locais com o Active Directory do Azure](active-directory-aadconnect.md)

