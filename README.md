# Welcome to AAB Curriculum

## What this is

AAB is a simulated training program for teaching new developers. It is focused on automating various banking processes through different scenarios. 

## Why this is

As software engineers, we find that learning automation skills to be crucial. We find ourselves piping data around from one system to another, and providing business values in taking that data, transforming it, kicking off other processes, and connecting 3rd parties together. 

We are not concerned with the programming language of choice in this curriculum. It was originally developed with .NET in mind, but can be done by any language that supports HTTP/REST.

"Real World Application" makes it feel more real. The scenarios are designed to be real business use cases. This living system gives the student an incentive to learn because they actually get to make a difference in this simulation. Given enough time, they will also experience creating systems that depend on other systems that they create. This gives them real experience in working with living systems and seeing upstream/downstream data.

## How this works

This GitHub organization contains all of the information you need to get started. 

There are various repo's here that represent the existing infrastructure that the simulation runs in. 

- BankersChoice
    - This is the primary Banking 3rd party service that is used. It contains all of the banking information, including accounts and transactions. It also allows updating data via a locking mechanism 
    - REST interface
    - Webhooks
- PersonablePeople
    - A CRM-like system that contains information about customers, employees, and leads
    - REST interface
    - Webhooks
- Checkers
    - A 3rd party check clearance system. Handles checks being cashed or transferred.
    - REST interface
- Verified
    - A customer verification system. Will return in some time for a request.
    - REST interface
        - No webhook, but expects a url to hit
- MailGrid
    - A simple email service with a templating engine
    - REST interface

Please note that NONE of these system talk to each other. They are all owned and operated externally by different 3rd party vendors. Currently, any work that needs to move from one system to another is done manually by hand (i.e. if a check clears in Checkers, an accountant has to type the transaction into BankersChoice). The point of this simulation is to work through scenarios that create automation between these systems. 

In addition to these 3rd party applications, there are a variety of generator applications. These generators power the simulation. They interact with the various 3rd party applications to create activity and give the illusion of working in a live, active system.

- BankersChoice-Live
    - Creates and updates accounts
    - Creates transactions
- PersonablePeople-Live
    - Creates and updates people
    - Converts leads into customers
- Checkers-Live
    - Creates cashed checks

All of these applications exist as .NET Core 3.1 applications (simply because that is my framework of choice). Hopefully you won't need to edit any of these applications yourself. The expect to run inside of a Kubernetes cluster. Each application has a Helm chart in the aab-helm repository. 

## Installation

1. Have a Kubernetes cluster
    - I am not going to go into how to set this up. Use a cloud provider or if you know what you're doing build one yourself. I assume you have a cluster up and running. This was tested using a DigitalOcean Kubernetes cluster with 2 $10 nodes.
    - I would recommend a dedicated cluster for this simulation. Please don't install these helm charts into your prod cluster, and I can not be held accountable for damage that may do.
1. Install an API Gateway that supports prefix based routing
    - I recommend Ambassador Community because it is free, it works well in Kubernetes, and the free edition supports all the needs of this simulation.
1. Install the 3rd party applications
    - <TODO> insert command line instructions here once I have them
1. Configure your API Gateway with these settings
    - <TODO> ...the settings
1. Install the generator applications
    - <TODO> insert the command line instructions here