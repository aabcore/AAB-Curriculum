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
    - I recommend Gloo Community because it is free, it works well in Kubernetes, and the free edition supports all the needs of this simulation.
1. Install the 3rd party applications
    - <TODO> insert command line instructions here once I have them
1. Configure your API Gateway with these settings
    - <TODO> ...the settings
1. Install the generator applications
    - <TODO> insert the command line instructions here
    
## Adding a MongoDb user

Each application uses a Mongo database as it's persistent storage. As such, each application needs its own user with access to its database, in each environment.

- Shell into the expected mongo pod (https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)
    - `kubectl exec --stdin --tty --namespace aab-prod  mongodb-prod-7745bdb6f6-g8j5h  -- /bin/bash`
- Log into mongo as root. The password is set a secret in that pod generated by the image
    - `mongo -u root -p`
- Use the admin database
    - `use admin`
- Add the user. The user needs to be able to read admin, and read/write in the specified database. [TODO] Check that read admin is actually required
    - `db.createUser({user: "BankersChoiceApi", pwd: "WNHmwjKyapILd411IFE", roles: [{role: "read", db: "admin"},{role: "readWrite", db: "BankersChoiceDb"}]})`

## Adding a credential secret

(https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

To keep applications from containing secrets, and so they can be deployed multiple times in different configurations, Kubernetes secrets are used to set credentials and connection strings.

- Create the yaml file. Values of the data must be in Base64. Be sure to use the correct namespace
  - The connection string needs to be in the format `<<name of service>>.<<name of namespace>>:27017` to internally point to the correct instance.

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: mongodb-cred
    namespace: aab-dev
data:
    username: "dGhpcyBpcyBhIHRlc3Qgc3RyaW5nIGlmIHlvdSBwbGVhc2U="
    password: "dGhpcyBpcyBhIHRlc3Qgc3RyaW5nIGlmIHlvdSBwbGVhc2UgMg=="
    connectionstring: "dGhpcyBpcyBhIHRlc3Qgc3RyaW5nICA="
```

- Apply the secret to your cluster
    - `kubectl apply -f secrets.yaml`


## Add an application

- Assuming the app and helm charts are built and distributed, and secret has been generated and applied
- `helm install --namespace aab-dev bankers-choice aab/aab-bankers-choice`

## Update an application
- `helm repo updage`
- `helm upgrade bankers-choice --namespace aab-dev aab/aab-bankers-choice`

## Add app to gloo

- If virutal service hasn't been created (this should only have to be done once per environment)
    - `glooctl create virtualservice aab-dev --display-name aab-dev --domains prod.aab.*`
- Find the upstream
    - `glooctl get upstreams`
- Create the route to the appropriate upstream
    - No prefix rewrite means the app has to know ahead of time what path base it can use (`app.UsePathBase("/bankerschoice");`)
    - `glooctl add route --path-prefix /bankerschoice/ --dest-name aab-dev-bankers-choice-aab-bankers-choice-80 --name aab-dev`
