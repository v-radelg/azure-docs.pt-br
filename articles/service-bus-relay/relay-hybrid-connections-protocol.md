---
title: "Guia de Protocolo de Conexões Híbridas de Retransmissão do Azure | Microsoft Docs"
description: "Guia de protocolo de Conexões Híbridas de Retransmissão do Azure."
services: service-bus-relay
documentationcenter: na
author: clemensv
manager: timlt
editor: 
ms.assetid: 149f980c-3702-4805-8069-5321275bc3e8
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/14/2017
ms.author: sethm;clemensv
translationtype: Human Translation
ms.sourcegitcommit: 4a972b9b8b52a90f27afda98d8bdc661016d1fe1
ms.openlocfilehash: f5fd4c6c0b8db3fe91d8b57a68fe33dcff353a59
ms.lasthandoff: 02/28/2017


---
# <a name="azure-relay-hybrid-connections-protocol"></a>Protocolo de Conexões Híbridas de Retransmissão do Azure
A Retransmissão do Azure é um dos principais pilares de funcionalidades da plataforma do Barramento de Serviço do Azure. A nova funcionalidade "Conexões Híbridas" da Retransmissão é uma evolução segura e de protocolo aberto com base em HTTP e WebSockets. Ele substitui o antigo recurso chamado igualmente de "Serviços BizTalk", criado sobre uma base de protocolo proprietário. A integração do Conexões Híbridas aos Serviços de Aplicativos do Azure continuará a funcionar no estado em que se encontra.

O “Conexão Híbridas” possibilita uma comunicação com transmissão bidirecional binária entre dois aplicativos em rede, sendo que um ou ambos podem residir atrás de NATs ou de firewalls. Este artigo descreve as interações do lado do cliente com a retransmissão de Conexões Híbridas para conectar clientes nas funções de ouvinte e o remetente e o modo como os ouvintes aceitam novas conexões.

## <a name="interaction-model"></a>Modelo de interação
A retransmissão do Conexões Híbridas conecta duas partes, fornecendo um ponto de encontro na nuvem do Azure que as partes podem descobrir e ao qual podem se conectar da perspectiva da sua própria rede. Esse ponto de encontro é chamado de "Conexão Híbrida" nesta e em outras documentações, nas APIs e também no Portal do Azure. O ponto de extremidade de serviço de Conexões Híbridas será referido como "serviço" no restante deste artigo. O modelo de interação depende da nomenclatura estabelecida por muitas outras APIs de rede:

Há um ouvinte que primeiro indica a prontidão para lidar com conexões de entrada e, subsequentemente, aceita-as assim que chegam. Do outro lado, há um cliente conectado que se conecta ao ouvinte, esperando que essa conexão seja aceita para estabelecer um caminho de comunicação bidirecional.
"Conectar", "Ouvir" e "Aceitar" são os mesmos termos que você encontrará na maioria das APIs de soquete.

Qualquer modelo de comunicação retransmitida faz com que uma das partes realize conexões de saída em direção a um ponto de extremidade de serviço, o que torna o "ouvinte" também um "cliente" em uso coloquial e também pode causar outras sobrecargas de terminologia; assim, a terminologia precisa que usamos para as conexões híbridas é da seguinte maneira:

Os programas em ambos os lados de uma conexão são chamados de "cliente", pois são clientes para o serviço. O cliente que aguarda e aceita conexões é o "ouvinte", ou diz-se que faz a "função de ouvinte". O cliente que inicia uma nova conexão em direção a um ouvinte por meio do serviço é chamado de "remetente" ou faz a "função de remetente".

### <a name="listener-interactions"></a>Interações de ouvinte
O ouvinte tem quatro interações com o serviço; todos os detalhes de conexão são descritos neste documento na seção de referência.

#### <a name="listen"></a>Escutar
Para indicar a preparação para o serviço sinalizando que um ouvinte está pronto para aceitar conexões, ele cria uma conexão de soquete da Web de saída. O handshake de conexão recebe o mesmo nome de uma Conexão Híbrida configurada no namespace de Retransmissão, além de um token de segurança que confere o direito "Listen" (de escuta) nesse nome.
Quando o soquete da Web é aceito pelo serviço, o registro é concluído e o soquete da Web estabelecido é mantido ativo como "canal de controle" para habilitar todas as interações subsequentes. O serviço permite até 25 ouvintes simultâneos em uma Conexão Híbrida. Se houver 2 ou mais ouvintes ativos, as conexões de entrada serão balanceadas entre eles em ordem aleatória; não há garantia de distribuição justa.

#### <a name="accept"></a>Aceitar
Sempre que um remetente abrir uma nova conexão no serviço, o serviço escolherá e notificará um os ouvintes ativos na Conexão Híbrida. A notificação é enviada para o ouvinte no canal de controle aberto como uma mensagem JSON que contém a URL do ponto de extremidade de soquete da Web ao qual o ouvinte deve se conectar para aceitar a conexão.

A URL pode e deve ser usada diretamente pelo ouvinte sem qualquer trabalho extra; as informações codificadas são válidas apenas por um curto período de tempo, essencialmente o tempo pelo qual o remetente está disposto a esperar para que a conexão seja estabelecida de ponta a ponta, até um máximo de 30 segundos. A URL pode ser usada apenas para uma tentativa de conexão bem-sucedida. Assim que a conexão de soquete da Web com a URL da reunião é estabelecida, todas as atividades adicionais neste soquete da Web são retransmitidas de e para o remetente, sem nenhuma intervenção ou interpretação pelo serviço.

#### <a name="renew"></a>Renew
O token de segurança que deve ser usado para registrar o ouvinte e manter o canal de controle pode expirar enquanto o ouvinte está ativo. A expiração do token não afetará conexões contínuas, mas fará com que o canal de controle seja descartado pelo serviço ou logo após o momento da expiração. A operação "renew" é uma mensagem JSON que o ouvinte pode enviar para substituir o token associado ao canal de controle, para que o canal de controle possa ser mantido por períodos longos.

#### <a name="ping"></a>Ping
Se o canal de controle permanecer ocioso por muito tempo, intermediários no caminho como balanceadores de carga ou NATs poderão remover a conexão TCP. A operação "ping" evita isso por meio do envio de uma pequena quantidade de dados no canal, que lembra a todos na rota de rede que a conexão deve estar ativa e também serve como um teste de atividade para o ouvinte. Se o ping falhar, o canal de controle deverá ser considerado inutilizável e o ouvinte deverá se reconectar.

### <a name="sender-interaction"></a>Interação de remetente
O remetente tem apenas uma única interação com o serviço, ele se conecta.

#### <a name="connect"></a>Connect
A operação "connect" abre um soquete da Web no serviço, fornecendo o nome da Conexão Híbrida e um token de segurança (opcional, mas necessário por padrão) concedendo a permissão "Send" na cadeia de caracteres de consulta. O serviço interagirá com o ouvinte no modo descrito anteriormente e fará com que o ouvinte crie uma conexão de encontro que será associada a este soquete da Web. Após o soquete da Web ser aceito, todas as outras interações no soquete da Web serão, portanto, com um ouvinte conectado.

### <a name="interaction-summary"></a>Resumo da interação
O resultado desse modelo de interação é que o cliente remetente sai do handshake com um soquete da Web "limpo", que é conectado a um ouvinte e que não precisa de nenhum preâmbulo ou preparação adicionais. Isso permite que praticamente qualquer implementação de cliente de soquete da Web existente tire proveito imediatamente do serviço de Conexões Híbridas fornecendo apenas uma URL construída corretamente na camada de clientes do seu soquete da Web.

O soquete da Web da conexão de reunião que o ouvinte obtém por meio de interação com accept também é limpa e pode ser entregue a qualquer implementação de servidor de soquete da Web existente com alguma abstração extra mínima, que faz a distinção entre operações "accept" nos ouvintes de rede local da sua estrutura e operações "accept" remotas do Conexões Híbridas.

## <a name="protocol-reference"></a>Referência de protocolo

Esta seção descreve os detalhes das interações de protocolo descritas acima.

Todas as conexões de soquete da Web são feitas na porta 443 como uma atualização do HTTPS 1.1, o que é normalmente abstraído por alguma API ou estrutura de soquete da Web. A descrição aqui é mantida neutra em termos de implementação, sem sugerir uma estrutura específica.

### <a name="listener-protocol"></a>Protocolo de ouvinte
O protocolo de ouvinte consiste em dois gestos de conexão e três operações de mensagem.

#### <a name="listener-control-channel-connection"></a>Conexão de canal de controle do ouvinte
O canal de controle é aberto com a criação de uma conexão de soquete da Web para:

```
wss://{namespace-address}/$hc/{path}?sb-hc-action=...[&sb-hc-id=...]&sb-hc-token=...
```

O `namespace-address` é o nome de domínio totalmente qualificado do namespace de Retransmissão do Azure que hospeda a Conexão Híbrida, normalmente no formato `{myname}.servicebus.windows.net`.

As opções de parâmetro de cadeia de caracteres de consulta são conforme demonstrado a seguir.

| Parâmetro | Obrigatório | Descrição |
| --- | --- | --- |
| sb-hc-action |Sim |Para a função de ouvinte, o parâmetro deve ser **sb-hc-action=listen** |
| {caminho} |Sim |O caminho de namespace em formato codificado de URL da Conexão Híbrida pré-configurada na qual registrar este ouvinte. Esta expressão é acrescentada à parte do caminho `$hc/` fixa. |
| sb-hc-token |Sim\* |O ouvinte deve fornecer um Token de Acesso válido, compartilhado com o Barramento de Serviço, em formato codificado de URL para o namespace ou Conexão Híbrida que confere o direito **Listen** (escutar). |
| sb-hc-id |Não |Essa ID opcional fornecida pelo cliente permite o rastreamento de diagnóstico de ponta a ponta. |

Se a conexão de soquete da Web falhar porque o caminho de Conexão Híbrida não está sendo registrado, porque há um token inválido ou ausente ou por algum outro erro, os comentários de erro serão fornecidos usando o modelo comum de comentários de status HTTP 1.1. A descrição do status conterá uma ID de rastreamento de erro que poderá ser comunicada ao Suporte do Azure:

| Código | Erro | Descrição |
| --- | --- | --- |
| 404 |Não encontrado |O **caminho** da Conexão Híbrida é inválido ou a URL base está malformada. |
| 401 |Não Autorizado |O token de segurança está ausente ou malformado ou inválido. |
| 403 |Proibido |O token de segurança não é válido para esse caminho para essa ação. |
| 500 |Erro Interno |Algo deu errado no serviço. |

Se a conexão de soquete da Web for desligada intencionalmente pelo serviço depois que ele tiver sido inicialmente configurado, o motivo para fazer isso será comunicado usando código de erro de protocolo de soquete da Web apropriado juntamente com uma mensagem de erro descritiva, que também incluirá uma ID de rastreamento. O serviço não desligará o canal de controle sem encontrar uma condição de erro. Qualquer desligamento normal é controlado de cliente.

| Status WS | Descrição |
| --- | --- |
| 1001 |O caminho de Conexão Híbrida foi excluído ou desabilitado. |
| 1008 |O token de segurança expirou e a política de autorização foi, portanto, violada. |
| 1011 |Algo deu errado no serviço. |

### <a name="accept-handshake"></a>Handshake de aceitação
A notificação de aceitação é enviada pelo serviço ao ouvinte pelo canal de controle estabelecido anteriormente como uma mensagem JSON em um quadro de texto de soquete da Web. Não há resposta para esta mensagem.

A mensagem contém um objeto JSON chamado "accept", que define as seguintes propriedades neste momento:

* **endereço** – a cadeia de caracteres de URL a ser usada para estabelecer o soquete da Web com o serviço para aceitar uma conexão de entrada.
* **ID** – o identificador exclusivo para esta conexão. Se a ID tiver sido fornecida pelo cliente do remetente, será o valor fornecido pelo remetente; caso contrário, será um valor gerado pelo sistema.
* **connectHeaders** – todos os cabeçalhos HTTP que foram fornecidos pelo remetente ao ponto de extremidade de retransmissão, o que também inclui os cabeçalhos Sec-WebSocket-Protocol e Sec-WebSocket-Extensions.

#### <a name="accept-message"></a>Mensagem de Aceitação
```json
{                                                           
    "accept" : {
        "address" : "wss://168.61.148.205:443/$hc/{path}?..."    
        "id" : "4cb542c3-047a-4d40-a19f-bdc66441e736",  
        "connectHeaders" : {                                         
            "Host" : "...",                                                
            "Sec-WebSocket-Protocol" : "...",                              
            "Sec-WebSocket-Extensions" : "..."                             
        }                                                            
     }                                                         
}
```

A URL do endereço fornecida na mensagem JSON é usada pelo ouvinte para estabelecer o soquete da Web para aceitar ou rejeitar o soquete do remetente.

#### <a name="accepting-the-socket"></a>Aceitando o soquete
Para aceitar, o ouvinte estabelece uma conexão de soquete da Web com o endereço fornecido.

Observe que, para o período de visualização, o URI do endereço pode usar um endereço IP sem colchetes e o certificado TLS fornecido pelo servidor falhará na validação desse endereço.
Isso será corrigido durante a visualização.

Se a mensagem "accept" transportar um cabeçalho "Sec-WebSocket-Protocol", espera-se que o ouvinte só aceite o soquete da Web se ele der suporte a esse protocolo e que ele defina o cabeçalho conforme o soquete da Web for estabelecido.

O mesmo se aplica ao cabeçalho "Sec-WebSocket-Extensions". Se a estrutura der suporte a uma extensão, ela deverá definir o cabeçalho como a resposta do lado do *servidor* do handshake "Sec-WebSocket-Extensions" necessário para a extensão.

A URL deve ser usada no estado em que se encontra para estabelecer o soquete de aceitação, mas contém os seguintes parâmetros:

| Parâmetro | Obrigatório | Descrição |
| --- | --- | --- |
| sb-hc-action |Sim |Para aceitar um soquete, o parâmetro deverá ser `sb-hc-action=accept` |
| {caminho} |Sim |(confira no parágrafo a seguir) |
| sb-hc-id |Não |Consulte a descrição da **id** acima. |

O `{path}` é o namespace em formato codificado de URL da Conexão Híbrida pré-configurada na qual este ouvinte deve ser registrado. Esta expressão é acrescentada à parte do caminho `$hc/` fixa. 

A expressão de caminho MAY ser estendida com um sufixo e uma expressão de cadeia de caracteres de consulta que segue o nome registrado após uma barra de separação. Isso permite que o cliente remetente passe argumentos de expedição para o ouvinte destinatário quando não é possível incluir cabeçalhos HTTP. A expectativa é que a estrutura do ouvinte analisará a parte fixa do caminho e o nome registrado do caminho e fará o restante, possivelmente sem nenhum argumento da cadeia de caracteres de consulta precedido por "sb-", disponível para o aplicativo para decidir se deseja aceitar a conexão.

Para obter mais detalhes, consulte a seção "Sender Protocol" (Protocolo de remetente) a seguir.

Se houver um erro, o serviço poderá responder da seguinte maneira:

| Código | Erro | Descrição |
| --- | --- | --- |
| 403 |Proibido |A URL não é válida. |
| 500 |Erro Interno |Algo deu errado no serviço |

Depois que a conexão tiver sido estabelecida, o servidor será desligado do soquete da Web quando o soquete da Web do remetente desligar ou com o status a seguir.

| Status WS | Descrição |
| --- | --- |
| 1001 |O cliente remetente encerra a conexão. |
| 1001 |O caminho de Conexão Híbrida foi excluído ou desabilitado. |
| 1008 |O token de segurança expirou e, portanto, a política de autorização foi violada. |
| 1011 |Algo deu errado no serviço. |

#### <a name="rejecting-the-socket"></a>Rejeitando o soquete
Rejeitar o soquete depois de inspecionar a mensagem "accept" requer um handshake semelhante, de modo que o código de status e a descrição de status comunicando o motivo da rejeição possam fluir de volta ao remetente.

A escolha de design do protocolo aqui é usar um handshake de soquete da Web (projetado para terminar em um estado de erro definido) para que as implementações do ouvinte cliente possam continuar dependendo de um cliente de soquete da Web e não precisem empregar um cliente HTTP básico adicional.

Para rejeitar o soquete, o cliente usa o endereço URI da mensagem "accept" e acrescenta dois parâmetros da cadeia de caracteres de consulta a ele:

| Param | Obrigatório | Descrição |
| --- | --- | --- |
| statusCode |Sim |Código de status HTTP numérico. |
| statusDescription |Sim |Motivo da rejeição legível por humanos. |

O URI resultante, em seguida, é usado para estabelecer uma conexão WebSocket; novamente, observe que o certificado TLS pode não corresponder ao endereço durante a visualização, portanto, a validação talvez precise ser desabilitada.

Ao ser concluído corretamente, esse handshake falhará intencionalmente com um código de erro HTTP 410, pois nenhum soquete da Web terá sido estabelecido. Se ocorrer um erro, estas serão as opções:

| Código | Erro | Descrição |
| --- | --- | --- |
| 403 |Proibido |A URL não é válida. |
| 500 |Erro Interno |Algo deu errado no serviço. |

### <a name="listener-token-renewal"></a>Renovação de tokens do ouvinte
Quando o token do ouvinte estiver prestes a expirar, ele poderá ser substituído enviando uma mensagem de quadro de texto ao serviço por meio do canal de controle estabelecido. A mensagem contém um objeto JSON chamado "renewToken", que define a seguinte propriedade neste momento:

* **token** – um Token de Acesso válido, compartilhado com o Barramento de Serviço, em formato codificado de URL para o namespace ou Conexão Híbrida e que confere o direito **Listen** (escutar).

#### <a name="renewtoken-message"></a>Mensagem renewToken
```json
{                                                                                                                                                                        
    "renewToken" : {                                                                                                                                                      
        "token" : "SharedAccessSignature sr=http%3a%2f%2fcontoso.servicebus.windows.net%2fhyco%2f&amp;sig=XXXXXXXXXX%3d&amp;se=1471633754&amp;skn=SasKeyName"  
    }                                                                                                                                                                     
}
```

Se a validação de token falhar, o acesso será negado e serviço de nuvem fechará o soquete da Web do canal de controle com um erro; caso contrário, não haverá nenhuma resposta.

| Status WS | Descrição |
| --- | --- |
| 1008 |O token de segurança expirou e a política de autorização foi, portanto, violada. |

## <a name="sender-protocol"></a>Protocolo de remetente
O protocolo de remetente é efetivamente idêntico ao modo como um ouvinte é estabelecido.
O objetivo é o máximo de transparência para o soquete da Web de ponta a ponta. O endereço ao qual se conectar é o mesmo do ouvinte, mas a "ação" é diferente e o token precisa de uma permissão diferente:

```
wss://{namespace-address}/$hc/{path}?sb-hc-action=...&sb-hc-id=...&sbc-hc-token=...
```

O *namespace-address* é o nome de domínio totalmente qualificado do namespace de Retransmissão do Azure que hospeda a Conexão Híbrida, normalmente no formato `{myname}.servicebus.windows.net`.

A solicitação pode conter cabeçalhos HTTP adicionais arbitrários, incluindo aqueles definidos pelo aplicativo. Todos os cabeçalhos fornecidos fluem para o ouvinte e podem ser encontrados no objeto "connectHeader" da mensagem de controle "accept".

As opções de parâmetro de cadeia de consulta são conforme demonstrado a seguir

| Param | Obrigatório? | Descrição |
| --- | --- | --- |
| sb-hc-action |Sim |Para a função de remetente, o parâmetro deve ser `action=connect`. |
| {caminho} |Sim |(confira no parágrafo a seguir) |
| sb-hc-token |Sim\* |O ouvinte deve fornecer um Token de Acesso válido, compartilhado com o Barramento de Serviço, em formato codificado de URL para o namespace ou Conexão Híbrida que confere o direito **Send**. |
| sb-hc-id |Não |Uma ID opcional que possibilita o rastreamento de diagnóstico de ponta a ponta e é disponibilizada para o ouvinte durante o handshake de aceitação. |

O {caminho} é o caminho de namespace em formato codificado de URL da Conexão Híbrida pré-configurada na qual registrar este ouvinte. A expressão do caminho PODE ser estendida com um sufixo e uma expressão de cadeia de caracteres de consulta para se comunicar ainda mais. Se a Conexão Híbrida for registrada no caminho "hyco", a expressão de caminho poderá ser `hyco/suffix?param=value&...` seguida por parâmetros de cadeia de caracteres da consulta definidos aqui. Assim, uma expressão completa pode ser:

```
wss://{namespace-address}/$hc/hyco/suffix?param=value&sb-hc-action=...[&sb-hc-id=...&]sbc-hc-token=...
```

A expressão de caminho é passada por meio do ouvinte no URI do endereço contido na mensagem de controle "accept".

Se a conexão de soquete da Web falhar porque o caminho de Conexão Híbrida não está sendo registrado, porque há um token inválido ou ausente ou por algum outro erro, os comentários de erro serão fornecidos usando o modelo comum de comentários de status HTTP 1.1. A descrição do status conterá uma ID de rastreamento de erro que poderá ser comunicada ao Suporte do Azure:

| Código | Erro | Descrição |
| --- | --- | --- |
| 404 |Não encontrado |O `path` da Conexão Híbrida é inválido ou a URL base está malformada. |
| 401 |Não Autorizado |O token de segurança está ausente ou malformado ou inválido. |
| 403 |Proibido |O token de segurança não é válido para esse caminho para essa ação. |
| 500 |Erro Interno |Algo deu errado no serviço. |

Se a conexão de soquete da Web for desligada intencionalmente pelo serviço depois que ele tiver sido inicialmente configurado, o motivo para fazer isso será comunicado usando código de erro de protocolo de soquete da Web apropriado juntamente com uma mensagem de erro descritiva, que também incluirá uma ID de rastreamento.

| Status WS | Descrição |
| --- | --- |
| 1000 |O ouvinte desligou o soquete. |
| 1001 |O caminho de Conexão Híbrida foi excluído ou desabilitado. |
| 1008 |O token de segurança expirou e, portanto, a política de autorização foi violada. |
| 1011 |Algo deu errado no serviço. |

## <a name="next-steps"></a>Próximas etapas
* [Perguntas frequentes sobre retransmissão](relay-faq.md)
* [Criar um namespace](relay-create-namespace-portal.md)
* [Introdução ao .NET](relay-hybrid-connections-dotnet-get-started.md)
* [Introdução ao Node](relay-hybrid-connections-node-get-started.md)


