
---
title: "Usar o Azure v2.0 para acessar recursos protegidos sem interação do usuário | Microsoft Docs"
description: "Criar aplicativos Web usando a implementação do Azure AD do protocolo de autenticação OAuth 2.0."
services: active-directory
documentationcenter: 
author: dstrockis
manager: mbaldwin
editor: 
ms.assetid: 9b7cfbd7-f89f-4e33-aff2-414edd584b07
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/07/2017
ms.author: dastrock
translationtype: Human Translation
ms.sourcegitcommit: 08e2e561b6b939de4d829c816c1948361e16f6c4
ms.openlocfilehash: 2bbfed6e14a77964a32b84ae3f78f3529f13043c


---
# <a name="azure-active-directory-v20-and-the-oauth-20-client-credentials-flow"></a>Fluxo de credenciais do Azure Active Directory v2.0 e de cliente OAuth 2.0
Você pode usar a [concessão de credenciais do cliente OAuth 2.0](http://tools.ietf.org/html/rfc6749#section-4.4), às vezes chamada de *OAuth de duas pernas*, para acessar os recursos hospedados na Web usando a identidade de um aplicativo. Esse tipo de concessão normalmente é usado para interações de servidor para servidor que devem ser executadas em segundo plano, sem interação imediata com um usuário. Esses tipos de aplicativo normalmente são mencionados como *daemons* ou *contas de serviço*.

> [!NOTE]
> O ponto de extremidade v2.0 não dá suporte a todos os cenários e recursos do Azure Active Directory. Para determinar se você deve usar o ponto de extremidade v2.0, leia sobre as [limitações da v2.0](active-directory-v2-limitations.md).
> 
> 

No tipo mais comum de *OAuth com três pernas*, um aplicativo cliente tem permissão para acessar um recurso em nome de um usuário específico. A permissão é delegada do usuário para o aplicativo, geralmente durante o processo de [consentimento](active-directory-v2-scopes.md). No entanto, no fluxo de credenciais do cliente, as permissões são concedidas diretamente ao próprio aplicativo. Quando o aplicativo apresenta um token a um recurso, esse impõe que o próprio aplicativo tenha autorização para executar uma ação, e não que algum usuário tenha essa autorização.

## <a name="protocol-diagram"></a>Diagrama de protocolo
O fluxo de credenciais de cliente inteira é similar ao próximo diagrama. Descrevemos cada uma das etapas neste artigo.

![Fluxo de credenciais do cliente](../../media/active-directory-v2-flows/convergence_scenarios_client_creds.png)

## <a name="get-direct-authorization"></a>Obter autorização direta
Um aplicativo normalmente recebe autorização direta para acessar um recurso de uma destas duas maneiras: por meio de uma ACL (lista de controle de acesso) no recurso ou pela atribuição de permissão de aplicativo no Azure AD (Azure Active Directory). Esses dois métodos são os mais comuns no Azure AD e são recomendados para clientes e recursos que executam o fluxo de credenciais de cliente. Um recurso pode optar por autorizar seus clientes de outras maneiras, no entanto. Cada servidor de recursos pode escolher o método que faz mais sentido para seu aplicativo.

### <a name="access-control-lists"></a>Listas de controle de acesso
Um provedor de recursos pode impor uma verificação de autorização com base em uma lista de IDs de Aplicativo que ele conhece e conceder nível de acesso específico. Quando o recurso recebe um token do ponto de extremidade v2.0, ele pode decodificar o token e extrair a ID do Aplicativo do cliente das declarações `appid` e `iss`. Em seguida, ele compara o aplicativo em relação a uma ACL que ele mantém. Granularidade e o método a ACL podem variar significativamente entre os recursos.

Um caso de uso comum é usar uma ACL para executar testes para um aplicativo web ou para uma API Web. A API Web pode conceder apenas um subconjunto das permissões completas para um cliente específico. Para executar testes de ponta a ponta na API, um cliente de teste é criado para adquirir tokens do ponto de extremidade v2.0 e então os enviar para a API. A API então verifica a ACL para a ID do Aplicativo do cliente de teste a fim de ter acesso completo à funcionalidade total da API. Se você usar esse tipo de ACL, certifique-se de validar não apenas o valor `appid` do chamador. Também valide que o valor `iss` do token é confiável.

Esse tipo de autorização é comum para daemons e contas de serviço que precisam acessar dados pertencentes a usuários consumidores com contas pessoais da Microsoft. Para dados de propriedade de organizações, é recomendável que você obtenha a autorização necessária por meio de permissões de aplicativo.

### <a name="application-permissions"></a>Permissões de aplicativo
Em vez de usar ACLs, você pode usar APIs para expor um conjunto de permissões do aplicativo. Uma permissão de aplicativo é concedida a um aplicativo por um administrador de uma organização e só pode ser usada para acessar os dados pertencentes a essa organização e a seus funcionários. Por exemplo, o Microsoft Graph expõe várias permissões de aplicativo para fazer o seguinte:

* Ler emails em todas as caixas de correio
* Ler e gravar mensagens em todas as caixas de correio
* Enviar emails como qualquer usuário
* Ler dados do diretório

Para saber mais sobre permissões de aplicativo, vá para [Microsoft Graph](https://graph.microsoft.io).

Para usar permissões de aplicativo em seu aplicativo, siga as etapas que discutiremos nas próximas seções.

#### <a name="request-the-permissions-in-the-app-registration-portal"></a>Solicitar as permissões no portal de registro do aplicativo
1. Vá para seu aplicativo no [Portal de Registro de Aplicativo](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList) ou [crie um aplicativo](active-directory-v2-app-registration.md) se ainda não tiver feito isso. Você precisará usar pelo menos um segredo do aplicativo quando você cria seu aplicativo.
2. Localize a seção **Permissões diretas do aplicativo** e adicione as permissões que seu aplicativo requer.
3. **Salve** o registro do aplicativo.

#### <a name="recommended-sign-the-user-in-to-your-app"></a>Recomendado: conectar o usuário ao aplicativo
Normalmente, quando você cria um aplicativo que usa as permissões de aplicativo, o aplicativo exige uma página ou exibição na qual o administrador aprova as permissões do aplicativo. Esta página pode ser parte do fluxo de entrada do aplicativo, parte das configurações do aplicativo ou pode ser um fluxo dedicado de "conexão". Em muitos casos, faz sentido que o aplicativo somente mostre o modo de exibição "conectar" depois que o usuário entra com uma conta corporativa ou de estudante da Microsoft.

Se você conectar o usuário ao seu aplicativo, você pode identificar a organização à qual o usuário pertence antes de você perguntar ao usuário para aprovar as permissões do aplicativo. Embora não seja estritamente necessário, isso pode ajudá-lo a criar uma experiência mais intuitiva para os usuários. Para conectar o usuário, siga nossos [tutoriais de protocolo v2.0](active-directory-v2-protocols.md).

#### <a name="request-the-permissions-from-a-directory-admin"></a>Solicitar permissões de um administrador de diretório
Quando você estiver pronto para solicitar permissões de administrador da organização, redirecione o usuário para o *ponto de extremidade de consentimento do administrador* v2.0.

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
```

```
// Pro tip: Try pasting the following request in a browser!
```

```
https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions
```

| Parâmetro | Condição | Descrição |
| --- | --- | --- |
| locatário |Obrigatório |O locatário do diretório para o qual você deseja solicitar permissão. Pode estar no formato de nome amigável ou de GUID. Se você não souber a qual locatário o usuário pertence e se quiser deixá-lo entrar com qualquer locatário, use `common`. |
| client_id |Obrigatório |A ID de aplicativo do [Portal de Registro de Aplicativo](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList) atribuída ao seu aplicativo. |
| redirect_uri |Obrigatório |O URI de redirecionamento onde você deseja que a resposta seja enviada para ser tratada pelo aplicativo. Ele deve corresponder exatamente a um dos URIs de redirecionamento que você registrou no portal, com exceção de que ele deve ser codificado por URL e podem ter mais segmentos de caminho. |
| state |Recomendadas |Um valor incluído na solicitação que também é retornado na resposta do token. Pode ser uma cadeia de caracteres de qualquer conteúdo desejado. O estado é usado para codificar as informações sobre o estado do usuário no aplicativo antes da solicitação de autenticação ocorrida, como a página ou exibição em que ele estava. |

Neste ponto, o Azure AD impõe que somente um administrador de locatário pode entrar para concluir a solicitação. O administrador deverá aprovar todas as permissões diretas do aplicativo que você solicitou para o aplicativo no portal de registro de aplicativos.

##### <a name="successful-response"></a>Resposta bem-sucedida
Se o administrador aprovar as permissões para o seu aplicativo, a resposta bem-sucedida terá esta aparência:

```
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| Parâmetro | Descrição |
| --- | --- | --- |
| locatário |O locatário do diretório que concedeu as permissões solicitadas, no formato GUID. |
| state |Um valor incluído na solicitação que também é retornado na resposta do token. Pode ser uma cadeia de caracteres de qualquer conteúdo desejado. O estado é usado para codificar as informações sobre o estado do usuário no aplicativo antes da solicitação de autenticação ocorrida, como a página ou exibição em que ele estava. |
| admin_consent |Definido como **true**. |

##### <a name="error-response"></a>Resposta de erro
Se o administrador não aprovar as permissões para o seu aplicativo, a resposta de falha terá esta aparência:

```
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| Parâmetro | Descrição |
| --- | --- | --- |
| error |Uma cadeia de caracteres de código de erro que você pode usar para classificar os tipos de erros, e que você pode usar para reagir a erros. |
| error_description |Uma mensagem de erro específica que pode ajudar você a identificar a causa raiz de um erro. |

Depois de receber uma resposta bem-sucedida do ponto de extremidade de provisionamento do aplicativo, o aplicativo terá as permissões diretas solicitadas. Agora você pode solicitar um token para o recurso desejado.

## <a name="get-a-token"></a>Obter um token
Após adquirir a autorização necessária para seu aplicativo, continue adquirindo os tokens de acesso para as APIs. Para obter um token usando a concessão de credenciais do cliente, envie uma solicitação POST para o ponto de extremidade v2.0 do `/token`:

```
POST /common/oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials
```

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials' 'https://login.microsoftonline.com/common/oauth2/v2.0/token'
```

| Parâmetro | Condição | Descrição |
| --- | --- | --- |
| client_id |Obrigatório |A ID de aplicativo do [Portal de Registro de Aplicativo](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList) atribuída ao seu aplicativo. |
| scope |Obrigatório |O valor transmitido ao parâmetro `scope` na solicitação deve ser o identificador de recurso (URI de ID do Aplicativo) do recurso que você deseja, com o sufixo `.default`. Para o exemplo do Microsoft Graph, o valor é `https://graph.microsoft.com/.default`. Esse valor informa ao ponto de extremidade v2.0 que em relação a todas as permissões diretas de aplicativo que você configurou para seu aplicativo, ele deverá emitir um token para as associadas ao recurso que você deseja usar. |
| client_secret |Obrigatório |O Segredo do Aplicativo que você gerou para seu aplicativo no portal de registro do aplicativo. |
| grant_type |Obrigatório |Deve ser `client_credentials`. |

##### <a name="successful-response"></a>Resposta bem-sucedida
Uma resposta bem-sucedida tem a seguinte aparência:

```
{
  "token_type": "Bearer",
  "expires_in": 3599,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBP..."
}
```

| Parâmetro | Descrição |
| --- | --- |
| access_token |O token de acesso solicitado. O aplicativo pode usar esse token para se autenticar no recurso protegido, como uma API Web. |
| token_type |Indica o valor do tipo de token. O único tipo com suporte do Azure AD é o `bearer`. |
| expires_in |Por quanto tempo o token de acesso é válido (em segundos). |

##### <a name="error-response"></a>Resposta de erro
Uma resposta de erro tem esta aparência:

```
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/.default is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Parâmetro | Descrição |
| --- | --- |
| error |Uma cadeia de caracteres de códigos de erro que você pode usar para classificar tipos de erro que ocorrem e para responder aos erros. |
| error_description |Uma mensagem de erro específica que pode ajudar um desenvolvedor a identificar a causa raiz de um erro de autenticação. |
| error_codes |Uma lista de códigos de erro específicos do STS que pode ajudar no diagnóstico. |
| timestamp |A hora na qual o erro ocorreu. |
| trace_id |Um identificador exclusivo para a solicitação que pode ajudar no diagnóstico. |
| correlation_id |Um identificador exclusivo para a solicitação que pode ajudar no diagnóstico entre os componentes. |

## <a name="use-a-token"></a>Usar um token
Agora que você já adquiriu um token, use-o para fazer solicitações ao recurso. Quando o token expirar, repita a solicitação para o ponto de extremidade `/token` a fim de adquirir um novo token de acesso.

```
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

```
// Pro tip: Try the following command! (Replace the token with your own.)
```

```
curl -X GET -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q" 'https://graph.microsoft.com/v1.0/me/messages'
```

## <a name="code-sample"></a>Exemplo de código
Para ver um exemplo de aplicativo que implementa a concessão de credenciais de cliente usando o ponto de extremidade de consentimento de administrador, veja nosso [exemplo de código de daemon para v2.0](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2).




<!--HONumber=Jan17_HO4-->


