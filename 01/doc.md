# Módulo 1 - Explore as opções de computação e armazenamento para cargas de trabalho de engenharia de dados

Este módulo ensina maneiras de estruturar o data lake e otimizar os arquivos para exploração, streaming e cargas de trabalho em lote. O aluno aprenderá a organizar o data lake em níveis de refinamento de dados à medida que transforma arquivos por meio de processamento em lote e fluxo. Em seguida, eles aprenderão a criar índices em seus conjuntos de dados, como arquivos CSV, JSON e Parquet, e usá-los para uma possível consulta e aceleração de carga de trabalho.

Neste módulo, o aluno será capaz de:

- Combine streaming e processamento em lote com um único pipeline
- Organize o data lake em níveis de transformação de arquivos
- Indexar armazenamento de data lake para aceleração de consulta e carga de trabalho

## Laboratório 1 - Arquitetura do Delta Lake

Neste laboratório, você usará um espaço de trabalho do Azure Databricks e executará o streaming estruturado com trabalhos em lote usando o Delta Lake. Você precisa concluir os exercícios em um Databricks Notebook. Para começar, você precisa ter acesso a um espaço de trabalho do Azure Databricks. Se você não tiver um espaço de trabalho disponível, siga as instruções abaixo. Caso contrário, você pode pular para o final da página para [Clone the Databricks archive](#clone-the-databricks-archive).

### Antes do laboratório prático

> **Observação:** conclua apenas as etapas "Antes do laboratório prático" se você **não** estiver usando um ambiente de laboratório hospedado e estiver usando sua própria assinatura do Azure. Caso contrário, pule para o Exercício 1.

Antes de percorrer os exercícios deste laboratório, verifique se você tem acesso a um espaço de trabalho do Azure Databricks com um cluster disponível. Execute as tarefas abaixo para configurar o espaço de trabalho.

#### Tarefa 1: Criar e configurar o espaço de trabalho Azure Databricks

**Se você não estiver usando um ambiente de laboratório hospedado**, siga as [lab 01 setup instructions](https://github.com/solliancenet/microsoft-data-engineering-ilt-deploy/blob/main/setup/01/lab-01-setup.md) para criar e configurar manualmente a área de trabalho.

### Exercício 1: Preencha o caderno de laboratório

#### Tarefa 1: Clone o arquivo Databricks

1. Se você não tiver o espaço de trabalho do Azure Databricks aberto no momento: no portal do Azure, navegue até o espaço de trabalho do Azure Databricks implantado e selecione **Launch Workspace**.
2. No painel esquerdo, selecione **Workspace** > **Users** e selecione seu nome de usuário (a entrada com o ícone de casa).
3. No painel exibido, selecione a seta ao lado do seu nome e selecione **Importar**.

![A opção de menu para importar o arquivo](imagens/import-archive.png)

1. Na caixa de diálogo **Import Notebooks**, selecione o URL e cole o seguinte URL:

 ```
  https://github.com/solliancenet/microsoft-learning-paths-databricks-notebooks/blob/master/data-engineering/DBC/11-Delta-Lake-Architecture.dbc?raw=true
 ```
2. Select **Import**.
3. Select the **11-Delta-Lake-Architecture** folder that appears.

#### Task 2: Complete the following notebook

Open the **1-Delta-Architecture** notebook. Make sure you attach your cluster to the notebook before following the instructions and running the cells within.

Within the notebook, you will explore combining streaming and batch processing with a single pipeline.

> After you've completed the notebook, return to this screen, and continue to the next lab.

## Lab 2 - Working with Apache Spark in Synapse Analytics

This lab demonstrates the experience of working with Apache Spark in Azure Synapse Analytics. You will learn how to connect an Azure Synapse Analytics workspace to an Azure Data Explorer workspace using a Linked Service and then load data from one of its databases using a Spark notebook. You will also learn how to use libraries like Hyperspace and MSSparkUtil to optimize the experience of working with Data Lake storage accounts from Spark notebooks. In addition to Data Explorer and Data Lake storage, the data enrichment process will also use historical data from a SQL Pool. In the end, you will learn how to publish the enriched data back into the Data Lake and consume it with the SQL Built-in Pool and Power BI.

After completing the lab, you will understand the main steps of an end-to-end data enrichment process that uses Spark in an Azure Synapse Analytics workspace.

### Before the hands-on lab

> **Note:** Only complete the `Before the hands-on lab` steps if you are **not** using a hosted lab environment, and are instead using your own Azure subscription. Otherwise, skip ahead to Exercise 1.

Before stepping through the exercises in this lab, make sure you have properly configured your Azure Synapse Analytics workspace. Perform the tasks below to configure the workspace.

#### Task 1: Create and configure the Azure Synapse Analytics workspace

>**NOTE**
>
>If you have already created and configured the Synapse Analytics workspace while running one of the other labs available in this repo, you must not perform this task again and you can move on to the next task. The labs are designed to share the Synapse Analytics workspace, so you only need to create it once.

**If you are not using a hosted lab environment**, follow the instructions in [Deploy your Azure Synapse Analytics workspace](https://github.com/solliancenet/microsoft-data-engineering-ilt-deploy/blob/main/setup/01/asa-workspace-deploy.md) to create and configure the workspace.

#### Task 2: Create and configure additional resources for this lab

**If you are not using a hosted lab environment**, follow the instructions in [Deploy resources for Lab 02](https://github.com/solliancenet/microsoft-data-engineering-ilt-deploy/blob/main/setup/01/lab-02-deploy.md) to deploy additional resources for this lab. Once deployment is complete, you are ready to proceed with the exercises in this lab.

### Exercise 1: Load and data with Spark

#### Task 1: Index the Data Lake storage with Hyperspace

When loading data from Azure Data Lake Gen 2, searching in the data is one of the most resource consuming operations. [Hyperspace](https://github.com/microsoft/hyperspace) introduces the ability for Apache Spark users to create indexes on their datasets, such as CSV, JSON, and Parquet, and use them for potential query and workload acceleration.

Hyperspace lets you create indexes on records scanned from persisted data files. After they're successfully created, an entry that corresponds to the index is added to the Hyperspace's metadata. This metadata is later used by Apache Spark's optimizer during query processing to find and use proper indexes. If the underlying data changes, you can refresh an existing index to capture that.

Also, Hyperspace allows users to compare their original plan versus the updated index-dependent plan before running their query.

1. Open Synapse Studio (<https://web.azuresynapse.net/>).

2. Select the **Develop** hub.

    ![The develop hub is highlighted.](imagens/develop-hub.png "Develop hub")

3. Select **+**, then **Notebook** to create a new Synapse notebook.

    ![The new notebook menu item is highlighted.](imagens/new-notebook.png "New Notebook")

4. Enter **Hyperspace** for the notebook name **(1)**, then select the **Properties** button above **(2)** to hide the properties pane.

    ![The notebook properties are displayed.](imagens/notebook-properties.png "Properties")

5. Attach the notebook to the Spark cluster and make sure that the language is set to **PySpark (Python)**.

    ![The cluster is selected and the language is set.](imagens/notebook-attach-cluster.png "Attach cluster")

6. Add the following code to a new cell in your notebook:

    ```python
    from hyperspace import *  
    from com.microsoft.hyperspace import *
    from com.microsoft.hyperspace.index import *

    # Disable BroadcastHashJoin, so Spark will use standard SortMergeJoin. Currently, Hyperspace indexes utilize SortMergeJoin to speed up query.
    spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

    # Replace the value below with the name of your primary ADLS Gen2 account for your Synapse workspace
    datalake = 'REPLACE_WITH_YOUR_DATALAKE_NAME'

    dfSales = spark.read.parquet("abfss://wwi-02@" + datalake + ".dfs.core.windows.net/sale-small/Year=2019/Quarter=Q4/Month=12/*/*.parquet")
    dfSales.show(10)

    dfCustomers = spark.read.load("abfss://wwi-02@" + datalake + ".dfs.core.windows.net/data-generators/generator-customer-clean.csv", format="csv", header=True)
    dfCustomers.show(10)

    # Create an instance of Hyperspace
    hyperspace = Hyperspace(spark)
    ```

    Replace the `REPLACE_WITH_YOUR_DATALAKE_NAME` value with the name of your primary ADLS Gen2 account for your Synapse workspace. To find this, do the following:

    1. Navigate to the **Data** hub.

        ![The data hub is highlighted.](imagens/data-hub.png "Data hub")

    2. Select the **Linked** tab **(1)**, expand the Azure Data Lake Storage Gen2 group, then make note of the primary ADLS Gen2 name **(2)** next to the name of the workspace.

        ![The primary ADLS Gen2 name is displayed.](imagens/adlsgen2-name.png "ADLS Gen2 name")

7. Run the new cell. It will load the two DataFrames with data from the data lake and initialize Hyperspace.

    ![Load data from the data lake and initialize Hyperspace](imagens/lab-02-ex-02-task-02-initialize-hyperspace.png "Initialize Hyperspace")

    > **Note**: You may select the Run button to the left of the cell, or enter `Shift+Enter` to execute the cell and create a new cell below.
    >
    > The first time you execute a cell in the notebook will take a few minutes since it must start a new Spark cluster. Each subsequent cell execution should be must faster.

8. Add a new code cell to your notebook with the following code:

    ```python
    #create indexes: each one contains a name, a set of indexed columns and a set of included columns
    indexConfigSales = IndexConfig("indexSALES", ["CustomerId"], ["TotalAmount"])
    indexConfigCustomers = IndexConfig("indexCUSTOMERS", ["CustomerId"], ["FullName"])

    hyperspace.createIndex(dfSales, indexConfigSales)			# only create index once
    hyperspace.createIndex(dfCustomers, indexConfigCustomers)	# only create index once
    hyperspace.indexes().show()
    ```

9. Run the new cell. It will create two indexes and display their structure.

    ![Create new indexes and display their structure](imagens/lab-02-ex-02-task-02-create-indexes.png "New indexes")

10. Add another new code cell to your notebook with the following code:

    ```python
    df1 = dfSales.filter("""CustomerId = 200""").select("""TotalAmount""")
    df1.show()
    df1.explain(True)
    ```

11. Run the new cell. The output will show that the physical execution plan is not taking into account any of the indexes (performs a file scan on the original data file).

    ![Hyperspace explained - no indexes used](imagens/lab-02-ex-02-task-02-explain-hyperspace-01.png)

12. Now add another new cell to your notebook with the following code (notice the extra line at the beginning used to enable Hyperspace optimization in the Spark engine):

    ```python
    # Enable Hyperspace - Hyperspace optimization rules become visible to the Spark optimizer and exploit existing Hyperspace indexes to optimize user queries
    Hyperspace.enable(spark)
    df1 = dfSales.filter("""CustomerId = 200""").select("""TotalAmount""")
    df1.show()
    df1.explain(True)
    ```

13. Run the new cell. The output will show that the physical execution plan is now using the index instead of the original data file.

    ![Hyperspace explained - using an index](imagens/lab-02-ex-02-task-02-explain-hyperspace-02.png)

14. Hyperspace provides an Explain API that allows you to compare the execution plans without indexes vs. with indexes. Add a new cell with the following code:

    ```python
    df1 = dfSales.filter("""CustomerId = 200""").select("""TotalAmount""")

    spark.conf.set("spark.hyperspace.explain.displayMode", "html")
    hyperspace.explain(df1, True, displayHTML)
    ```

15. Run the new cell. The output shows a comparison `Plan with indexes` vs. `Plan without indexes`. Observe how, in the first case the index file is used while in the second case the original data file is used.

    ![Hyperspace explained - plan comparison](imagens/lab-02-ex-02-task-02-explain-hyperspace-03.png)

16. Let's investigate now a more complex case, involving a join operation. Add a new cell with the following code:

    ```python
    eqJoin = dfSales.join(dfCustomers, dfSales.CustomerId == dfCustomers.CustomerId).select(dfSales.TotalAmount, dfCustomers.FullName)

    hyperspace.explain(eqJoin, True, displayHTML)
    ```

17. Run the new cell. The output shows again a comparison `Plan with indexes` vs. `Plan without indexes`, where indexes are used in the first case and the original data files in the second.

    ![Hyperspace explained - plan comparison for join](imagens/lab-02-ex-02-task-02-explain-hyperspace-04.png)

    In case you want to deactivate Hyperspace and cleanup the indexes, you can run the following code:

    ```python
    # Disable Hyperspace - Hyperspace rules no longer apply during query optimization. Disabling Hyperspace has no impact on created indexes because they remain intact
    Hyperspace.disable(spark)

    hyperspace.deleteIndex("indexSALES")
    hyperspace.vacuumIndex("indexSALES")
    hyperspace.deleteIndex("indexCUSTOMERS")
    hyperspace.vacuumIndex("indexCUSTOMERS")
    ```

#### Task 2: Explore the Data Lake storage with the MSSparkUtil library

Microsoft Spark Utilities (MSSparkUtils) is a builtin package to help you easily perform common tasks. You can use MSSparkUtils to work with file systems, to get environment variables, and to work with secrets.

1. Continue with the same notebook from the previous task and add a new cell with the following code:

    ```python
    from notebookutils import mssparkutils

    #
    # Microsoft Spark Utilities
    #
    # https://docs.microsoft.com/en-us/azure/synapse-analytics/spark/microsoft-spark-utilities?pivots=programming-language-python
    #

    # Azure storage access info
    blob_account_name = datalake
    blob_container_name = 'wwi-02'
    blob_relative_path = '/'
    linkedServiceName = datalake
    blob_sas_token = mssparkutils.credentials.getConnectionStringOrCreds(linkedServiceName)

    # Allow SPARK to access from Blob remotely
    spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name), blob_sas_token)

    files = mssparkutils.fs.ls('/')
    for file in files:
        print(file.name, file.isDir, file.isFile, file.path, file.size)

    mssparkutils.fs.mkdirs('/SomeNewFolder')

    files = mssparkutils.fs.ls('/')
    for file in files:
        print(file.name, file.isDir, file.isFile, file.path, file.size)
    ```

2. Run the new cell and observe how `mssparkutils` is used to work with the file system.

### Resources

To learn more about the topics covered in this lab, use these resources:

- [Apache Spark in Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-overview)
- [Announcing Azure Data Explorer data connector for Azure Synapse](https://techcommunity.microsoft.com/t5/azure-data-explorer/announcing-azure-data-explorer-data-connector-for-azure-synapse/ba-p/1743868)
- [Connect to Azure Data Explorer using Apache Spark for Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics/quickstart-connect-azure-data-explorer)
- [Azure Synapse Analytics shared metadata](https://docs.microsoft.com/azure/synapse-analytics/metadata/overview)
- [Introduction of Microsoft Spark Utilities](https://docs.microsoft.com/azure/synapse-analytics/spark/microsoft-spark-utilities?pivots=programming-language-python)
- [Hyperspace - An open source indexing subsystem that brings index-based query acceleration to Apache Spark™ and big data workloads](https://github.com/microsoft/hyperspace)


