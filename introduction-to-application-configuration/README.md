# Introduction to Application Configuration

We'll establish some general definitions and approaches for managing application configuration.

## What is configuration?

The [12 Factor App](https://12factor.net/config) defines application configuration as:

> everything that is likely to vary between deploys (staging, production, developer environments, etc)

In other words, configuration refers to a set of options that can be defined & configured to alter or enable the behavior of an application. For example, one could configure:

- How many threads to use for a particular task (You may want to use more threads in production compared to your development environments).
- What themes to use, the font size, key mappings, resolution etc.
- Where & how to save results.
- What features to enable or disable ([feature flags](https://martinfowler.com/articles/feature-toggles.html)) e.g. running a JVM with experimental or preview features.
- URLs for external services.
- etc

## What are secrets?

From an application's perspective, the configuration also includes secrets. [Hashicorp](https://www.hashicorp.com/resources/what-is-secret-sprawl-why-is-it-harmful) defines secrets as:

> anything that gives us access to a system or allows us to authenticate or authorize ourself.

As such, secrets are pieces of information that should be kept confidential. Moreover, Secrets should be stored securely and access to them should be audited & restricted. They can include things like:

- Usernames/Passwords
- API keys
- Encryption Keys
- Certificates
- etc

## Static Configuration

Static configuration refers to the configuration that doesn't change that often and is typically checked into source control. Such configuration could be hardcoded or bundled with an application as a property file. For example, the URL for an external service could be hardcoded in the code. Updating static configuration typically requires **both** rebuilding and redeploying the application.

## Dynamic (Externalized) Configuration

Dynamic or externalized configuration on the other hand refers to configuration that can be provided or updated at runtime. For example, an external configuration file, command line arguments, environment variables, a centralized configuration management service, etc. can be used to supply dynamic configuration.

Consequently, applications **do not** need to be rebuilt and typically only require a redeployment or restart. Furthermore, applications that support configuration reloading don't require a redeployment either!

### Configuration Sources

A configuration source can be defined as anything that an application interacts with or anything that is provided to the application to initialize its configuration. Some configuration sources include:

- Command Line Arguments (CLI)
- Files (Externalized & Bundled: *.json, *.yaml, *.yml, *.xml, *.properties, *.ini, *.toml, *.conf, .env, config etc.)
- Operating System Environment Variables
- System Properties (JVM etc.)
- git repositories
- Configuration Management Services / Distributed KV Stores (e.g. Consul, Zookeeper, etcd etc.)
- Secrets Management Services (e.g. Vault, Cloud Secret Managers like AWS Secrets Manager)
- Databases
- etc.

### Configuration Reloading

Configuration reloading refers to the ability of an application to automatically update its internal configuration state without a redeployment when its configuration source is updated. This can be useful for applications that need to be highly available as it allows them to be updated without any downtime. For instance, consider NGINX which can reload its configuration without having to be restarted by sending it a SIGHUP signal i.e. `nginx -s reload`. Other examples of applications with high availability requirements include emergency services applications, financial software (trading), e-commerce apps, etc.

Depending on the type of application, however, configuration reloading may not be required. For starters, it adds a layer of complexity, the implementation can be error-prone, and it can be hard to reason about when things go wrong. Moreover, suppose that an application's configuration is changed at runtime while the application is processing some request. If the application is not designed to handle this correctly, it is possible that the new configuration will not be applied to the request, or that the request will be processed using the old configuration or even a mixture of both old and new configurations. This can lead to inconsistent behavior or even data loss.

Your experience may vary, but configuration reloading is probably unnecessary for a lot of applications, and simply reading and initializing configuration on startup should work fine in most cases. To summarize, avoid implementing configuration reloading unless there is a need for it.

Moreover, if there are very high availability requirements, you could also consider rolling updates or [rolling deployments](https://www.techtarget.com/searchitoperations/definition/rolling-deployment]. Assuming you have multiple instances of your application behind a load balancer, the idea is that you update instances gradually instead of updating all instances simultaneously.

### Multiple Configuration Sources

Typically, when using a framework like [Spring](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config), [Micronaut](https://docs.micronaut.io/1.3.0.M1/guide/index.html#config), etc. you don't have to implement or maintain the functionality to use a particular configuration source. Moreover, with clear overriding rules defined by the framework, it is straightforward to use multiple configuration sources, and you can happily mix and match as your needs dictate.

However:

> With great flexibility comes great complexity.

To better understand some of the potential issues, consider [secrets sprawl](https://www.hashicorp.com/resources/what-is-secret-sprawl-why-is-it-harmful). Hashicorp defines secrets sprawl as the distribution of secrets "littered about our infrastructure". This can lead to a lack of visibility, limited access control, and it can also make it harder to remediate breaches. Similarly, using a lot of configuration sources can make it harder to reason about the desired configuration of an application. This can be especially true when managing multiple environments and can also lead to environment drift.

### What configuration source to use?

So we've established that just because you can use multiple configuration sources that does not mean you necessarily should. So what do you use?

The [12 Factor App](https://12factor.net/config) suggests using environment variables instead of other configuration sources as they are:

> easy to change between deploys without changing any code; unlike config files, there is little chance of them being checked into the code repo accidentally; and unlike custom config files, or other config mechanisms such as Java System Properties, they are a language- and OS-agnostic standard.

In other words, environment variables are easy to change, secure, and portable compared to other configuration sources. They also avoid issues related to secrets sprawl due to their nature compared to something like configuration files which can easily be "scattered about in different places and different formats".

#### Using a distributed KV store to supply environment variables

Given that we're using environment variables, we have to consider:

1. Where and how are environment variables managed?
2. Who or what supplies them to the application at runtime?

To answer the first question, we can use a distributed key-value store such as Consul, Vault, etcd, and so on to centrally manage our configuration. The goal is to have one single source of truth for configuration.

Secondly, regarding supplying environment variables, we can delegate the responsibility to an external tool such as [consul-template](https://developer.hashicorp.com/consul/tutorials/developer-configuration/consul-template)/[envconsul](https://developer.hashicorp.com/vault/tutorials/app-integration/application-integration) to talk to the KV store and inject environment variables.

While not strictly environment variables, also consider that the application can directly talk to a configuration service to get its configuration. This requires integrating your application with the service using a library or maintaining a custom solution. Let's assume that you have a lot of applications written in multiple languages. You would then need to maintain an integration for all those applications. In contrast, when using an external tool, our application remains blissfully unaware of what configuration service is being used and the overall maintenance overhead is lower.

##### Local Development

In addition to deployment, you also have to consider local development.

Developers can simply use the same tool used for injecting environment variables during deployments by using short-lived tokens tied to their account which they can use to authenticate with the KV-Store.

**Note**: One potential approach is to check in a configuration file with sane defaults for local development and then override the configuration for deployments using environment variables. However, this means you have to support multiple configuration sources and have to deal with all the issues discussed earlier. Additionally, there is a risk of configuration drift if the developer forgets to update either the KV store or the configuration file.

## References / Additional Reading

1. "12 Factor App", Adam Wiggins, [https://12factor.net/](https://12factor.net/)
2. "Feature Toggles", Pete Hodgson, [martinfowler.com/articles/feature-toggles.html](https://martinfowler.com/articles/feature-toggles.html)
3. "What is 'secret sprawl' and why is it harmful?", Hashicorp, [hashicorp.com/resources/what-is-secret-sprawl-why-is-it-harmful](https://www.hashicorp.com/resources/what-is-secret-sprawl-why-is-it-harmful)
4. "Controlling NGINX Processes at Runtime", nginx, [docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control](https://docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control)
5. "rolling deployment", Stephen J. Bigelow, [techtarget.com/searchitoperations/definition/rolling-deployment](https://www.techtarget.com/searchitoperations/definition/rolling-deployment)
6. "7.2. Externalized Configuration", SpringBoot, [docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config)
7. "4 Application Configuration", Micronaut,[docs.micronaut.io/1.3.0.M1/guide/index.html#config](https://docs.micronaut.io/1.3.0.M1/guide/index.html#config)
8. "Service Configuration with Consul Template", Hashicorp,[developer.hashicorp.com/consul/tutorials/developer-configuration/consul-template](https://developer.hashicorp.com/consul/tutorials/developer-configuration/consul-template)
9. "Use Consul Template and Envconsul with Vault", Hashicorp, [developer.hashicorp.com/vault/tutorials/app-integration/application-integration](https://developer.hashicorp.com/vault/tutorials/app-integration/application-integration)
10. "Dynamic Configuration Management in Distributed Systems — Part 1", Abdelmoula Eya, Med Ali Jardak and Ghaida Bouchâala, [ghaidabouchala.medium.com/dynamic-configuration-management-in-distributed-systems-part-1-3218b8317b68](https://ghaidabouchala.medium.com/dynamic-configuration-management-in-distributed-systems-part-1-3218b8317b68)
