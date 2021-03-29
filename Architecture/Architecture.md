# Architecture

This section describes the architecture and operation of the VMC software which consists of four main modules: Knowledge Collector, Asset Collector, Vulnerability Collector and Processing Module. Figure below shows the flow of information. The first three modules are responsible for collecting data from various parts of the network, while the fourth, computational one, is responsible for delivering results as close to real time as possible.

![VMC Architecture](./arch-vmc.png)

All modules work independently of each other, communicating asynchronously via a queue system. Thanks to this, depending on the amount of data received by the module, it can be scaled vertically, independently of other components.

The system is configured from the administrator panel. The order and timing of data download from individual sources are controlled from the Scheduler module, while the Task Monitor allows you to view the current system status.

The entire solution has been prepared for operations in the cloud environment and is based on the Docker container technology. The data is stored in ElasticSearch, which enables its processing in full-text mode, while the Kibana tool is used for the results' presentation layer. The Python language was used to implement the components described below due to its flexibility, and the possibility of efficient data processing on the server side. The entire project includes support for tenants and allows for full data separation between specific tenants. VMC operates in operational and historical modes. The operating mode allows the user to view all changes observed by the system in real time, while the historical mode enables viewing and performing computational operations on historical data, thus improving the quality and scope of the analysis of events in the system. 

In order to simplify the principle of operation and the VMC architecture, the data processing process has been divided into individual stages following one another.

## Knowledge Collector
The Knowledge Collector module is responsible for downloading, updating and normalizing information from publicly available databases on known exploits, weaknesses (CWE - Common Weakness Enumeration) and vulnerabilities (CVE - Common Vulnerabilities and Exposures). Information sources include the National Vulnerabilities Database (NVD) and the Exploits Database. Below is the structure of the document responsible for storing information on the url of a published exploit, the structure responsible for storing information about the software (CPE - Common Platform Enumeration), and a standardized structure for the vulnerability, which also contains relations to the above-described documents. The document stores the entire vector that makes up the vulnerability assessment, in order to speed up the calculations and enable easier searching for the vulnerability by its characteristic features, e.g. the possibility of remote exploitation of the vulnerability. After passing the normalization process, the collected information can be displayed in the presentation layer.

```python
class ExploitDocument:
    id = Keyword()
    url = Keyword()

```

In the document representing the collected information about publicly available Exploits, the following fields are distinguished.

|Name| Description                                                         |
|----|---------------------------------------------------------------------|
|id  |exploit identifier based on address from https://www.exploit-db.com/ |
|url |URL to a page describing the identified exploit                      |

```python
class CpeDocument:
    name = Keyword()
    vendor = Keyword()
```

In the document representing the collected information about the software, based on the CPE, we distinguish the following fields.

|Name   | Description                                                      |
|-------|------------------------------------------------------------------|
|name   |software name based on CPE                                        |
|vendor |the name of the software manufacturer based on CPE                |

```python
class CveDocument:
    id = Keyword()
    base_score_v2 = Float()
    base_score_v3 = Float()
    summary = Keyword()
    access_vector_v2 = TupleValueField(choice_type=metrics.AccessVectorV2)
    access_complexity_v2 = TupleValueField(choice_type=metrics.AccessComplexityV2)
    authentication_v2 = TupleValueField(choice_type=metrics.AuthenticationV2)
    confidentiality_impact_v2 = TupleValueField(choice_type=metrics.ImpactV2)
    integrity_impact_v2 = TupleValueField(choice_type=metrics.ImpactV2)
    availability_impact_v2 = TupleValueField(choice_type=metrics.ImpactV2)
    attack_vector_v3 = TupleValueField(choice_type=metrics.AttackVectorV3)
    attack_complexity_v3 = TupleValueField(choice_type=metrics.AttackComplexityV3)
    privileges_required_v3 = EnumField(choice_type=metrics.PrivilegesRequiredV3)
    user_interaction_v3 = TupleValueField(choice_type=metrics.UserInteractionV3)
    scope_v3 = EnumField(choice_type=metrics.ScopeV3)
    confidentiality_impact_v3 = TupleValueField(choice_type=metrics.ImpactV3)
    integrity_impact_v3 = TupleValueField(choice_type=metrics.ImpactV3)
    availability_impact_v3 = TupleValueField(choice_type=metrics.ImpactV3)
    published_date = Date()
    last_modified_date = Date()

    exploits = ExploitDoc()
    cpe = CpeDoc()
    cwe = CweDoc()
```
In the document representing the collected information on vulnerabilities (CVE), we distinguish the following fields.

|Name                      | Description                                                      |
|--------------------------|------------------------------------------------------------------|
|id	                       |unique CVE vulnerability identifier
|base_score_v2             |calculated vulnerability criticality assessment based on CVSS version 2
|base_score_v3             |the calculated vulnerability criticality assessment based on CVSS version 3
|summary                   |a detailed description of the vulnerability found
|access_vector_v2          |access vector for vulnerability exploitation, based on CVSS version 2
|access_complexity_v2      |access complexity vector for vulnerability exploitation, based on CVSS version 2
|authentication_v2         |the authentication factor and the level of authorization needed to exploit the vulnerability, based on the CVSS version 2
|confidentiality_impact_v2 |the impact on the confidentiality of the system when a vulnerability is exploited, based on CVSS version 2
|integrity_impact_v2       |impact on system integrity when a vulnerability is exploited, based on CVSS version 2
|availability_impact_v2    |impact on the system availability in case of using a vulnerability, based on CVSS version 2
|attack_vector_v3          |access vector for vulnerability exploitation, based on CVSS version 3
|attack_complexity_v3      |access complexity vector for vulnerability exploitation, based on CVSS version 3
|privileges_required_v3    |the authentication factor and the level of privileges needed to exploit the vulnerability, based on the CVSS version 3
|user_interaction_v3       |user interaction factor for vulnerability exploitation, based on CVSS version 3
|scope_v3                  |the factor of the impact of using the vulnerability on other components not directly targeted by the attack
|confidentiality_impact_v3 |the impact on the confidentiality of the system when a vulnerability is exploited, based on CVSS version 3
|integrity_impact_v3       |impact on system integrity when a vulnerability is exploited, based on CVSS version 3
|availability_impact_v3    |impact on system availability in case of using a vulnerability, based on CVSS version 3
|published_date            |vulnerability publication date
|last_modified_date        |date of the last vulnerability modification

![Kibana Sample](./kibana_sample.png)

## Asset Collector
The Asset Collector module is responsible for managing data on detected assets and assets defined for the monitored network. It has two data sources. The first source is the Ralph tool, a system that meets the requirements of corporate networks allowing for the management of the life cycle of a particular component of the environment. The second source of data is a vulnerability scanner which, in addition to scanning, has the functionality of detecting IT infrastructure components. In this way, VMC is able to inform the operator about the discrepancy between the data contained in the asset database and the information provided with the scan results. Listing below shows how the asset is represented in the database. The confidentiality requirement, integrity requirement, availability requirement fields allow you to calculate an environmental risk assessment for individual assets. The business owner and technical owner fields are used to assign specific people or organizational units responsible for particular assets, both from a technical and business perspective. If any of the above-mentioned values is missing, an alarm will be generated in the system, informing the operator that the data is inconsistent.

```python
class AssetDocument:
    ip_address = Keyword()
    mac_address = Keyword()
    os = Keyword()
    confidentiality_requirement = TupleValueField(choice_type=Impact)
    integrity_requirement = TupleValueField(choice_type=Impact)
    availability_requirement = TupleValueField(choice_type=Impact)
    business_owner = Nested(OwnerInnerDoc, include_in_parent=True)
    technical_owner = Nested(OwnerInnerDoc, include_in_parent=True)
    service = Keyword()
    environment = Keyword()
    hostname = Keyword()
    tenant = Keyword()
    tags = Keyword()
    url = Keyword()
    source = Keyword()
   last_scan_date = Date()
```

In the document representing the collected information about the asset, we distinguish the following fields.

|Name                        | Description                                                      |
|----------------------------|------------------------------------------------------------------|
|ip_address	                 |IP address of the asset
|mac_address                 |MAC address of the asset
|os                          |operating system version
|confidentiality_requirement |criticality factor of the confidentiality of the asset and the data processed
|integrity_requirement       |criticality factor for the integrity of the asset and the processed data
|availability_requirement    |availability criticality factor of the asset and the processed data
|business_owner              |the persons or units responsible for the asset from the business point of view
|technical_owner             |persons or units responsible for the asset from a technological point of view
|service                     |type of asset
|environment                 |the nave of the environment in which the asset is located
|hostname                    |name identifying the asset
|tenant                      |the name of the tenant
|tags                        |tag specifying the state of the machine (DISCOVERED - asset detected during vulnerability scanning, DELETED - asset has been removed from the asset management database)
|url                         |URL for the defined asset in the IT asset management database
|source                      |the name of the configuration the data comes from
|last_scan_date              |last scan date

## Vulnerability Collector
The Vulnerability Collector module is responsible for downloading, updating and normalizing data from Nessus and OpenVas vulnerability scanners. In order to optimize the results, during data processing, vulnerabilities classified as informational are ignored. Listing below shows the structure of the document of the detected vulnerability, after being processed by the module. This structure includes information such as the port number, protocol, service name, vulnerability description and information on how the vulnerability should be fixed. This information is a recommendation for system owners and other people implementing the corrective mechanisms.
```python
class VulnerabilityDocument:
    port = Integer()
    svc_name = Keyword()
    protocol = Keyword()
    name = Keyword()
    description = Keyword()
    solution = Keyword()
    environmental_score_v2 = Float()
    environmental_score_vector_v2 = Keyword()
    environmental_score_v3 = Float()
    environmental_score_vector_v3 =  Keyword()
    cve = CveInnerDoc()
    asset = AssetInnerDoc()
    tags = ListField()
    source = Keyword()
    tenant = Keyword()
    scan_file_url = Keyword()
    scan_date = Date()
```

In the document describing the vulnerability in the system found, we distinguish the following fields.

|Name                         | Description                                                      |
|-----------------------------|------------------------------------------------------------------|
|port                         |port of the vulnerable service
|svc_name                     |the name of the vulnerable service
|protocol                     |the transport layer protocol on which the service is listening
|name                         |vulnerability short name
|description                  |a detailed description of the vulnerability found
|solution                     |recommended method of mitigation
|environmental_score_v2       |environmental valuation factor for the vulnerability detected, based on CVSS version 2, in numerical notation
|environmental_score_vector_v2|environmental valuation factor for the vulnerability detected, based on CVSS version 2, in vector notation
|environmental_score_v3       |the environmental valuation factor for the vulnerability detected, based on CVSS version 3, in numerical notation
|environmental_score_vector_v3|environmental valuation factor for the vulnerability detected, based on CVSS version 3, in vector notation
|cve                          |assigned vulnerability described in the document representing the collected information on vulnerabilities (CVE)
|asset                        |an asset that is described in a document representing collected information about the asset
|tags                         |list of tags describing the vulnerability status. If the vulnerability is fixed, VMC assigns the FIXED tag
|source                       |the name of the configuration the data comes from
|tenant                       |the name of the tenant
|scan_file_url                |url to the zip file containing the downloaded report file
|scan_date                    |date of scanning, detection of the vulnerability

## Processing Module
Processing Module is responsible for making calculations and updating the assessment of detected vulnerabilities, taking into account data obtained from previous modules. For each new or updated vulnerability, the obtained calculation results are entered in the fields environmental score v2 for CVSS 2.0 and environmental score v3 for CVSS 3.0. Additionally, in the fields environmental score vector v2 and environmental score vector v3 there is a vector notation of the vulnerability assessment, thanks to which it is possible to easily analyze each of the individual values affecting the final environmental assessment.
