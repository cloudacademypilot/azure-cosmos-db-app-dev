# Load Data Into Cosmos DB with ADF

In this lab, you will populate an Azure Cosmos DB container from an existing set of data using tools built in to Azure. After importing, you will use the Azure portal to view your imported data.

> If you have not already completed setup for the lab content see the instructions for [Account Setup](00-account_setup.md) before starting this lab.  This will create an Azure Cosmos DB database and container that you will use throughout the lab. You will also use an **Azure Data Factory (ADF)** resource to import existing data into your container.

## Create Azure Cosmos DB Database and Container

You will now create a database and container within your Azure Cosmos DB account.

1. Navigate to the [Azure Portal](https://portal.azure.com)

1. On the left side of the portal, select the **Resource groups** link.

    ![Resource groups is highlighted](./assets/03-resource_groups.jpg "Select the Resource Groups")

1. In the **Resource groups** blade, locate and select the **cosmoslabs** resource group.

    ![The cosmoslabs resource group is highlighted](./assets/03-lab_resource_group.jpg "Select the cosmoslabs resource group")

1. In the **cosmoslabs** blade, select the **Azure Cosmos DB** account you recently created.

    ![The Cosmos DB resource is highlighted](./assets/03-cosmos_resource.jpg "Select the cosmoslabs resource")

1. In the **Azure Cosmos DB** blade, locate and select the **Overview** link on the left side of the blade. At the top select the **Add Container** button.

    ![Add container link is highlighted](./assets/03-add_collection.jpg "Add a new container")

1. In the **Add Container** popup, perform the following actions:

    1. In the **Database id** field, select the **Create new** option and enter the value **ImportDatabase**.

    2. Do not check the **Provision database throughput** option.

        > Provisioning throughput for a database allows you to share the throughput among all the containers that belong to that database. Within an Azure Cosmos DB database, you can have a set of containers which shares the throughput as well as containers, which have dedicated throughput.

    3. In the **Container Id** field, enter the value **FoodCollection**.

    4. In the **Partition key** field, enter the value ``/foodGroup``.

    5. In the **Throughput** field, enter the value ``11000``. *Note: we will reduce this to 400 RU/s after the data has been imported*

    6. Select the **OK** button.

1. Wait for the creation of the new **database** and **container** to finish before moving on with this lab.

## Import Lab Data Into Container

You will use **Azure Data Factory (ADF)** to import the JSON array stored in the **nutrition.json** file from Azure Blob Storage.


1. On the left side of the portal, select the **Resource groups** link.

    > To learn more about copying data to Cosmos DB with ADF, please read [ADF's documentation](https://docs.microsoft.com/azure/data-factory/connector-azure-cosmos-db).

    ![Resource groups link is highlighted](./assets/03-resource_groups.jpg "Select Resource Groups")

1. In the **Resource groups** blade, locate and select the Cloud Academy resource group containing the lab resources.

1. Amoung the resources you will find a storage account with name **cosmoslabstrgacct** and a data factory with name **cosmosLabAdf** with unique random string appended.

    ![Cosmos Lab Resources](./assets/03-storageacct-adf.jpg "Cosmos Lab Resources")

1. Open the storage account and click on containers link on the left navigation pane. You will find the container **nutritiondata**
    ![Cosmos Lab storage account](./assets/03-storageacct-container.jpg "Cosmos Lab storage account")

1. Click on the container name to navigate to the container detail screen. On the top you will see an upload button.
    ![Cosmos Lab storage account](./assets/03-storageacct-container-upload.jpg "Cosmos Lab storage account upload")

1. Click on the upload button, this will open up a form to upload blob file on the right hand side. Click on the **select file** button and nagivate to ***C:->Labs->setup folder*** on VM and select **NutritionData** json file using the file browser. Use the upload button to upload the file to the container  
    ![Cosmos Lab storage account](./assets/03-storageacct-file-upload.jpg "Cosmos Lab storage account file upload")
    
1. Right click on nutrition contatainer select **Generate SAS** and select Permissions dropdown check List and reader, then Click Generate SAS token and URL
    ![Cosmos Lab storage generate SAS](./assets/02_SASTokens.jpg "Cosmos Lab storage Generate SAS")
    
1. After file is uploaded, click on link **shared access tokens** to open the dialog for generating SAS token. Click on 
button **Generate SAS token and URL** to generate container SAS. Copy the **Blob SAS URL**. 
     ![Cosmos Lab storage account get SAS](./assets/03-storageacct-get-sas.jpg "Cosmos Lab storage account get SAS")
   
1. Open the Data Factory. Select **Open Azure Data Factory Studio** to launch ADF studio.

    ![The overview blade is displayed for ADF](./assets/03-adf-open.jpg "Select Author and Monitor link")

1. Click on  **Ingest**. This open the copy data tool

    ![The main workspace page is displayed for ADF](./assets/03-adf-ingest.jpg "Lauch copy data tool")

   - We will be using ADF for a one-time copy of data from a source JSON file on Azure Blob Storage to a database in Cosmos DB’s SQL API. ADF can also be used for more frequent data transfers from Cosmos DB to other data stores.Select to **Run once now**, then select **Next**

    ![Built in copy task is displayed](./assets/03-adf-built-in-copy.jpg "Select built in copy activity")

1. **Create a new connection** and select **Azure Blob Storage**. We will import data from a json file on Azure Blob Storage. In addition to Blob Storage, you can use ADF to migrate from a wide variety of sources. We will not cover migration from these sources in this tutorial.

    ![Create new connection link is highlighted](./assets/03-adf_blob.jpg "Create a new connection")

    ![Azure Blog Storage is highlighted](./assets/03-adf_blob2.jpg "Select the Azure Blob Storage connection type")

1. Name the source **NutritionJson** and select **SAS URI** as the Authentication method. Please use the SAS URI generated from the previous step for read-only access to the Blob Storage container:

     `https://cosmoslabstrgacctmyx5zl.blob.core.windows.net/nutritiondata?sp=r&st=2022-10-17T14:41:58Z&se=2022-10-17T22:41:58Z&spr=https&sv=2021-06-08&sr=c&sig=IZBcvj4HjjIO4K0lJ4ROMprH6rypsWZHO64bCPHzCoE%3D`

    ![The New linked service dialog is displayed](./assets/03-adf_connecttoblob.jpg "Enter the SAS url in the dialog")

1. Select **Create**
1. Select **Next**
1. In the **File or Folder** textbox, enter the folder name as ``nutirion-data`` and then click on **Browse** to select the **nutrition-data** folder. Finally select **NutritionData.json** file.

    ![The nutritiiondata folder is displayed](./assets/03-adf_choosestudents.jpg "Select the NutritionData.json file")

1. Un-check **Copy file recursively** or **Binary Copy** if they are checked. Also ensure that other fields are empty. Click **Next**

    ![The input file or folder dialog is displayed](./assets/03-adf_source_next.jpg "Ensure all other fields are empty, select next")

1. Select the file format as **JSON format**. Then select **Next**.

    !["The file format settings dialog is displayed"](./assets/03-adf_source_dataset_format.jpg "Ensure JSON format is selected, then select Next")

1. You have now successfully connected the Blob Storage container with the nutrition.json file as the source.

1. For the **Destination data store** add the Cosmos DB target data store by selecting **Create new connection** and selecting **Azure Cosmos DB (SQL API)**.

    !["The New Linked Service dialog is displayed"](./assets/03-adf_selecttarget.jpg "Select the Azure Cosmos DB service type")

1. Name the linked service **targetcosmosdb** and select your Azure subscription and Cosmos DB account. You should also select the Cosmos DB **ImportDatabase** that you created earlier.

    !["The linked service configuration dialog is displayed"](./assets/03-adf_selecttargetdb.jpg "Select the ImportDatabase database")

1. Select your newly created **targetcosmosdb** connection as the Destination data store.

    !["The destination data source dialog is displayed"](./assets/03-adf_destconnectionnext.jpg "Select your recently created data source")

1. Select your **FoodCollection** container from the drop-down menu. You will map your Blob storage file to the correct Cosmos DB container. Select **Next** to continue.

    !["The table mapping dialog is displayed"](./assets/03-adf_correcttable.jpg "Select the FoodCollection container")

1. There is no need to change any `Settings`. Select **next**.

    !["The settings dialog is displayed"](./assets/03-adf_settings.jpg "Review the dialog, select next")

1. Select **Next** to begin deployment After deployment is complete, select **Monitor**.

    !["The pipeline runs are displayed"](./assets/03-adf_progress.jpg "Notice the pipeline is In progress")

1. After a few minutes, refresh the page and the status for the ImportNutrition pipeline should be listed as **Succeeded**.

    !["The pipeline runs are displayed"](./assets/03-adf_progress_complete.jpg "The pipeline has succeeded")

1. Once the import process has completed, close the ADF. You will now proceed to validate your imported data.

## Validate Imported Data

The Azure Cosmos DB Data Explorer allows you to view documents and run queries directly within the Azure Portal. In this exercise, you will use the Data Explorer to view the data stored in our container.

You will validate that the data was successfully imported into your container using the **Items** view in the **Data Explorer**.

1. Return to the **Azure Portal** (<http://portal.azure.com>).

1. On the left side of the portal, select the **Resource groups** link.

    ![Resource groups link is highlighted](./assets/03-resource_groups.jpg "Select your resource group")

1. In the **Resource groups** blade, locate and select the **cosmoslabs** resource group.

    ![The Lab resource group is highlighted](./assets/03-lab_resource_group.jpg "Select the resource group")

1. In the **cosmoslabs** blade, select the **Azure Cosmos DB** account you recently created.

    ![The Cosmos DB resource is highlighted](./assets/03-cosmos_resource.jpg "Select the Cosmos DB resource")

1. In the **Azure Cosmos DB** blade, locate and select the **Data Explorer** link on the left side of the blade.

    ![The Data Explorer link was selected and is blade is displayed](./assets/03-data_explorer_pane.jpg "Select Data Explorer")

1. In the **Data Explorer** section, expand the **ImportDatabase** database node and then expand the **FoodCollection** container node.

    ![The Container node is displayed](./assets/03-collection_node.jpg "Expand the ImportDatabase node")

1. Within the **FoodCollection** node, select the **Scale and Settings** link to view the throughput for the container. Reduce the throughput to **400 RU/s**.

    ![Scale and Settings](./assets/03-collection-settings.png "Reduce throughput")

1. Within the **FoodCollection** node, select the **Items** link to view a subset of the various documents in the container. Select a few of the documents and observe the properties and structure of the documents.

    ![Items is highlighted](./assets/03-documents.jpg "Select Items")

    ![An Example document is displayed](./assets/03-example_document.jpg "Select a document")

> If this is your final lab, follow the steps in [Removing Lab Assets](11-cleaning_up.md) to remove all lab resources.
