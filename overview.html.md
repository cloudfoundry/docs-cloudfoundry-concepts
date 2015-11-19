---
title: Cloud Foundry Overview
---

## The Industry-Standard Cloud Platform

Cloud platforms let anyone deploy network applications or services and make them available to the world in a few minutes. When an app becomes popular, the cloud easily scales it to handle more traffic, replacing with a few keystrokes the build-out and migration efforts that once took months.

Not all cloud platforms are created equal. Some have limited language and framework support, lack key application services, or restrict deployment to a single cloud. Cloud Foundry (CF) has become the industry standard. It’s an [open source](https://github.com/cloudfoundry) platform that you can deploy to run your apps on your own computing infrastructure, or deploy on an IaaS like AWS or vSphere. You can also use a PaaS deployed by a commercial CF cloud [provider](https://www.cloudfoundry.org/learn/providers/). A broad [community](https://www.cloudfoundry.org/community/) contributes to and supports Cloud Foundry. The platform’s openness and extensibility prevent its users from being locked into a single framework, set of application services, or cloud.

Cloud Foundry is ideal for anyone interested in removing the cost and complexity of configuring infrastructure for their applications. Developers can deploy their applications to Cloud Foundry using their existing tools and with zero modification to their code.

##Cloud Foundry Subsystems

Clouds balance their processing loads over multiple machines, optimizing for efficiency and resilience against point failure. A Cloud Foundry installation accomplishes this at three levels:

1. The [BOSH](http://bosh.io) system creates and deploys virtual machines (VMs) on top of a physical computing infrastructure, and deploys and runs Cloud Foundry on top of this cloud. To configure the deployment, it follows a manifest document.

1. The CF [Cloud Controller](./architecture/cloud-controller.html) runs the applications and other processes on the cloud’s VMs, balancing demand and managing app lifecycles.

1. The [(Go)router](./architecture/router.html) routes incoming traffic from the world to the VMs that are running the applications that the traffic demands, usually working with a customer-provided load balancer.

To meet demand, multiple VMs may run duplicate instances of the same application. This means the apps must be portable. Cloud Foundry distributes application source code to VMs with everything needed to compile and run the apps locally. This includes 1) the OS [stack](./stacks.html) that the application runs on, and 2) a [buildpack](../buildpacks/index.html) containing all languages, libraries, services etc. that the app uses. Before sending an app to a VM, the Cloud Controller [stages](./how-applications-are-staged.html) it for delivery by combining stack, buildpack, and source code into a [droplet](./glossary.html) that the VM can unpack, compile, and run. (For simple, standalone apps with no dynamic pointers, the droplet can contain a pre-compiled executable instead of source code, language, and libraries.)

To organize user access to the cloud and to control resource use, a cloud operator defines [Orgs and Spaces](./roles.html) within an installation and assigns Roles to all users: admin, developer, manager, auditor, etc. The [User Authentication and Authorization](./architecture/uaa.html) (UAA) server supports access control as an [OAuth2](http://oauth.io) service, and can store user information internally or connect to external user stores through LDAP or SAML.

Cloud Foundry uses the git system on [github](http://github.org) to version-control all source code, buildpacks, documentation, and everything else. Developers on the platform also use github, for their own applications, custom configurations, etc.

As the cloud operates, the Cloud Controller VM, router VM, and all VMs running applications continuously generate logs and metrics. The [Loggregator](../loggregator/architecture.html) system corrals this information into structured, usable form. This high-volume stream of event data, the Firehose, can be tapped in full, or else directed to specific uses by applying Nozzles---for example to monitor system internals or analyze user behavior. 

Cloud Foundry components communicate with each other internally using the [NATS](./architecture/messaging-nats.html) messaging system.

Typical applications depend on free or metered [services](../services/overview.html) such as databases or third-party APIs. To incorporate these into an application, a developer writes a Service Broker, an API that publishes to the Cloud Controller the ability to list service offerings, provision the service, and enable applications to make calls out to it.

## Who should use Cloud Foundry?

Cloud Foundry is ideal for any developer interested in removing the cost and complexity of configuring infrastructure for their applications. Cloud Foundry allows developers to deploy and scale their applications without being locked in to a single cloud. Developers can deploy their applications to Cloud Foundry using their existing tools and with zero modification to their code.


