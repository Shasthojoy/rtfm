---
layout: post
title: A revised approach to application & infrastructure deployments
author: Ben Fairless
---

A revised approach to application & infrastructure deployment
=============================================================

Over the last few weeks the Web Operations team has been working to try and define a standardised method for running applications in a live environment for Digital by Default services.

Currently, there is a lack of visibility for the development teams of what their Pre-Production and Production regions look like, what versions of their applications are deployed into those regions, and timing application releases so that compatible versions are deployed together. Making corresponding infrastructure changes necessary to implement new service features in a timely manner is also difficult.

As we move into developing a more mature, highly-available platform, such limitations will become more problematic and hinder the delivery and stability of our services.

In order to eliminate these issues we have started on a path to deliver an improved solution, allowing easy deployment and providing more control and flexibility to development teams.



## The old setup - a monolithic Puppet configuration

![Diagram of old Puppet setup]({{ site.baseurl }}/assets/old-puppet.svg)

Our current configuration is relatively monolithic, with the majority of the infrastructure configuration being contained within a single code repository.

There is a branch in version control for each application region. When changes are pushed to a branch, they are then propagated to all servers in the equivalent region. All services share a central configuration.

All application releases are managed independently and pulled straight from version control, deployed through Jenkins, and manged through various shell scripts.

Our focus in developing this solution was to provide as much shared configuration as possible; unfortunately, due to our limited smoke testing, this also means breaking as many things as possible when a change is made.


## Using modular Puppet & Hiera configurations to deploy applications

![Diagram of new Puppet setup]({{ site.baseurl }}/assets/new-puppet.svg)

Our proposal for a new configuration takes a far more modular approach to infrastructure management. Instead of having one centrally-managed configuration, the setup will be segregated with each service having it's own self-maintained infrastructure configuration. Instead of using branches to represent application regions the configuration will be versioned and a specific version or commit can be deployed to any particular region when appropriate.

To provide better visibility and control of application releases this can also be managed through the Puppet configuration. We are actively working on providing custom Puppet types to manage application deployments (initially focusing on Python WSGI applications).

The idea of managing server configurations may seem uncomfortable to some developers but our focus on building supporting modules should provide a suitable level of abstraction so that developers can easily achieve the tasks of releasing new application versions and setting environment variables without necessarily concerning themselves with needing to manage a highly-available setup or meeting security requirements.

The majority of the configuration is easily managed through Hiera's hierarchical data structure and human-readable YAML documents. Below is an example of a web server configuration:

```yaml
---
    wsgi_applications:
        service-frontend:
            port: 5000
            revision: 1.1
            variables:
                API_URL: 'http://localhost:5001/'
        service-api:
            port: 5001
            revision: 1.2
            variables:
              SQLALCHEMY_DATABASE_URI: 'postgresql+pg8000://admin:password@localhost:5432/sample_data'

    postgres_databases:
        sample_data:
            user: admin
            password: md5b84f21541ef11f3e5d07840ed6f98cd9
            owner: admin

```
