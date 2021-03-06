---
title: Conectar-se ao SQL Data Warehouse do Azure - VSTS | Microsoft Docs
description: Consultar o SQL Data Warehouse com o Visual Studio.
services: sql-data-warehouse
documentationcenter: NA
author: barbkess
manager: jhubbard
editor: 
ms.assetid: daace889-95e5-4826-b2fc-047eac9d6d95
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: get-started-article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 10/31/2016
ms.author: barbkess
translationtype: Human Translation
ms.sourcegitcommit: 77474214c6fafe7f591030d30f6a46c66fbc5c09
ms.openlocfilehash: 71a56d0e99308d3f7f514283792a2155a05a7172


---
# <a name="connect-to-sql-data-warehouse-with-visual-studio-and-ssdt"></a>Conectar-se ao SQL Data Warehouse com o Visual Studio e o SSDT
> [!div class="op_single_selector"]
> * [Power BI](sql-data-warehouse-get-started-visualize-with-power-bi.md)
> * [Aprendizado de Máquina do Azure](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md)
> * [Visual Studio](sql-data-warehouse-query-visual-studio.md)
> * [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md) 
> * [SSMS](sql-data-warehouse-query-ssms.md)
> 
> 

Use o Visual Studio para consultar o SQL Data Warehouse do Azure em apenas alguns minutos. Esse método usa a extensão SSDT (SQL Server Data Tools) no Visual Studio. 

## <a name="prerequisites"></a>Pré-requisitos
Para usar este tutorial, você precisará de:

* Um SQL Data Warehouse existente. Para criar um, confira [Criar um SQL Data Warehouse][Criar um SQL Data Warehouse].
* SSDT para Visual Studio. Se você tiver o Visual Studio, provavelmente já terá isso. Para obter instruções e opções de instalação, consulte [Instalar o Visual Studio e o SSDT][Instalar o Visual Studio e o SSDT].
* O nome de servidor SQL totalmente qualificado. Para encontrar isso, confira [Conectar-se ao SQL Data Warehouse][Conectar-se ao SQL Data Warehouse].

## <a name="1-connect-to-your-sql-data-warehouse"></a>1. Conectar-se ao SQL Data Warehouse
1. Abra o Visual Studio 2013 ou 2015.
2. Abra o Pesquisador de Objetos do SQL Server. Para fazer isso, selecione **Exibir** > **Pesquisador de Objetos do SQL Server**.
   
    ![Pesquisador de Objetos do SQL Server][1]
3. Clique no ícone **Adicionar SQL Server** .
   
    ![Adicionar SQL Server][2]
4. Preencha os campos na janela Conectar ao Servidor.
   
    ![Conectar-se ao servidor][3]
   
   * **Nome do servidor**. Insira o **nome do servidor** identificado anteriormente.
   * **Autenticação**. Selecione **Autenticação do SQL Server** ou **Autenticação Integrada do Active Directory**.
   * **Nome de Usuário** e **Senha**. Insira o nome de usuário e senha se a Autenticação do SQL Server foi selecionada acima.
   * Clique em **Conectar**.
5. Para explorar, expanda seu servidor do SQL Azure. Você pode exibir os bancos de dados associados ao servidor. Expanda o AdventureWorksDW para ver as tabelas no banco de dados de exemplo.
   
    ![Explorar o AdventureWorksWeb][4]

## <a name="2-run-a-sample-query"></a>2. Executar uma consulta de exemplo
Agora que uma conexão foi estabelecida com o banco de dados, escreveremos uma consulta.

1. Clique com o botão direito do mouse em seu banco de dados no Gerenciador de Objetos do SQL Server.
2. Selecione **Nova Consulta**. Uma nova janela de consulta é aberta.
   
    ![Nova Consulta][5]
3. Copie esta consulta TSQL na janela de consulta:
   
    ```sql
    SELECT COUNT(*) FROM dbo.FactInternetSales;
    ```
4. Execute a consulta. Para fazer isso, clique na seta verde ou use este atalho: `CTRL`+`SHIFT`+`E`.
   
    ![Executar consulta][6]
5. Examine os resultados da consulta. Neste exemplo, a tabela FactInternetSales tem 60398 linhas.
   
    ![Resultados da consulta][7]

## <a name="next-steps"></a>Próximas etapas
Agora que você pode se conectar e consultar, tente [visualizar os dados com o PowerBI][visualizar os dados com o PowerBI].

Para configurar seu ambiente para a autenticação do Azure Active Directory, confira [Autenticar no SQL Data Warehouse][Autenticar no SQL Data Warehouse].

<!--Arcticles-->
[Conectar-se ao SQL Data Warehouse]: sql-data-warehouse-connect-overview.md
[Criar um SQL Data Warehouse]: sql-data-warehouse-get-started-provision.md
[Instalar o Visual Studio e o SSDT]: sql-data-warehouse-install-visual-studio.md
[Autenticar no SQL Data Warehouse]: sql-data-warehouse-authentication.md
[visualizar os dados com o PowerBI]: sql-data-warehouse-get-started-visualize-with-power-bi.md  

<!--Other-->
[portal do Azure]: https://portal.azure.com

<!--Image references-->

[1]: media/sql-data-warehouse-query-visual-studio/open-ssdt.png
[2]: media/sql-data-warehouse-query-visual-studio/add-server.png
[3]: media/sql-data-warehouse-query-visual-studio/connection-dialog.png
[4]: media/sql-data-warehouse-query-visual-studio/explore-sample.png
[5]: media/sql-data-warehouse-query-visual-studio/new-query2.png
[6]: media/sql-data-warehouse-query-visual-studio/run-query.png
[7]: media/sql-data-warehouse-query-visual-studio/query-results.png



<!--HONumber=Nov16_HO3-->


