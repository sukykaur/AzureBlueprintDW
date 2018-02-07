# Azure Blueprint Automation: Data Warehouse for FedRAMP

## Overview

The Federal Risk and Authorization Management Program (FedRAMP), is a U.S. government-wide program that provides a standardized approach to security assessment, authorization, and continuous monitoring for cloud products and services. This Azure Blueprint Automation – Data Warehouse for FedRAMP provides guidance and automation scripts to deliver a Microsoft Azure data warehouse architecture that implements the FedRAMP High or Moderate set of controls depending on your compliance needs. This solution automates deployment and configuration of Azure resources for a common reference architecture, demonstrating ways in which customers can meet specific security and compliance requirements and serves as a foundation for customers to build and configure their own data warehouse solutions in Azure.

This architecture is intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment. Deploying an application into this environment without modification is not sufficient to completely meet the requirements of the FedRAMP High baseline. Please note the following:
- This architecture provides a baseline to help customers use Azure in a FedRAMP-compliant manner.
- Customers are responsible for conducting appropriate security and compliance assessment of any solution built using this architecture, as requirements may vary based on the specifics of each customer's implementation.

## Architecture Diagram and Components

This solution deploys a data warehouse reference architecture enabling you to quickly implement a high-performance and secure cloud data warehouse. There are two separate data tiers in this deployment: one where data is imported, stored, and staged within a clustered SQL environment, and another for the Azure SQL Data Warehouse where the data is loaded using an ETL tool (e.g. PolyBase T-SQL queries) for processing. Once data is stored in Azure SQL Data Warehouse, you can run analytics at massive scale. Data is stored in relational tables with columnar storage, and this format significantly reduces the data storage costs, and improves query performance. Compared to traditional database systems, analysis queries finish in seconds instead of minutes, or hours instead of days. Depending on your usage requirements, Azure SQL Data Warehouse compute resources can be scaled up or down, or shut off completely if there are no active processes needing compute resources.

A SQL load balancer is deployed to load balance and manage SQL traffic, ensuring high performance. Furthermore, all virtual machines in this reference architecture are deployed in an availability set, and SQL Server instances are configured in an AlwaysOn availability group for high-availability and disaster-recovery capabilities.

This data warehouse reference architecture also deploys an Active Directory tier for identity management. All virtual machines are domain-joined, and Active Directory group policies are used to enforce security and compliance configurations at the operating system level.

A virtual machine is deployed as a management jumpbox (bastion host) to provide a secure connection for administrators to access deployed resources. The data is loaded into the staging area through this management jumpbox. It is recommended that you configure a VPN or Azure ExpressRoute connection for management and data import into the reference architecture subnet.

[Diagram]

This solution uses the following Azure services. Details of the deployment architecture are in the Deployment Architecture section.

Azure Virtual Machines
-	(1) Jumpbox
-	(2) Active Directory domain controller
-	(2) SQL Server Cluster Node
-	(1) SQL Server Witness

Availability Sets
-	(1) Active Directory domain controllers
-	(1) SQL cluster nodes and witness

Virtual Network
-	(4) Subnets
-	(4) Network Security Groups

SQL Data Warehouse

Azure SQL Load Balancer

Recovery Services Vault

Azure Key Vault

Operations Management Suite (OMS)



### Deployment Architecture:

The following section details the development and implementation elements.

**SQL Data Warehouse**: SQL Data Warehouse is an Enterprise Data Warehouse (EDW) that leverages Massively Parallel Processing (MPP) to quickly run complex queries across petabytes of data. Import big data into SQL Data Warehouse with simple PolyBase T-SQL queries, and then use the power of MPP to run high-performance analytics.

**Jumpbox**: The jumpbox (bastion host) is the single point of entry that allows users to access the deployed resources in this environment. The jumpbox provides a secure connection to deployed resources by only allowing remote traffic from public IP addresses on a safe list. To permit remote desktop (RDP) traffic, the source of the traffic needs to be defined in the Network Security Group (NSG).

A virtual machine was created as a domain-joined jumpbox with the following configurations:
-	Antimalware extension
-	OMS extension
-	Azure Diagnostics extension
-	Azure Disk Encryption using Azure Key Vault (respects Azure Government, PCI DSS, HIPAA and other requirements).
-	An auto-shutdown policy to reduce consumption of virtual machine resources when not in use.
-	Windows Defender Credential Guard enabled so that credentials and other secrets run in a protected environment that is isolated from the running operating system.

#### Virtual Network
This deployment defines a private virtual network with an address space of 10.0.0.0/16.

**Network Security Groups**: NSGs contain Access Control Lists (ACLs) that allow or deny traffic within a VNet. NSGs can be used to secure traffic at a subnet or individual VM level. The following NSGs exist:
  -	An NSG for the Data Tier (SQL Server Clusters, SQL Server Witness, and SQL Load Balancer)
  -	An NSG for the SQL Data Warehouse
  -	An NSG for the management jumpbox (bastion host)
  -	An NSG for Active Directory

Each of the NSGs have specific ports and protocols opened for the secure and correct working of the solution. In addition, the following configurations are enabled for each NSG:
  -	Enabled diagnostic logs and events are stored in a storage account
  -	Connected OMS Log Analytics to the NSG's diagnostics

**Subnets**: Each subnet is associated with its corresponding NSG.

#### Data at rest
The architecture protects data at rest by using encryption, database auditing, and other measures.

**Azure Storage**
To meet encrypted data-at-rest requirements, all Azure Storage uses Storage Service Encryption.
Azure Disk Encryption
Azure Disk Encryption leverages the BitLocker feature of Windows to provide volume encryption for OS and data disks. The solution is integrated with Azure Key Vault to help control and manage the disk-encryption keys.

**Azure SQL Database**
The Azure SQL Database instance uses the following database security measures:
-	AD Authentication and Authorization allows you to centrally manage the identities of database users and other Microsoft services in one central location.
-	SQL database auditing tracks database events and writes them to an audit log in your Azure storage account.
-	SQL Database is configured to use Transparent Data Encryption (TDE) which performs real-time encryption and decryption of data and log files to protect information at rest. TDE provides assurance that stored data has not been subject to unauthorized access.
-	Firewall rules prevent all access to your database server until you specify which computers have permission. The firewall grants access to databases based on the originating IP address of each request.
-	SQL Threat Detection enables you to detect and respond to potential threats as they occur by providing security alerts upon suspicious database activities, potential vulnerabilities, and SQL injection attacks, as well as anomalous database access patterns.
-	Always Encrypted columns ensure that sensitive data never appears as plaintext inside the database system. After data encryption is enabled, only client applications or app servers that have access to the keys can access plaintext data.
-	SQL Database dynamic data masking, using the post-deployment PowerShell script. Note: Customers will need to adjust dynamic data masking settings to adhere to their database schema.

#### Business Continuity
High Availability: Server workloads are grouped in a Availability Set to help ensure high availability of virtual machines in Azure. This configuration helps ensure that during a planned or unplanned maintenance event at least one virtual machine will be available and meet the 99.95% Azure SLA.

Recovery Services Vault: The Recovery Services Vault houses backup data and protects all configurations of Azure Virtual Machines in this deployment. Using a Recovery Services Vault, you can restore files and folders from an IaaS VM without restoring the entire VM, which enables faster restore times.

#### Logging and Audit
Operations Management Suite (OMS) provides extensive logging of system and user activity as well as system health. Log Analytics is a service in Operations Management Suite (OMS) that enables collection and analysis of data generated by resources in Azure and on-premises environments.
- Activity Logs: Activity logs provide insight into the operations that were performed on resources in your subscription.
- Diagnostic Logs: Diagnostic logs are all logs emitted by every resource. These logs include Windows event system logs, Azure Blob storage, tables, and queue logs.
- Firewall Logs: The Application Gateway provides full diagnostic and access logs. Firewall logs are available for Application Gateway resources that have WAF enabled.
- Log Archiving: All diagnostic logs are configured to write to a centralized and encrypted Azure storage account for archival with a defined retention period (2 days). Logs are then connected to Azure Log Analytics for processing, storing, and dashboarding. Log Analytics is an OMS service that helps collect and analyze data generated by resources in your cloud and on-premises environments.
The following OMS solutions are installed as a part of this deployment:
-	AD Assessment: The Active Directory Health Check solution assesses the risk and health of your server environments on a regular interval, and provides you with a prioritized list of recommendations specific to your deployed server infrastructure.
-	Antimalware Assessment: The Antimalware solution reports on malware, threats, and protection status.
-	Azure Automation: The Azure Automation solution allows you to store, run, and manage runbooks.
-	Security and Audit: The Security and Audit dashboard provides a high-level insight into the security state of your resources by providing metrics on security domains, notable issues, detections, threat intelligence, and common security queries.
-	SQL Assessment: The SQL Health Check solution assesses the risk and health of your server environments on a regular interval, and provides you with a prioritized list of recommendations specific to your deployed server infrastructure.
-	Update Management: The Update Management solution allows you to manage operating system security updates, including a status of available updates, as well as managing the process of installing required updates.
-	Agent Health: The Agent Health solution reports how many agents are deployed and their geographic distribution, as well as the number of agents that are unresponsive and submitting operational data.
-	Azure Activity Logs: The Activity Log Analytics solution helps you analyze and search the Azure activity log across all your Azure subscriptions.
-	Change Tracking: The Change Tracking solution allows you to easily identify changes in your environment.

#### Identity Management
The following technologies provide identity management capabilities in the Azure environment:
-	Azure Active Directory (Azure AD) is the Microsoft's multi-tenant cloud-based directory and identity management service. All users for the solution were created in Azure Active Directory, including users accessing the SQL Database.
-	Authentication to the application is performed using Azure AD. For more information, see Integrating applications with Azure Active Directory. Additionally, the database column encryption also uses Azure AD to authenticate the application to Azure SQL Database. For more information, see Always Encrypted: Protect sensitive data in SQL Database.
-	Azure Active Directory Identity Protection detects potential vulnerabilities affecting your organization’s identities, configures automated responses to detected suspicious actions related to your organization’s identities, and investigates suspicious incidents and takes appropriate action to resolve them.
-	Azure Role-based Access Control (RBAC) enables precisely focused access management for Azure. Subscription access is limited to the subscription administrator, and Azure Key Vault access is restricted to all users.
To learn more about using the security features of Azure SQL Database, see the Contoso Clinic Demo Application sample.

#### Security
Secrets Management
The solution uses Azure Key Vault to manage keys and secrets. Azure Key Vault helps safeguard cryptographic keys and secrets used by cloud applications and services.

Malware Protection
Microsoft Antimalware for Virtual Machines provides real-time protection capability that helps identify and remove viruses, spyware, and other malicious software, with configurable alerts when known malicious or unwanted software attempts to install or run on protected virtual machines.

Patch Management
Windows virtual machines deployed by this Blueprint Automation are configured by default to receive automatic updates from Windows Update Service. This solution also deploys the OMS Azure Automation solution through which update deployments can be created to deploy patches to Windows servers when needed.


## Guidance and Recommendations
ExpressRoute or a secure VPN tunnel will need to be configured to securely establish a connection to the resources deployed as a part of this data warehouse reference architecture. As ExpressRoute connections do not go over the Internet, these connections offer more reliability, faster speeds, lower latencies, and higher security than typical connections over the Internet. By appropriately setting up an ExpressRoute or a VPN, you can add a layer of protection for data in transit.

The Azure Commercial cloud maintains a FedRAMP JAB P-ATO at the Moderate Impact Level. This instance of Microsoft Azure offers a wide variety of services to assist with formatted and unformatted data storage and staging for use in data warehousing, including:
-	Azure Data Factory is a managed cloud service that's built for complex hybrid extract-transform-load (ETL), extract-load-transform (ELT), and data integration projects. Using Azure Data Factory, you can create and schedule data-driven workflows (called pipelines) that can ingest data from disparate data stores. You can then process and transform the data for output into data stores such as Azure SQL Data Warehouse.
-	Azure Data Lake Store enables you to capture data of any size, type, and ingestion speed in one single place for operational and exploratory analytics. Azure Data Lake Store is compatible with most open source components in the Hadoop ecosystem. It also integrates nicely with other Azure services such as Azure SQL Data Warehouse.

## Threat Model

## Compliance Documentation
Customer Responsibility Matrix
The Customer Responsibility Matrix (Excel Workbook) lists all security controls required by the FedRAMP High baseline. The matrix denotes, for each control (or control subpart), whether implementation responsibly for the control is the responsibility of Microsoft, the customer, or shared between the two.

Control Implementation Matrix
The Control Implementation Matrix (Excel Workbook) lists all security controls required by the FedRAMP High baseline. The matrix denotes, for each control (or control subpart) that is designated a customer-responsibly in the customer responsibilities matrix, 1) if the Blueprint Automation implements the control, and 2) a description of how the implementation aligns with the control requirement(s). This content is also available here.

## Deploy the Solution
Coming soon.
## Disclaimer

 - This document is for informational purposes only. MICROSOFT MAKES NO WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, AS TO THE INFORMATION IN THIS DOCUMENT. This document is provided "as-is." Information and views expressed in this document, including URL and other Internet website references, may change without notice. Customers reading this document bear the risk of using it.
 - This document does not provide customers with any legal rights to any intellectual property in any Microsoft product or solutions.
 - Customers may copy and use this document for internal reference purposes.
 - Certain recommendations in this document may result in increased data, network, or compute resource usage in Azure, and may increase a customer's Azure license or subscription costs.
 - This architecture is intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment.
 - This document is developed as a reference and should not be used to define all means by which a customer can meet specific compliance requirements and regulations. Customers should seek legal support from their organization on approved customer implementations.
