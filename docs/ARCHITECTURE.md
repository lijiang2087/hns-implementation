# Architecture

- [Architecture](#architecture)
  - [Overview](#overview)
  - [Requirements](#requirements)
    - [TLD management:](#tld-management)
  - [Components](#components)
    - [ENS Fronted](#ens-fronted)
    - [ENS Web3 Backend](#ens-web3-backend)
    - [Web 2 Registry](#web-2-registry)
    - [Web 2 Domain Name Server](#web-2-domain-name-server)
  - [Reviews](#reviews)
    - [Registry - Nomulus](#registry---nomulus)
    - [Name Servers - Criteria and CodeBases for Review](#name-servers---criteria-and-codebases-for-review)
    - [core-dns (go) - Ranking 1](#core-dns-go---ranking-1)
    - [trust-dns (rust) - Ranking 3](#trust-dns-rust---ranking-3)
    - [solana-dns (javascript) - Ranking 4](#solana-dns-javascript---ranking-4)
    - [project (language) - Ranking X](#project-language---ranking-x)
  - [Standards](#standards)
    - [Basic operations](#basic-operations)
    - [Update operations](#update-operations)
    - [Secure DNS operations](#secure-dns-operations)



## Overview


## Requirements

### TLD management: 

- This allows us to create a registry that manages the TLD, such that we can accept requests from registrars (user-facing businesses that let people look up available domain names and buy domains)
    - The requests from registrars are sent via [EPP commands](https://datatracker.ietf.org/doc/html/rfc3731). Nomulus has built-in tools to parse and process these commands.
- We will need to add web3 capabilities to the registry managed by Nomulus, so it reads and writes the blockchain when it is processing EPP requests (and other types of requests) from registrars.
- Since existing registrars are not taking the necessary web3 information from users (which at minimum, should include wallet address and a signature to authorize a request), we will also need to build a simple registrar to demonstrate how it should be done.
    - In the non-custodial case, the registrar can be a UI-only client. This should be what we build first
    - If the registrar wants to operates in a conventional way and takes the custody of the domains for users without web3 wallets, it will also need a server. We should build a simple example of that as well, so we can partner with traditional registrars


## Components

### ENS Fronted

Lets users register their domains. Currently Harmony has developed a demonstration application deployed at https://ens.demo.harmony.one/ using the codebase https://github.com/polymorpher/.1.country/tree/main/client. This has limited functionality where a site can be rented for 3 months, with the ability for others to surpass the rental by paying double the last rental amount. Initial rental amount is 100 ONE. This does not support Top Level Domains (TLDs) instead Tier 2 domains under `.1.country` are suppored e.g. `alice.1.country`.

*An indepth review of the exisisting ENS Application can be found in [HNS-Design](./HNS-DESIGN.md).*

### ENS Web3 Backend

Provides the smart contract functionality for registering and renting domains. Currently Harmony has developed a single smart contract [D1DC.sol](https://github.com/polymorpher/.1.country/blob/main/contracts/contracts/D1DC.sol). To provide this functionality.

*An indepth review of the exisisting ENS Smart Contracts can be found in [HNS-Design](./HNS-DESIGN.md).*


### Web 2 Registry

This provides functionality for registering web 2 domains. Capturing the domain owner, contact details and period of ownership. It does not manage the Domains DNS entries which is the responsiblity of the Domain Name Server. Currently Harmony uses [UNI NAMING AND REGISTRY](https://uniregistry.link/) (UNR). There has also been an initial Review of Open Source Code Registry [Nomulus by Google](https://github.com/google/nomulus) which can be found in [Review -> Registry](#registry).

### Web 2 Domain Name Server

This provides the managment of DNS entries, contact information and other technical and administrative information for a domain. Below in [Reviews -> Name Servers](#name-servers) are high level criteria for the functionality required to support web3 as well as open source code bases under evaluation.


## Reviews

### Registry - Nomulus
<details><summary><b> Nomulus Technical Review</b></summary>
    
- Actors
    - User (Alice) ⇒ Registers Tier 2 Domain alice.country
    - Registrar
        - [https://ens.demo.harmony.one/unsupportedNetwork](https://ens.demo.harmony.one/unsupportedNetwork) ([client](https://github.com/polymorpher/.1.country/tree/main/client))
        - Front End (Registrar) ⇒ GoDaddy, Google Domains
    - TLD Owner Manual tier2  `.1.country`
        - TLD ⇒ .country, .io etc
    - Web3 Backend([D1DC](https://github.com/polymorpher/.1.country/blob/main/contracts/contracts/D1DC.sol))
        - Review ENS Contracts (Optional)
             
            1. [DNS Registrar guide](https://docs.ens.domains/dns-registrar-guide)
            
            2. [dns-sec-oracle](https://github.com/ensdomains/ens-contracts/tree/master/contracts/dnssec-oracle) : [implementation](https://github.com/ensdomains/ens-contracts/blob/master/contracts/dnssec-oracle/DNSSECImpl.sol), [algorithms](https://github.com/ensdomains/ens-contracts/tree/master/contracts/dnssec-oracle/algorithms) and [digests](https://github.com/ensdomains/ens-contracts/tree/master/contracts/dnssec-oracle/digests)
                
            3.  [DNSRegistrar.sol](https://github.com/ensdomains/ens-contracts/blob/master/contracts/dnsregistrar/DNSRegistrar.sol)
                
    - Web2 Backend(Registry) ⇒ (Harmony Nomulus)
        - github: [Nomulus](https://github.com/google/nomulus)
        - Messaging: [EPP](https://datatracker.ietf.org/doc/html/rfc3731)
            - https://github.com/domainr/epp
            - https://github.com/hiqdev/heppy
            - https://github.com/NeustarRegistry/nsr-toolkit
- Scenarios
    - Scenario 1: Alice Registers [alice.country](http://alice.country) (non-custodial)
        1. Availability Check
        2. Price Calculation
        3. Signature Request
        4. Tier 2 Registration
    - Scenario 2: Alice Registers [alice.io](http://alice.io) (custodial)
        1. Registrar (Harmony Nomulus) integrates with (TLD .io owner)
            a. [see registrar-faq](https://github.com/google/nomulus/blob/master/docs/registrar-faq.md) [domain-create-flow](https://github.com/google/nomulus/blob/master/docs/flows.md#domaincreateflow)
        2. Registrar (Harmony Nomulus integrates with [ICANN](https://www.icann.org/)
            - see [task-queues](https://github.com/google/nomulus/blob/master/docs/architecture.md#task-queues)
                - `brda` -- Queue for tasks to upload weekly Bulk Registration Data Access (BRDA) files to a location where they are available to ICANN. The `RdeStagingReducer` (part of the RDE MapReduce) creates these tasks at the end of generating an RDE dump.
                - `rde-report` -- Queue for tasks to upload RDE reports to ICANN following successful upload of full RDE files to the escrow provider. Tasks are enqueued by `RdeUploadAction` and executed by `RdeReportAction`.
        3.. Registry (TLD .io owner) integrates with Registrar(Harmony Nomulus)
- Deployment Tasks
    - Local Development
        1.  [Ganche](https://www.npmjs.com/package/ganache)
        2.  Deploy [Contracts](https://github.com/polymorpher/.1.country/tree/main/contracts) Locally
        3.  Deploy [Client Frontend](https://github.com/polymorpher/.1.country/tree/main/client) Locally
        4.  Deploy [Nomulus](https://github.com/google/nomulus)
            1. [Installation](https://github.com/google/nomulus/blob/master/docs/install.md)
            2. [First Steps](https://github.com/google/nomulus/blob/master/docs/first-steps-tutorial.md)
    - Prod Deployment
        - [Architecture](https://github.com/google/nomulus/blob/master/docs/architecture.md)
        - [operational-procedures](https://github.com/google/nomulus/blob/master/docs/operational-procedures.md)
        - [proxy-setup](https://github.com/google/nomulus/blob/master/docs/proxy-setup.md) for [epp](https://www.rfc-editor.org/rfc/rfc5730.html) and [whois](https://www.rfc-editor.org/rfc/rfc3912) support
        - [Configuration](https://github.com/google/nomulus/blob/master/docs/configuration.md)
- Code Changes
    1. Contracts
        - Review ENS Contracts (Optional)
                
            1. [DNS Registrar guide](https://docs.ens.domains/dns-registrar-guide)
                
            2. [dns-sec-oracle](https://github.com/ensdomains/ens-contracts/tree/master/contracts/dnssec-oracle) : [implementation](https://github.com/ensdomains/ens-contracts/blob/master/contracts/dnssec-oracle/DNSSECImpl.sol), [algorithms](https://github.com/ensdomains/ens-contracts/tree/master/contracts/dnssec-oracle/algorithms) and [digests](https://github.com/ensdomains/ens-contracts/tree/master/contracts/dnssec-oracle/digests)
                
            3.  [DNSRegistrar.sol](https://github.com/ensdomains/ens-contracts/blob/master/contracts/dnsregistrar/DNSRegistrar.sol)
                
    2. Client (Registrar)
        1. Integrate with Nomulus for Availability Check
            1. [registrar-faq.md](https://github.com/google/nomulus/blob/master/docs/registrar-faq.md)
            2. [EPP Integration](https://datatracker.ietf.org/doc/html/rfc3731)
                1. [verisign epp sdks](https://www.verisign.com/en_US/channel-resources/domain-registry-products/epp-sdks/index.xhtml) (Java and c++)
                2. https://github.com/domainr/epp (go)
                3. https://github.com/hiqdev/heppy (python)
                4. [Other epp-clients on github](https://github.com/topics/epp-client)
            3. Capture signature before calling EPP Messages
            4. Enhance EPP Messages to capture signature and `ownerAddress`
                1. Review [domainCreateFlow](https://github.com/google/nomulus/blob/master/docs/flows.md#domaincreateflow)
                2. Review [EPP <create> Command](https://datatracker.ietf.org/doc/html/rfc3731#section-3.2.1)
                3. Review [epp-1-0.xsd](https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/epp-1.0.xsd) and [domain-1-0.xsd](https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/domain-1.0.xsd) [infDataType](https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/domain-1.0.xsd#L281)
                    - Sample message adding ownerAddress
                            
                        ```xml
                        <?xml version="1.0" encoding="UTF-8" standalone="no"?>
                           <epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
                                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                                xsi:schemaLocation="urn:ietf:params:xml:ns:epp-1.0
                                epp-1.0.xsd">
                             <command>
                               <create>
                                 <domain:create
                                  xmlns:domain="urn:ietf:params:xml:ns:domain-1.0"
                                  xsi:schemaLocation="urn:ietf:params:xml:ns:domain-1.0
                                  domain-1.0.xsd">
                                   <domain:name>example.com</domain:name>
                                   <domain:period unit="y">2</domain:period>
                                   <domain:ns>
                                     <domain:hostObj>ns1.example.com</domain:hostObj>
                                     <domain:hostObj>ns1.example.net</domain:hostObj>
                                    </domain:ns>
                                    <domain:registrant>jd1234</domain:registrant>
                                   <domain:contact type="admin">sh8013</domain:contact>
                                   <domain:contact type="tech">sh8013</domain:contact>
                                   <domain:authInfo>
                                     <domain:pw>2fooBAR</domain:pw>
                                   </domain:authInfo>
                                 </domain:create>
                               </create>
                               <clTRID>ABC-12345</clTRID>
                             </command>
                           </epp>
                        ```

       2. [create](https://datatracker.ietf.org/doc/html/rfc3731#section-3.2.1) using authInfo [normalizedString](https://www.w3schools.com/xml/schema_dtypes_string.asp)
            - authInfo
            - A <domain:authInfo> element that contains authorization information to be associated with the domain object.  This mapping includes a password-based authentication mechanism, but the schema allows new mechanisms to be defined in new schemas.
                - existing
                
                ```xml
                           <domain:authInfo>
                             <domain:pw>2fooBAR</domain:pw>
                           </domain:authInfo>
                ```
                
                - new
                
                ```xml
                <domain:authInfo>
                    <domain:w3address>663727970746f000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000</domain:w3address>
                    <domain:w3signature>0x13407e3b6e4cb59cf12ce9f138b0528cf9d13e8cef87e954b258245ae8224e69</domain:w3signature>
                    <domain:w3receipt>0x367fdc3a623ec216ff227a22ee863cc4aedd059ac67e07ac98016240602b1d82</domain:w3receipt>
                </domain:authInfo>
                ```
                
                - [domain-1.0.xsd](https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/domain-1.0.xsd#L106) changes
                
                ```xml
                <complexType name="authInfoType">
                        <choice>
                            <element name="pw" type="eppcom:pwAuthInfoType"/>
                            <element name="ext" type="eppcom:extAuthInfoType"/>
                            <element name="w3address" type="eppcom:pwAuthInfoType"/>
                            <element name="w3signature" type="eppcom:pwAuthInfoType"/>
                            <element name="w3receipt" type="eppcom:pwAuthInfoType"/>
                        </choice>
                    </complexType>
                ```
                
                - [eppcom-1.0.xsd](https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/eppcom-1.0.xsd#L12) changes (optional new w3authInfoType)
                
                ```xml
                <complexType name="w3AuthInfoType">
                    <simpleContent>
                      <extension base="normalizedString">
                      </extension>
                    </simpleContent>
                  </complexType>
                ```
                
            - Fields
                - wallet address
                - signature,
                - [web3] registration transaction receipt id

        3. Integrate with Nomulus for Tier 2 Registration
        4. Link to Nomulus for DNS Management etc
    3. Nomulus
        1. Update the DB to hold contract  addresses
            1. [database](https://github.com/google/nomulus/blob/master/db/README.md) (schema)
                1. `public.Domain` add `ownerAddress`
                2. `[public.Contact](http://public.Contact)` optionally add  `ownerAddress`
                3. `public.Tld` add `web3` flag to indicate capture of `ownerAddress`
        2. Modify [Registry](https://github.com/google/nomulus/tree/master/core/src/main/java/google/registry) flows
            1. Review [Nomulus EPP Command API Documentation](https://github.com/google/nomulus/blob/master/docs/flows.md)
            2. Review [Registry flows for Domain](https://github.com/google/nomulus/tree/master/core/src/main/java/google/registry/flows/domain)
            3. Modify custom flows e.g. [DomainCreateFlowCustomLogic.java](https://github.com/google/nomulus/blob/master/core/src/main/java/google/registry/flows/custom/DomainCreateFlowCustomLogic.java)
</details>

### Name Servers - Criteria and CodeBases for Review

**Criteria**

- Built-in authentication systems - we need to transform this to web3 authentication
- Gated APIs for updating records
- Support common DNS record types (A, CNAME, MX, TXT)
- Ideally in modern languages (Golang, Rust, C++17 or later, Python, Node.js, Scala)
- We could consider just implementing a full web3 version and store all records on smart contracts (extending ENS contracts or building our own), if this can be done in 1-2 weeks

<details><summary><b>Name Server Code Bases for Review</b></summary>

- https://github.com/topics/dns-server
    - [Rust](https://github.com/topics/dns-server?l=rust)
        - https://github.com/bluejekyll/trust-dns
        - https://github.com/Revertron/Alfis (blockchain in title)
    - [Go](https://github.com/topics/dns-server?l=go)
        - https://github.com/coredns/coredns
    - [C++](https://github.com/topics/dns-server?l=c%2B%2B)
        - https://github.com/PowerDNS/pdns (GNU)
    - [C](https://github.com/topics/dns-server?l=c)
        - https://github.com/NLnetLabs/nsd
        - https://github.com/gdnsd/gdnsd
        - https://github.com/samboy/MaraDNS
        - https://github.com/isc-projects/bind9
        - https://github.com/CZ-NIC/knot  [docs](https://www.knot-dns.cz/docs/3.2/html/introduction.html#what-is-knot-dns)
        - https://github.com/dnomd343/ClearDNS (Chinese README.md)
    - [C#](https://github.com/topics/dns-server?l=c%23)
        - https://github.com/TechnitiumSoftware/DnsServer (C#)
        - https://github.com/fanliang11/surging
    - [Javascript](https://github.com/topics/dns-server?l=javascript)
        - https://github.com/song940/node-dns
        - https://github.com/Monadical-SAS/solana-dns
    - [Python](https://github.com/topics/dns-server?l=python)
        - https://github.com/ctxis/SnitchDNS

</details>


### core-dns (go) - Ranking 1

**Summary**
[core-dns](https://github.com/coredns/coredns) is a popular repository with over 10,000 stars and 1800 forks. It has a comprehensive Plugin Approach.<sup>[1](#cd1)</sup><sup>[2](#cd2)</sup> which enables the chaining of plugins to customize logic when responding to a client request to `ServeDNS()`. "Core" Plugins have been developed by CoreDNS<sup>[3](#cd3)</sup> and also a number of specialized Plugins have been developed by third party developers<sup>[4](#cd4)</sup>

Of particular relevance is the ENS Plugin<sup>[5](#cd5)</sup> which enables nameservers to retrieve information from the Ethereum Blockchain seamlessly with the existing infrastructure. This uses go-ens<sup>[6](#cd6)</sup> to interact with the ens-contracts.<sup>[7](#cd7)</sup> All repositories appear to be actively maintained with recent commits.

Development Approach: Develop a harmony specific plugin for coredns leveraging the existing codebase.

**Evaluation Criteria**
1. Authentication: Is currently supported by the acl-plugin<sup>[7](#cd7)</sup> which can optionally be enabled to `users are able to block or filter suspicious DNS queries by configuring IP filter rule sets, i.e. allowing authorized queries or blocking unauthorized queries.` A similar plugin could be written to enable authentication via web3.
2. APIs
3. DNS Record Type Support
4. Modern Language: Yes
5. Development TimeFrame: Although the coreDns framework is large the architecture allowing building of a specific plugin as well as the reference ENS plugin and codebase should reduce development time.



**References**

<a name="cd1">[1]</a>  [CoreDNS Plugin Approach](https://coredns.io/2017/03/01/how-to-add-plugins-to-coredns/): CoreDNS is a DNS server that chains plugins. A plugin is defined as a method: ServeDNS() that gets a request and either responds to the client or passes it on to the next plugin. If none of the plugins handle the request a default response of SERVFAIL is returned.

<a name="cd2">[2]</a>  [CoreDNS Enabling Plugins](https://coredns.io/2017/07/25/compile-time-enabling-or-disabling-plugins/): CoreDNS' plugins (or external plugins) can be enabled or disabled on the fly by specifying (or not specifying) it in the Corefile. But you can also compile CoreDNS with only the plugins you need and leave the rest completely out.

<a name="cd3">[3]</a>  [CoreDNS Developed Plugins ](https://coredns.io/plugins/): Provides a list of all Plugins developed by CoreDNS. source code can be found on [github](https://github.com/coredns/coredns/tree/master/plugin).

<a name="cd4">[4]</a>  [CoreDNS External Plugins](https://coredns.io/explugins/): Provides a list of External Plugins.

<a name="cd5">[5]</a>  [ENS Plugin](https://coredns.io/explugins/ens/): The ens plugin serves DNS records from the Ethereum Name Service. Source code can be found on [github](https://github.com/wealdtech/coredns-ens). With a detailed description found in [EthDNS: an Ethereum backend for the Domain Name System](http://www.wealdtech.com/articles/ethdns-an-ethereum-backend-for-the-domain-name-system/)

> The nameservers for the my.xyz domain have been configured to fetch information not from a local store but from the Ethereum blockchain, shown as the Ethereum symbol in the bottom-right corner of the diagram. Note that the name resolution process is exactly the same as in the previous diagram, and that the client is unaware that the information has been returned from the blockchain. This makes EthDNS a drop-in replacement for existing nameservers, working seamlessly with the existing infrastructure.

<a name="cd6">[6]</a> [go-ens](https://github.com/wealdtech/go-ens): Go module to simplify interacting with the Ethereum Name Service contracts.

<a name="cd7">[7]</a> [ens-contracts](https://github.com/ensdomains/ens-contracts): This repo doubles as an npm package with the compiled JSON contracts for ENS.

<a name="cd8">[8]</a> [acl-plugin](https://coredns.io/plugins/acl/): acl enforces access control policies on source ip and prevents unauthorized access to DNS servers. Source code is located on [github](https://github.com/coredns/coredns/tree/master/plugin/acl).

<a name="cd9">[8]</a>  [A first look at CoreDNS](https://jpmens.net/2017/09/09/coredns/): Initial review of coreDNS functionality.

<a name="cd10">[9]</a>  [EPP Integration Approach](https://github.com/coredns/coredns/issues/131): As much as I would love to have an EPP client written in go, it isn't likely that an EPP client would be useful to many users. Only registrars typically have access to the EPP endpoints at registries. And even then, it is likely that they don't allow individual services to connect directly to the EPP endpoints since the registries impose connection count limits.


### trust-dns (rust) - Ranking 3

Summary: [trust-dns](https://github.com/bluejekyll/trust-dns): is a widely used project with 2,600 stars, 313 forks and 1077 users of the project. It is built using rust and has a modular framwork producing 8 rust crates<sup>[1](#td1)</sup> which can be used by other rust applications (similar to npm packages for javascript) as well as being able to be deployed as a standalone DNS authoritative server. It's Goals<sup>[2](#td2)</sup> focus on safety, security and simplicity. It has widespread support for Internet Engineering Task Force (IETF) Request for Comments (RFC's)<sup>[3](#td3)</sup>. 

Development Approach: Deploy Standalone Version and modify the [server crate](https://github.com/bluejekyll/trust-dns/tree/main/crates/server)<sup>[1](#td1)</sup>,  to use Harmony Blockchain instead of sql-lite for [store](https://github.com/bluejekyll/trust-dns/tree/main/crates/server/src/store) functionality. Note: there are minimal ENS rust codebases<sup>[4](#td4)</sup><sup>[5](#td5)</sup><sup>[6](#td6)</sup>. Which can be reviewed for devleopment of the rust to Harmony ENS backend. Optionally a harmony-specific server crate can be published with could be used by the 1684 projects currently using trust-dns-server.

**Evaluation Criteria**
1. Authentication: 
2. APIs: 
3. DNS Record Type Support: Good
4. Modern Language: Yes Rust
5. Development TimeFrame: The development approach is fairly clear. Factors to consider is the availibility of Rust developers and the fact that there are minimal ens references to leverage.


**References**

<a name="td1">[1]</a>  [Trust-DNS Crates](https://trust-dns.org/): Rust crates developed by Trust-DNS
* [Trust-DNS](https://crates.io/crates/trust-dns): Binaries for running a DNS authoritative server.
* [Proto](https://crates.io/crates/trust-dns-proto): trust-dns-proto Raw DNS library, exposes an unstable API and only for use by the other Trust-DNS libraries, not intended for end-user use.
* [Client](https://crates.io/crates/trust-dns-client): trust-dns-client Used for sending query, update, and notify messages directly to a DNS server.
* [Server](https://crates.io/crates/trust-dns-server): trust-dns-server Use to host DNS records, this also has a named binary for running in a daemon form.
* [Resolver](https://crates.io/crates/trust-dns-resolver): trust-dns-resolver Utilizes the client library to perform DNS resolution. Can be used in place of the standard OS resolution facilities.
* [Rustls](https://crates.io/crates/trust_dns_rustls): trust-dns-rustls Implementation of DNS over TLS protocol using the rustls and ring libraries.
* [NativeTls](https://crates.io/crates/trust_dns_native_tls): trust-dns-native-tls Implementation of DNS over TLS protocol using the Host OS’ provided default TLS libraries
* [OpenSsl](https://crates.io/crates/trust_dns_openssl): trust-dns-openssl Implementation of DNS over TLS protocol using OpenSSL

<a name="td2">[2]</a>  [Trust-DNS Goals](https://github.com/bluejekyll/trust-dns#goals): 
* Build a safe and secure DNS server and client with modern features.
* No panics, all code is guarded
* Use only safe Rust, and avoid all panics with proper Error handling
* Use only stable Rust
* Protect against DDOS attacks (to a degree)
* Support options for Global Load Balancing functions
* Make it dead simple to operate

<a name="td3">[3]</a>  [Trust-DNS RFCs implemented](https://github.com/bluejekyll/trust-dns/blob/main/README.md#rfcs-implemented): A list of RFCs implemented and in progress by trust-dns.


<a name="td4">[4]</a>  [rust ens-rainbow](https://github.com/graphprotocol/ens-rainbow): Developed by graphprotocol in 2019 is an upload utility for ens_names.sql.gz

<a name="td5">[5]</a>  [rust-ens crate](https://crates.io/crates/ens): Used by ens-rainbos it is a Rust ENS(Ethereum Name Service) interface, based on rust-web3.

<a name="td6">[6]</a>  [rust ens-api](https://github.com/EnormousCloud/ens-api): Ethereum Name Service RESTful API developed and last modified in December 2021.

<a name="td7">[7]</a>  [trust-dns-server Dependency graph](https://github.com/bluejekyll/trust-dns/network/dependents?dependents_after=MjMwODY3NDk4MDM): Repositories that depend on trust-dns-server.

### solana-dns (javascript) - Ranking 4

Summary: [solana-dns](https://github.com/Monadical-SAS/solana-dns) was built more as a conceptual prototype for Solana 3 years ago by Nick Sweeting<sup>[1](#sd1)</sup>. It was never completed but has some good points<sup>[2](#sd2)</sup> to consider.

**Evaluation Criteria**
1. Authentication: No
2. APIs: Not Fully Developed
3. DNS Record Type Support
4. Modern Language: Yes Javascript 
5. Development TimeFrame: Would be long as would be building from scratch without much code that could be leveraged.


**References**

<a name="sd1">[1]</a> [Nick Sweeting Twitter](https://twitter.com/thesquashSH): Reached out to Nick regarding the status of the project and he responded with 
> [The Project] Just stopped, it was a fun side project proposal to our friends on the solana team for a potential use case for the platform. They ended up giving us other solana work instead but thought dns was a cool idea, never got funding to do the DNS thing though. Feel free to rip off all/some/any of it, it's up for grabs if you want to build something similar! 🙂

<a name="sd2">[2]</a> [How does DNS work normally?](https://github.com/Monadical-SAS/solana-dns#how-does-dns-work-normally): The repository's README.md gives an overview of the approach and potential problems. It also contains a [Data Flow](https://github.com/Monadical-SAS/solana-dns#data-flow) and [Execution Flow](https://github.com/Monadical-SAS/solana-dns#execution-flow).

> **So what does Solana add to the equation?**
>                    
>    It removes the root DNS servers from the trust equation (e.g. the `.com` TLD servers). Remember that they have ultimate control over all record authentication because they could choose to ignore or maliciously change the DNSSEC public key that you published via the registrar. This implicit trust can be removed in Solana DNS because the public key is the user's wallet public key, which is intrinsically tied to their identity on the platform. Solana has no power to change the public key because the chain forms an immutable record, and only later statements signed with a revocation key are recognized by clients as authorized to change the public key in control of a given domain.
>                    
>    This isn't just a case of "slap a blockchain on it and hope it gets better", this is a true novel distributed database-style solution that hasn't been feasible with traditional blockchains. Running usable DNS on blockchains is now feasible with the advent of fast proof-of-history chains, because it enables updates to happen within DNS TTL windows like 30sec, which are well below the 10min verification time that a chain like Bitcoin would require. Because changes can also be synchronously applied to appear to everyone across the world at the same time, we can also lose the "eventual consistency" and propagation delay issues that plagued standard DNS systems in the past.
>                     
>    Solana DNS can also retain the nice distributed properties of past DNS systems. Because all records are signed all the way up to the root using the domain owner's public key, any node can serve up records without users having to implicitly trust them. The query results will always arrive signed with a key that can be verified to ensure malicous middle-boxes cant get away with modifying records.
>                    
> **Potential problems?**
>                    
>    - Solana may be fast, but the RPC communication with the Solana chain may be significantly slower than DNS over UDP
>    - If RPC communication becomes the bottleneck, we end up having to implement time-synchronized caching servers with fairly complex validation/staking mechanics to penalize clock drift and inadherence to signed record TTL expiry times. This problem becomes slightly simpler if we only allow caching on localhost, with no middleboxes between the local Solana DNS server and the Solana chain API.
>    - It may not be necessary to replace DNSSEC entirely, it's possible to just sign the same root public key used on the TLD in Solana, and use DNSSEC from there on down the chain (checking against the Solana root key instead of the implicitly trusted root-server-published key)
>    - The mechanics around domain ownership verification need to be figured out (or whether it's even necessary with Solana DNS)
>    - The mechanics of pinning identities to keys needs to be figured out (whether via staking, 3rd party identity providers, or something else)
>    - The mechanics of key issuing, rotating, and revocation need to be figured out (proper revocation is haaard, we dont want domains being lost forever to the ether because someone lost a private key)
>                    
> Given the power of an immutable, globally sychronized database, with key-based identity baked in, many of the distributed-systems and authentication problems that have plagued DNS in the past become drastically easier to solve. Unfortunately, designing DNS system to seamlessly augment or entirely replace existing DNS authentication mechanisms is incredibly complex. Though our ambitions are big, this project will likely have to make some decisions early on to narrow the scope and pick a few core features to focus on as a proof-of-concept.

### project (language) - Ranking X

Summary: [repo](link)

**Evaluation Criteria**
1. Authentication: 
2. APIs
3. DNS Record Type Support
4. Modern Language: 
5. Development TimeFrame: 


**References**


## Standards

- [RFC 8499](https://tools.ietf.org/html/rfc8499): No more master/slave, in honor of [Juneteenth](https://en.wikipedia.org/wiki/Juneteenth)

### Basic operations

- [RFC 1035](https://tools.ietf.org/html/rfc1035): Base DNS spec (see the Resolver for caching)
- [RFC 2308](https://tools.ietf.org/html/rfc2308): Negative Caching of DNS Queries (see the Resolver)
- [RFC 2317](https://tools.ietf.org/html/rfc2317): Classless IN-ADDR.ARPA delegation
- [RFC 2782](https://tools.ietf.org/html/rfc2782): Service location
- [RFC 3596](https://tools.ietf.org/html/rfc3596): IPv6
- [RFC 6891](https://tools.ietf.org/html/rfc6891): Extension Mechanisms for DNS
- [RFC 6761](https://tools.ietf.org/html/rfc6761): Special-Use Domain Names (resolver)
- [RFC 6762](https://tools.ietf.org/html/rfc6762): mDNS Multicast DNS (experimental feature: `mdns`)
- [RFC 6763](https://tools.ietf.org/html/rfc6763): DNS-SD Service Discovery (experimental feature: `mdns`)
- [RFC ANAME](https://tools.ietf.org/html/draft-ietf-dnsop-aname-02): Address-specific DNS aliases (`ANAME`)

### Update operations

- [RFC 1995](https://tools.ietf.org/html/rfc1995): Incremental Zone Transfer
- [RFC 1996](https://tools.ietf.org/html/rfc1996): Notify secondaries of update
- [RFC 2136](https://tools.ietf.org/html/rfc2136): Dynamic Update
- [RFC 7477](https://tools.ietf.org/html/rfc7477): Child-to-Parent Synchronization in DNS
- [Update Leases](https://tools.ietf.org/html/draft-sekar-dns-ul-01): Dynamic DNS Update Leases
- [Long-Lived Queries](https://tools.ietf.org/html/draft-sekar-dns-llq-01): Notify with bells

### Secure DNS operations

- [RFC 3007](https://tools.ietf.org/html/rfc3007): Secure Dynamic Update
- [RFC 4034](https://tools.ietf.org/html/rfc4034): DNSSEC Resource Records
- [RFC 4035](https://tools.ietf.org/html/rfc4035): Protocol Modifications for DNSSEC
- [RFC 4509](https://tools.ietf.org/html/rfc4509): SHA-256 in DNSSEC Delegation Signer
- [RFC 5155](https://tools.ietf.org/html/rfc5155): DNSSEC Hashed Authenticated Denial of Existence
- [RFC 5702](https://tools.ietf.org/html/rfc5702): SHA-2 Algorithms with RSA in DNSKEY and RRSIG for DNSSEC
- [RFC 6844](https://tools.ietf.org/html/rfc6844): DNS Certification Authority Authorization (CAA) Resource Record
- [RFC 6698](https://tools.ietf.org/html/rfc6698): The DNS-Based Authentication of Named Entities (DANE) Transport Layer Security (TLS) Protocol: TLSA
- [RFC 6840](https://tools.ietf.org/html/rfc6840): Clarifications and Implementation Notes for DNSSEC
- [RFC 6844](https://tools.ietf.org/html/rfc6844): DNS Certification Authority Authorization Resource Record
- [RFC 6944](https://tools.ietf.org/html/rfc6944): DNSKEY Algorithm Implementation Status
- [RFC 6975](https://tools.ietf.org/html/rfc6975): Signaling Cryptographic Algorithm Understanding
- [RFC 7858](https://tools.ietf.org/html/rfc7858): DNS over TLS (feature: `dns-over-rustls`, `dns-over-native-tls`, or `dns-over-openssl`)
- [RFC DoH](https://tools.ietf.org/html/draft-ietf-doh-dns-over-https-14): DNS over HTTPS, DoH (feature: `dns-over-https-rustls`)
- [DNSCrypt](https://dnscrypt.org): Trusted DNS queries
- [S/MIME](https://tools.ietf.org/html/draft-ietf-dane-smime-09): Domain Names For S/MIME

[RFC2135](https://www.rfc-editor.org/rfc/rfc2136): Dynamic Updates in the Domain Name System (DNS UPDATE)


