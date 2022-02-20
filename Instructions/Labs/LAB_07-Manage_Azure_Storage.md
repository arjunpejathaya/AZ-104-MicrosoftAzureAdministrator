---
lab:
    title: '07 - Manage Azure storage'
    module: 'Module 07 - Azure Storage'
---

# Lab 07 - Manage Azure Storage
# Student lab manual

## Lab scenario

You need to evaluate the use of Azure storage for storing files residing currently in on-premises data stores. While majority of these files are not accessed frequently, there are some exceptions. You would like to minimize cost of storage by placing less frequently accessed files in lower-priced storage tiers. You also plan to explore different protection mechanisms that Azure Storage offers, including network access, authentication, authorization, and replication. Finally, you want to determine to what extent Azure Files service might be suitable for hosting your on-premises file shares.

## Objectives

In this lab, you will:

+ Task 1: Provision the lab environment
+ Task 2: Create and configure Azure Storage accounts
+ Task 3: Manage blob storage
+ Task 4: Manage authentication and authorization for Azure Storage
+ Task 5: Create and configure an Azure Files shares
+ Task 6: Manage network access for Azure Storage
+ Task 7: Implement File Sync in Azure

## Estimated timing: 40 minutes

## Architecture diagram

![image](../media/lab07.png)


## Instructions

## Exercise 1

### Task 1: Provision the lab environment

In this task, you will deploy an Azure virtual machine that you will use later in this lab.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. In the Azure portal, open the **Azure Cloud Shell** by clicking on the icon in the top right of the Azure Portal.

1. If prompted to select either **Bash** or **PowerShell**, select **PowerShell**.

    >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and click **Create storage**.

1. In the toolbar of the Cloud Shell pane, click the **Upload/Download files** icon, in the drop-down menu, click **Upload** and upload the files **\\Allfiles\\Labs\\07\\az104-07-vm-template.json** and **\\Allfiles\\Labs\\07\\az104-07-vm-parameters.json** into the Cloud Shell home directory.

1. Edit the **Parameters** file you just uploaded and change the password. If you need help editing the file in the Shell please ask your instructor for assistance. As a best practice, secrets, like passwords, should be more securely stored in the Key Vault. 

1. From the Cloud Shell pane, run the following to create the resource group that will be hosting the virtual machine (replace the '[Azure_region]' placeholder with the name of an Azure region where you intend to deploy the Azure virtual machine)

    >**Note**: To list the names of Azure regions, run `(Get-AzLocation).Location`
    >**Note**: Each command below should be typed separately

    ```powershell
    $location = '[Azure_region]'
    ```
  
    ```powershell
     $rgName = 'az104-07-rg0'
    ```

    ```powershell
    New-AzResourceGroup -Name $rgName -Location $location
    ```
    
1. From the Cloud Shell pane, run the following to deploy the virtual machine by using the uploaded template and parameter files:

   ```powershell
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-07-vm-template.json `
      -TemplateParameterFile $HOME/az104-07-vm-parameters.json `
      -AsJob
   ```

    >**Note**: Do not wait for the deployments to complete, but proceed to the next task.

1. Close the Cloud Shell pane.

### Task 2: Create and configure Azure Storage accounts

In this task, you will create and configure an Azure Storage account.

1. In the Azure portal, search for and select **Storage accounts**, and then click **+ Create**.

1. On the **Basics** tab of the **Create storage account** blade, specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | the name of a **new** resource group **az104-07-rg1** |
    | Storage account name | any globally unique name between 3 and 24 in length consisting of letters and digits |
    | Region | the name of an Azure region where you can create an Azure Storage account  |
    | Performance | **Standard** |
    | Redundancy | **Geo-redundant storage (GRS)** |

1. Click **Next: Advanced >**, on the **Advanced** tab of the **Create storage account** blade, review the available options, accept the defaults, and click **Next: Networking >**.

1. On the **Networking** tab of the **Create storage account** blade, review the available options, accept the default option **Public endpoint (all networks}** and click **Next: Data protection >**.

1. On the **Data protection** tab of the **Create storage account** blade, review the available options, accept the defaults, click **Review + Create**, wait for the validation process to complete and click **Create**.

    >**Note**: Wait for the Storage account to be created. This should take about 2 minutes.

1. On the deployment blade, click **Go to resource** to display the Azure Storage account blade.

1. On the Storage account blade, in the **Data management** section, click **Geo-replication** and note the secondary location. 

1. On the Storage account blade, in the **Settings** section, select **Configuration**, in the **Replication** drop-down list select **Locally redundant storage (LRS)** and save the change.

1. Switch back to the **Geo-replication** blade and note that, at this point, the Storage account has only the primary location.

1. Display again the **Configuration** blade of the Storage account, set **Blob access tier (default)** to **Cool**, and save the change.

    > **Note**: The cool access tier is optimal for data which is not accessed frequently.

### Task 3: Manage blob storage

In this task, you will create a blob container and upload a blob into it.

1. On the Storage account blade, in the **Data storage** section, click **Containers**.

1. Click **+ Container** and create a container with the following settings:

    | Setting | Value |
    | --- | --- |
    | Name | **az104-07-container**  |
    | Public access level | **Private (no anonymous access)** |

1. In the list of containers, click **az104-07-container** and then click **Upload**.

1. Browse to **\\Allfiles\\Labs\\07\\LICENSE** on your lab computer and click **Open**.

1. On the **Upload blob** blade, expand the **Advanced** section and specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Authentication type | **Account key**  |
    | Blob type | **Block blob** |
    | Block size | **4 MB** |
    | Access tier | **Hot** |
    | Upload to folder | **licenses** |

    > **Note**: Access tier can be set for individual blobs.

1. Click **Upload**.

    > **Note**: Note that the upload automatically created a subfolder named **licenses**.

1. Back on the **az104-07-container** blade, click **licenses** and then click **LICENSE**.

1. On the **licenses/LICENSE** blade, review the available options.

    > **Note**: You have the option to download the blob, change its access tier (it is currently set to **Hot**), acquire a lease, which would change its lease status to **Locked** (it is currently set to **Unlocked**) and protect the blob from being modified or deleted, as well as assign custom metadata (by specifying an arbitrary key and value pairs). You also have the ability to **Edit** the file directly within the Azure portal interface, without downloading it first. You can also create snapshots, as well as generate a SAS token (you will explore this option in the next task).

### Task 4: Manage authentication and authorization for Azure Storage

In this task, you will configure authentication and authorization for Azure Storage.

1. On the **licenses/LICENSE** blade, on the **Overview** tab, click **Copy to clipboard** button next to the **URL** entry.

1. Open another browser window by using InPrivate mode and navigate to the URL you copied in the previous step.

1. You should be presented with an XML-formatted message stating **ResourceNotFound** or **PublicAccessNotPermitted**.

    > **Note**: This is expected, since the container you created has the public access level set to **Private (no anonymous access)**.

1. Close the InPrivate mode browser window, return to the browser window showing the **licenses/LICENSE** blade of the Azure Storage container, and switch to the the **Generate SAS** tab.

1. On the **Generate SAS** tab of the **licenses/LICENSE** blade, specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Signing key | **Key 1** |
    | Permissions | **Read** |
    | Start date | yesterday's date |
    | Start time | current time |
    | Expiry date | tomorrow's date |
    | Expiry time | current time |
    | Allowed IP addresses | leave blank |
    

1. Click **Generate SAS token and URL**.

1. Click **Copy to clipboard** button next to the **Blob SAS URL** entry.

1. Open another browser window by using InPrivate mode and navigate to the URL you copied in the previous step.

    > **Note**: If you are using Microsoft Edge, you should be presented with the **The MIT License (MIT)** page. If you are using Chrome, Microsoft Edge (Chromium) or Firefox, you should be able to view the content of the file by downloading it and opening it with Notepad.

    > **Note**: This is expected, since now your access is authorized based on the newly generated the SAS token.

    > **Note**: Save the blob SAS URL. You will need it later in this lab.

1. Close the InPrivate mode browser window, return to the browser window showing the **licenses/LICENSE** blade of the Azure Storage container, and from there, navigate back to the **az104-07-container** blade.

1. Click the **Switch to the Azure AD User Account** link next to the **Authentication method** label.

    > **Note**: You can see an error when you change the authentication method (the error is *"You do not have permissions to list the data using your user account with Azure AD"*). It is expected.  

    > **Note**: At this point, you do not have permissions to change the Authentication method.

1. On the **az104-07-container** blade, click **Access Control (IAM)**.

1. On the **Check access** tab, click **Add role assignment**.

1. On the **Add role assignment** blade, specify the following settings:

    | Setting | Value |
    | --- | --- |
    | Role | **Storage Blob Data Owner** |
    | Assign access to | **User, group, or service principal** |
    | Members | the name of your user account |

1. Click **Review + Assign** and then **Review + assign**, and return to the **Overview** blade of the **az104-07-container** container and verify that you can change the Authentication method to (Switch to Azure AD User Account).

    > **Note**: It might take about 5 minutes for the change to take effect.

### Task 5: Create and configure an Azure Files shares

In this task, you will create and configure Azure Files shares.

> **Note**: Before you start this task, verify that the virtual machine you provisioned in the first task of this lab is running.

1. In the Azure portal, navigate back to the blade of the storage account you created in the first task of this lab and, in the **Data storage** section, click **File shares**.

1. Click **+ File share** and create a file share with the following settings:

    | Setting | Value |
    | --- | --- |
    | Name | **az104-07-share** |

1. Click the newly created file share and click **Connect**.

1. On the **Connect** blade, ensure that the **Windows** tab is selected. Below you will find a grey textbox with a script, in the bottom right corner of that box hover over the pages icon and click **Copy to clipboard**.

1. In the Azure portal, search for and select **Virtual machines**, and, in the list of virtual machines, click **az104-07-vm0**.

1. On the **az104-07-vm0** blade, in the **Operations** section, click **Run command**.

1. On the **az104-07-vm0 - Run command** blade, click **RunPowerShellScript**.

1. On the **Run Command Script** blade, paste the script you copied earlier in this task into the **PowerShell Script** pane and click **Run**.

1. Verify that the script completed successfully.

1. Replace the content of the **PowerShell Script** pane with the following script and click **Run**:

   ```powershell
   New-Item -Type Directory -Path 'Z:\az104-07-folder'

   New-Item -Type File -Path 'Z:\az104-07-folder\az-104-07-file.txt'
   ```

1. Verify that the script completed successfully.

1. Navigate back to the **az104-07-share** file share blade, click **Refresh**, and verify that **az104-07-folder** appears in the list of folders.

1. Click **az104-07-folder** and verify that **az104-07-file.txt** appears in the list of files.

### Task 6: Manage network access for Azure Storage

In this task, you will configure network access for Azure Storage.

1. In the Azure portal, navigate back to the blade of the storage account you created in the first task of this lab and, in the **Security + Networking** section, click **Networking** and then click **Firewalls and virtual networks**.

1. Click the **Selected networks** option and review the configuration settings that become available once this option is enabled.

    > **Note**: You can use these settings to configure direct connectivity between Azure virtual machines on designated subnets of virtual networks and the storage account by using service endpoints.

1. Click the checkbox **Add your client IP address** and save the change.

1. Open another browser window by using InPrivate mode and navigate to the blob SAS URL you generated in the previous task.

1. You should be presented with the content of **The MIT License (MIT)** page.

    > **Note**: This is expected, since you are connecting from your client IP address.

1. Close the InPrivate mode browser window, return to the browser window showing the **Networking** blade of the Azure Storage account.

1. In the Azure portal, open the **Azure Cloud Shell** by clicking on the icon in the top right of the Azure Portal.

1. If prompted to select either **Bash** or **PowerShell**, select **PowerShell**.

1. From the Cloud Shell pane, run the following to attempt downloading of the LICENSE blob from the **az104-07-container** container of the storage account (replace the `[blob SAS URL]` placeholder with the blob SAS URL you generated in the previous task):

   ```powershell
   Invoke-WebRequest -URI '[blob SAS URL]'
   ```
1. Verify that the download attempt failed.

    > **Note**: You should receive the message stating **AuthorizationFailure: This request is not authorized to perform this operation**. This is expected, since you are connecting from the IP address assigned to an Azure VM hosting the Cloud Shell instance.

1. Close the Cloud Shell pane.

### Task 7: Implement File Sync in Azure

1. Create an Azure Storage Account with the following configuration.

    | Setting | Value |
    | -- | -- |
    | Storage Account name | az104-filesync-**"your initials"** |
    | Region | **East US** |
    | Performance | **Standard** |
    | Redundancy | **Locally Redundant Storage (LRS)** |

2. Click on **Review + Create** and wait till the storage account is created.

3. Create a **File Share** in the storage account with the name **test**. Accept defaults for the other settings.

4. In the Azure Portal, search for the **Storage Sync Services** resource.

5. Click on **+ Create** and select the Resource Group and enter a name for the Sync Service. Select the region as **East US**.

6. Click on **Review + Create** and create the Sync Service.

3. From the **All services** blade in the Portal Menu, search for and select **Virtual machines**, and then click **+Add, +Create, +New** and choose **+Virtual machine** from the drop down.

4. On the **Basics** tab, fill in the following information (leave the defaults for everything else):

    | Settings | Values |
    |  -- | -- |
    | Subscription | **Use default supplied** |
    | Resource group | **Create new resource group** |
    | Virtual machine name | **myVM** |
    | Region | **(US) East US**|
    | Availability options | No infrastructure redundancy options required|
    | Image | **Windows Server 2019 Datacenter - Gen2**|
    | Size | **Standard D2s v3**|
    | Administrator account username | **azureuser** |
    | Administrator account password (type in carefully!) | **Enter your own password**|
    | Inbound port rules - | **Allow select ports **|
    | Select inbound ports | **RDP (3389)**| 

5. Switch to the Networking tab to ensure **RDP (3389)** are selected in section **Select inbound ports**.

6. Switch to the Management tab, and in its **Monitoring** section, select the following setting:

    | Settings | Values |
    | -- | -- |
    | Boot diagnostics | **Disable**|

7. Leave the remaining values on the defaults and then click the **Review + create** button at the bottom of the page.

8. Once Validation is passed click the **Create** button. It can take anywhere from five to seven minutes to deploy the virtual machine.

9. You will receive updates on the deployment page and via the **Notifications** area (the bell icon in the top menu bar).

14. Open the **Storage Sync Services** resource created and from the **Overview** page, select **+ Sync Group**. Enter the following details

    | Setting | Value |
    | -- | -- |
    | Sync Group Name | Local to Cloud Group |
    | Storage Account | Click on **Select Storage Account** and select the account and file share you created |

15. Click on **Create** and wait till the Cloud Endpoint is created.

16. Once created, Click on the Sync Group and notice that you cannot add another Cloud Endpoint. 

17. Go back to the previous blade and from the left pane, select **Registered Servers** and right click, copy the **File Sync Agent** URL from the top instructions.

In the next set of steps, we will connect to our new virtual machine using RDP (Remote Desktop Protocol). 

18. Click on bell icon from the upper blue toolbar, and select 'Go to resource' when your deployment has succeded. 

    **Note**: You could also use the **Go to resource** link on the deployment page 

19. On the virtual machine **Overview** blade, click **Connect** button and choose **RDP** from the drop down.

    **Note**: The following directions tell you how to connect to your VM from a Windows computer. On a Mac, you need an RDP client such as this Remote Desktop Client from the Mac App Store and on a Linux computer you can use an open source RDP client.

2. On the **Connect to virtual machine** page, keep the default options to connect with the public IP address over port 3389 and click **Download RDP File**. A file will download on the bottom left of your screen.

3. **Open** the downloaded RDP file (located on the bottom left of your lab machine) and click **Connect** when prompted. 

4. In the **Windows Security** window, sign in using your Azure Portal Global Administrator account (the one you used to redeem your Azure Pass) and it's password. 

5. You may receive a warning certificate during the sign-in process. Click **Yes** or to create the connection and connect to your deployed VM. You should connect successfully.

6. A new Virtual Machine (myVM) will launch inside your Lab. Close the Server Manager and dashboard windows that pop up (click "x" at top right). You should see the blue background of your virtual machine. **Congratulations!** You have deployed and connected to a Virtual Machine running Windows Server.

25. In the **Server Manager**, on the left pane, select **Local Server**

26. In the newly opened tab, click on **On** next to **IE Enahanced Security Configuration** and select **Off** for Administrators.

27. Open **Internet Explorer**. Click on **Ask me Later** if prompted and paste the URL in the Address Bar.

28. Check the **StorageSyncAgent_WS2019.msi** entry from the list and click on Next.

29. In the pop-up blocked alert from the bottom pane, click on **Options for this site** and **Always Allow**. The file should be saved to your **Downloads** directory. Run the downloaded file and follow the steps to install the same by accepting all default settings.

30. Go to the Desktop, Right Click and create a new folder and give the name **onpremtest**. Right Click on the folder and go to properties. Go to the **Sharing** section and click on the **Share** button and the **Share** button in the bottom pane again. Click on **Yes, turn on network discovery and file sharing for all public networks** in the prompt and click on done and close the open window for the folder.

31. Once the agent installation finishes, it will check for updates. Click on **OK** and you should be prompted for registration.

32. Select the Azure Environment as **AzureCloud** and click on Sign in.

33. Login with your Azure Pass Subscription Account.

34. Select your subscription, Resource Group and Storage Sync Service and click on **Register**.
    >**Note**: It may take some time for the Storage Sync Service to appear. In that case, you may check back later in 10-15 minutes. Click on Start and click on **Azure Storage Sync Agent Updater** to relaunch the process.
35. Once the registration succeeds, click on **Copy Azure Resource URL** and minimise your RDP Window.

36. In the Storage Sync Service resource, go to the **Sync Group** and click on **Add Server endpoint**.

37. Select the registered server from the dropdown.
    >**Note**: It may take some time for the server to show up in the dropdown. You may wait for 10-15 minutes and refresh your tab and continue the process.

38. Specify the path as **C:\Users\azureuser\Desktop\onpremtest**.

39. You may enable tiering if you want to cache your information in the Virtual Machine (On Premises Server equivalent in this case). Keep it as **Disabled** for now and click on **Create**. Wait till the Server Endpoint is added. **Cloud Tiering is a feature which stores only frequently accessed files on the Local Server to enhance it's performance.**

40. Wait till the Health of the server endpoint changes from **Pending** to **Healthy** and try to add the server endpoint again and note your observations.

41. In the RDP Window, create a new text file in the folder and see if it gets synced to the File Share in the Storage Account. Note your observations with regards to the time and the directionality of sync.

### Clean up resources

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

>**Note**:  Don't worry if the lab resources cannot be immediately removed. Sometimes resources have dependencies and take a long time to delete. It is a common Administrator task to monitor resource usage, so just periodically review your resources in the Portal to see how the cleanup is going. You might also try to delete the Resource Group where the resources reside. That is a quick Administrator shortcut. If you have concerns speak to your instructor.

1. In the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. List all resource groups created throughout the labs of this module by running the following command:

   ```powershell
   Get-AzResourceGroup -Name 'az104-07*'
   ```

1. Delete all resource groups you created throughout the labs of this module by running the following command:

   ```powershell
   Get-AzResourceGroup -Name 'az104-07*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**Note**: The command executes asynchronously (as determined by the -AsJob parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the resource groups are actually removed.

2. To remove the Sync Service, Go to the sync group and next to the **Cloud Endpoint** and **Server Endpoint**, select **Delete**. Wait until the process is finished.

1. Go to the Registered Servers and next to the server **Unregister server**. Wait until the de-provisioning happens.

3. Remove the Sync Group by going to the Sync Group and selecting **Delete** in the top pane.

4. Finally, go to the previous blade and remove the Sync Group by clicking on **Delete**.

    >**Note**: The above four steps need to be executed after the completion of the previous steps. Ex: Do not execute step 5 before step 4 execution is completed.

### Review Questions
- If you have vendors outside your organisation, what method will you suggest to give them Granular access to your files? Is there any way you can invalidate their access if required?
- What is the difference between Blob Storage and File Shares?
- How can you configure your files in Hot Storage to move to Cool Storage Tier based on a rule?
- What is the mapping of the Cloud Endpoint to the Storage Endpoint?

### Review

In this lab, you have:

- Provisioned the lab environment
- Created and configured Azure Storage accounts
- Managed blob storage
- Managed authentication and authorization for Azure Storage
- Created and configured an Azure Files shares
- Managed network access for Azure Storage
- Implemented a File Sync Service