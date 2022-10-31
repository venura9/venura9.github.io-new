---
title: What's up Azure? (July 2020)
key: 20200723
author: Venura Athukorala
tags: azure
---

With all the partners heading to Microsoft Inspire (Virtual), the last 30 days have been a bit quiet with the extravagant updates, which is most likely to be released in the next 30 days. 

As usual the updates were pleanty, with key updates for this month skewed towards;

* Azure Storage 
* Azure Arc
* Azure Functions
* KeyVaults
* Data Lake 

Let's dig in, What's Up Azure?

### [Customer-initiated Storage account failover is now generally available](https://azure.microsoft.com/en-us/updates/azure-storage-account-failover-ga/)

* For GRS and RA-GRS Storage you can now trigger the failover manually without Microsoft having to detect an issue with the primary region. 
At the point of writing this post the feature is broken and Australia doesn't yet support it. 

![Storage Account in Australia](https://whatcloud.xyz/assets/current-manual-storage-failover-status.png "Storage Account in Australia")

* [You can read about how the failover is triggered](https://docs.microsoft.com/en-au/azure/storage/common/storage-initiate-account-failover?tabs=azure-portal)

### [Public Preview for Azure Monitor for VMs on Arc Enabled Servers](https://azure.microsoft.com/en-us/updates/public-preview-for-azure-monitor-for-vms-on-arc-enabled-servers/)

* Azure Arc allows you to manage your on-premises infrastucutre the same we as you would manage resource on Azure. This includes your Servers, Kubernetes Clusters and Azure Data Services Running on Arc. As announced at Inspire, Azure Arc is getting closer to GA. Arc will provide number of operational benefits to end-users and partners alike. 

* This feature addition effectively extends Azure Monitor for VMs to Azure Arc Instances. 

![Azure Arc](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/6673cca7-4ea2-43be-af88-11ffec6a6659.png)

[Learn more about Azure Arc](https://azure.microsoft.com/en-au/services/azure-arc/#features)

### [Durable Functions support for Python](https://azure.microsoft.com/en-us/updates/durable-functions-now-supports-python/)

Might not make everyone excited. But,I am.
Durable functions allow you to extend the capabilities of Azure Functions to new heights. 

- Function Chaining
- Fan out/fan in
- Async HTTP APIs
- Monitor
- Human interaction

E.g. Fan out/in

![Fan out/in](https://docs.microsoft.com/en-au/azure/azure-functions/durable/media/durable-functions-concepts/fan-out-fan-in.png)

```python
import azure.functions as func
import azure.durable_functions as df


def orchestrator_function(context: df.DurableOrchestrationContext):
    parallel_tasks = []

    # Get a list of N work items to process in parallel.
    work_batch = yield context.call_activity("F1", None)

    for i in range(0, len(work_batch)):
        parallel_tasks.append(context.call_activity("F2", work_batch[i]))
    
    outputs = yield context.task_all(parallel_tasks)

    # Aggregate all N outputs and send the result to F3.
    total = sum(outputs)
    yield context.call_activity("F3", total)


main = df.Orchestrator.create(orchestrator_function)
```
[Read More about Durable Functions](https://docs.microsoft.com/en-au/azure/azure-functions/durable/durable-functions-overview?tabs=python)

### [Azure Monitor for KeyVaults in Preview](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/key-vault-insights-overview)

New Section introduced in Azure Monitor. Also, avaialble under insights of individual instances. 
![Azure Monitor KeyVaults](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/media/key-vaults-insights-overview/overview.png)


### [Azure Data Lake Storage](https://azure.microsoft.com/en-au/updates/2020/06/?status=nowavailable,inpreview,indevelopment&query=data%20lake%20storage)

Few new additions to the capability set; 

#### Preview
- Static website
- File snapshots
- Immutable storage

#### GA 
- Archive tier


### [Azure Well Architected Framework](https://azure.microsoft.com/en-au/blog/introducing-the-microsoft-azure-wellarchitected-framework/)

- Rings a bell!? - yup yup >> https://aws.amazon.com/architecture/well-architected/

### Other

- [Azure Load Balancer - Backend Pools by IP](https://docs.microsoft.com/en-us/azure/load-balancer/backend-pool-management#configuring-backend-pool-by-ip-address-and-virtual-network)
- [Block Blob size increased to 200TB](https://azure.microsoft.com/en-us/updates/azure-storage-200-tb-block-blob-size-is-now-in-preview/)
- [Announcing the general availability of Azure shared disks and new Azure Disk Storage enhancements](https://azure.microsoft.com/en-au/blog/announcing-the-general-availability-of-azure-shared-disks-and-new-azure-disk-storage-enhancements/)
- [App Gateway - URL rewrite and wildcard host names in listener for Azure Application Gateway are now available in preview.](https://azure.microsoft.com/en-us/updates/url-rewrite-wildcard-listener-preview/)

### [Azure Pipelines Sprint 171](https://docs.microsoft.com/en-us/azure/devops/release-notes/2020/sprint-171-update)
- Pipeline resources get additional filter with the the support for 'Tags'
- The hosted agents how support Linux/ARM64
- The allowed list of tasks for pipelines now can be configured at: `https://dev.azure.com/<your_org>/_settings/pipelinessettings` adding more meat to the goverenece of Azure DevOps. 


### [Azure Pipelines Sprint 172](https://docs.microsoft.com/en-us/azure/devops/release-notes/2020/sprint-172-update)
- Exclusive deployment lock policy - only one pipeline can can deploy to an environment at a given point of time. 
- Pipeline resources get additional filter with the the support for 'Stages'
- Generic webhook based triggers for YAML pipelines - Trigger deployment based on external events via webhooks. Syntax now supports a resource called `webhooks` 
- YAML resource trigger issues support and traceability - You can now find out as to why a pipeline was not triggered. 
- Banner for live site incidents impacting pipelines - No comments!
