---
title: "Conectar-se ao banco de dados SQL ‒ SQL Server Management Studio | Microsoft Docs"
description: Saiba como se conectar a um banco de dados SQL no Azure usando o SSMS (SQL Server Management Studio). Em seguida, execute uma consulta de exemplo usando o Transact-SQL (T-SQL).
metacanonical: 
keywords: conectar-se ao banco de dados sql, sql server management studio
services: sql-database
documentationcenter: 
author: CarlRabeler
manager: jhubbard
editor: 
ms.assetid: 7cd2a114-c13c-4ace-9088-97bd9d68de12
ms.service: sql-database
ms.custom: development
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/01/2017
ms.author: sstein;carlrab
translationtype: Human Translation
ms.sourcegitcommit: 8d988aa55d053d28adcf29aeca749a7b18d56ed4
ms.openlocfilehash: a5eaf43aa01e5d30171ea038db7ba985c9684fb7
ms.lasthandoff: 02/16/2017


---
# <a name="connect-to-sql-database-with-sql-server-management-studio-and-execute-a-sample-t-sql-query"></a>Conectar-se ao Banco de Dados SQL com o SQL Server Management Studio e executar um exemplo de consulta T-SQL

Este artigo mostra como se conectar a um banco de dados SQL do Azure usando o SQL Server Management Studio (SSMS). Depois de nos conectarmos com êxito, executamos uma consulta Transact-SQL (T-SQL) simples para verificar a comunicação com o banco de dados.

[!INCLUDE [SSMS Install](../../includes/sql-server-management-studio-install.md)]

1. Se você ainda não o fez, baixe e instale a versão mais recente do SSMS em [Baixar o SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx). Para se manter atualizado, a versão mais recente do SSMS avisará você quando houver uma nova versão disponível para download.

2. Depois de instalar, digite **Microsoft SQL Server Management Studio** na caixa de pesquisa do Windows e clique em **Enter** para abrir o SSMS:

    ![SQL Server Management Studio](./media/sql-database-get-started/ssms.png)
3. Na caixa de diálogo Conectar ao servidor, insira as informações necessárias para se conectar ao SQL server usando a Autenticação do SQL Server.

    ![conectar-se ao servidor](./media/sql-database-get-started/connect-to-server.png)
4. Clique em **Conectar**.

    ![conectado ao servidor](./media/sql-database-get-started/connected-to-server.png)
5. No Pesquisador de Objetos, expanda **Bancos de dados** e qualquer bancos de dados para exibir objetos nele.

    ![novos objetos de banco de dados de exemplo com o ssms](./media/sql-database-get-started/new-sample-db-objects-ssms.png)
6. Clique com o botão direito do mouse neste banco de dados e depois clique em **Nova Consulta**.

    ![nova consulta de banco de dados de exemplo com o ssms](./media/sql-database-get-started/new-sample-db-query-ssms.png)
7. Na janela de consulta, digite o seguinte:

   ```select * from sys.objects```
   
8.  Na barra de ferramentas, clique em **Executar** para retornar uma lista de todos os objetos de sistema no banco de dados de exemplo.

    ![novos objetos de sistema de consulta de banco de dados de exemplo com o ssms](./media/sql-database-get-started/new-sample-db-query-objects-ssms.png)

> [!Tip]
> Para conferir um tutorial, consulte [Tutorial: Provisionar e acessar um banco de dados SQL do Azure usando o Portal do Azure e o SQL Server Management Studio](sql-database-get-started.md).    
>

## <a name="next-steps"></a>Próximas etapas

- Você pode usar instruções T-SQL para criar e gerenciar bancos de dados no Azure da mesma forma que faz no SQL Server. Se você já sabe como usar o T-SQL com o SQL Server, consulte [Informações de Transact-SQL de Banco de Dados SQL do Azure](sql-database-transact-sql-information.md) para obter um resumo das diferenças.
- Se não tiver experiência com o T-SQL, consulte [Tutorial: Escrevendo instruções Transact-SQL](https://msdn.microsoft.com/library/ms365303.aspx) e a [Referência do Transact-SQL (mecanismo de banco de dados)](https://msdn.microsoft.com/library/bb510741.aspx).
- Para obter uma introdução ao tutorial de autenticação do SQL Server, confira [Autenticação e autorização do SQL](sql-database-control-access-sql-authentication-get-started.md)
- Para obter uma introdução ao tutorial de autenticação do Azure Active Directory, confira [Autenticação e autorização do Azure AD](sql-database-control-access-aad-authentication-get-started.md)
- Para saber mais sobre o SSMS, consulte [Usar o SQL Server Management Studio](https://msdn.microsoft.com/library/ms174173.aspx).


