---
title: Configurar a StorSimple Virtual Array como um servidor de arquivos | Microsoft Docs
description: "Este terceiro tutorial na implantação de StorSimple Virtual Array lhe instrui a configurar um dispositivo virtual como servidor de arquivos."
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: 
ms.assetid: adc1c942-dd4e-4589-bc10-18ae3f7cbcdc
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/26/2016
ms.author: alkohli
translationtype: Human Translation
ms.sourcegitcommit: 78daa5a75b3414e2761333ea6ad91945596553c8
ms.openlocfilehash: e1863b43706ffc200bb94c4a26ae75080a6dd857


---
# <a name="deploy-storsimple-virtual-array---set-up-as-file-server"></a>Implantar o StorSimple Virtual Array –Preparar como servidor de arquivos
![](./media/storsimple-ova-deploy3-fs-setup/fileserver4.png)

## <a name="introduction"></a>Introdução
Este artigo se aplica ao Microsoft Azure StorSimple Virtual Array (também conhecido como o dispositivo virtual local StorSimple ou dispositivo virtual StorSimple) que executa a versão GA (disponibilidade geral) de março de 2016. Este artigo descreve como executar a configuração inicial, registrar o servidor de arquivos do StorSimple, concluir a configuração do dispositivo, criar compartilhamentos SMB e conectar-se a eles. Este é o último artigo da série de tutoriais de implantação necessários para implantar completamente sua matriz virtual como um servidor de arquivos ou um servidor iSCSI.

O processo de preparação e configuração pode levar aproximadamente 10 minutos para ser concluído.

## <a name="setup-prerequisites"></a>Pré-requisitos de configuração
Antes de configurar e configurar o dispositivo virtual StorSimple, certifique-se de que:

* Você provisionou um dispositivo virtual e se conectou a ele, conforme detalhado em [Provisionar um StorSimple Virtual Array no Hyper-V](storsimple-ova-deploy2-provision-hyperv.md) ou [Provisionar um StorSimple Virtual Array no VMware](storsimple-ova-deploy2-provision-vmware.md).
* Você tem a chave de registro do serviço StorSimple Manager que você criou para gerenciar dispositivos virtuais StorSimple. Para obter mais informações, veja [Etapa 2: Obter a chave de registro do serviço](storsimple-ova-deploy1-portal-prep.md#step-2-get-the-service-registration-key) para o StorSimple Virtual Array.
* Se esse for o segundo dispositivo virtual ou dispositivo virtual subsequente que você está registrando com um serviço StorSimple Manager existente, você deve ter a chave de criptografia de dados do serviço. Essa chave foi gerada quando o primeiro dispositivo foi registrado com êxito com esse serviço. Caso tenha perdido essa chave, veja [Obter a chave de criptografia de dados de serviço](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) do StorSimple Virtual Array.

## <a name="step-by-step-setup"></a>Configuração passo a passo
Use as instruções passo a passo a seguir para preparar e configurar seu dispositivo virtual StorSimple.

## <a name="step-1-complete-the-local-web-ui-setup-and-register-your-device"></a>Etapa 1: concluir a configuração de interface do usuário da Web local e registrar seu dispositivo
#### <a name="to-complete-the-setup-and-register-the-device"></a>Para concluir a configuração e registrar o dispositivo
1. Abra uma janela do navegador e conecte-se à interface do usuário da Web local. Digite:    
   
   `https://<ip-address of network interface>`
   
   Use a URL de conexão observada na etapa anterior. Você verá um erro indicando que há um problema com o certificado de segurança do site. Clique em **Continuar para essa página da Web**.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image2.png)
2. Entre na interface do usuário da Web de seu dispositivo virtual como **StorSimpleAdmin**. Digite a senha do administrador do dispositivo que você alterou na Etapa 3: iniciar o dispositivo virtual em [Provisionar um StorSimple Virtual Array no Hyper-V](storsimple-ova-deploy2-provision-hyperv.md) ou em [Provisionar um StorSimple Virtual Array no VMware](storsimple-ova-deploy2-provision-vmware.md).
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image3.png)
3. Você será levado à página **Inicial** . Esta página descreve as várias configurações necessárias para configurar e registrar o dispositivo virtual com o serviço StorSimple Manager. Observe que **Configurações de rede**, **Configurações de proxy Web** e **Configurações de hora** são opcionais. As únicas configurações obrigatórias são as **Configurações do dispositivo** e **Configurações de nuvem**.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image4.png)
4. Na página **Configurações de rede**, em **Adaptadores de rede**, DATA 0 será configurado automaticamente para você. Cada interface de rede é definida por padrão para obter um endereço IP automaticamente (DHCP). Assim, um endereço IP, a sub-rede e gateway serão atribuídos automaticamente (tanto para IPv4 quanto para IPv6).
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image5.png)
   
   Se você adicionou mais de uma interface de rede durante o provisionamento do dispositivo, você pode configurá-las aqui. Observe que você pode configurar a interface de rede apenas como IPv4 ou como IPv4 e IPv6. Não há suporte para configurações somente IPv6.
5. Os servidores DNS são necessários porque eles são usados quando o dispositivo tenta se comunicar com seus provedores de serviço de armazenamento de nuvem, ou então para resolver seu dispositivo por nome caso ele esteja configurado como um servidor de arquivos. Na página **Configurações de rede**, em **Servidores DNS**:
   
   1. Um servidor DNS primário e um secundário serão configurados automaticamente. Se você optar por configurar endereços IP estáticos, você pode especificar servidores DNS. Para alta disponibilidade, recomendamos que você configure um servidor DNS primário e um secundário.
   2. Clique em **Aplicar**. Isso aplicará e validará as configurações de rede.
6. Na página **Configurações do dispositivo** :
   
   1. Atribua um **Nome** exclusivo ao seu dispositivo. Esse nome pode ter de 1 a 15 caracteres e pode conter letras, números e hifens.
   2. Clique no ícone ![](./media/storsimple-ova-deploy3-fs-setup/image6.png) do **Servidor de arquivos** para o **Tipo** de dispositivo que você está criando. Um servidor de arquivos permitirá que você crie pastas compartilhadas.
   3. Como o dispositivo é um servidor de arquivos, você precisará ingressar o dispositivo em um domínio. Insira um **Nome de domínio**.
   4. Clique em **Aplicar**.
7. Uma caixa de diálogo aparecerá. Insira suas credenciais de domínio no formato especificado. Clique no ícone de verificação. As credenciais de domínio serão verificadas. Você verá uma mensagem de erro se as credenciais estiverem incorretas.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image7.png)
8. Clique em **Aplicar**. Isso aplicará e validará as configurações do dispositivo.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image8.png)
   
   > [!NOTE]
   > Certifique-se de que a matriz virtual esteja em sua própria OU (unidade organizacional) do Active Directory e que nenhum GPO (objeto de política de grupo) seja aplicado a ela ou herdado. A política de grupo pode instalar aplicativos como, software antivírus, no StorSimple Virtual Array. Não há suporte para a instalação de software adicional e isso pode levar a dados corrompidos. 
   > 
   > 
9. Opcionalmente, configure seu servidor proxy da Web. Embora a configuração do proxy da Web seja opcional, saiba que se você usar um proxy da Web, só poderá configurá-lo aqui.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image9.png)
   
   Na página **Proxy Web** :
   
   1. Forneça a **URL do proxy Web** neste formato: *http://&lt;endereço IP do host ou FDQN&gt;: Número de porta*. Observe que não há suporte para URLs HTTPS.
   2. Especifique a **Autenticação** como **Básica** ou **Nenhuma**.
   3. Se você estiver usando autenticação, você também precisará fornecer um **Nome de Usuário** e **Senha**.
   4. Clique em **Aplicar**. Isso validará e aplicará as configurações de proxy Web definidas.
10. Opcionalmente, defina as configurações de hora para seu dispositivo, como o fuso horário e os servidores NTP primários e secundários. Os servidores NTP são necessários, pois seu dispositivo deve sincronizar a hora para que ele possa se autenticar com seus provedores de serviço de nuvem.
    
    ![](./media/storsimple-ova-deploy3-fs-setup/image10.png)
    
    Na página **Configurações de hora** :
    
    1. Na lista suspensa, selecione o **Fuso horário** com base na localização geográfica na qual o dispositivo está sendo implantado. O fuso horário padrão para o seu dispositivo é PST. Seu dispositivo usará esse fuso horário para todas as operações agendadas.
    2. Especifique um **Servidor NTP primário** para seu dispositivo ou aceite o valor padrão de time.windows.com. Verifique se sua rede permite que o tráfego NTP passe do data center para a Internet.
    3. Opcionalmente, especifique um **Servidor NTP secundário** para o dispositivo.
    4. Clique em **Aplicar**. Isso validará e aplicará as configurações de hora definidas.
11. Defina as configurações de nuvem para seu dispositivo. Nesta etapa, você concluirá a configuração de dispositivo local e, em seguida, registrará o dispositivo com o serviço StorSimple Manager.
    
    1. Insira a **Chave de registro do serviço** que você obteve na [Etapa 2: obter a chave de registro do serviço](storsimple-ova-deploy1-portal-prep.md#step-2-get-the-service-registration-key) para a StorSimple Virtual Array.
    2. Ignore esta etapa se esse for o primeiro dispositivo registrando-se com esse serviço e vá para a próxima etapa. Se não for o primeiro dispositivo que você está registrando com esse serviço, você precisará fornecer a **Chave de criptografia de dados de serviço**. Essa chave é necessária com a chave de registro do serviço para registrar dispositivos adicionais no serviço StorSimple Manager. Para obter mais informações, consulte obter a [chave de criptografia de dados de serviço](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) em sua interface do usuário da Web local.
    3. Clique em **Registrar**. Isso reiniciará o dispositivo. Talvez seja necessário aguardar de 2 a 3 minutos até que o dispositivo seja registrado com êxito. Depois que o dispositivo for reiniciado, você será levado à página de entrada.
       
       ![](./media/storsimple-ova-deploy3-fs-setup/image13.png)
12. Retorne ao portal clássico do Azure. Na página **Dispositivos** , verifique se o dispositivo conectou com êxito o serviço pesquisando o status. O status do dispositivo deve ser **Ativo**.

![](./media/storsimple-ova-deploy3-fs-setup/image12.png)

## <a name="step-2-complete-the-required-device-setup"></a>Etapa 2: concluir a configuração obrigatória do dispositivo
Para concluir a configuração de dispositivo do seu dispositivo StorSimple, é necessário:

* Selecionar uma conta de armazenamento para associar ao seu dispositivo.
* Escolha as configurações de criptografia para os dados que são enviados para a nuvem.

Execute as etapas a seguir no [portal clássico do Azure](https://manage.windowsazure.com/) para concluir a configuração obrigatória do dispositivo.

#### <a name="to-complete-the-minimum-device-setup"></a>Para concluir a configuração mínima do dispositivo
1. Na página **Dispositivos** , selecione o dispositivo que você acabou de criar. Este dispositivo apareceria como **Ativo**. Clique na seta ao lado do nome do dispositivo e, em seguida, clique em **Início Rápido**.
2. Clique em **concluir a instalação do dispositivo** para iniciar o assistente Configurar dispositivo.
3. No assistente Configurar dispositivo, na página **Configurações Básicas** , faça o seguinte:
   
   1. Especifique uma conta de armazenamento para ser usada com seu dispositivo. Você pode selecionar uma conta de armazenamento existente nessa assinatura da lista suspensa, ou você pode especificar **Adicionar mais** para escolher uma conta de uma assinatura diferente.
   2. Defina as configurações de criptografia para todos os dados em repouso (criptografia AES) que serão enviados para a nuvem. Para criptografar seus dados, selecione a caixa de combinação para **habilitar a chave de criptografia de armazenamento em nuvem**. Insira uma chave de criptografia de armazenamento em nuvem que contenha 32 caracteres. Redigite a chave para confirmá-la. Uma chave AES de 256 bits será ser usada com a chave de criptografia definida pelo usuário.
   3. Clique no ícone de verificação ![](./media/storsimple-ova-deploy3-fs-setup/image15.png).
      
      ![](./media/storsimple-ova-deploy3-fs-setup/image16.png)

As configurações agora serão atualizadas. Depois que as configurações forem atualizadas com êxito, o botão concluir configuração de dispositivo estará esmaecido. Você será retornado para a página **Início Rápido** do dispositivo.

 ![](./media/storsimple-ova-deploy3-fs-setup/image17.png)

> [!NOTE]
> Você pode modificar todas as outras configurações do dispositivo a qualquer momento acessando a página **Configurar** .
> 
> 

## <a name="step-3-add-a-share"></a>Etapa 3: adicionar um compartilhamento
Execute as etapas a seguir no [portal clássico do Azure](https://manage.windowsazure.com/) para criar um compartilhamento.

#### <a name="to-create-a-share"></a>Para criar um compartilhamento
1. Na página **Início Rápido** do dispositivo, clique em **Adicionar um compartilhamento**. Isso inicia o assistente Adicionar um volume.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image17.png)
2. Na página **Configurações Básicas** , faça o seguinte:
   
   1. Especifique um nome exclusivo para seu compartilhamento. O nome deve ser uma cadeia de caracteres contendo entre 3 e 127 caracteres.
   2. Forneça, opcionalmente, uma descrição para o volume. A descrição ajudará a identificar os proprietários de compartilhamento.
   3. Selecione um tipo de uso para o compartilhamento. O tipo de uso pode ser **Em camadas** ou **Localmente afixado**, sendo que Em camadas é o padrão. Para cargas de trabalho que exigem garantias locais, menos latências e um melhor desempenho, selecione um compartilhamento **Fixado localmente** . Para todos os outros dados, selecione um compartilhamento **Em camadas** .
   
   Um compartilhamento fixado localmente é provisionado estaticamente e garante que os dados primários no compartilhamento permaneçam como locais para o dispositivo e não sejam divulgados na nuvem. Um compartilhamento em camadas, por outro lado, é provisionado dinamicamente. Quando você cria um volume em camadas, aproximadamente 10% do espaço é provisionado na camada local e 90% do espaço é provisionado na nuvem. Por exemplo, se você provisionar um volume de 1 TB, 100 GB residiriam no espaço local e 900 GB seriam usados na nuvem quando os dados fossem distribuídos em camadas. Isso, por sua vez, implica que se você ficar sem todo o espaço local no dispositivo, você não poderá provisionar um compartilhamento em camadas.
3. Especifique a capacidade provisionada para o seu compartilhamento. Observe que a capacidade especificada deve ser menor do que a capacidade disponível. Se estiver usando um compartilhamento em camadas, o tamanho do compartilhamento deve ser entre 500 GB e 20 TB. Para um compartilhamento fixado localmente, especifique um tamanho de compartilhamento entre 50 GB e 2 TB. Use a capacidade disponível como um guia para provisionar um compartilhamento. Se a capacidade local disponível for 0 GB, você não terá permissão para provisionar compartilhamentos em camadas ou fixados localmente.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image18.png)
4. Clique no ícone de seta ![](./media/storsimple-ova-deploy3-fs-setup/image19.png) para ir para a próxima página.
5. Na página **Configurações Adicionais** , atribua as permissões para o usuário ou o grupo que terá acesso a esse compartilhamento. Especifique o nome do usuário ou grupo de usuários no formato *<mailto:john@contoso.com>*. É recomendável que você use um grupo de usuários (em vez de um único usuário) para conceder privilégios de administrador para acessar esses compartilhamentos. Depois de atribuir as permissões aqui, você pode usar o Windows Explorer para modificar essas permissões.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image20.png)
6. Clique no ícone de verificação ![](./media/storsimple-ova-deploy3-fs-setup/image21.png). Será criado um compartilhamento com as configurações especificadas. Por padrão, monitoramento e backup estarão habilitados para o compartilhamento.

## <a name="step-4-connect-to-the-share"></a>Etapa 4: conectar-se ao compartilhamento
Agora, você precisará conectar-se ao(s) compartilhamento(s) que você criou na etapa anterior. Execute estas etapas no host do Windows Server.

#### <a name="to-connect-to-the-share"></a>Para conectar-se ao compartilhamento
1. Pressione ![](./media/storsimple-ova-deploy3-fs-setup/image22.png) + R. Na janela Executar, especifique *\\<file server name>* como o caminho, substituindo *nome do servidor de arquivos* com o nome do dispositivo que você atribuiu ao seu servidor de arquivos. Clique em **OK**.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image23.png)
2. Isso abrirá o Explorer. Agora você deverá ser capaz de ver os compartilhamentos que criou como pastas. Selecione e clique duas vezes em um compartilhamento (pasta) para exibir o conteúdo.
   
   ![](./media/storsimple-ova-deploy3-fs-setup/image24.png)
3. Agora você pode adicionar arquivos a esses compartilhamentos e fazer um backup.

![ícone de vídeo](./media/storsimple-ova-deploy3-fs-setup/video_icon.png) **Vídeo disponível**

Assista ao vídeo para ver como você pode configurar e registrar uma StorSimple Virtual Array como um servidor de arquivos.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Configure-a-StorSimple-Virtual-Array/player]
> 
> 

## <a name="next-steps"></a>Próximas etapas
Aprenda como [usar a interface do usuário da Web local para administrar sua StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).




<!--HONumber=Jan17_HO5-->


