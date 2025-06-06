---
title: Chargeback Process Using Azure Cost Analysis
description: Learn how to use Microsoft Defender for Cloud's chargeback feature to allocate cloud expenses accurately across teams using Azure Cost Analysis.
author: dcurwin
contributors:
ms.topic: conceptual
ms.date: 03/17/2025
ms.author: dacurwin
---

# Microsoft Defender for Cloud chargeback process using Azure Cost Analysis

## Introduction

The chargeback feature in Defender for Cloud lets organizations allocate cloud expenses accurately across teams or departments. Defender for Cloud integrates with Azure Cost Analysis to analyze detailed billing data and calculate precise chargebacks. Implementing this approach fosters transparency, financial accountability, and efficient management of resources within your organization.

The chargeback process uses the [Azure Billing Tags](/azure/cost-management-billing/costs/billing-tags) infrastructure.

## How to distribute the cost for chargeback

There are two main ways to split shared costs. You can use the [Azure cost allocation rules](/azure/cost-management-billing/costs/allocate-costs). Alternatively, you can manually split costs using the customized or built-in views of the detailed billing report generated by [Azure Cost Analysis](/azure/cost-management-billing/costs/cost-analysis-built-in-views).

## Understanding the detailed billing report

At the core of the manual chargeback process is Azure Cost Analysis, which provides an extensive billing report. This report details each billed asset, including essential information required for precise cost distribution:

### Report details for the selected period

- **Asset Name:** Identifies each asset.
- **Asset Type:** Specifies the category of the asset.
- **Resource Group:** Lists the resource group containing the asset.
- **Location:** Indicates the geographic location of the asset.
- **Subscription:** Identifies the subscription associated with the asset.
- **Tags:** Includes metadata for quick identification and attribution of cost responsibility.
- **Cost:** Displays the total expense incurred by each asset for the reporting period.

## Importance of tags

Tags are critical for efficient cost allocation. They function as labels to link assets directly to responsible departments, projects, or individuals. Common tags include:

- **Department:** Identifies the department that uses the asset.
- **Project:** Connects the asset with a specific project.
- **Owner:** Pinpoints the individual or team responsible for asset usage.

> [!NOTE]
> Both Defender for Endpoint machine tags and cloud tags are used as billing tags. Therefore, you'll see both tag types listed as billing tags in the detailed reports.

## Manual chargeback calculation steps

Follow these steps to execute the chargeback process:

> [!NOTE]
> The following process can be conducted via the Azure portal user interface or programmatically through the [Azure Consumption API](/rest/api/consumption/usage-details/list).

1. **Review the report:** Thoroughly examine the detailed billing report to identify all assets and their associated costs.
1. **Identify responsible parties:** Use the provided tags to identify the department, project, or individual responsible for each asset.
1. **Allocate costs:** Assign expenses accurately to each relevant team or department based on tag data.
1. **Generate statements:** Create detailed chargeback statements for each department or team, clearly reflecting their respective costs.

## Example of chargeback implementation

Consider a scenario where your organization's enterprise subscription costs $20,000 monthly and includes billing for all direct onboarded virtual machines (VMs). Your goal is to allocate charges per asset owner:

- Use the Azure Consumption API to obtain a detailed billing report.
- Programmatically filter assets based on the tag "owned_by:[group_name]."

For example, the detailed billing report for the selected period might look like this:

- Asset Name: VM-Prod-01
- Asset Type: Virtual Machine
- Resource Group: Production-Group
- Location: East US
- Subscription: Enterprise Subscription
- Tags: owned_by:Dev-Team-A
- Cost: $500 for the selected billing period

Use the Defender for Cloud chargeback process:

1. Retrieve and review this asset's billing details using the API.
1. Identify that Finance-Team-A owns this asset based on the tag.
1. Allocate the $500 cost to Finance-Team-A's budget.
1. Generate and send a chargeback report to the finance department detailing the allocated costs.

## Benefits of implementing chargeback

Employing the chargeback system in Defender for Cloud offers substantial benefits:

- **Enhanced accountability:** Clearly assigns responsibility for cloud usage, fostering a culture of financial accountability.
- **Improved budget management:** Allows for detailed expense tracking, facilitating accurate budgeting and forecasting.
- **Resource efficiency:** Encourages teams to optimize their resource use, effectively controlling and reducing costs.

## Conclusion

The Defender for Cloud chargeback process provides a structured, equitable way to distribute cloud expenses across an organization. Using detailed billing reports and strategic tagging helps achieve transparency, accountability, and improved budget management.
