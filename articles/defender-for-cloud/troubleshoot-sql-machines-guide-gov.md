---
title: Troubleshoot Defender for SQL on Machines deployment government
description: Troubleshoot deployment issues for SQL Servers on machines using the Azure Monitoring Agent (AMA) autoprovisioning process.
ms.date: 04/23/2025
ms.topic: how-to
ms.custom: references_regions
author: ElazarK
ms.author: elkrieger
#customer intent: As a security professional, I want to ensure that my deployment of SQL servers on machines is correct and protects my resources.
---

# Troubleshoot machine protection in Defender for SQL on Machines government

> [!IMPORTANT]
> This article applies to government clouds. If you're using commercial clouds, see the [Troubleshoot Defender for SQL on Machines deployment](troubleshoot-sql-machines-guide.md) article.

If you enable Defender for SQL Server on Machines and some SQL instances aren't in a protected state, use this article to troubleshoot deployment issues.

Before starting the troubleshooting steps, ensure you have:
- Followed the steps to [enable Defender for SQL on Machines](defender-for-sql-usage-gov.md).
- Reviewed the [protection status of databases running on protected machines](verify-machine-protection-gov.md).

## Step 1: Understand how resources are created

Defender for SQL Servers on Machines automatically creates resources as shown in the diagram.

:::image type="content" source="media/troubleshoot-sql-machines-guide-gov/resource-level.png" alt-text="Diagram that shows resources and the levels that they're created on." lightbox="media/troubleshoot-sql-machines-guide-gov/resource-level.png":::

Resources are summarized in the table:

| Resource type | Level created |
|--|--|
| **Resource group** - Created in East US Azure region | Subscription level |
| **Managed identity** - A user-assigned managed identity is created in each Azure region | Subscription level |
| **Log Analytics workspace** - Use the default or a custom workspace. | Subscription level |
| **Data collection rule (DCR)** - Created for each workspace | Subscription level |
| **Data collection rule association (DCRA)** | Defined on each SQL Server instance |
| **Azure Monitoring Agent (AMA)**  | The extension is installed on each SQL Server instance |
| **Defender for SQL extension**  | The extension is installed on each SQL Server instance |

## Step 2: Make sure extensions are allowed

To ensure protection works as expected, make sure your organizational deny policy doesn't block these extensions: 

- Defender for SQL (IaaS and Arc)
    - Publisher: Microsoft.Azure.AzureDefenderForSQL
    - Type: AdvancedThreatProtection.Windows 
- SQL IaaS Extension (IaaS) 
    - Publisher: Microsoft.SqlServer.Management
    -  Type: SqlIaaSAgent 
- SQL IaaS Extension (Arc)
    - Publisher: Microsoft.AzureData
    - Type: WindowsAgent.SqlServer
- AMA extension (IaaS and Arc)
    - Publisher: Microsoft.Azure.Monitor
    - Type: AzureMonitorWindowsAgent 

## Step 3: Ensure East US region is allowed

When you enable the plan, Azure creates a resource group in the East US region. Ensure no deny policies block this region.

## Step 4: Verify resource naming conventions

Defender for SQL Server on Machines uses specific naming conventions for resources. Ensure your organization doesn't block these naming conventions and don't modify any automatically created resources.

- DCR: `MicrosoftDefenderForSQL--dcr` 
- DCRA: `/Microsoft.Insights/MicrosoftDefenderForSQL-RulesAssociation` 
- Resource group: `DefaultResourceGroup-` 
- Log Analytics workspace: `D4SQL--`

Defender for SQL uses *MicrosoftDefenderForSQL* as a *createdBy* database tag. 

## Step 5: Identify misconfigurations at subscription level

To identify which subscriptions have misconfigurations, use the [SQL Servers on Machines AMA Helper workbook](https://ms.portal.azure.com/#view/AppInsightsExtension/UsageNotebookBlade/ComponentId/Azure%20Security%20Center/ConfigurationId/community-Workbooks%2FAzure%20Security%20Center%2FSQL%20Servers%20on%20Machines%20AMA%20Helper/WorkbookTemplateName/SQL%20Servers%20on%20Machines%20AMA%20Helper).

1. Open the [SQL Servers on Machines AMA Helper workbook](https://ms.portal.azure.com/#view/AppInsightsExtension/UsageNotebookBlade/ComponentId/Azure%20Security%20Center/ConfigurationId/community-Workbooks%2FAzure%20Security%20Center%2FSQL%20Servers%20on%20Machines%20AMA%20Helper/WorkbookTemplateName/SQL%20Servers%20on%20Machines%20AMA%20Helper).

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/ama-helper-workbook.png" alt-text="Screenshot of the SQL Servers on Machines AMA Helper workbook main page." lightbox="media/troubleshoot-sql-machines-guide-gov/ama-helper-workbook.png":::

1. In **Subscriptions Overview** review misconfigurations at subscription level. 

    - **SQL Servers on Azure Virtual Machines** - shows subscriptions that contain Azure VMs.
    - **Arc-Enabled SQL Servers** - shows subscriptions that contain Azure Arc-enabled VMs.

    Subscriptions appear on these tabs in accordance with your specific environment.

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/navigate-sections.png" alt-text="Screenshot that shows where to navigate to on the SQL Servers on Azure Virtual Machines workbook page." lightbox="media/troubleshoot-sql-machines-guide-gov/navigate-sections.png":::
    
1. Review component configurations for each subscription.

    - The number of SQL Server instances in the subscription.
    - Instances with the Defender for SQL extension installed.
    - Instances with the AMA extension installed.
    - DCRs created for each workspace in the subscription (across all regions).
    - DCRAs created for each SQL instance.
    - Managed identity created for each region at the subscription level.
    - Log Analytics workspace created for each region at the subscription level.
    - AMA autoprovisioning enabled for the subscription.
    - Defender for SQL enabled for the subscription.

1. For each subscription, check which component doesn't match the expected configuration, such as 0/1, 10/15, or No. In our example screenshot, the Demo subscription has misconfigurations in DCRA 0/1. 

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/ama-helper-workbook-results.png" alt-text="Screenshot of the SQL Servers on Machines AMA Helper workbook results." lightbox="media/troubleshoot-sql-machines-guide-gov/ama-helper-workbook-results.png":::

After locating a subscription with misconfigurations, resolve the misconfigurations first at the subscription level, then at the resource level and extension installation level.

## Step 6: Resolve misconfigurations at the subscription level

After identifying misconfigurations, start by fixing DCR issues, then workspace issues, and finally identity issues at the subscription level.

Fix misconfigurations in the correct order. DCR resolution relies on workspace resolution, and workspace resolution relies on identity resolution. If you try to resolve these misconfigurations out of order, they aren't resolved.


1. Navigate to **Policy** > **Compliance**.

1. Select **Scope**.
    
    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/scope.png" alt-text="Screenshot that shows where to select scope on the policy and compliance page." lightbox="media/troubleshoot-sql-machines-guide-gov/scope.png":::

1. In Scope, select the relevant subscription.

1. On the Compliance page, select the policy in accordance with your workspace configuration:
    - Default workspace: **Defender for SQL on SQL VMs and Arc-enabled SQL Servers**.
    - Custom workspace: **Defender for SQL on SQL VMs and Arc-enabled SQL Servers-custom**.

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/select-misconfiguration.png" alt-text="Screenshot that shows where the misconfiguration is found on the page." lightbox="media/troubleshoot-sql-machines-guide-gov/select-misconfiguration.png":::

1. Search for and resolve each noncompliant issue in this order **Identity** > **Workspace** > **DCR**.

1. Fix each issue as follows:
    - **Identity** - `Create and assign a built-in user-assigned managed identity`.
    - **Workspace** - `Configure the Microsoft Defender for SQL Log Analytics workspace`.
    - **DCR** - `Configure SQL Virtual Machines to automatically install Microsoft Defender for SQL and DCR with a Log Analytics workspace` or `Configure Arc-enabled SQL Servers to automatically install Microsoft Defender for SQL and DCR with a Log Analytics workspace`.

1. For each policy that is noncompliant, review the compliance reason and select **Create remediation task** to resolve it.
    
    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/remediation-task.png" alt-text="Screenshot that shows where to create a remediation task on the page." lightbox="media/troubleshoot-sql-machines-guide-gov/remediation-task.png":::

1. Fill in the relevant information.

1. Select **Remediate**.

1. Repeat these steps for each noncompliant policy and subscription.

### Input custom values with PowerShell deployment script

If you couldn't resolve subscription issues with the workbook, Defender for SQL Servers on Machines provides a PowerShell deployment script that enables you to input your own values for workspace, DCR, and user Identity. To use the PowerShell script, follow the [instructions on this page](enable-defender-sql-at-scale.md).

## Step 7: Resolve misconfigurations at the resource level

After resolving misconfigurations at the subscription level, resolve misconfigurations at the resource level, including DCRA misconfigurations and incomplete AMA or Defender for SQL extension deployment.

### Troubleshoot extension misconfigurations

1. In the Azure portal, navigate to **Policy** > **Compliance**.

1. Select **Scope**.

1. In the dropdown, select the subscription with misconfigurations.

1. Search for and select **Defender for SQL on SQL VMs and Arc-enabled SQL Servers initiative**.

1. Select the noncompliant policy name.

    - **Defender for SQL extension** - `Create and assign a built-in user-assigned managed identity`. 
    - **AMA extension**: - `Configure SQL Virtual Machines to automatically install Azure Monitor Agent` or `Configure Arc-enabled SQL Servers to automatically install Azure Monitor Agent`.

1. For each policy that is noncompliant, review the compliance reason and select **Create remediation task** to resolve it.

### Troubleshoot DCRA misconfigurations

1. In the Azure portal, Search for and select **[Data collection rules](https://ms.portal.azure.com/#browse/microsoft.insights%2Fdatacollectionrules)**.

1. Select **Subscription equals** > select the relevant subscription.

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/subscription-equals.png" alt-text="Screenshot that shows where to select subscription equals." lightbox="media/troubleshoot-sql-machines-guide-gov/subscription-equals.png":::

1. Select **Apply**.

1. Locate and select the relevant DCR. The DCR naming convention follows this format: `MicrosoftDefenderForSQL-region-dcr`.

1. Select **Configuration** > **Resources**.

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/resources.png" alt-text="Screenshot that shows you where to select configuration and resources." lightbox="media/troubleshoot-sql-machines-guide-gov/resources.png":::

1. Select **+ Add**.

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/add.png" alt-text="Screenshot that shows where the add button is located." lightbox="media/troubleshoot-sql-machines-guide-gov/add.png":::

1. In the **Resource types** dropdown table, select **Machines - Azure Arc** and **Virtual machines** in accordance with your deployment.

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/resource-type-filter.png" alt-text="Screenshot that shows where to filter by Machines Azure Arc and Virtual machines." lightbox="media/troubleshoot-sql-machines-guide-gov/resource-type-filter.png":::

1. Expand each resource group and select each machine.

    :::image type="content" source="media/troubleshoot-sql-machines-guide-gov/select-machines.png" alt-text="Screenshot that shows each machine selected individually." lightbox="media/troubleshoot-sql-machines-guide-gov/select-machines.png":::

1. Select **Apply**.

## Step 8: Reverify protection status

After completing all the steps on this page, [reverify the protection status of each SQL Server instance](verify-machine-protection-gov.md).
