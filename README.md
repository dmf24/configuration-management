## Configuration Management

```
BootstrappedNode = Bootstrapper(Provisioner(BaremetalNode))
NodeFacts=FactGatherer(BootstrappedNode)
ConfiguredNode = Configurator(ConfigCompiler(ConfigRepository, SiteEnvironment, NodeFacts),
                              BootstrappedNode,
                              SiteEnvironment,
                              SecureData))
SiteEnvironment.update(ReportingAgent(ConfiguredNode))
```

### Node

A Node is a computer on a network that is a member of a Site.

### Site

A Site is group of related nodes that share a single environment and configuration repository.

### SecureData

SecureData are Secrets that require additional attention to reduce risk of exposure.

### BareMetalNode

A Node that has power and network but no operating system.

### ProvisionedNode

A ProvisionedNode has an installed operating system and access to network resources.

### BootstrappedNode

A BootstrappedNode is a ProvisionedNode that is ready to apply ConfigInstructions

### NodeFacts

NodeFacts are pieces of information about a Node that are discovered, rather than set, prior to configuration.  Note that NodeFacts might be set manually at some other time and are only static in the context of an initiated configuration run.

### FactGatherer

The FactGatherer discovers facts about nodes and provides them to the ConfigCompiler.

### ConfiguredNode

A ConfiguredNode is a Node that has been configured and is ready to work.

### ConfiguredSite

A ConfiguredSite is a Site that has been configured and is ready to work.

### ConfigCompiler

The Configuration Compiler transforms the code in the ConfigRepository into instructions the configurator can use to directly make changes to a node.

The ConfigCompiler needs access to

  * ConfigRepository
  * NodeFacts
  * SiteEnvironment

### ConfigDeployer/Retriever

A Node must have an updated copy of either ConfigInstructions or the whole ConfigRepository.  Therefore, there must be some method defined for transmitting it to nodes.

### Secure Data Deployer/Retriever

Assuming we have a secure data service, we need a method to move that data from the store to where it is needed.

### Cofigurator

This is how nodes apply their configuration, once they know what it is.  The configurator must be able to run as root on the BootstrappedNode it will configure, and also needs defined methods for accessing:

  * ConfigInstructions
  * BootstrappedNode
  * SecureData
  * SiteEnvironment (if not handled by ConfigCompiler)

Technically, configurator will also need access to information in the NodeFacts data structure, but this is covered 

### ReportingAgent

It is desirable for Nodes to have a method to update the SiteEnvironment with NodeFacts and services so other nodes know they exist and what they are for.

### ConfigRepository

This is a primarily plain-text, human-written, source-controlled, codified specification of configuration for the Site.

### SiteEnvironment (Dynamic Site Inventory)

This is a site-wide data store, accessible from the entire cluster, that contains comprehensive information about each node in the cluster, including its role, services offered, and current state.

A SiteEnvironment should be accessible via a simple, straightforward API that must be usable by:
  * ConfigCompiler
  * Configurator
  * ReportingAgent

Since changes to the SiteEnvironment can circumvent the source-control of the ConfigRepository to make changes to the Site, it may require rigorous auditing features.

**Other possible features:**

  * Node discovery
  * Service discovery
  * 3rd party integrations

### Secure Data service

Configuration management and preferably arbitrary nodes need a good way to obtain secure configuration data, such as a root password hash, so that we do not have to put it directly in the configuration repository.  No matter how much you might lock down github with 2-factor auth, it is way too easy to just `git clone` a config repo into a world-readable directory.

Ideally, this could also be an authoritative place for storing files like website SSL certificates although that is less of a concern for the HPC cluster.

The core challenge here is answering the question of how an automated process on an arbitrary can access secure data while minimizing risk.

### Provisioning and Bootstrapping

Although provisioning itself is worth its own topic, there are a few important questions from a configuration management perspective:
  * What is the expected state of a provisioned node?
  * How is a provisioned node bootstrapped into config management?
  * How is the provisioning system itself managed?
