# "Vestigial objects" one-pager

Parent epic: [Regularly find and delete vestigial objects in .NET Engineering Services Subscriptions](https://github.com/dotnet/arcade/issues/8814)

The goal of this effort is to develop a consistent, approachable method to understand the existing Azure inventory 

This effort must answer these specific questions:

- *What resources do we have now?* (Or, what resources exist that are not related to running our services.)
- *What resources have changed recently?* (Or, Something has broken, have there been any recent configuration changes?)

## Stakeholders

- Operations lead
- First Responder lead

It is expected the primary audience of this effort are First Responders, for service failure diagnosis, and Leadership, for cost analysis. 

## Risks

Risk is low.

All data already exists in Azure, and Azure itself provides almost all of the infrastructure necessary for use. The goals of this effort are supported and expected use of this data.

All Azure data stores involved are Kusto-like, which is a well-understood technology in dnceng.

This effort will also use Grafana and Power BI, which are well-established tools in dnceng.

## The Plan

Azure provides infrastructure and features we can leverage to achieve our goals. 

- [Azure Resource Graph](https://learn.microsoft.com/en-us/azure/governance/resource-graph/overview)
- [Event Logs](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log) and [Resource Logs](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/resource-logs)
- [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/overview) and Log Analytics
- [Resource Tags](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources)

The Resource Graph, Event Logs, and Resource Logs are stores of audit and activity data generated by Azure. In some cases, like the Resource Graph, this information is already generated and available directly for query. In other cases, like specific resource auditing, logs must be enabled by a resource owner. Generally, this uses Azure Monitor and is listed as the "Export Log" feature.

### Understanding current inventory

The Azure Resource Graph provides a Kusto-like query interface to all resources in a subscription. 

Azure allows "Tags", which are simply key-value pairs, to be arbitrarily associated with almost any resource. Internally, Microsoft already uses this feature to track assets and configuration across subscriptions (e.g., AzSecPack, NRMSException). "Inventory" is an explicit use case supported in Azure.

This effort will develop a set of standard Tags that will be applied to resources. These Tags will identify, at minimum, resources that are necessary for operation of our services. Application of tags should automated where possible. Resources deployed by ARM or as part of the weekly deployment, for example, will have their deployment process modified to attach these tags with each deployment.

The inverse is also interesting: It will highlight resources that are _not_ necessary for the operation of our services. These will be analyzed and their purpose understood, creating more operational Tags as needed. They may also be deleted to eliminate unnecessary spending.

### Understanding changes to inventory

While the Resource Graph provides a (mostly) static view of resources, the Event and Resource Logs provide a view of changes to a resource.

The specific Log information recorded varies from resource to resource, and a resource can generally be configured to log at different level of details. Coarse logging is enabled at the Subscription level. More detailed, resource-specific logging (for example, which specific secrets are accessed from a Key Vault) may be enabled at the resource level. 

This effort shall ensure that, at minimum, creation and deletion of Resources are logged. Additionally, an appropriate level of configuration change shall be captured. This should be at least include the existance of a change and who made that change, but may include more detailed as desired. We may adjust the level of detail over time, though likely any change will need to be decided deliberately (Changes and other detailed activity can be verbose and are generally partitioned in Resource configurations to manage bandwidth use).

Records between Resource Graph and Event Log may be joined using a correlation ID provided for this purpose.

Once inventory Tags are stable, this data will make plain any new resources created, when they were created and who created them.

## Tasks

- [ ] Define tags schema
  - [ ] Indicating that an asset is an operational asset
  - [ ] Consider deployment information
- [ ] Modify service deployments to publish inventory tags with deployed resources
- [ ] Manually add inventory tags to resources not automatically deployed
- [ ] Configure subscriptions to export to a designated Log Analytics Workspace
  - [ ] Understand implications of Azure datacenter location and ingestion costs
  - [ ] Evaluate the possibility of using deployments to enforce audit configuration
  - [ ] Evaluate including Key Vault auditing information
- [ ] Develop Dashboards presenting data to expected audience. Consider Grafana and Power BI (or a mix of both).
  - [ ] FR can quickly view changes to an arbitrary resource
  - [ ] New resources or resources with an unknown purpose are identifiable from a dashboard
- [ ] Develop documentation 
  - [ ] On use, targeting expected audience. Where to find information. How to interpret dashboards. Basic, pertinent information on how to interpret the raw Azure data. How to develop and run custom audit-like queries.
  - [ ] For new services, to ensure new resoures are properly inventoried and audited

## Ideas discarded

Adopt Azure deployment technologies like ARM, Bicep.

Pros: 

- Supports configuration as code
- Explicit and absolute control over all entities and their configuration within a resource group
- Re-deployment overwrites any changes back to their expected state

Cons:

- Has no existing pattern within dnceng
- Would require significant effort initially establishing configuration
- Not be aligned with team's current operational goals

## Telemetry and Monitoring

This effort is itself telemetry and monitoring. Almost all data exists in and is managed by Azure. There is nothing intrinsic to monitor and thus requires no alerting.

## FR Handoff

FR is expected to be a critical user of audit information (specifcally, "recent resource changes"). Their acceptance of the final product is required.

## Appendix: Resources

[Azure resource inventory helps manage operational efficiency and compliance](https://www.microsoft.com/en-us/insidetrack/azure-resource-inventory-helps-manage-operational-efficiency-and-compliance)

[Azure security logging and auditing](https://docs.microsoft.com/en-us/azure/security/fundamentals/log-audit)

[Azure Activity Log event schema](https://docs.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log-schema)

[Use tags to organize your Azure resources and management hierarchy](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources)

[Explore your Azure resources with Resource Graph](https://learn.microsoft.com/en-us/azure/governance/resource-graph/concepts/explore-resources)

[Query changes made to resource properties](https://learn.microsoft.com/en-us/azure/governance/resource-graph/how-to/get-resource-changes)

[Azure Resource Graph table and resource type reference](https://learn.microsoft.com/en-us/azure/governance/resource-graph/reference/supported-tables-resources)

<!-- Begin Generated Content: Doc Feedback -->
<sub>Was this helpful? [![Yes](https://helix.dot.net/f/ip/5?p=Documentation%5CProject-Docs%5Cvestigial-objects-onepager.md)](https://helix.dot.net/f/p/5?p=Documentation%5CProject-Docs%5Cvestigial-objects-onepager.md) [![No](https://helix.dot.net/f/in)](https://helix.dot.net/f/n/5?p=Documentation%5CProject-Docs%5Cvestigial-objects-onepager.md)</sub>
<!-- End Generated Content-->