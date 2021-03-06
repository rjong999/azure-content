<properties
	pageTitle="Connect Windows computers to Log Analytics | Microsoft Azure"
	description="This article covers the steps needed for you to connect the Windows computers in your on-premises infrastructure directly to OMS by using a customized version of the Microsoft Monitoring Agent (MMA)."
	services="log-analytics"
	documentationCenter=""
	authors="bandersmsft"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="log-analytics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/28/2016"
	ms.author="banders"/>


# Connect Windows computers to Log Analytics

This article covers the steps needed for you to connect the Windows computers in your on-premises infrastructure directly to OMS by using a customized version of the Microsoft Monitoring Agent (MMA). You need to install and connect agents for all of the computers that you want to on board to OMS in order for them to send data to OMS and to view and act on that data in the OMS portal.

You can install agents using Setup, command line, or with Desired State Configuration (DSC) in Azure Automation.  

On computers with Internet connectivity, the agent will use the connection to the Internet to send data to OMS. For computers that do not have Internet connectivity, you can use a proxy or the OMS Log Analytics Forwarder.

Connecting your Windows computers to OMS is straightforward using 3 simple steps:

1. Download the agent setup file
2. Install the agent using the method you choose
3. Configure the agent, if necessary

The following diagram shows the relationship between your Windows computers and OMS after you’ve installed and configured agents.

![oms-direct-agent-diagram](./media/log-analytics-windows-agents/oms-direct-agent-diagram.png)


## System requirements and required configuration
Before you install or deploy agents, review the following details to ensure you meet necessary requirements.

- You can only install the OMS MMA on computers running Windows Server 2008 SP 1 or later or Windows 7 SP1 or later.
- You'll need an OMS subscription.  For additional information, see [Get started with Log Analytics](log-analytics-get-started.md).
- Each Windows computer must be able to connect to the Internet. This connection can be via a proxy or through the  OMS Log Analytics Forwarder.
- You can install the OMS MMA on stand-alone computers, servers, and virtual machines. If you want to connect Azure-hosted virtual machines to OMS, see [Connect Azure storage to Log Analytics](log-analytics-azure-storage.md).
- The agent needs to use TCP port 443 for various resources. For more information, see [Configure proxy and firewall settings in Log Analytics](log-analytics-proxy-firewall.md).

## Download the agent setup file from OMS
1. In the OMS portal, on the **Overview** page, click the **Settings** tile.  Click the **Connected Sources** tab at the top.  
    ![Connected Sources tab](./media/log-analytics-windows-agents/oms-direct-agent-connected-sources.png)
2. Under **Attach Computers Directly**, click **Download Windows Agent** applicable to your computer processor type to download the setup file.
3. On the right of  **Workspace ID**, click the copy icon and paste the ID into Notepad.
4. On the right of  **Primary Key**, click the copy icon and paste the ID into Notepad.     
    ![copy Workspace ID and Primary Key](./media/log-analytics-windows-agents/oms-direct-agent-primary-key.png)

## Install the agent using setup
1. Run Setup to install the agent on a computer that you want to manage.
2. On the Welcome page, click **Next**.
3. On the License Terms page, read the license and then click **I Agree**.
4. On the Destination Folder page, change or keep the default installation folder and then click **Next**.
5. On the Agent Setup Options page, you can choose to connect the agent to Operational Insights (OMS), Operations Manager, or you can leave the choices blank if you want to configure the agent later. Click **Next**.   
    - If you chose to connect to Operational Insights (OMS), paste the **Workspace ID** and **Workspace Key (Primary Key)** that you copied into Notepad in the previous procedure and then click **Next**.  
        ![paste Workspace ID and Primary Key](./media/log-analytics-windows-agents/oms-mma-aoi-setup.png)
    - If you chose to connect to Operations Manager, type the **Management Group Name**, **Management Server** name, and **Management Server Port**, and then click **Next**. On the Agent Action Account page, choose either the Local System account or a local domain account and then click **Next**.  
        ![management group configuration](./media/log-analytics-windows-agents/oms-mma-om-setup01.png)![agent action account](./media/log-analytics-windows-agents/oms-mma-om-setup02.png)

6. On the Ready to Install page, review your choices and then click **Install**.
7. On the Configuration completed successfully page, click **Finish**.
8. When complete, the **Microsoft Monitoring Agent** appears in **Control Panel**. You can review your configuration there and verify that the agent is connected to Operational Insights (OMS). When connected to OMS, the agent displays a message stating: **The Microsoft Monitoring Agent has successfully connected to the Azure Operational Insights (OMS) service.**

## Install the agent using the command line
- Modify and then use the following example to install the agent using the command line.

    ```
    MMASetup-AMD64.exe /C:"setup.exe /qn ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_ID=<your workspace id> OPINSIGHTS_WORKSPACE_KEY=<your workspace key> AcceptEndUserLicenseAgreement=1"
    ```

## Install the agent using DSC in Azure Automation
1. Import the Azure Automation DSC module by installing the xPSDesiredStateConfiguration Azure PowerShell DSC Module from [http://www.powershellgallery.com/packages/xPSDesiredStateConfiguration](http://www.powershellgallery.com/packages/xPSDesiredStateConfiguration) into Azure Automation.  

2.	Create Azure Automation variables for *OPSINSIGHTS_WS_ID* and *OPSINSIGHTS_WS_KEY*. Set *OPSINSIGHTS_WS_ID* to your OMS Log Analytics workspace ID and set *OPSINSIGHTS_WS_KEY* to the primary key of your workspace.
3.	Use the script below and save it as DSC-HRfrontend.PS1
4.	Modify and then use the following example to install the agent using DSC in Azure Automation. Import  DSC-HRfrontend.PS1 into Azure Automation by using the Azure Automation interface or cmdlet.
5.	Assign a node to the configuration. Within 15 minutes the node will check its configuration and the MMA will be pushed to the node.

```
Configuration HRfrontend
{
    $OIPackageLocalPath = "C:\MMASetup-AMD64.exe"
    $OPSINSIGHTS_WS_ID = Get-AutomationVariable -Name "OPSINSIGHTS_WS_ID"
    $OPSINSIGHTS_WS_KEY = Get-AutomationVariable -Name "OPSINSIGHTS_WS_KEY"


    Import-DscResource -ModuleName xPSDesiredStateConfiguration

    Node HRfrontend {
        Service OIService
        {
            Name = "HealthService"
            State = "Running"
        }

        xRemoteFile OIPackage {
            Uri = "https://opsinsight.blob.core.windows.net/publicfiles/MMASetup-AMD64.exe"
            DestinationPath = $OIPackageLocalPath
        }

        Package OI {
            Ensure = "Present"
            Path  = $OIPackageLocalPath
            Name = "Microsoft Monitoring Agent"
            ProductId = "E854571C-3C01-4128-99B8-52512F44E5E9"
            Arguments = '/C:"setup.exe /qn ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_ID=' + $OPSINSIGHTS_WS_ID + ' OPINSIGHTS_WORKSPACE_KEY=' + $OPSINSIGHTS_WS_KEY + ' AcceptEndUserLicenseAgreement=1"'
            DependsOn = "[xRemoteFile]OIPackage"
        }
    }
}  


```


## Configure an agent manually
If you've installed agents but did not configure them or if you've made a configuration change, you can use the following information to enable or reconfigure them. After you've configured the agent, it will register with the agent service and will get necessary configuration information and management packs that contain solution information.

1. After you've installed the Microsoft Monitoring Agent, open **Control Panel**.
2. Open **Microsoft Monitoring Agent** and then click **Connect to Azure Operational Insights** (OMS).   
3. Paste the **Workspace ID** and **Workspace Key (Primary Key)** that you copied into Notepad in a previous procedure and then click **OK**.  
    ![configure Operational Insights](./media/log-analytics-windows-agents/oms-mma-aoi.png)

After data is collected from computers monitored by the agent, the number of computers monitored by OMS will appear in the OMS portal on the **Connected Sources** tab in **Settings** as **Servers Connected**.


### To disable an agent
1. After installing the agent, open **Control Panel**.
2. Open Microsoft Monitoring Agent and then click the **Azure Operational Insights (OMS)** tab.
3. Clear **Connect to Azure Operational Insights**.

## Configure an agent using the command line

- You can use Windows PowerShell with the following example.
    ```
    $healthServiceSettings = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
    $healthServiceSettings.EnableAzureOperationalInsights('workspacename', 'workspacekey')
    $healthServiceSettings.ReloadConfiguration()
    ```

## Optionally, configure agents to report to an Operations Manager management group

If you use Operations Manager in your IT infrastructure, you can also use the MMA agent as an Operations Manager agent.

### To configure MMA agents to report to an Operations Manager management group
1.	On the computer where the agent is installed, open **Control Panel**.
2.	Open **Microsoft Monitoring Agent** and then click the **Operations Manager** tab.
    ![Microsoft Monitoring Agent Operations Manager tab](./media/log-analytics-windows-agents/oms-mma-om01.png)
3.	If your Operations Manager servers have integration with Active Directory, click Automatically update management group assignments from AD DS.
4.	Click **Add** to open the **Add a Management Group** dialog box.  
    ![Microsoft Monitoring Agent Add a Management Group](./media/log-analytics-windows-agents/oms-mma-om02.png)
5.	In **Management group name** box, type the name of your management group.
6.	In the **Primary management server** box, type the computer name of the primary management server.
7.	In the **Management server port** box, type the TCP port number.
8.	Under **Agent Action Account**, choose either the Local System account or a local domain account.
9.	Click **OK** to close the **Add a Management Group** dialog box and then click **OK** to close the **Microsoft Monitoring Agent Properties** dialog box.

## Optionally, configure agents to use the OMS Log Analytics Forwarder

If you have servers or clients that do not have a connection to the Internet, you can still have them send data to OMS by using the OMS Log Analytics Forwarder.  When you use the forwarder, all data from agents is sent through a single server that has access to the Internet. The Forwarder transfers data from the agents to OMS directly without analyzing any of the data that is transferred.

See [OMS Log Analytics Forwarder](https://blogs.technet.microsoft.com/msoms/2016/03/17/oms-log-analytics-forwarder) to learn more about the forwarder, including set up and configuration.

For information about how to configure your agents to use a proxy server, which in this case is the OMS Forwarder, see [Configure proxy and firewall settings in Log Analytics](log-analytics-proxy-firewall.md).

## Optionally, configure proxy and firewall settings
If you have proxy servers or firewalls in your environment that restrict access to the Internet, see [Configure proxy and firewall settings in Log Analytics](log-analytics-proxy-firewall.md) to enable your agents to communicate to the OMS service.

## Next steps

- [Add Log Analytics solutions from the Solutions Gallery](log-analytics-add-solutions.md) to add functionality and gather data.
- [Configure proxy and firewall settings in Log Analytics](log-analytics-proxy-firewall.md) if your organization uses a proxy server or firewall so that agents can communicate with the Log Analytics service.
