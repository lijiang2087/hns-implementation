# core-dns plugin development

- [core-dns plugin development](#core-dns-plugin-development)
  - [Overview](#overview)
  - [Development Components](#development-components)
    - [hns-contracts](#hns-contracts)
    - [hns-go](#hns-go)
    - [coredns-hns](#coredns-hns)
    - [core-dns](#core-dns)
  - [References](#references)
    - [core-dns (go) - Evaluation](#core-dns-go---evaluation)


## Overview

This document gives an overview of the development framework and build process to develop a web3 backend plugin for core-dns. 

*For running a local environment please see [ens-app-v3/docs/Developer.md](https://github.com/harmony-domains/ens-app-v3/blob/multichain-docker-simplified/docs/Developer.md)*

## Development Components

### [hns-contracts](https://github.com/harmony-domains/hns-contracts)

**Summary**

This contains the web3 backend logic using smart contracts. The current version simply contains the contracts used for http://ens.demo.harmony.one/ copied from [.1.country/contracts](https://github.com/polymorpher/.1.country/tree/main/contracts). However for full integration we need to review the ens contracts held in [harmony/domains/ens-contracts](https://github.com/harmony-domains/ens-contracts). 

**Next Steps**

1. Review ens-contracts to determine contracts required
2. Build and test contracts


### [hns-go](https://github.com/harmony-domains/hns-go)

**Summary**

This contains a go package `github.com/harmony-domains/hns-go` which is initially [published](https://go.dev/doc/modules/publishing) as `v0.0.2`. The package provides the ability for go codebases to interact with the [hns-contracts](#hns-contracts) including the [abi's](https://github.com/harmony-domains/hns-go/blob/main/contracts/auctionregistrar/contract.abi) and [generators](https://github.com/harmony-domains/hns-go/blob/main/contracts/auctionregistrar/generate.go) to build the [contract.go](https://github.com/harmony-domains/hns-go/blob/main/contracts/auctionregistrar/contract.go) files. The contract logic is then wrapped in files such as [auctionregistrar.go](https://github.com/harmony-domains/hns-go/blob/main/contracts/auctionregistrar/contract.go) which provide the type definitions and functions to access the contracts.

The intial version was copied from [go-ens](https://github.com/wealdtech/go-ens) and refactord to use the package name hns-go. 

**Next Steps**
1. Import the abi's from hns-contracts
2. Generate new contract.go files
3. Write contract logic files 
4. Test
5. Publish Files


### [coredns-hns](https://github.com/harmony-domains/coredns-hns)

**Summary**

This codebase provides a [core-dns plugin](https://coredns.io/2016/12/19/writing-plugins-for-coredns/) allowing [core-dns](https://github.com/coredns/coredns) a popular domain name server to interact with the hns web backend for domain information including the domain owners address (see [this article](http://www.wealdtech.com/articles/ethdns-an-ethereum-backend-for-the-domain-name-system/) for an overview). It leverage hns-go for web3 backend integration.

The inital version has been modified to leverage [hns-go].

**Function Overview**

An Overview of functions can be found [here](./assets/hns-go-docs.pdf) and below
```
type HNS
    func (e HNS) HasRecords(domain string, name string) (bool, error)
    func (e HNS) IsAuthoritative(domain string) bool
    func (e HNS) Name() string
    func (e HNS) Query(domain string, name string, qtype uint16, do bool) ([]dns.RR, error)
    func (e HNS) Ready() bool
    func (e HNS) ServeDNS(ctx context.Context, w dns.ResponseWriter, r *dns.Msg) (int, error)
type Result
    func Lookup(server Server, state request.Request) ([]dns.RR, []dns.RR, []dns.RR, Result)
type Server
Package files
dname.go hns.go server.go setup.go wildcard.go
```

**Sample DNS Retrieval of an A Record from web3 backend**
1. core-dns-hns performs a [Query](https://github.com/harmony-domains/coredns-hns/blob/main/hns.go#L66)
   
   `func (e HNS) Query(domain string, name string, qtype uint16, do bool) ([]dns.RR, error) {`
2. for entry types SOA,NS,TXT,A,AAAA it performs [obtainContentHash](https://github.com/harmony-domains/coredns-hns/blob/main/hns.go#L80)

    `contentHash, err = e.obtainContentHash(name, domain)`
3. for A record types if calls [e.handleA](results, err = e.handleA(name, domain, contentHash)

   `results, err = e.handleA(name, domain, contentHash)`
4. handleA calls [e.obtainARRSet(name, domain)](https://github.com/harmony-domains/coredns-hns/blob/main/hns.go#L224)

   `aRRSet, err := e.obtainARRSet(name, domain)`
5. obtainARRSet calls [getDNSResolver](https://github.com/harmony-domains/coredns-hns/blob/main/hns.go#L310) to get the resolver

   `resolver, err := e.getDNSResolver(ethDomain)`
6. getDNSResolver calls [NewDNSResolver](https://github.com/harmony-domains/coredns-hns/blob/main/hns.go#L368)

   `resolver, err := hns.NewDNSResolver(e.Client, domain)`
   1. hns.NewDNSResolver calls [NewDNSResolver](https://github.com/harmony-domains/hns-go/blob/main/dnsresolver.go#L38) in hns-go
   
      `func NewDNSResolver(backend bind.ContractBackend, domain string) (*DNSResolver, error) `
      1. NewDNSResolver creates a [NewRegistry](https://github.com/harmony-domains/hns-go/blob/main/dnsresolver.go#L39) instance 
   
            `registry, err := NewRegistry(backend)`
      2. The registry is used to get the [ResolverAddress](https://github.com/harmony-domains/hns-go/blob/main/dnsresolver.go#L43) using hns-go to get the resolvers contract address
   
            `address, err := registry.ResolverAddress(domain)`

      3. [ResolverAddress](https://github.com/harmony-domains/hns-go/blob/main/registry.go#L69) is a function in registry.go which interacts with the Registry contract to get the Resolver address

            `return r.Contract.Resolver(nil, nameHash)`

            1. [resolver](https://github.com/ensdomains/ens-contracts/blob/master/contracts/registry/ENSRegistry.sol#L170) is a function in ENSRegistry.sol in ens-contracts whch returns the resolvers address based on the hash passed

                ```
                function resolver(bytes32 node) public view virtual override returns (address) {
                    return records[node].resolver;
                }
                ```
      4. ResolverAddress is used to create a [NewDNSResolverAt](https://github.com/harmony-domains/hns-go/blob/main/dnsresolver.go#L48) Resolver contract instance

            `return NewDNSResolverAt(backend, domain, address)`
7. obtainARRSet calls [resolver.Record](https://github.com/harmony-domains/coredns-hns/blob/main/hns.go#L315) to get the A record

      `return resolver.Record(name, dns.TypeA)`
      1. resolver.Record calls [Record](https://github.com/harmony-domains/hns-go/blob/main/dnsresolver.go#L76) in hns.go
   
            `func (r *DNSResolver) Record(name string, rrType uint16) ([]byte, error) {`

         1. Record calls [r.Contract.DnsRecord](https://github.com/harmony-domains/hns-go/blob/main/dnsresolver.go#L81)
   
            `return r.Contract.DnsRecord(nil, nameHash, DNSWireFormatDomainHash(name), rrType)`

            1. [dnsRecord](https://github.com/ensdomains/ens-contracts/blob/master/contracts/resolvers/profiles/DNSResolver.sol#L117) is a function in DNSResolver.sol in ens-contracts
   
               ```   
               function dnsRecord(bytes32 node, bytes32 name, uint16 resource) public view virtual override returns (bytes memory) {
                    return versionable_records[recordVersions[node]][node][name][resource];
                }
                ```

**Next Steps**
1. Review [How to Add Plugins to CoreDNS](https://coredns.io/2017/03/01/how-to-add-plugins-to-coredns/)
2. Update [hns.go](https://github.com/harmony-domains/coredns-hns/blob/main/hns.go) functionality to reflect updated [hns-go](#hns-go) package.
3. Review [server.go](https://github.com/harmony-domains/coredns-hns/blob/main/server.go) functionality
4. Enable the plugin: [Compile Time Enabling or Disabling Plugins](https://coredns.io/2017/07/25/compile-time-enabling-or-disabling-plugins/)
5. End to End testing locally using [build-standalone.sh](https://github.com/harmony-domains/coredns-hns/blob/main/build-standalone.sh)
6. Document the plugin (see [ens](https://coredns.io/explugins/ens/))
7. Working Production Version on Harmony Mainnet
8. Publish the Plugin 

### [core-dns](https://github.com/harmony-domains/coredns)

**Summary**
Core DNS Domain Name Server. 

**Next Steps**
1. All testing will be done via [coredns-hns](#coredns-hns)



## References

### [core-dns (go) - Evaluation](https://github.com/harmony-domains/hns-implementation/blob/main/docs/ARCHITECTURE.md#core-dns-go---ranking-1)

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

<a name="cd10">[9]</a>  [EPP Integration Approach](https://github.com/coredns/coredns/issues/131): 
> As much as I would love to have an EPP client written in go, it isn't likely that an EPP client would be useful to many users. Only registrars typically have access to the EPP endpoints at registries. And even then, it is likely that they don't allow individual services to connect directly to the EPP endpoints since the registries impose connection count limits.