<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="Learn how to use Microsoft HPC Pack and Azure to set up a small, hybrid high performance computing (HPC) cluster" metaCanonical="" services="" documentationCenter="" title="Set up a Hybrid Cluster with Microsoft HPC Pack" authors="danlep" solutions="" manager="dongill" editor="mattshel" />



#Set up a Hybrid Cluster with Microsoft HPC Pack
This tutorial shows you how to use Microsoft HPC Pack 2012 R2 and Azure to set up a small, hybrid high performance computing (HPC) cluster. The cluster will consist of an on-premises head node (a computer running the Windows Server operating system and HPC Pack) and some compute nodes you deploy on-demand as worker role instances in an Azure cloud service. You can then run compute jobs on the hybrid cluster.
 
![Hybrid HPC cluster][Overview] 

This tutorial shows one approach, sometimes called cluster "burst to the cloud," to use scalable, on-demand compute resources in Azure to run compute intensive applications.

This tutorial assumes no prior experience with compute clusters or HPC Pack. It is intended only to help you deploy a hybrid cluster quickly for demonstration purposes. For considerations and steps to deploy a hybrid HPC Pack cluster at greater scale in a production environment, see the [detailed guidance](http://go.microsoft.com/fwlink/p/?LinkID=200493).

>[WACOM.NOTE] Azure offers a [range of sizes](http://go.microsoft.com/fwlink/p/?LinkId=389844) for your compute resources, suitable for different workloads. For example, the A8 and A9 instances combine high performance and access to a low latency, high throughput application network needed for certain HPC applications. For information, see [A8 and A9 Compute Intensive Instances: Quick Start with HPC Pack](http://go.microsoft.com/fwlink/p/?Linkid=328042). 

This tutorial walks you through these basic steps:

* [Prerequisistes](#BKMK_Prereq)
* [Install HPC Pack on the head node](#BKMK_DeployHN)
* [Prepare the Azure subscription](#BKMK_Prpare)
* [Configure the head node](#BKMK_ConfigHN)
* [Add Azure nodes to the cluster](#BKMK_worker)
* [Start the Azure nodes](#BKMK_start)
* [Run a command across the cluster](#BKMK_RunCommand)
* [Run a test job](#BKMK_RunJob)
* [Stop the Azure nodes](#BKMK_stop)

<h2 id="BKMK_Prereq">Prerequisites</h2>

>[WACOM.NOTE]To complete this tutorial, you need an Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="http://www.windowsazure.com/en-us/pricing/free-trial/" target="_blank">Azure Free Trial</a>.

In addition, you need the following for this tutorial.

* An on-premises computer that is running an edition of Windows Server 2012 R2 or Windows Server 2012. This computer will be the head node of the HPC cluster. If you are not already running Windows Server, you can download and install an [evaluation version](http://technet.microsoft.com/evalcenter/dn205286.aspx).

	* The computer must be joined to an Active Directory domain.

	* Ensure that no additional server roles or role services are installed.

	* To support HPC Pack, the operating system must be installed in one of these languages: English, Japanese, or Chinese (Simplified).

	* Verify that important and critical updates are installed.
	
* Installation files for HPC Pack 2012 R2, which is available free of charge. [Download](http://go.microsoft.com/fwlink/p/?linkid=389557) the full installation package and copy the files to the head node computer or to a network location. Choose installation files in the same language as your installation of Windows Server.

* A domain account with local Administrator permissions on the head node.

* TCP connectivity on port 443 from the head node to Azure.

<h2 id="BKMK_DeployHN">Install HPC Pack on the head node</h2>

You first install Microsoft HPC Pack on an on-premises computer that will be the head node of the cluster.

1. Log on to the head node by using a domain account that has local Administrator permissions.

2. Start the HPC Pack Installation Wizard by running Setup.exe from the HPC Pack installation files.

3. On the **HPC Pack 2012 R2 Setup** screen, click **New installation or add new features to an existing installation**.

	![HPC Pack 2012 Setup][install_hpc1]

4. On the **Microsoft Software User Agreement page**, click **Next**.

5. On the **Select Installation Type** page, click **Create a new HPC cluster by creating a head node**, and then click **Next**.

	![Select Installation Type][install_hpc2]

6. The wizard runs several pre-installation tests. Click **Next** on the **Installation Rules** page if all tests pass. Otherwise, review the information provided and make any necessary changes in your environment. Then run the tests again or if necessary start the Installation Wizard again. 

	![Installation Rules][install_hpc3]

7. On the **HPC DB Configuration** page, make sure **Head Node** is selected for all HPC databases, and then click **Next**.

	![DB Configuration][install_hpc4]

8. Accept default selections on the remaining pages of the wizard. On the **Install Required Components** page, click **Install**.

	![Install][install_hpc6]

9. After the installation completes, uncheck **Start HPC Cluster Manager** and then click **Finish**. (You will start HPC Cluster Manager in a later step to complete the configuration of the head node.)

	![Finish][install_hpc7]

<h2 id="BKMK_Prepare">Prepare the Azure subscription</h2>
Use the [Management Portal](https://manage.windowsazure.com) to perform the following steps with your Azure subscription. These are needed so you can later deploy Azure nodes from the on-premises head node. 

- Upload a management certificate (needed for secure connections between the head node and the Azure services)
 
- Create an Azure cloud service in which Azure nodes (worker role instances) will run

- Create an Azure storage account
	
	>[WACOM.NOTE]Also make a note of your Azure subscription ID, which you will need later. Find this in your Azure <a href="[https://account.windowsazure.com/Subscriptions">account information</a>.

<h3>Upload the default management certificate</h3>
HPC Pack installs a self-signed certificate on the head node, called the Default Microsoft HPC Azure Management certificate, that you can upload as an Azure management certificate. This certificate is provided for testing purposes and proof-of-concept deployments.

1. From the head node computer, sign in to the [Management Portal](https://manage.windowsazure.com).

2. Click **Settings**, and then click **Management Certificates**.

3. On the command bar, click **Upload**.

	![Certificate Settings][upload_cert1]

4. Browse on the head node for the file C:\Program Files\Microsoft HPC Pack 2012\Bin\hpccert.cer. Then, click the **Check** button.

	![Upload Certificate][install_hpc10]

You will see **Default HPC Azure Management** in the list of management certificates.

<h3>Create an Azure cloud service</h3> 

>[WACOM.NOTE]For best performance, create the cloud service and the storage account in the same geographic region or affinity group (if configured).

1. In the Management Portal, on the command bar, click **New**.

2. Click **Compute**, click **Cloud Service**, and then click **Quick Create**.

3. Type a URL for the cloud service, and then click **Create Cloud Service**.

	![Create Service][createservice1]

<h3>Create an Azure storage account</h3>

1. In the Management Portal, on the command bar, click **New**.

2. Click **Data Services**, click **Storage**, and then click **Quick Create**.

3. Type a URL for the account, and then click **Create Storage Account**.

	![Create Storage][createstorage1]

<h2 id="BKMK_ConfigHN">Configure the head node</h2>

To use HPC Cluster Manager to deploy Azure nodes and to submit jobs, first perform some required cluster configuration steps. 

1. On the head node, start HPC Cluster Manager. If the **Select Head Node** dialog box appears, click **Local Computer**. The **Deployment To-do List** appears.

2. Under **Required deployment tasks**, click **Configure your network**.

	![Configure Network][config_hpc2]

3. In the Network Configuration Wizard, select **All nodes only on an enterprise network** (Topology 5).

	![Topology 5][config_hpc3] 

	>[WACOM.NOTE]This is the simplest configuration for demonstration purposes, because the head node only needs a single network adapter to connect to Active Directory and the Internet. This tutorial does not cover cluster scenarios that require additional networks. 
	 
4. Click **Next** to accept default values on the remaining pages of the wizard. Then, on the **Review** tab, click **Configure** to complete the network configuration.

5. In the **Deployment To-do List**, click **Provide installation credentials**. 

6. In the **Installation Credentials** dialog box, type the credentials of the domain account that you used to install HPC Pack. Then click **OK**.

	![Installation Credentials][config_hpc6]

	>[WACOM.NOTE]HPC Pack services only use installation credentials to deploy on-premises compute nodes.
	
7. In the **Deployment To-do List**, click **Configure the naming of new nodes**. 

8. In the **Specify Node Naming Series** dialog box, accept the default naming series and click **OK**.

	![Node Naming][config_hpc8]

	>[WACOM.NOTE]The naming series only generates names for domain-joined compute nodes. Azure nodes are named automatically.
	  
9. In the **Deployment To-do List**, click **Create a node template**. You will use the node template to add Azure nodes to the cluster.

10. In the Create Node Template Wizard, do the following:

	a. On the **Choose Node Template Type** page, click **Azure node template**, and then click **Next**.

	![Node Template][config_hpc10]

	b. Click **Next** to accept the default template name.

	c. On the **Provide Subscription Information** page, enter your Azure subscription ID (available in your Azure <a href="[https://account.windowsazure.com/Subscriptions">account information</a>). Then, in **Management certificate**, click **Browse** and select **Default HPC Azure Management.** Then click **Next**.

	![Node Template][config_hpc12]

	d. On the **Provide Service Information** page, select the cloud service and the storage account that you created in a previous step. Then click **Next**.

	![Node Template][config_hpc13]

	e. Click **Next** to accept default values on the remaining pages of the wizard. Then, on the **Review** tab, click **Create** to create the node template.

	>[WACOM.NOTE]By default, the Azure node template includes settings for you to start (provision) and stop the nodes manually. You can also configure a schedule to start and stop the Azure nodes automatically. 
	
<h2 id="#BKMK_worker">Add Azure nodes to the cluster</h2>

You now use the node template to add Azure nodes to the cluster. Adding the nodes to the cluster stores their configuration information so that you can start (provision) them at any time as role instances in the cloud service. Your subscription only gets charged for Azure nodes after the role instances are running in the cloud service.

For this tutorial you will add two Small nodes.

1. In HPC Cluster Manager, in **Node Management**, in the **Actions** pane, click **Add Node**. 

	![Add Node][add_node1]

2. In the Add Node Wizard, on the **Select Deployment Method** page, click **Add Azure nodes**, and then click **Next**.

	![Add Azure Node][add_node1_1]

3. On the **Specify New Nodes** page, select the Azure node template you created previously (called by default **Default AzureNode Template**). Then specify **2** nodes of size **Small**, and then click **Next**.

	![Specify Nodes][add_node2]

	For details about the available virtual machine sizes, see [Virtual Machine and Cloud Service Sizes for Azure](http://msdn.microsoft.com/library/windowsazure/dn197896.aspx).

4. On the **Completing the Add Node Wizard** page, click **Finish**. 

	 Two Azure nodes, named **AzureCN-0001** and **AzureCN-0002**, now appear in HPC Cluster Manager. They are both in the **Not-Deployed** state.

	![Added Nodes][add_node3]

<h2 id="BKMK_Start">Start the Azure nodes</h2>
When you want to use the cluster resources in Azure, use HPC Cluster Manager to start (provision) the Azure nodes and bring them online.

1.	In HPC Cluster Manager, in **Node Management**, click one or both nodes and then, in the **Actions** pane, click **Start**. 

	![Start Nodes][add_node4]

2. In the **Start Azure Nodes** dialog box, click **Start**.

	![Start Nodes][add_node5]
 
	The nodes transition to the **Provisioning** state. You can view the provisioning log to track the provisioning progress.

	![Provision Nodes][add_node6]

3. After a few minutes, the Azure nodes finish provisioning and are in the **Offline** state. In this state the role instances are running but will not yet accept cluster jobs.

4. To confirm that the role instances are running, in the [Management Portal](https://manage.windowsazure.com), click **Cloud Services**, click the name of your cloud service, and then click **Instances**. 

	![Running Instances][view_instances1]

	You will see two worker role instances are running in the service. HPC Pack also automatically deploys two **HpcProxy** instances (size Medium) to handle communication between the head node and Azure.

5. To bring the Azure nodes online to run cluster jobs, select the nodes, right-click, and then click **Bring Online**.

	![Offline Nodes][add_node7]
 
	HPC Cluster Manager indicates that the nodes are in the **Online** state.

<h2 id="BKMK_RunCommand">Run a command across the cluster</h2>	
You can use the HPC Pack **clusrun** command to run a command or application on one or more cluster nodes. As a simple example, use **clusrun** get the IP configuration of the Azure nodes.

1. On the head node, open a command prompt.

2. Type the following command:

	`clusrun /nodes:azurecn* ipconfig`

3. You will see output similar to the following.

	![Clusrun][clusrun1]

<h2 id="BKMK_RunJob">Run a test job</h2>

You can submit a test job that runs on the hybrid cluster. This example is a simple "parametric sweep" job (a type of intrinsically parallel computation) which runs subtasks that add an integer to itself by using the **set /a** command. All the nodes in the cluster contribute to finishing the subtasks for integers from 1 to 100. 

1. In HPC Cluster Manager, in **Job Management**, in the **Actions** pane, click **New Parametric Sweep Job**.

	![New Job][test_job1]

2. In the **New Parametric Sweep Job** dialog box, in **Command line**, type `set /a *+*` (overwriting the default command line that appears). Leave default values for the remaining settings, and then click **Submit** to submit the job.

	![Parametric Sweep][param_sweep1]

3. When the job is finished, double-click the **My Sweep Task** job.

4. Click **View Tasks**, and then click a subtask to view the calculated output of that subtask.

	![Task Results][view_job361]

5. To see which node performed the calculation for that subtask, click **Allocated Nodes**. (Your cluster might show a different node name.)

	![Task Results][view_job362]

<h2 id="BKMK_stop">Stop the Azure nodes</h2>

After you try out the cluster, you can use HPC Cluster Manager to stop the Azure nodes to avoid unnecessary charges to your account. This stops the cloud service and removes the Azure role instances. 

1. In HPC Cluster Manager, in **Node Management,** select both Azure nodes. Then, in the **Actions** pane, click **Stop**. 

	![Stop Nodes][stop_node1]

2. In the **Stop Azure Nodes** dialog box, click **Stop**.

	![Stop Nodes][stop_node2]

3. The nodes transition to the **Stopping** state. After a few minutes, HPC Cluster Manager shows that the nodes are **Not-Deployed**.

	![Not Deployed Nodes][stop_node4]

4. To confirm that the role instances are no longer running in Azure, in the [Management Portal](https://manage.windowsazure.com), click **Cloud Services**, click the name of your cloud service, and then click **Instances**. No instances will be deployed in the production environment.

	![No Instances][view_instances2]

	This completes the tutorial.

<h2 id="">Related resources</h2>

* [HPC Pack 2012 R2 and HPC Pack 2012](http://go.microsoft.com/fwlink/p/?LinkID=263697)
* [Burst to Azure with Microsoft HPC Pack](http://go.microsoft.com/fwlink/p/?LinkID=200493)
* [Deploying Applications to Azure Nodes](http://go.microsoft.com/fwlink/p/?LinkId=325317)
* [Microsoft HPC Pack in an Azure Virtual Machine](http://go.microsoft.com/fwlink/p/?linkid=330375)


[Overview]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/hybrid_cluster_overview.png
[install_hpc1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc1.png
[install_hpc2]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc2.png
[install_hpc3]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc3.png
[install_hpc4]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc4.png
[install_hpc6]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc6.png
[install_hpc7]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc7.png
[install_hpc10]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc10.png
[upload_cert1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/upload_cert1.png
[createstorage1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/createstorage1.png
[createservice1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/createservice1.png
[config_hpc2]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc2.png
[config_hpc3]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc3.png
[config_hpc6]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc6.png
[config_hpc8]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc8.png
[config_hpc10]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc10.png
[config_hpc12]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc12.png
[config_hpc13]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc13.png
[add_node1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node1.png
[add_node1_1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node1_1.png
[add_node2]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node2.png
[add_node3]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node3.png
[add_node4]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node4.png
[add_node5]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node5.png
[add_node6]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node6.png
[add_node7]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node7.png
[view_instances1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/view_instances1.png
[clusrun1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/clusrun1.png
[test_job1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/test_job1.png
[param_sweep1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/param_sweep1.png
[view_job361]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/view_job361.png
[view_job362]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/view_job362.png
[stop_node1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/stop_node1.png
[stop_node2]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/stop_node2.png
[stop_node4]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/stop_node4.png
[view_instances2]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/view_instances2.png
