<properties linkid="manage-services-hdinsight-version" urlDisplayName="HDInsight Hadoop Version" pageTitle="What's new in the cluster versions provided by HDInsight? | Azure" metaKeywords="hdinsight, hadoop, hdinsight hadoop, hadoop azure" description="HDInsight supports multiple Hadoop cluster versions deployable at any time. See the Hadoop and HortonWorks Data Platform (HDP) distribution versions supported." services="HDInsight" umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" title="What's new in the cluster versions provided by HDInsight?" authors="bradsev" />


#What's new in the cluster versions provided by HDInsight?

Azure HDInsight now supports Hadoop 2.2 with HDInsight cluster version 3.0 and takes full advantage of these platform to provide a range of significant benefits to customers. These include, most notably:

- **Microsoft Avro Library**: This library implements the Apache Avro data serialization system for the Microsoft.NET environment. Apache Avro provides a compact binary data interchange format for serialization. It uses JSON to define language agnostic schema that underwrites language interoperability. Data serialized in one language can be read in another. Currently C, C++, C#, Java, PHP, Python, and Ruby are supported. The Apache Avro serialization format is widely used in Azure HDInsight to represent complex data structures within a Hadoop MapReduce job.

- **YARN**: A new, general-purpose, distributed, application management framework that has replaced the classic Apache Hadoop MapReduce framework for processing data in Hadoop clusters. It effectively serves as the Hadoop operating system, and takes Hadoop from a single-use data platform for batch processing to a multi-use platform that enables batch, interactive, online and stream processing. This new management framework improves scalability and cluster utilization according to criteria such as capacity guarantees, fairness, and service-level agreements.

- **High Availability**: A second headnode has been added to the Hadoop clusters deployed by HDInsight to increase the availability of the service. Standard implementations of Hadoop clusters typically have a single headnode. HDInsight removes this single point of failure with the addition of a secondary headnode. The switch to new HA cluster configuration doesn’t change the price of the cluster, unless customer provision clusters with extra large head node.

- **Hive performance**: Order of magnitude improvements to Hive query response times (up to 40x) and to data compression (up to 80%) using the **Optimized Row Columnar** (ORC) format.

- **Pig, Sqoop, Qozie, Ambari**: Component version upgrades for HDInsight cluster version 3.0 ((HDP 2.0/Hadoop 2.2) that provide parity with HDInsight cluster version 2.1 (HDP 1.3/Hadoop 1.2). See the version tables below for specifics. Note that HBase, Mahout, Flume are not included.


**Deployment**	
Creation of HDInsight 3.0 clusters on Hadoop 2.2 is supported by the Azure Portal, the HDInsight SDK, and by Azure PowerShell. Note the HDInsight 2.1 clusters are created by default on Hadoop 1.2, so the users must specify HDInsight the 3.0 cluster version to create an Hadoop 2.2 cluster.

**Global Availability**		
With the release of Azure HDInsight on Hadoop 2.2, Microsoft has made HDInsight available in all major Azure geographies with the exception of Greater China. Specifically, west Europe and southeast Asia data centers have been brought online. This enables customers to locate clusters in a data center that is close and potentially in a zone of similar compliance requirements. 

**Breaking Changes**	
Only the "wasb://" syntax is supported in HDInsight 3.0 clusters. The older "asv://" syntax is supported in HDInsight 2.1 and 1.6 clusters, but it is not supported in HDInsight 3.0 clusters and it will not be supported in later versions. This means that any jobs submitted to an HDInsight 3.0 cluster that explicitly use the “asv://” syntax will fail. The wasb:// syntax should be used instead. Also, jobs submitted to any HDInsight 3.0 clusters that are created with an existing metastore that contains explicit references to resources using the asv:// syntax will fail. These metastores will need to be recreated using the wasb:// to address resources.

##HDInsight versions
HDInsight supports multiple Hadoop cluster versions that can be deployed at any time. Each version choice provisions a specific version of the Hortonworks Data Platform (HDP) distribution and a set of components that are contained within that distribution. The component version associated with each HDInsight cluster version are itemized in the following table. Note that the default cluster version used by [Azure HDInsight](http://go.microsoft.com/fwlink/?LinkID=285601) is currently 2.1 based on HDP 1.3.


<table border="1">
<tr><th>Component</th><th>Version 3.0</th><th>Version 2.1 (Default)</th><th>Version 1.6</th></tr>
<tr><td>Hortonworks Data Platform (HDP)</td><td>2.2.</td><td>1.3</td><td>1.1</td></tr>
<tr><td>Apache Hadoop</td><td>2.2.0</td><td>1.2.0</td><td>1.0.3</td></tr>
<tr><td>Apache Hive</td><td>0.12.0</td><td>0.11.0</td><td>0.9.0</td></tr>
<tr><td>Apache Pig</td><td>0.12.0</td><td>0.11.0</td><td>0.9.3</td></tr>
<tr><td>Apache Sqoop</td><td>1.4.4</td><td>1.4.3</td><td>1.4.2</td></tr>
<tr><td>Apache Oozie</td><td>4.0.0</td><td>3.2.2</td><td>3.2.0</td></tr>
<tr><td>Apache HCatalog</td><td>Merged with Hive</td><td>Merged with Hive</td><td>0.4.1</td></tr>
<tr><td>Apache Templeton</td><td>Merged with Hive</td><td>Merged with Hive</td><td>0.1.4</td></tr>
<tr><td>Ambari</td><td>1.4.1</td><td>API v1.0</td><td>no version
</td></tr>
</table>


### Select a version when provisioning an HDInsight cluster

When creating a cluster through the HDInsight PowerShell Cmdlets or the HDInsight .NET SDK, you can choose the version for the HDInsight Hadoop cluster using the "Version" parameter.

If you use the **Quick Create** option, you will get the version 2.1 of HDInsight Hadoop cluster by default. If you use the **Custom Create** option from the Azure Portal, you can choose the version of the cluster you will deploy from the **HDInsight Version** drop-down on the **Cluster Details** page. Version 3.0 of HDInsight Hadoop cluster is only available as an option on the **Custom Create** wizard.

![HDI.Versioning.VersionScreen][image-hdi-versioning-versionscreen]


## Supported versions
The following table lists the versions of HDInsight currently available, the corresponding Hortonworks Data Platform (HDP) versions that they use, and their release dates. When known, their deprecation dates will also be provided. Highly available clusters with two head nodes are deployed by default for HDInsight 2.1, and 3.0 clusters. They are not available for HDInsight 1.6 clusters.

<table border="1">
<tr><th>HDInsight Version</th><th>HDP Version</a><th>High Availability</th></th><th>Release Date</th><th>Support Expiration Date</th><th>Deprecation Date</th></tr>
<tr><td>HDI 3.0</td><td>HDP 2.0</td><td>Yes</td><td>02/11/2014</td><td>08/11/2014</td><td></td></tr>
<tr><td>HDI 2.1</td><td>HDP 1.3</td><td>Yes</td><td>10/28/2013</td><td>05/12/2014</td><td>05/01/2015</td></tr>
<tr><td>HDI 1.6</td><td>HDP 1.1</td><td>No</td><td>10/28/2013</td><td>04/28/2014</td><td>05/01/2014</td></tr>
</table><br/>


### The Service-Level Agreement (SLA) for HDInsight cluster versions 

The SLA is defined in terms of a "Support Window". A Support Window refers to the period of time that an HDInsight cluster version is supported by Microsoft Customer Support.  An HDInsight cluster is outside the Support Window if its version has a **Support Expiration Date** past the current date.  A list of supported HDInsight cluster versions may be found in the table above.  The Support Expiration Date for a given HDInsight version (denoted as version X) is calculated as the later of:  

- Formula 1:  Add 180 days to the date HDInsight cluster version X was released
- Formula 2: Add 90 days to the date HDInsight cluster version X+1 (the subsequent version after X) is made available in the Azure Management Portal.

The **Deprecation Date** is the date after which the cluster version can no be created on HDInsight.

> [WACOM.NOTE] Both HDInsight 2.1 and 3.0 cluster run on Azure Guest OS [Family 4](http://msdn.microsoft.com/en-us/library/azure/ee924680.aspx#explanation) which uses the 64-bit version of Windows Server 2012 R2 and supports .NET Framework 4.0, 4.5. and 4.5.1. 

### Additional notes and information on versioning	

* The SQL Server JDBC Driver is used internally by HDInsight and is not used for external operations. If you wish to connect to HDInsight using ODBC, please use the Microsoft Hive ODBC driver. For more information on using Hive ODBC, [Connect Excel to HDInsight with the Microsoft Hive ODBC Driver][connect-excel-with-hive-ODBC].

* The ports used by the HDInsight service have been changed. The port numbers which were being used were within the Windows OS ephemeral port range. Ports are allocated automatically from a predefined ephemeral range for short-lived internet protocol-based communications. The new set of allowed HDP service port numbers are now outside of this range to avoid encountering conflicts that could arise with the ports used by services running on the headnode. The new port numbers should not cause any breaking changes. The numbers used now are as follows:


 **HDP1.1**
<table border="1">
<tr><th>Name</th><th>Value</th></tr>
<tr><td>dfs.http.address</td><td>namenodehost:30070</td></tr>
<tr><td>dfs.datanode.address</td><td>0.0.0.0:30010</td></tr>
<tr><td>dfs.datanode.http.address</td><td>0.0.0.0:30075</td></tr>
<tr><td>dfs.datanode.ipc.address</td><td>0.0.0.0:30020</td></tr>
<tr><td>dfs.secondary.http.address</td><td>0.0.0.0:30090</td></tr>
<tr><td>mapred.job.tracker.http.address</td><td>jobtrackerhost:30030</td></tr>
<tr><td>mapred.task.tracker.http.address</td><td>0.0.0.0:30060</td></tr>
<tr><td>mapreduce.history.server.http.address</td><td>0.0.0.0:31111</td></tr>
<tr><td>templeton.port</td><td>30111</td></tr>
</table>


 **HDP2.0 and 2.1**
<table border="1">
<tr><th>Name</th><th>Value</th></tr>
<tr><td>dfs.namenode.http-address</td><td>namenodehost:30070</td></tr>
<tr><td>dfs.namenode.https-address</td><td>headnodehost:30470</td></tr>
<tr><td>dfs.datanode.address</td><td>0.0.0.0:30010</td></tr>
<tr><td>dfs.datanode.http.address</td><td>0.0.0.0:30075</td></tr>
<tr><td>dfs.datanode.ipc.address</td><td>0.0.0.0:30020</td></tr>
<tr><td>dfs.namenode.secondary.http-address</td><td>0.0.0.0:30090</td></tr>
<tr><td>yarn.nodemanager.webapp.address</td><td>0.0.0.0:30060</td></tr>
<tr><td>templeton.port</td><td>30111</td></tr>
</table>



* HDInsight cluster version 3.0 uses an Hadoop distribution that is based on the [Hortonworks Data Platform 2.0][hdp-2-0-8].

* HDInsight cluster version 2.1 uses an Hadoop distribution that is based on the [Hortonworks Data Platform 1.3][hdp-1-3-0]. This is the default Hadoop cluster created when using the Azure HDInsight portal.

* HDInsight cluster version 1.6 uses an Hadoop distribution that is based on the [Hortonworks Data Platform 1.1][hdp-1-1-0]. 

* The component versions associated with HDInsight cluster versions may change in future updates to HDInsight. One way to determine the available components and to verify which versions are being used for a cluster to use the Ambari REST API. The GetComponentInformation command can be used to retrieve information about a service component. For details, see the [Ambari documentation][ambari-docs]. Another way to obtain this information is to login to a cluster using remote desktop and examine the contents of the "C:\apps\dist\" directory directly.

[image-hdi-versioning-versionscreen]: ./media/hdinsight-component-versioning/hdi-versioning-version-screen.png

[wa-forums]: http://www.windowsazure.com/en-us/support/forums/

[connect-excel-with-hive-ODBC]: /en-us/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver

[hdp-2-0-8]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html

[hdp-1-3-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html

[hdp-1-1-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-Win-1.1/bk_releasenotes_HDP-Win/content/ch_relnotes-hdp-win-1.1.0_1.html

[ambari-docs]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md

[zookeeper]: http://zookeeper.apache.org/ 
