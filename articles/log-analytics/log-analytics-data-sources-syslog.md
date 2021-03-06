---
title: Coletar e analisar mensagens do Syslog no Log Analytics do OMS | Microsoft Docs
description: "O Syslog é um protocolo de registro de eventos em log que é comum para o Linux.   Este artigo descreve como configurar a coleta de mensagens do Syslog no Log Analytics e os detalhes dos registros que eles criam no repositório do OMS."
services: log-analytics
documentationcenter: 
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: f1d5bde4-6b86-4b8e-b5c1-3ecbaba76198
ms.service: log-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/23/2017
ms.author: bwren
translationtype: Human Translation
ms.sourcegitcommit: 653696779e612726ed5b75829a5c6ed2615553d7
ms.openlocfilehash: 6e92a79c0b7ea35f110c779922255d6ddc93ed7c
ms.lasthandoff: 01/24/2017


---
# <a name="syslog-data-sources-in-log-analytics"></a>Fontes de dados de Syslog no Log Analytics
O Syslog é um protocolo de registro de eventos em log que é comum para o Linux.  Os aplicativos enviarão mensagens que podem ser armazenadas no computador local ou entregues a um coletor de Syslog.  Quando o agente do OMS para Linux é instalado, ele configura o daemon do Syslog local para encaminhar mensagens para o agente.  O agente então envia a mensagem para o Log Analytics, no qual um registro correspondente é criado no repositório OMS.  

> [!NOTE]
> O Log Analytics dá suporte à coleta de mensagens enviadas por rsyslog ou syslog-ng. O daemon syslog padrão na versão 5 do Red Hat Enterprise Linux, CentOS e na versão Oracle Linux (sysklog) não tem suporte para a coleta de eventos de syslog. Para coletar dados de syslog nessa versão das distribuições, o [daemon rsyslog](http://rsyslog.com) deverá ser instalado e configurado para substituir sysklog.
> 
> 

![Coleção do Syslog](media/log-analytics-data-sources-syslog/overview.png)

## <a name="configuring-syslog"></a>Configurando Syslog
O Agente do OMS para Linux coletará apenas os eventos com os recursos e severidades especificadas em sua configuração.  Você pode configurar o Syslog por meio do portal do OMS ou gerenciando arquivos de configuração em seus agentes do Linux.

### <a name="configure-syslog-in-the-oms-portal"></a>Configurar o Syslog no portal do OMS
Configure o Syslog do menu [Dados nas Configurações do Log Analytics](log-analytics-data-sources.md#configuring-data-sources).  Essa configuração é entregue ao arquivo de configuração em cada agente do Linux.

Você pode adicionar um novo recurso, digitando seu nome e clicando em **+**.  Para cada recurso, somente mensagens com as severidades selecionadas serão coletados.  Marque as severidades para o recurso específico que você deseja coletar.  Você não pode fornecer quaisquer critérios adicionais para filtrar mensagens.

![Configurar Syslog](media/log-analytics-data-sources-syslog/configure.png)

Por padrão, todas as alterações de configuração são automaticamente envidas por push para todos os agentes.  Se você quiser configurar o Syslog manualmente em cada agente do Linux, desmarque a caixa *Aplicar as configurações abaixo aos computadores Linux*.

### <a name="configure-syslog-on-linux-agent"></a>Configurar o Syslog no agente do Linux
Quando o [agente do OMS é instalado em um cliente Linux](log-analytics-linux-agents.md), ele instala um arquivo de configuração de Syslog padrão, que define o recurso e a severidade das mensagens que são coletadas.  Você pode modificar esse arquivo para alterar a configuração.  O arquivo de configuração é diferente, dependendo do daemon do Syslog que o cliente tem instalado.

> [!NOTE]
> Se você editar a configuração de syslog, deverá reiniciar o daemon syslog para que as alterações entrem em vigor.
> 
> 

#### <a name="rsyslog"></a>rsyslog
O arquivo de configuração para rsyslog está localizado em **/etc/rsyslog.d/95-omsagent.conf**.  Seu conteúdo padrão é mostrado abaixo.  Isso coleta mensagens do syslog enviadas do agente local para todos os recursos com um nível de aviso ou superior.

    kern.warning       @127.0.0.1:25224
    user.warning       @127.0.0.1:25224
    daemon.warning     @127.0.0.1:25224
    auth.warning       @127.0.0.1:25224
    syslog.warning     @127.0.0.1:25224
    uucp.warning       @127.0.0.1:25224
    authpriv.warning   @127.0.0.1:25224
    ftp.warning        @127.0.0.1:25224
    cron.warning       @127.0.0.1:25224
    local0.warning     @127.0.0.1:25224
    local1.warning     @127.0.0.1:25224
    local2.warning     @127.0.0.1:25224
    local3.warning     @127.0.0.1:25224
    local4.warning     @127.0.0.1:25224
    local5.warning     @127.0.0.1:25224
    local6.warning     @127.0.0.1:25224
    local7.warning     @127.0.0.1:25224

Você pode remover um recurso removendo sua seção do arquivo de configuração.  Você pode limitar as severidades coletadas para um recurso específico, modificando a entrada desse recurso.  Por exemplo, para limitar o recurso do usuário a mensagens com uma severidade de erro ou superior, você modificaria essa linha do arquivo de configuração para o seguinte:

    user.error    @127.0.0.1:25224


#### <a name="syslog-ng"></a>syslog-ng
O arquivo de configuração para syslog-ng é a localização em **/etc/syslog-ng/syslog-ng.conf**.  Seu conteúdo padrão é mostrado abaixo.  Isso coleta mensagens do syslog enviadas do agente local para todos os recursos e todas as severidades.   

    #
    # Warnings (except iptables) in one file:
    #
    destination warn { file("/var/log/warn" fsync(yes)); };
    log { source(src); filter(f_warn); destination(warn); };

    #OMS_Destination
    destination d_oms { udp("127.0.0.1" port(25224)); };

    #OMS_facility = auth
    filter f_auth_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(auth); };
    log { source(src); filter(f_auth_oms); destination(d_oms); };

    #OMS_facility = authpriv
    filter f_authpriv_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(authpriv); };
    log { source(src); filter(f_authpriv_oms); destination(d_oms); };

    #OMS_facility = cron
    filter f_cron_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(cron); };
    log { source(src); filter(f_cron_oms); destination(d_oms); };

    #OMS_facility = daemon
    filter f_daemon_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(daemon); };
    log { source(src); filter(f_daemon_oms); destination(d_oms); };

    #OMS_facility = kern
    filter f_kern_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(kern); };
    log { source(src); filter(f_kern_oms); destination(d_oms); };

    #OMS_facility = local0
    filter f_local0_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(local0); };
    log { source(src); filter(f_local0_oms); destination(d_oms); };

    #OMS_facility = local1
    filter f_local1_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(local1); };
    log { source(src); filter(f_local1_oms); destination(d_oms); };

    #OMS_facility = mail
    filter f_mail_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(mail); };
    log { source(src); filter(f_mail_oms); destination(d_oms); };

    #OMS_facility = syslog
    filter f_syslog_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(syslog); };
    log { source(src); filter(f_syslog_oms); destination(d_oms); };

    #OMS_facility = user
    filter f_user_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(user); };
    log { source(src); filter(f_user_oms); destination(d_oms); };

Você pode remover um recurso removendo sua seção do arquivo de configuração.  Você pode limitar as severidades coletadas para um recurso específico, removendo-as de sua lista.  Por exemplo, para limitar o recurso exclusivamente a mensagens críticas e de alerta, você modificaria essa seção do arquivo de configuração para o seguinte:

    #OMS_facility = user
    filter f_user_oms { level(alert,crit) and facility(user); };
    log { source(src); filter(f_user_oms); destination(d_oms); };


### <a name="changing-the-syslog-port"></a>Alterando a porta de Syslog
O agente do OMS escuta as mensagens do Syslog no cliente local, na porta 25224.  Você pode alterar essa porta adicionando a seção a seguir ao arquivo de configuração do agente do OMS localizado em **/etc/opt/microsoft/omsagent/conf/omsagent.conf**.  Substitua 25224, na entrada **porta** , pelo número da porta que você deseja.  Observe que você também precisará modificar o arquivo de configuração para que o daemon do Syslog envie mensagens para essa porta.

    <source>
      type syslog
      port 25224
      bind 127.0.0.1
      protocol_type udp
      tag oms.syslog
    </source>


## <a name="data-collection"></a>Coleta de dados
O agente do OMS escuta as mensagens do Syslog no cliente local, na porta 25224. O arquivo de configuração para o daemon do Syslog encaminha mensagens do Syslog enviadas do aplicativo para essa porta, na qual elas são coletadas pelo Log Analytics.

## <a name="syslog-record-properties"></a>Propriedades de registro do syslog
Os registros do syslog têm um tipo de **Syslog** e têm as propriedades na tabela a seguir.

| Propriedade | Descrição |
|:--- |:--- |
| Computador |Computador do qual o evento foi coletado. |
| Recurso |Define a parte do sistema que gerou a mensagem. |
| HostIP |Endereço IP do sistema que envia a mensagem. |
| HostName |Nome do sistema enviando a mensagem. |
| SeverityLevel |Nível de severidade do evento. |
| SyslogMessage |Texto da mensagem. |
| ProcessID |A ID do processo que gerou a mensagem. |
| EventTime |Data e hora em que o alerta foi gerado. |

## <a name="log-queries-with-syslog-records"></a>Consultas do log com registros do Syslog
A tabela a seguir fornece diferentes exemplos de consultas de log que recuperam registros do Syslog.

| Consultar | Descrição |
|:--- |:--- |
| Type=Syslog |Todos os Syslogs. |
| Type=Syslog SeverityLevel=error |Todos os registros do Syslog com a severidade de erro. |
| Type=Syslog &#124; measure count() by Computer |Contagem de registros do Syslog por computador. |
| Type=Syslog &#124; measure count() by Facility |Contagem de registros do Syslog por recurso. |

## <a name="next-steps"></a>Próximas etapas
* Saiba mais sobre [pesquisas de log](log-analytics-log-searches.md) para analisar os dados coletados de fontes de dados e soluções. 
* Use [campos personalizados](log-analytics-custom-fields.md) para analisar dados dos registros do syslog em campos individuais.
* [Configure Agentes do Linux](log-analytics-linux-agents.md) para coletar outros tipos de dados. 


