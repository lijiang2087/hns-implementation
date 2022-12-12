# ONE Name Service Design Document
This document covers the high level development tasks to create similar functionality as [ens.domains](https://ens.domains/) for [Harmony](https://www.harmony.one/).

*Note: This project is separate from [hnsdomains](https://github.com/hnsdomains) which was a Harmony community lead project.*

**Table of Contents**
- [Harmony Name Service Design Document](#harmony-name-service-design-document)
  - [Focus areas](#focus-areas)
  - [Requirements](#requirements)
  - [Technical Components](#technical-components)
  - [Deployment Strategy](#deployment-strategy)
    - [Prerequisites](#prerequisites)
    - [Local Build](#local-build)
    - [Customize Local Build for Harmony](#customize-local-build-for-harmony)
    - [Local Deploy](#local-deploy)
    - [Testnet Deploy](#testnet-deploy)
  - [Additional Tasks](#additional-tasks)
  - [Assumptions](#assumptions)
  - [References](#references)
    - [Websites](#websites)
    - [ENS Github](#ens-github)

## Focus areas

* Migration of ENS codebase to Harmony codebase
* Support of Deployment on Harmony.one
* Support for `.country` domain (instead of `.ens`)
* Blended web2 and web3 support for `.country` domains
* NFT support


## Requirements

1. Name Registration: see [Registering & Renewing Names](https://docs.ens.domains/dapp-developer-guide/registering-and-renewing-names)
2. Sub Domain Registration: see [Creating SubDomains](https://docs.ens.domains/dapp-developer-guide/managing-names#creating-subdomains)
3. NFT Regsitration: All registered domains are created as NFT's. (see [ENS as NFT](https://docs.ens.domains/dapp-developer-guide/ens-as-nft)). Trading support can also be built out on Harmony NFT Marketplaces similar to [ENS on OpenSea](https://opensea.io/collection/ens).
4. `.country` support: see [DNS Registrar Guide](https://docs.ens.domains/dns-registrar-guide)
5. Wallet Integration: Integrating the creation of Harmony Name Service Domains in OneWallet or SMS Wallet can be build following the framework documented in [ENS Enabling your DApp](https://docs.ens.domains/dapp-developer-guide/ens-enabling-your-dapp)

## Technical Components

* Smart Contracts
    * [ens-contracts](https://github.com/ensdomains/ens-contracts): This repo doubles as an npm package with the compiled JSON contracts
        * [ens](https://github.com/ensdomains/ens): Prior Version of Implementations for registrars and local resolvers for the Ethereum Name Service.
    * [ENS Offchain Resolver](https://github.com/ensdomains/offchain-resolver): smart contracts and a node.js gateway server that together allow hosting ENS names offchain using EIP 3668 and ENSIP 10.
    * [l2gateway-demo](https://github.com/ensdomains/l2gateway-demo/): A demonstration and MVP of an Ethereum <-> Optimism bridge for resolving ENS names (see [medium article](https://medium.com/the-ethereum-name-service/mvp-of-ens-on-l2-with-optimism-demo-video-how-to-try-it-yourself-b44c390cbd67)). Not Needed for harmony-domains, but worth investegating if we want Harmony to effectively be an L2 for Ethereum ENS.
    * [subdomain-registrar](https://github.com/ensdomains/subdomain-registrar): set of smart contracts and corresponding webapp that facilitates easy registration of ENS subdomains for users.
* SDK
    * [ensjs-v3](https://github.com/ensdomains/ensjs-v3): The ultimate ENS javascript library, with ethers.js under the hood.
        * [emsjs](https://github.com/ensdomains/ensjs): Prior version of ensjs
* API
    * [ens-avatar](https://github.com/ensdomains/ens-avatar): Avatar resolver library for both nodejs and browser.
        * [ens-avatar-worker](https://github.com/ensdomains/ens-avatar-worker): API [Cloudflare serverless worker](https://developers.cloudflare.com/workers/) for getting avataRs
    * [ens-metadata-service](https://github.com/ensdomains/ens-metadata-service): API to get ens metadata
* Fronted
    * [ens-app-v3](https://github.com/ensdomains/ens-app-v3): The all new, all cool version of the ENS manager.
        * [ens-app](https://github.com/ensdomains/ens-app): Prior version of ENS Application
* Reporting
    * [ens-subgraph](https://github.com/ensdomains/ens-subgraph): sources events from the ENS contracts.
        * See also Harmony's support of [The Graph - Subgraphs](https://docs.harmony.one/home/developers/web3-foundations/tutorials/the-graph-subgraphs)
* Documentation
    * [docs](https://github.com/ensdomains/docs): ens documentation   

## Deployment Strategy

*Note: See [Deploying ENS on a Private Chain](https://docs.ens.domains/deploying-ens-on-a-private-chain) as a starting point (modified version is in [DEPLOYMENT.md](./DEPLOYMENT.md)).*

### Prerequisites
- [x] Set up Github Organization [harmony-domains](https://github.com/harmony-domains)
- [x] Copy required repositories (not fork) from [ensdomains](https://github.com/ensdomains)
- [ ] Setup Hosting accounts on AWS
- [ ] Regsister Domain to support ENS e.g. [harmony.domains](https://harmony.domains)


### Local Build

- [x] Clone and build the following repositories
    - [x] [ens-contracts](https://github.com/ensdomains/ens-contracts): This repo doubles as an npm package with the compiled JSON contracts 
    - [x] [ens-app-v3](https://github.com/ensdomains/ens-app-v3): The all new, all cool version of the ENS manager. Dependencies Include when running `dev:nlocal`
        - [ ] Local Node : `ganache -m "test test test test test test test test test test test junk"'`
        - [ ] [Metadata Service](https://github.com/ensdomains/ens-metadata-service): [docs](https://metadata.ens.domains/docs)
        - [ ] Avatar Uploader: [ens-avatar-worker](https://github.com/ensdomains/ens-avatar-worker)
        - [ ] [ens-subgraph](https://github.com/ensdomains/ens-subgraph)
        - [ ] [offChain-resolver](https://github.com/ensdomains/offchain-resolver)

### Customize Local Build for Harmony

- [ ] Customize Repositories for Harmony
    - [ ] Address Format Customizations
    - [ ] `.country` customizations
    - [ ] document updates
### Local Deploy

*Note: Need to review the [ens-test-env](https://github.com/ensdomains/ensjs-v3/tree/main/packages/ens-test-env/) which uses docker. We may be able to run a local test environment by running components locally in terminal windows as below*

Original Fronted [ens-app](https://github.com/ensdomains/ens-app)

```
# Start ganache locally (terminal window 1)
/Users/john/one-wallet/ens/ens-deployer/env
./ganche-new.sh

# Deploy locally (terminal window 2)
cd /Users/john/one-wallet/ens/ens-deployer/contract
yarn deploy

# Start the Frontend (separate terminal window)
cd /Users/john/one-wallet/ens/ens-app

# To start pointing to Goerli or Mainnet using ens infrastructure
yarn start

# To test locally



# Runs local node with additional developer options under settings
pnpm dev:nlocal


# View the frontend at http://localhost:3000/


```

New Fronted [ens-app-v3](https://github.com/ensdomains/ens-app-v3)


```
# Start Hardhat Locally (terminal window 1)
cd /Users/john/one-wallet/ens/ens-contracts
ganachem='ganache -m "test test test test test test test test test test test junk"

# Deploy Contracts locally using [harmony-deploy.js](https://github.com/harmony-domains/ens-contracts/blob/jw-harmony-build/scripts/harmony-deploy.js)(terminal window 2) use this or the deployer below NOT BOTH
cd /Users/john/one-wallet/ens/ens-contracts
npx hardhat run scripts/harmony-deploy.js --network localhost

# Deploy contracts using [ensDeployer.ts](https://github.com/harmony-domains/ens-deployer/blob/main/contract/deploy/ensDeployer.ts) (terminal window 2) use this or the deployer above NOT BOTH
cd /Users/john/one-wallet/ens/ens-deployer/contract 
yarn deploy


# Start the Frontend (separate terminal window)
cd /Users/john/one-wallet/ens/ens-app-v3

# Runs Local node connected to Goerli (currently stalls when registering a name)
pnpm dev

# Runs local node with additional developer options under settings
pnpm dev:nlocal


# View the frontend at http://localhost:3000/


```
### Testnet Deploy

## Additional Tasks

* Setting up an Harmony.domains
* Setting up community channels 
  * twitter
  * discord
  * medium


## Assumptions
1. We will create a separate github organization for this project [harmony-domains](https://github.com/harmony-domains)
2. Relevant repositories will be copied (not forked) from [ensdomains](https://github.com/ensdomains)
3. We will host his under the domain name https://harmony.domains
4. Hosting Services will be under AWS and hosted by `modulo.so`
5. Transtion plan to a Harmony Domains DAO will be created for ongoing support and maintenance of the project.

## References

[ENS DAO Constitution](./assets/constitution.pdf)
[ENS on OpenSea](https://opensea.io/collection/ens)

### Websites
* [ens.domains](https://ens.domains/)
* [app.ens.domains](https://app.ens.domains/)
* [docs.ens.domains](https://docs.ens.domains/)
* [reserve.ens.domains](https://reserve.ens.domains/)
* [github.com/ensdomains](https://github.com/ensdomains)
* [twitter.com/ensdomains](https://twitter.com/ensdomains)
* [medium](https://medium.com/the-ethereum-name-service)

### ENS Github

* [ens-contracts](https://github.com/ensdomains/ens-contracts): This repo doubles as an npm package with the compiled JSON contracts
    * [ens](https://github.com/ensdomains/ens): Prior Version of Implementations for registrars and local resolvers for the Ethereum Name Service.
* [ENS Offchain Resolver](https://github.com/ensdomains/offchain-resolver): smart contracts and a node.js gateway server that together allow hosting ENS names offchain using EIP 3668 and ENSIP 10.
* [subdomain-registrar](https://github.com/ensdomains/subdomain-registrar): set of smart contracts and corresponding webapp that facilitates easy registration of ENS subdomains for users.
* [ensjs-v3](https://github.com/ensdomains/ensjs-v3): The ultimate ENS javascript library, with ethers.js under the hood.
    * [emsjs](https://github.com/ensdomains/ensjs): Prior version of ensjs
* [ens-app-v3](https://github.com/ensdomains/ens-app-v3): The all new, all cool version of the ENS manager.
    * [ens-app](https://github.com/ensdomains/ens-app): Prior version of ENS Application
* [ensdomains-v2](https://github.com/ensdomains/ensdomains-v2): ENS Homepage V2 supports the website [ens.domains](https://ens.domains/)
    * [ens.domains](https://github.com/ensdomains/ens.domains): Prior version of ENS Website
* [ui](https://github.com/ensdomains/ui): Reusable functions and components for the ENS apps
* [ens-avatar](https://github.com/ensdomains/ens-avatar): Avatar resolver library for both nodejs and browser.
    * [ens-avatar-worker](https://github.com/ensdomains/ens-avatar-worker): API [Cloudflare serverless worker](https://developers.cloudflare.com/workers/) for getting avataRs
* [ens-metadata-service](https://github.com/ensdomains/ens-metadata-service): API to get ens metadata
* [ens-subgraph](https://github.com/ensdomains/ens-subgraph): sources events from the ENS contracts.
* [address-encoder](https://github.com/ensdomains/address-encoder): typescript library encodes and decodes address formats for various cryptocurrencies.
* [ens-print](https://github.com/ensdomains/ens-print): a micro app to generate your ENS sticker and print.
* [name-reservations](https://github.com/ensdomains/name-reservations): Support for ENS Short Name Claim Tool [reserve.ens.domains](https://reserve.ens.domains/) 
* [docs](https://github.com/ensdomains/docs): ens documentation
