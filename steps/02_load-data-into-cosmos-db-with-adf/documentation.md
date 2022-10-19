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

    1. In the **Database id** field, select the **Create new** option and enter the value **NutritionDatabase**.

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

    
1. After file is uploaded, click on link **shared access tokens** to open the dialog for generating SAS token and select Permissions dropdown check **List** and **reader**,  Click on **Generate SAS token and URL** button to generate container SAS. Copy the **Blob SAS URL**. 

     ![Cosmos Lab storage account get SAS](./assets/02_SASToken.jpg "Cosmos Lab storage account get SAS")
   
1. Open the Data Factory. Select **Open Azure Data Factory Studio** to launch ADF studio.

    ![The overview blade is displayed for ADF](./assets/03-adf-open.jpg "Select Author and Monitor link")

1. Click on  **Ingest**. This open the copy data tool

    ![The main workspace page is displayed for ADF](./assets/03-adf-ingest.jpg "Lauch copy data tool")

   - We will be using ADF for a one-time copy of data from a source JSON file on Azure Blob Storage to a database in Cosmos DB’s SQL API. ADF can also be used for more frequent data transfers from Cosmos DB to other data stores.Select to **Run once now**, then select **Next**

    ![Built in copy task is displayed](./assets/03-adf-built-in-copy.jpg "Select built in copy activity")

1. Select Source type as **Azure Blob Storage**. We will import data from a json file on Azure Blob Storage. In addition to Blob Storage, you can use ADF to migrate from a wide variety of sources. We will not cover migration from these sources in this tutorial.

    ![Create new connection link is highlighted](./assets/02_SourceType.jpg "Create a new connection")
    
1. Click on **New Connection** , Name it as **NutritionJson** and select **SAS URI** as the Authentication method. Please use the SAS URI generated from the previous step for read-only access to the Blob Storage container.

    ![The New linked service dialog is displayed](./assets/02_Connectionstring.jpg "Enter the SAS url in the dialog")

1. Select **Create**
1. Select **Next**
1. In the **File or Folder** textbox, enter the folder name as ``nutritiondata`` and then click on **Browse** to open **NutritionData.json** file.

    ![The nutritiiondata folder is displayed](./assets/02_Selectnutrition.jpg "Select the NutritionData.json file")

1. Un-check **Copy file recursively** or **Binary Copy** if they are checked. Also ensure that other fields are empty. Click **Next**

    ![The input file or folder dialog is displayed](./assets/02-uncheck.jpg "Ensure all other fields are empty, select next")

1. Select the file format as **JSON format**. Then select **Next**.

    !["The file format settings dialog is displayed"](./assets/03-adf_source_dataset_format.jpg "Ensure JSON format is selected, then select Next")

1. You have now successfully connected the Blob Storage container with the nutrition.json file as the source.

1. Under **Destination data store** , select destination type as  **Azure Cosmos DB (SQL API)**.

    !["The New Linked Services dialog is displayed"](./assets/02_Destination-Sqlapi.jpg "Select the Azure Cosmos DB service type")

1. For Connection, click on **New connection**, Name the linked service targetcosmosdb and select Cosmos DB account name. You should also select the Cosmos DB NutritionDatabase that you created earlier,click on Create.

    !["The New Linked Service dialog is displayed"](./assets/02_Destination_sub.jpg "Select the Azure Cosmos DB linked service type")

1. Select your **FoodCollection** container from the drop-down menu. You will map your Blob storage file to the correct Cosmos DB container. Select **Next** to continue.

    !["The table mapping dialog is displayed"](./assets/02_Destination_foodcollection.jpg "Select the FoodCollection container")

1. You can give any name for Task name or there is no need to change any `Settings`. Select **next**.

    !["The settings dialog is displayed"](./assets/02-cosmos_task_Name.jpg "Review the dialog, select next")

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

    ![The Data Explorer link was selected and is blade is displayed](./assets/02-cosmos_dataexplorer_without.jpg "Select Data Explorer")

1. In the **Data Explorer** section, expand the **NutritionDatabase** database node and then expand the **FoodCollection** container node.

    ![The Container node is displayed](./assets/02-cosmos_dataexplorer.jpg "Expand the ImportDatabase node")

1. Within the **FoodCollection** node, select the **Scale and Settings** link to view the throughput for the container. Reduce the throughput to **400 RU/s**.

    ![Scale and Settings](./assets/03-collection-settings.png "Reduce throughput")

1. Within the **FoodCollection** node, select the **Items** link to view a subset of the various documents in the container. Select a few of the documents and observe the properties and structure of the documents.

    ![Items is highlighted](./assets/02-cosmos_items.jpg "Select Items")
    

    ![An Example document is displayed](./assets/02-cosmos_items_structure.jpg "Select a document")

> If this is your final lab, follow the steps in [Removing Lab Assets](11-cleaning_up.md) to remove all lab resources.
