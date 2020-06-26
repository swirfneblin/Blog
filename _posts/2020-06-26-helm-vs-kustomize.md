---
layout: post
title:  "Administrar complexidade entre Helm e Kustomize"
date:   2020-06-26 13:40:00 -0300
categories: 
  - helm
  - kustomize
---

# Helm vs Kustomize: Managing Complexity

> How Helm Charts and Kustomize enable you to manage complexity.

When [Kustomize](https://github.com/kubernetes-sigs/kustomize) came into the spotlight conversations quickly moved to compare it to [Helm](https://helm.sh/). We have it in our nature to debate tools. Just look at the long standing Vim vs Emacs debates. While these tools are really tangential to each other, there are elements to this discussion worth bringing to the surface and thinking though. One such worthy area is to look at who has to handle which parts of the complexity.

Kubernetes Is Complex
---------------------

Before we talk about managing complexity we need to look at the complexity itself.

Earlier this year there were a number of posts and conversions on social media about Kubernetes complexity. There was [an article in The New Stack that briefly covered it](https://thenewstack.io/has-kubernetes-already-become-too-unnecessarily-complex-for-enterprise-it/). There is a nice section in the article that hits home with the situation:

> “Kubernetes was designed by systems engineers, for systems engineers,” stated Kate Kuchin, an engineer with Heptio, during the last KubeCon. “Which is great, if you’re a systems engineer. For the rest of us, Kubernetes is really, really intimidating.

Not everyone is a systems engineer. Not everyone needs to think about systems engineering problems. Julia Evans, on the Stripe blog, did an excellent job highlighting this thinking in her blog post titled [“Learning to operate Kubernetes reliably”](https://stripe.com/blog/operating-kubernetes). In the section _Making cron jobs easy to use_ she notes:

> Our original goal was to design a system for running cron jobs that our team was confident operating and maintaining. Once we had established our confidence in Kubernetes, we needed to make it easy for our fellow engineers to configure and add new cron jobs. We developed a simple YAML configuration format so that our users didn’t need to understand anything about Kubernetes’ internals to use the system.

The engineers working on business logic who needed cron jobs didn’t need to know anything about Kubernetes to run them. Their scope isn’t systems engineering. Stripe came up with a solution that hid Kubernetes knowledge away to make it simpler for those engineers. They managed who had to deal with which parts of the complexity. **Application engineers didn’t have to deal with systems engineer tasks or the knowledge to do them.**

Everyone Is Trying To Manage Complexity
---------------------------------------

[DHH](http://david.heinemeierhansson.com/), the creator of [Ruby on Rails](https://rubyonrails.org/), recently gave a keynote at RailsConf. [His tweet, with a link to the video on it, highlights what the talk was about](https://twitter.com/dhh/status/996541891020640257):

> My RailsConf 2018 keynote about conceptual compression, liberating the best ideas from the grasp of complexity, the beauty of leaky abstractions, how our industry is backsliding, and facing alienation from the product of our labor. – DHH

The keynote directly targets managing complexity. Managing complexity is a common problem that’s not unique to Kubernetes.

I like to think of it in terms of a separation of concerns. Who needs to be concerned with what? Chip designers aren’t concerned with with app business logic and vice versa. There are different concerns that all apply to making the same thing work. Some concerns are far enough away we may not even think of them.

In the cloud space there are some more practical example separations that reduce complexity:

1.  If you use an IaaS to manage your infrastructure you don’t have to be concerned with racking and stacking of hardware. There is no need to think about network cables, switches, cooling, or those other concerns. The complexity and knowledge to handle it has been cleanly separated into layers that communication over an API
2.  Functions as a Services, a subset of serverless, also push on this separation. Serverless isn’t really server less because the code has to run somewhere. But, for business logic developers it is a case of _I don’t need to be concerned with servers_. A separation of concerns has been clearly defined and an API put in place to manage communication

In both of these cases complexity is managed by separating concerns and defining channels of communication. **Those on each side of the separation can focus on their part of the problem.**

Application Complexity
----------------------

On the Helm website under the section _“What is Helm?"_ we have our basis for the conversation on complexity.

> Helm helps you manage Kubernetes applications

The basis for a discussion on complexity management with Helm and Kustomize needs to start with applications. Managing the complexity for operating Kubernetes itself is an entirely different issue.

When it comes to managing complexing we should look at who needs to manage it. Here are a few roles:

*   _Application developers_ - the people who write the business logic for an application. The applications they write could be run in Kubernetes, on a VM, or somewhere else. Their focus is on the business logic
*   _Application operators_ - those who need to operate an application in an environment
*   _Environment developers and operators_ - the systems engineers who build and operate an environment like Kubernetes, Mesos, or even a public cloud

How much complexity do different people need to know. Do application developers need to know much, if anything, about Kubernetes? Do systems engineering handling the environment need to know how to write a modern we app? If we manage the complexity well we can separate the concerns.

### MySQL Complexity Example

Examples help this to hit home and a common example is the long standing MySQL. MySQL is a great example because it is in wide use, can run in a container, VM, or on bare metal, and fits nicely with our next example.

To operate MySQL in Kubernetes is the job of the _application operator_ role. This person needs to know both business logic for the application and how the environment (Kubernetes) works.

When it comes to the business logic they don’t need to know everything. But, they do need to know how to configure it including handling things like replication, backups, and performance tuning.

To operate MySQL in Kubernetes they don’t need to know everything about Kubernetes. Rather, they need to know about a handful of features around running workloads, using storage, and exposing services. They may also need to know how to handle running the application in different types of environments. You can’t easily run MySQL the exact same in minikube as you can in an HA environment.

Their expertise is where the two things come together. A bit of the layers above and below them in the stack.

### Complexity Of Operating WordPress

WordPress is a common example people are familiar with. It also depends on a database allowing us to look at the next level of complexity.

Like MySQL, an _application operator_ needs to know the business logic of WordPress and the workload features of Kubernetes to make it run. **But, WordPress depends on a database.** That means someone needs to know the business logic of running the database and how to run it in Kubernetes. Is that the same application operator that’s handling WordPress? Is there a way to encapsulate the complexity for a database, like MySQL, so the WordPress application operator doesn’t need to be a MySQL operator as well?

With something simple, like WordPress, it can be easy to put it all on one person. But, what if the system has 10, 20, or more moving parts? The explosion of microservices means this is not uncommon.

Helm, Kustomize, and Complexity
-------------------------------

Managing knowledge and complexity are a bit different between Kustomize and Helm. With all of this background we can now talk a little about them.

### Helm

Helm encapsulates the components needed to operate an application into a package called a chart. The interface to the chart is a set of properties that can be passed into the chart at the command line or in a file. Helm charts are [semantically versioned](https://semver.org/) based around the API change to this values file.

In a chart, the business logic for the application and operational knowledge for Kuberentes are bundled so that the consumer of the chart does not need to know those details.

Helm charts can have dependencies on other charts. For example, a WordPress chart can depend on a MySQL chart. It does so using version ranges. The parent can capture properties and pass them to the dependency. Dependency handling logic, such as a decision to use a SaaS database or a dependent chart, can be handled through this interface as well.

Helm charts provide a means of handling complexity by separating the concerns into packages with an API. The flexibility is there to handle situations such as running WordPress in a public cloud while using a SaaS in production and letting a developer in minikube use a locally run MySQL server via a dependent chart.

Helm intentionally focuses on application operation, separation of concerns, and managing complexity.

### Kustomize

Kustomize takes an entirely different approach. Since Kustomize is not a direct competitor to Helm this isn’t the case of one should win out over the other. It’s a matter of knowing how a tool does things to know when to use it.

The tag like for kustomize on GitHub hints at the purpose:

> Customization of kubernetes YAML configurations

The introduction to the Readme continues this idea where it says:

> `kustomize` lets you customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is.
> 
> `kustomize` targets kubernetes; it understands and can patch kubernetes style API objects.

With the focus on Kubernetes and Kubernetes style API objects it’s worth asking who that is for? Helm noted it starts with application management while kustomize starts with an API object style. Not telling us who the user is means we have to try to work that out.

In our earlier list there are two obvious roles who are concerned with these types of file. They are the _application operators_ and _Kubernetes operators_. Since we are focused on application operation here I’m going to focus on that.

If an application operator has to manage their application they have to work with Kubernetes API objects and kustomize is a powerful tool to do that. **But, this is about managing complexity and not about the direct ability to work on YAML.**

To manage complexity we can look at a couple cases.

#### Dependency Handling

If we look at the WordPress example and have a dependency on a database how would that be handled? Kustomize works on Kubernetes API objects so you have those for the dependency in raw long form. That means a database dependency has one directly working with the raw configuration for a dependency.

To make changes to a dependency you need to understand the business logic for it and to change the YAML accordingly. Kustomize provides the tools to patch the YAML files and to patch them differently for different environments.

**The way kustomize works exposes you to all the raw complexity of the dependency tree.** Instead of encapsulating the complexity it puts it all on display and enables you to make changes to any of it.

For some, this is an exciting capability. If we’re looking to manage complexity it doesn’t so much help us. For some people in some situations this is ok. That’s an organizational decision.

#### Handling Situational Logic

We already noted the case of using a database as a service when running in production and running a local database for local development. Doing this requires different configuration. In one case your application needs a URI for a database and in other situation we need the Kubernetes API objects.

Handling these differences with kustomize is currently an exercise for the application operator to work out with other tools and processes. Encapsulating these situations is not handled.

This, again, highlights that kustomize isn’t providing a means to encapsulate complexity but rather puts it all on display and looks for other elements to manage that.

This also highlights how Helm and Kustomize are not direct competitors. They do different things and do them in different ways.

Summary
-------

I believe it’s worth an organization looking at how it manages complexity. A goal should be to make things simpler both in terms of daily working complexity and in terms of how much complexity is being worked on at a given time.

There may come a time where we work out how to have kustomize and Helm work together. This is a possibility.

Personally, when it comes to managing application operation I would use Helm charts. It provides a nice way to encapsulate complexity and enable it to work with other parts with higher level applications. Though, I am biased as I am a Helm maintainer. So, take it with a grain of salt.


[Source](https://codeengineered.com/blog/2018/helm-kustomize-complexity/)