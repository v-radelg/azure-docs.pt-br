---
title: "Enviar dados ao índice de pesquisa usando o Data Factory | Microsoft Docs"
description: "Saiba mais sobre como enviar dados por push ao Índice do Azure Search usando o Azure Data Factory."
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: monicar
ms.assetid: f8d46e1e-5c37-4408-80fb-c54be532a4ab
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/08/2017
ms.author: jingwang
translationtype: Human Translation
ms.sourcegitcommit: 55c988bf74ff0f2e519e895a735dc68f3dc99855
ms.openlocfilehash: e2deed13106db9467eef181f25a0a226034df5a2
ms.lasthandoff: 12/21/2016

---

# <a name="push-data-to-an-azure-search-index-by-using-azure-data-factory"></a>Enviar dados para um índice do Azure Search usando o Azure Data Factory
Este artigo descreve como usar a atividade de cópia para enviar dados por push de um armazenamento de dados local compatível com o serviço Data Factory ao índice do Azure Search. Armazenamentos de dados de origem com suporte listados na coluna de origem a [fontes e coletores com suporte](data-factory-data-movement-activities.md#supported-data-stores-and-formats) tabela. Este artigo se baseia no artigo de [atividades de movimentação de dados](data-factory-data-movement-activities.md) , que apresenta uma visão geral de movimentação de dados com a Atividade de Cópia e combinações de armazenamentos de dados com suporte.

## <a name="enabling-connectivity"></a>Habilitando a conectividade
Para permitir que o serviço se conecte a um armazenamento de dados local do Data Factory, instale o Gateway de Gerenciamento de Dados em seu ambiente local. Você pode instalar o gateway na mesma máquina que hospeda o banco de dados ou em uma máquina separada para evitar a concorrência por recursos com o armazenamento de dados.

O Gateway de Gerenciamento de Dados conecta fontes de dados locais a serviços de nuvem de maneira segura e gerenciada. Consulte o artigo [Mover dados entre local e nuvem](data-factory-move-data-between-onprem-and-cloud.md) para obter detalhes sobre o Gateway de Gerenciamento de Dados.

## <a name="copy-data-wizard"></a>Assistente para Copiar Dados
A maneira mais fácil de criar um pipeline que copie dados para o Azure Search de qualquer um dos armazenamentos de dados com suporte é usar o assistente para Copiar Dados. Veja o [Tutorial: Criar um pipeline usando o Assistente de Cópia](data-factory-copy-data-wizard-tutorial.md) para obter um passo a passo rápido.

O exemplo a seguir fornece as definições JSON de exemplo que você pode utilizar para criar um pipeline usando o [Portal do Azure](data-factory-copy-activity-tutorial-using-azure-portal.md), [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Eles mostram como copiar dados de um SQL Server local para um índice do Azure Search. No entanto, os dados podem ser copiados de qualquer um dos armazenamentos de dados locais indicados [aqui](data-factory-data-movement-activities.md#supported-data-stores-and-formats) usando a Atividade no Azure Data Factory.   

## <a name="sample-copy-data-from-on-premises-sql-server-to-azure-search-index"></a>Exemplo: copiar dados do SQL Server local para o índice do Azure Search

O exemplo a seguir mostra:

1.    Um serviço vinculado do tipo [AzureSearch](#azure-search-linked-service-properties).
2.    Um serviço vinculado do tipo [OnPremisesSqlServer](data-factory-sqlserver-connector.md#sql-server-linked-service-properties).
3.    Um [conjunto de dados](data-factory-create-datasets.md) de entrada do tipo [SqlServerTable](data-factory-sqlserver-connector.md#sql-server-dataset-type-properties).
4.    Um [conjunto de dados](data-factory-create-datasets.md) de saída do tipo [AzureSearchIndex](#azure-search-index-dataset-properties).
4.    Um [pipeline](data-factory-create-pipelines.md) com a atividade de cópia que usa [SqlSource](data-factory-sqlserver-connector.md#sql-server-copy-activity-type-properties) e [AzureSearchIndexSink](#azure-search-index-sink-properties).

O exemplo copia dados de série temporal de um banco de dados do SQL Server local para um índice do Azure Search por hora. As propriedades JSON usadas nesses exemplos são descritas nas seções após os exemplos.

Como uma primeira etapa, configure o gateway de gerenciamento de dados em seu computador local. As instruções estão no artigo [Mover dados entre fontes locais e a nuvem](data-factory-move-data-between-onprem-and-cloud.md) .

**Serviço vinculado ao Azure Search:**

```JSON
{
    "name": "AzureSearchLinkedService",
       "properties": {
        "type": "AzureSearch",
           "typeProperties": {
            "url": "https://<service>.search.windows.net",
            "key": "<AdminKey>"
        }
       }
}
```

**Serviço vinculado do SQL Server**

```JSON
{
  "Name": "SqlServerLinkedService",
  "properties": {
    "type": "OnPremisesSqlServer",
    "typeProperties": {
      "connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;",
      "gatewayName": "<gatewayname>"
    }
  }
}
```

**Conjunto de dados de entrada do SQL Server**

O exemplo supõe que você criou uma tabela "MyTable" no SQL Server e que ela contém uma coluna chamada "timestampcolumn" para dados de série temporal. Você pode consultar várias tabelas no mesmo banco de dados usando um único conjunto de dados, mas uma única tabela deve ser usada para a typeProperty de tableName do conjunto de dados.

Configurar "external": "true" informa ao serviço Data Factory que o conjunto de dados é externo ao Data Factory e não é produzido por uma atividade no Data Factory.

```JSON
{
  "name": "SqlServerDataset",
  "properties": {
    "type": "SqlServerTable",
    "linkedServiceName": "SqlServerLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Conjunto de dados de saída do Azure Search:**

O exemplo copia dados para um índice do Azure Search chamado **produtos**. O Data Factory não cria o índice. Para testar o exemplo, crie um índice com esse nome. Crie o índice do Azure Search com o mesmo número de colunas do que o conjunto de dados de entrada. Novas entradas são adicionadas ao índice do Azure Search a cada hora.

```JSON
{
    "name": "AzureSearchIndexDataset",
    "properties": {
        "type": "AzureSearchIndex",
        "linkedServiceName": "AzureSearchLinkedService",
         "typeProperties" : {
            "indexName": "products",
        },
        "availability": {
            "frequency": "Minute",
            "interval": 15
        }
   }
}
```

**Pipeline com a Atividade de cópia:**

O pipeline contém uma Atividade de Cópia que está configurada para usar os conjuntos de dados de entrada e saída e é agendada para ser executada a cada hora. Na definição de JSON do pipeline, o tipo de **fonte** está definido como **SqlSource** e o tipo de **coletor** está definido como **AzureSearchIndexSink**. A consulta SQL especificada para a propriedade **SqlReaderQuery** seleciona os dados na última hora a serem copiados.

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline for copy activity",
    "activities":[  
      {
        "name": "SqlServertoAzureSearchIndex",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": " SqlServerInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSearchIndexDataset"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
          },
          "sink": {
            "type": "AzureSearchIndexSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
     ]
   }
}
```

Se você estiver copiando dados de um armazenamento de dados de nuvem para o Azure Search, a propriedade `executionLocation` será necessária. Abaixo está a alteração necessária em Copiar atividade `typeProperties` como exemplo. Verifique a seção [Copiar dados entre armazenamentos de dados de nuvem](data-factory-data-movement-activities.md#global) para obter mais detalhes e valores com suporte.

```JSON
"typeProperties": {
  "source": {
    "type": "BlobSource"
  },
  "sink": {
    "type": "AzureSearchIndexSink"
  },
  "executionLocation": "West US"
}
```

## <a name="azure-search-linked-service-properties"></a>Propriedades do serviço vinculado Azure Search

A tabela a seguir fornece descrições dos elementos JSON específicos para o serviço vinculado Azure Search.

| Propriedade | Descrição | Obrigatório |
| -------- | ----------- | -------- |
| type | A propriedade type deve ser definida como: **AzureSearch**. | Sim |
| URL | URL para o serviço Azure Search. | Sim |
| chave | Chave de administração para o serviço Azure Search. | Sim |

## <a name="azure-search-index-dataset-properties"></a>Propriedades de conjunto de dados de índice do Azure Search

Para obter uma lista completa das seções e propriedades disponíveis para definir os conjuntos de dados, veja o artigo [Criando conjuntos de dados](data-factory-create-datasets.md) . As seções como structure, availability e policy de um conjunto de dados JSON são similares para todos os tipos de conjunto de dados. A seção **typeProperties** é diferente para cada tipo de conjunto de dados. A seção typeProperties para um conjunto de dados do tipo **AzureSearchIndex** tem as propriedades a seguir:

| Propriedade | Descrição | Obrigatório |
| -------- | ----------- | -------- |
| type | A propriedade type deve ser definida como: **AzureSearchIndex**.| Sim |
| indexName | Nome do índice do Azure Search. O Data Factory não cria o índice. O índice deve existir no Azure Search. | Sim |


## <a name="azure-search-index-sink-properties"></a>Propriedades do coletor de índice do Azure Search
Para obter uma lista completa das seções e propriedades disponíveis para definir as atividades, veja o artigo [Criando pipelines](data-factory-create-pipelines.md) . As propriedades, como nome, descrição, tabelas de entrada e saída, e diversas políticas, estão disponíveis para todos os tipos de atividade. As propriedades que estão disponíveis na seção typeProperties da atividade, por outro lado, variam de acordo com cada tipo de atividade. Para a Atividade de Cópia, elas variam de acordo com os tipos de fonte e coletor.

Na Atividade de Cópia, quando a fonte é do tipo **AzureSearchIndexSink**, as seguintes propriedades estão disponíveis na seção typeProperties:

| Propriedade | Descrição | Valores permitidos | Obrigatório |
| -------- | ----------- | -------------- | -------- |
| WriteBehavior | Especifica se deve mesclar ou substituir quando já existe um documento no índice. Veja a [propriedade WriteBehavior](#writebehavior-property).| Merge (padrão)<br/>Carregar| Não |
| WriteBatchSize | Carrega dados para o índice do Azure Search quando o tamanho do buffer atinge writeBatchSize. Veja a [propriedade WriteBatchSize](#writebatchsize-property) para obter detalhes. | 1 a 1.000. O valor padrão é 1000. | Não |

### <a name="writebehavior-property"></a>Propriedade WriteBehavior
Upsert do AzureSearchSink ao gravar dados. Em outras palavras, ao escrever um documento, se a chave do documento já existe no índice do Azure Search, o Azure Search atualiza o documento existente, em vez de lançar uma exceção de conflito.

O AzureSearchSink fornece estes dois comportamentos de upsert (usando o SDK da AzureSearch):

- **Mesclar**: combine todas as colunas no novo documento com a existente. Para colunas com valor nulo no novo documento, o valor existente é preservado.
- **Carregar**: o novo documento substituirá o existente. Para colunas não especificadas no novo documento, o valor será definido como nulo se houver um valor não nulo no documento existente ou não.

O comportamento padrão é **Mesclar**.

### <a name="writebatchsize-property"></a>Propriedade WriteBatchSize
O serviço de Azure Search oferece suporte a escrever documentos como um lote. Um lote pode conter de 1 a 1.000 ações. Uma ação manipula um documento para executar a operação de carregamento/mesclagem.

### <a name="data-type-support"></a>Suporte ao tipo de dados
A tabela a seguir especifica se um tipo de dados do Azure Search tem suporte ou não.

| Tipo de dados do Azure Search | Suporte no coletor do Azure Search |
| ---------------------- | ------------------------------ |
| Cadeia de caracteres | S |
| Int32 | S |
| Int64 | S |
| Duplo | S |
| Booliano | S |
| DataTimeOffset | S |
| Matriz de cadeia de caracteres | N |
| GeographyPoint | N |

## <a name="copy-from-a-cloud-source"></a>Copiar de uma origem de nuvem
Se você estiver copiando dados de um armazenamento de dados de nuvem para o Azure Search, a propriedade `executionLocation` será necessária. Abaixo está a alteração necessária em Copiar atividade `typeProperties` como exemplo. Verifique a seção [Copiar dados entre armazenamentos de dados de nuvem](data-factory-data-movement-activities.md#global) para obter mais detalhes e valores com suporte.

```JSON
"typeProperties": {
  "source": {
    "type": "BlobSource"
  },
  "sink": {
    "type": "AzureSearchIndexSink"
  },
  "executionLocation": "West US"
}
```

## <a name="specifying-structure-definition-for-rectangular-datasets"></a>Especifica a definição de estrutura para conjuntos de dados retangulares
A seção de estrutura em conjuntos de dados JSON é uma seção **opvional** para tabelas retangulares (com linhas e colunas) e contém uma coleção de colunas para a tabela. Use a seção de estrutura tanto para fornecer informações de tipo para conversões de tipo quanto para fazer mapeamentos de coluna. As seções a seguir descrevem esses recursos detalhadamente.

Cada coluna contém as seguintes propriedades:

| Propriedade | Descrição | Obrigatório |
| -------- | ----------- | -------- |
| name | Nome da coluna. | Sim |
| type | Tipo de dados da coluna. Para saber mais sobre quando deve você especificar informações de tipo, veja a seção [conversões de tipo](#type-conversion-sample). | Não |
| culture | Cultura baseada em .NET a ser usada quando o tipo é especificado e é o tipo .NET `Datetime` ou `Datetimeoffset`. O padrão é `en-us`.  | Não |
| formato | O formato de cadeia de caracteres a ser usado quando o tipo é especificado e é o tipo .NET `Datetime` ou `Datetimeoffset`. | Não |

O exemplo a seguir mostra a seção da estrutura JSON para uma tabela que tem três colunas `userid`, `name` e `lastlogindate`.

```JSON
"structure":
[
    { "name": "userid"},
    { "name": "name"},
    { "name": "lastlogindate"}
],
```
Use as diretrizes a seguir referentes a quando incluir informações de "estrutura" e o que incluir na seção **estrutura**.

- **Para fontes de dados estruturados** que armazenam as informações sobre o esquema e o tipo junto com os dados em si (por exemplo: SQL Server, Oracle, tabela do Azure etc.): especifique a seção "estrutura" apenas se quiser mapear colunas de origem específicas para colunas específicas no coletor e seus nomes não são iguais. Consulte detalhes na seção de mapeamento de coluna.

    Como mencionado acima, as informações de tipo são opcionais na seção "estrutura". Para fontes estruturadas, as informações de tipo estão disponíveis como parte da definição de conjunto de dados no repositório de dados, portanto, você não deve incluir informações de tipo quando você incluir a seção "estrutura".
- **Para esquema de fontes de dados de leitura (especificamente o blob do Azure)** , você pode optar por armazenar os dados sem armazenar nenhuma informação de tipo ou esquema juntamente com esses dados. Para esses tipos de fontes de dados, você deve incluir "estrutura" nos dois casos a seguir:
    - Mapeie colunas de origem para colunas de coletor.
    - Quando o conjunto de dados é uma fonte em uma atividade de cópia, você pode fornecer informações de tipo em "estrutura". O Data Factory usa essas informações de tipo para conversão em tipos nativos para o coletor. Para saber mais, veja [Mover dados de e para o Blob do Azure](data-factory-azure-blob-connector.md).

### <a name="supported-net-based-types"></a>Tipos baseados em .NET para os quais há suporte
O Data Factory dá suporte aos valores de tipo baseados em .NET compatíveis com CLS a seguir, para fornecer informações de tipo em "estrutura" para esquema de fontes de dados de leitura, como blob do Azure.

- Int16
- Int32
- Int64
- Single
- Duplo
- Decimal
- Bool
- string
- Datetime
- Datetimeoffset
- Timespan

Para Datetime e Datetimeoffset, você também pode especificar as cadeias de caracteres para "cultura" e "formato" para facilitar a análise de sua cadeia de caracteres Datetime personalizada. Consulte o exemplo de conversão de tipo na seção a seguir:


[!INCLUDE [data-factory-type-conversion-sample](../../includes/data-factory-type-conversion-sample.md)]

[!INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]


## <a name="performance-and-tuning"></a>Desempenho e ajuste  
Veja o [Guia de desempenho e ajuste da Atividade de Cópia](data-factory-copy-activity-performance.md) para saber mais sobre os principais fatores que afetam o desempenho e a movimentação dos dados (Atividade de Cópia), além de várias maneiras de otimizar.

## <a name="next-steps"></a>Próximas etapas
Confira os seguintes artigos:

* [Tutorial de atividade de cópia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obter instruções passo a passo para criação de um pipeline com uma Atividade de cópia.

