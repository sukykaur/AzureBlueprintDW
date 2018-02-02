# Azure Blueprint Automation: Data Warehouse for FedRAMP

## Overview

 This article provides guidance and automation scripts to deliver a Microsoft Azure data warehouse architecture that implements a subset of controls from the FedRAMP High baseline, based on NIST SP 800-53. This solution automates deployment and configuration of Azure resources for a common reference architecture, demonstrating ways in which customers can meet specific security and compliance requirements and serves as a foundation for customers to build and configure their own solutions on Azure. The solution 

## Architecture Diagram and Components


### Deployment Architecture:

**Jumpbox**: Also called a [bastion host](https://en.wikipedia.org/wiki/Bastion_host), which is a secure VM on the network that administrators use to connect to VMs in the production VNet. The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list. To permit remote desktop (RDP) traffic, the source of the traffic needs to be defined in the NSG. Management of production resources is via RDP using a secured Jumpbox VM.

**Network Security Groups**: [NSGs](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) contain Access Control Lists that allow or deny traffic within a VNet. NSGs can be used to secure traffic at a subnet or individual VM level.

## Guidance and Recommendations

### Business Continuity

### Logging and Audit

### Identity

### Security

## Compliance Documentation

## Deploy the Solution

## Disclaimer

 - This document is for informational purposes only. MICROSOFT MAKES NO WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, AS TO THE INFORMATION IN THIS DOCUMENT. This document is provided "as-is." Information and views expressed in this document, including URL and other Internet website references, may change without notice. Customers reading this document bear the risk of using it.
 - This document does not provide customers with any legal rights to any intellectual property in any Microsoft product or solutions.
 - Customers may copy and use this document for internal reference purposes.
 - Certain recommendations in this document may result in increased data, network, or compute resource usage in Azure, and may increase a customer's Azure license or subscription costs.
 - This architecture is intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment.
 - This document is developed as a reference and should not be used to define all means by which a customer can meet specific compliance requirements and regulations. Customers should seek legal support from their organization on approved customer implementations.
