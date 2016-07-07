<properties
	pageTitle=""
    description=""
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>

> [AZURE.LANGUAGE]
- [中文](/support/trust-center/security/)
- [English](/support/trust-center/security-en/)

# Security
 
* Design and operational security
* Security Controls and Capabilities
* Identity and access management
* Encryption
* Network security
* Threat management
* Monitoring, logging and reporting
* Penetration testing
 
 <tags ms.service="trust-center-en" ms.date="12/2015" wacn.date="12/2015" wacn.lang="en"/>
 
Microsoft Azure operated by 21Vianet runs in data centers located exclusively in mainland China. They are managed, monitored, and administered by 21Vianet operations staff that have years of experience in delivering secure online services with 24 x 7 continuity.

Security is built into Azure from the ground up, starting with the Secure Development Lifecycle, a mandatory development process that embeds security requirements into every phase of the development process. We help ensure that the Azure infrastructure is resilient to attack by mandating that our operational activities follow the rigorous security guidelines laid out in the security management process.

## Design and operational security

**Security Development Lifecycle (SDL).** The Azure technology licensed from Microsoft uses Security Development Lifecycle - a comprehensive approach for writing more secure, reliable and privacy-enhanced code. The Security Development Lifecycle (SDL) is a Microsoft software development process that aims to reduce the number and severity of vulnerabilities in software. Taking a holistic and practical approach since 2004, the SDL embeds security requirements into the entire lifecycle, from planning through deployment. After a product has shipped, the focus shifts to vulnerabilities. Teams respond to any incidents, and their findings inform the SDL to help improve future products. As technology evolves and criminals become more sophisticated, so does the SDL.

**Security Management.** The Security Management program provides an operational security baseline across all major cloud services, including Microsoft Azure operated by 21Vianet, helping ensure key risks are consistently mitigated.

**Assume Breach.** Specialized teams of our security engineers use pioneering security practices and operate with an "assume breach" mindset to identify potential vulnerabilities and proactively eliminate threats before they become risks to customers.

**Incident Response.** We operate a 24x7 event and incident response team to help mitigate threats from attacks and malicious activity. Our incident response team is on constant alert to identify, investigate, and resolve security incidents and vulnerabilities in the security of software. The detection of a security event mobilizes engineers and a communications team; while the engineering team conducts a detailed investigation of the issue and develops a solution, it works with the communications team who develops guidance for customers.

## Security Controls and Capabilities

Azure delivers a trusted foundation on which customers can design, build and manage their own secure cloud applications and infrastructure.

**Patching.** Integrated deployment systems manage the distribution and installation of security patches. Customers can apply similar patch management processes for Virtual Machines deployed in Azure.

**Zero standing privileges.** Access to customer data by our operations and support personnel is denied by default. When granted, access is carefully managed and logged. Data center access to the systems that store customer data is strictly controlled via lock box processes.

## Identity and access

Azure Active Directory enables customers to manage access to Azure, Office 365 and a world of other cloud apps. Azure Active Directory is a comprehensive identity and access management cloud solution that helps secure access to your data and on-premises and cloud applications, and helps simplify the management of users and groups. It combines core directory services, advanced identity governance, security and application access management. Azure Active Directory also makes it easy for developers to build policy-based identity management into their applications.

[Learn more](https://www.azure.cn/home/features/identity/)

Azure multi-factor authentication requires the use of more than one verification method to authenticate a user. Azure helps safeguard user access to data and applications with this extra layer of authentication for both on-premises and cloud applications. It delivers strong authentication with a range of easy verification options while meeting user demand for a simple sign-in process.

## Encryption

For data in transit, Azure uses industry-standard transport protocols between user devices and 21Vianet data centers, and within data centers themselves. You can enable encryption for traffic between your own virtual machines (VMs) and end users. With virtual networks, you can use industry-standard IPsec protocol to encrypt traffic between your corporate VPN gateway and Azure.

**Encrypted communications.** Built-in SSL and TLS cryptography enables customers to encrypt communications within and between deployments, from Azure to on-premises datacenters, and from Azure to administrators and users.

**Data encryption.** For data at rest, Azure offers a wide range of encryption capabilities up to AES-256.

## Network security

Azure networking provides the infrastructure necessary to securely connect VMs to one another and to connect on-site data centers with Azure VMs. Azure blocks unauthorized traffic to and within 21Vianet data centers, using a variety of technologies such as firewalls, partitioned local area networks (LANs) and the physical separation of backend servers from public-facing interfaces.

Azure Virtual Network extends your on-premises network to the cloud through site-to-site VPN, much like the way you'd set up and connect to a remote branch office. You control the network topology, including configuration of DNS and IP address ranges, and manage it just like your on-site infrastructure.

**Isolation.** Azure uses network isolation to prevent unwanted communications between deployments, and access controls block unauthorized users. Virtual Machines do not receive inbound traffic from the Internet unless customers configure them to do so.

**Azure Virtual Networks.** Customers can choose to assign multiple deployments to an isolated Virtual Network and allow those deployments to communicate with each other through private IP addresses.

[Learn more](/home/features/networking/)
[Learn Azure Network Security](https://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/documents/AzureNetworkSecurity_v3_Feb2015_CN_20151214.pdf)

## Threat management

To protect against online threats, Azure offers Antimalware for cloud services and virtual machines, and uses detection and mitigation techniques to protect against DDoS attacks.

**Intrusion detection and DDoS.** Intrusion detection and prevention systems, denial of service attack prevention, regular penetration testing, and forensic tools help identify and mitigate threats from both outside and inside of Azure.

**Antimalware for Azure with real-time protection and remediation.** Antimalware is a real-time protection capability for cloud services and virtual machines that helps identify and remove viruses, spyware, and other malware. It uses configurable alerts when malicious software attempts to install itself or run on Azure systems. When malware is detected, Antimalware automatically deletes or quarantines malicious files and cleans up malicious registry entries.

**Distributed denial-of-service (DDoS) defenses** are part of Azure's continuous monitoring and regular penetration testing to improve Azure's security controls and processes. Azure's DDoS defense system is not only designed to withstand attacks from the outside, but from other Azure tenants as well. Azure uses standard intrusion detection and mitigation techniques such as SYN cookies, rate limiting, and connection limits to protect against these attacks.

## Monitoring, logging and reporting

To help you manage the large amount of information generated by devices within the Azure environment, Azure offers centralized monitoring and analysis systems that provide continuous visibility and timely alerts to the teams who manage the service.

**24 hour monitored physical security.** Datacenters are physically constructed, managed, and monitored to shelter data and services from unauthorized access as well as environmental threats.

**Monitoring and logging.** Security is monitored with the aid of centralized monitoring, correlation, and analysis systems that manage the large amount of information generated by devices within the environment and providing timely alerts. In addition, multiple levels of monitoring, logging, and reporting are available to provide visibility to customers.

**Create an audit trail.** For applications that are deployed in Azure and virtual machines created from the Azure Virtual Machines Gallery, Azure enables a set of operating system security events by default. You can add, remove, or modify events to be audited by customizing the audit policy. In addition to generating Windows event logs, you can configure operating system components to generate logs that are important for security analysis and monitoring.

**Perform centralized analysis of large data sets.** Azure enables you to collect security events from Azure IaaS and PaaS. You can then use HDInsight to aggregate and analyze these events, and export them to on-premises security information and event management systems for ongoing monitoring.

[Learn more](/home/features/hdinsight/)

**Monitor access and usage reporting.** Azure logs administrative operations, including system access, to create an audit trail in case unauthorized or accidental changes are made. You can retrieve audit logs for your tenant and view access and usage reports. This will help you gain visibility into the integrity and security of your Azure Active Directory tenant, and help you better determine where possible security risks may lie. In the full Azure Management Portal, reports include anomalous sign-in events, user-specific reports, and activity reports.

**Export security alerts to on-premises SIEM.** Azure provides Azure Diagnostics (WAD) that can be configured to collect Windows security event logs and other security-specific logs. You can also export this data into a third-party, on-premises Security Information and Event Management (IEM) system for analysis and alerting.

[Learn more](/documentation/articles/cloud-services-dotnet-diagnostics/)

## Security Incident and Abuse Reporting

To report suspected security issues or abuse of Azure, please contact Azure Customer Support.

## Penetration testing

We conduct regular penetration testing to improve Azure security controls and processes. We understand that security assessment is also an important part of our customers' application development and deployment. Therefore, we have established a policy for customers to carry out authorized penetration testing on their applications hosted in Azure. Because such testing can be indistinguishable from a real attack, it is critical that customers conduct penetration testing only after obtaining approval in advance from Azure Customer Support. Penetration testing must be conducted in accordance with our terms and conditions. Requests for penetration testing should be submitted with a minimum of 7-day advanced notice.

To learn more or to initiate penetration testing, please download the [Penetration Testing Approval Form](https://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/documents/Penetration_Test_Questionnaire.docx), and then contact Azure Customer Support.
