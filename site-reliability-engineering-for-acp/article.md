# Site Reliability Engineering for the Apprenda Cloud Platform

## Summary
Site Reliability Engineering (SRE) is a notion originally developed at Google in the early 2000s.  SRE is the practice of using software engineering to apply automation to traditionally operations-centric, and often manual, activities.

 > Fundamentally, it’s what happens when you ask a software engineer to design an operations function [^1]

Much more information about the origin and evolution of SRE is available in the article referenced above.

Around the time that Google developed the SRE practice to support internal operations (the evolution of `borg`, `borgmon`, and other standards), initial R&D functions began at Apprenda on our cloud platform.  Within months we began engaging with enterprise customers who rolled out the platform in some very large operating environments.  One uniform challenge that all customers faced is the transformative nature of not only the platform technology, but the ecosystem and culture of operations around it.  Enterprises have decades of operational "baggage." From investments in existing tools to long-established workforce skill sets.  The systems and people impacted by paradigm shifts such as PaaS and cloud-native (containers, etc.) are often perceived as disruptive and met with friction.  However, through the right lens and with the right approach they are also an opportunity to course correct and build an IT organization that is ready to support app development and innovation well into the future.  Frankly, the industry trends fueling software development have outpaced traditional IT shops' ability to provide and support, and today's typical IT organization has some catching up to do.  This is something that Google recognized early on, and the resulting SRE practice and culture is a model that all organizations looking to introduce new services and consolidated platforms should embrace.  It is broadly applicable and adaptable.

At Apprenda, we've taken the Google foundation for SRE and adapted it for the Apprenda Cloud Platform.  This paper explains the key tenets of a holistic SRE approach leveraging ACP features inclusive of tangential systems and common requirements our customers have.  The result is a set of guidelines for how to build and operate an __IT function capable of and dedicated to ensuring reliable first-class platform services to application developers__.  Where appropriate, we've included specific technology recommendations and examples.  The beauty of the Apprenda approach is that due to the nature of our cloud platform's technological capabilities, many of the practices established at the onset can address the needs of existing ("legacy") apps and new cloud-first apps.  The concepts introduced in this paper apply whether you are undergoing an app migration project, building a cloud-first platform and/or moving to public cloud, or one of the many other use cases for ACP.

## Introduction

### The Need for SRE and Apprenda in the Enterprise
Over several years of implementing the Apprenda Cloud Platform in many large enterprise environments, we've observed the need for increased controls, automation, and response capability for various infrastructure and platform conditions.  While the industry-specific parameters may change between implementations, the pervasiveness of regulatory requirements and corporate control has a negative impact on service delivery that is amazingly consistent.  That's not to say these things are unimportant, these are the laws that are designed to keep our data safe.  As such, our approach to SRE is inclusive of these types of considerations.  We consider that the concept of reliability engineering needs to be accessible to the typical IT shop that doesn't have the luxury of building a practice from the ground up.

The needs that SRE addresses are not unique to ACP, but pertain to all platform initiatives (_X_-aaS) that are part of the enterprise IT landscape.  By their nature, such initiatives put IT in the role of service provider.  Despite the aforementioned necessary controls, there is no reason an internal service provider should act with any less agility than a startup building their operations for the first time.  Enterprise services have needs that are very much inline with SRE tenets of increased automation and allowing software to manage the influx of information that comes from large systems.  There is a common misconception in IT today that "control" and "human intervention" are synonymous, yet often times it is the manual and repetitive nature of current approaches, along with separation of responsibilities (i.e. "red tape"), that introduce complication, delays, and mistakes which cause an organization's IT functions to be unnecessarily slow and error prone (SRE refers to this as "toil").  The opposite of why the controls exist in the first place.  Arguably, the _only_ way to solve these issues is through the removal of manual activities, remove toil, under a shared discipline model. _Note: Although not the primary audience for this article, this point should not be overlooked by executives tasked with evolving software development practices and creating environments that foster innovation and transformation!_

Corroborating Google's initial intent for SRE, we've found that organizations who embrace operational automation learn that reliability in controls and the ability meet onerous requirements increase, while issues such as system outages and security violations decrease.  The way to achieve this is through software engineering.

### Apprenda's Cloud Platform and Site Reliability Engineering
At this point, we've reviewed SRE's historical context and made a case for the approach in today's enterprise using Apprenda's experiences as verification.  Let's now turn towards the Apprenda Cloud Platform PaaS itself.

Apprenda's stated product goals include "increasing developer productivity while lowering operating burden for IT."  To achieve this, the Product Management and Engineering teams at Apprenda have taken great care to incorporate feature sets into the platform that clearly serve the ability of both developers and operators to automate.  On the development side this means CI/CD pipelines and automated deployment through APIs.  On the platform operations side, this means the ability to mitigate issues automatically both within the platform itself and through integrations with external tools (there are examples of both in this paper).  ACP exposes APIs and produces data that make it possible to build an effective SRE function around the platform.  Some examples:
* [__Bootstrap Policies__](http://docs.apprenda.com/6-0/bootstrap-policies) - apply processing modules, engineered by SREs, that operate against application workloads that are in flight to their execution nodes.  Transform, validate, or even block the deployment based on requirements.
* [__Application Deployment Policies__](http://docs.apprenda.com/6-0/app-deployment-policies) - apply rule sets to ACP's deployment decision engine to ensure that ACP executes application bits in a safe, secure, and compliant manner on machines that meet the needs of the application.
* [__Platform Operations API__](http://docs.apprenda.com/restapi/platformops/v1) - retrieve information from the core ACP data stores about current platform state.  Use this information to take informed automated actions including updating platform configuration in real time, also using the API.
* [__Health Sweeps__](http://docs.apprenda.com/6-0/Managing-Apprenda-Infrastructure#health) - ACP executes periodic health checks against all nodes.  Nodes report back information they collect about their surroundings - disk space warnings, network reliability, database access, and more.  This information along with external infrastructure monitoring is invaluable in running a reliable infrastructure environment for ACP and its guest apps.

There are many more.  It is important to understand that SRE with ACP is about creating a function whose responsibility it is to utilize ACP product features along with external tooling and custom software to create solutions to everyday operational tasks.  

For example: If a node's disk nears capacity, ACP will detect this during a _health sweep__ and automatically put that node into _Reserved_ mode, which prevents future deployments from reaching that node. The ACP API will report two things: a) the node failed a health sweep, b) the node was placed in reserved mode automatically.  _ACP has already taken automatic steps to prevent further issues_.  An SRE solution should react by corroborating this information (`/NEED NODE HEALTH CHECK ENDPOINT HERE`) with infrastructure monitoring tools and executing the cleanup or expansion of storage on that node before instructing ACP to put it back into the __Online__ node pool (`PUT /api/v1/hosts/{hostname}/Online`).  __SREs should consider ACP an active participant in any solution they design and implement__.

### Make SRE Someone's Job
Long ago we developed a practice at Apprenda of establishing a [_Center of Excellence_](https://apprenda.com/blog/how-to-staff-paas-excellence-center/) within our client projects. Members of the client _COE_ are trained on administrative workflows and operational support for ACP, application ingress processes for app on-boarding, how to evangelize PaaS internally, and many other things.  We now make strong recommendations that our clients establish site reliability engineering as part of their COE practice, and we offer coaching frameworks (expanding on this paper) to show them how to do it.  We find that the best stewards of organizational transformation are SREs who have exposure to both the engineering side of the organization and the operational, or system administration, side.  By definition, site reliability engineers bridge that traditional gap that has existed between the app developers and system operators.  This also aligns with Google's prescription for ideal SREs.  It is highly recommended that an SRE function is created and assigned to one or more individuals as a full time job at the onset of platform implementation.  In the nascent stages of the project these
individuals will likely spend more time on operational tasks such as system configuration and setup, but as the implementation progresses they will parlay this experience into a focus on engineering solutions.  By participating in the initial build out, their knowledge of the fundamental platform stack will grow to include:
* Underlying infrastructure systems: networking, storage, virtualization, operating systems
* System integrations: I&AM, CMDB, file storage, SDN
* Developer tooling: IDEs, CI/CD pipeline tools, build systems
* Internal policy: Password policies, certificate issuance, security

These are all entities within IT that represent direct or indirect dependencies to the SRE function.  As SREs transition from the early stages of the project they will align more with the intent of spending the bulk of their time writing software.

## Reference Architecture

Now we'll introduce a reference architecture that we've used as a baseline for initial SRE with ACP.  _Note:_ The specifics of the platform implementation internals (# of worker nodes, cache cluster, databases) are intentionally omitted.  This information is unimportant to the conversation about how to operate the environment because any monitoring and response solution should be scalable enough to deal with variances in the platform implementation.  As platform adoption grows, so will it's usage and capacity, and our reference architecture here should remain unaffected.

![ref-arch-monitoring.png](https://github.com/mammerman/docs/blob/master/site-reliability-engineering-for-acp/ref-arch-monitoring.png)

The approach is to use a set of micro services, each responsible for one particular piece of information.  They leverage the ACP Platform Operations APIs to poll for data at a regular interval.  We set these up as Docker containers and use `crontab` on a remote machine to start the containers at regular intervals.  Each micro service does the following:
1. Startup
2. Load environmental configuration (API urls, etc.)
3. Connect to Apprenda API endpoint and retrieve current data
4. Parse data and construct a corresponding [Prometheus](https://prometheus.io) metric object
5. Push the metric object to a configured `pushgateway`
6. Clean up and shut down

The jobs are short-lived, existing only for a couple hundred milliseconds.  The data they push to Prometheus becomes part of the _time-series_ for that particular metric and the rest of the analysis and reporting is handled by Prometheus and other tools.

### System Availability

### Monitoring, Alerting, Logging, and Auditing

### Situational Response

### Capacity Planning

## Availability of the Platform

## Monitoring, Alerting, Logging, and Auditing

An important aspect of monitoring a distributed system such as ACP is watching the platform's perception of reality over time.  We watch this not only to detect anomalies, but to uncover scenarios where the platform's perception of reality doesn't match what's really happening, or the platform isn't able to discern symptoms from root causes.  These types of situations almost always arise from forces outside of ACP, such as changes in integrated systems (I&AM, networking, group policy, etc.)  Often times these tangential systems are managed by disparate teams, and it's a fact of life that changes will occur in these systems without warning. ACP has built in node health sweeping to protect it against unpredictable culprits.  It will, for example, proactively put nodes in reserved or maintenance modes if health reports dictate that they cannot take new workloads.  Our SRE guidelines take that one step further by observing this activity, corroborating it with known environmental issues, and taking corrective or compensatory action.

We do this by using _time series data_.  At regular intervals, monitoring systems pull data from ACPs APIs.  These monitoring systems are custom microservices hosted outside the platform. The data they pull is flowed into a time series store.  We use [Prometheus](http://prometheus.io) to handle this information because we can then analyze the data over time, which is important as there are scenarios where temporary conditions are expected. [Prometheus](http://prometheus.io) lets us then build multi-dimensional reports against the data we've captured.  If we're looking to identify issues with a particular node, or a combination of multiple nodes, we can do that; and then we can use these reports to generate real-time dashboards and feed into alerting systems.

One added benefit of using [Prometheus](http://prometheus.io) is that it works very well with Kubernetes also, and aggregating Kubernetes cluster data with ACP cluster data and application metrics gives us a unified holistic picture of our platform for existing/legacy (brownfield) apps and cloud native (greenfield) apps, which is what Apprenda is all about.

#### Alerting

Alerting from this system is quite practical due to the fact that we are dealing with time-series data.  Given the data captured and stored by [Prometheus](http://prometheus.io) as part of our monitoring activities, we can build an entirely separate and flexible alerting model.  This is important because we don't want to be overly aggressive with our alerting (waking up on call support personnel for non-severe incidents, for example), but we also need to ensure that issues are treated with the proper urgency and emergency response is swift and effective.  The [Prometheus](http://prometheus.io) [Alertmanager](https://github.com/prometheus/alertmanager) allows us to build rules for alerting that process inbound alerts from [Prometheus](http://prometheus.io) and make smart decisions about what to do with them.  For example, we can silence certain alerts if other conditions are met.  We can group alert notifications to prevent notification spam.  Some conditions in ACP are cause for concern only after they've persisted for a certain amount of time - for example, a node that has been put into _Maintenance Mode_ might actually be undergoing maintenance.  We don't want to raise urgency or take action unless we are crossing a maintenance window threshold.

## Situational Response

Interestingly, SRE isn't entirely about _preventing_ failure, and in many cases failure is expected and tolerable.

#### Postmortems and Case Reviews

## Capacity Planning










A model for monitoring and alerting.

##### Monitoring Key Platform Metrics with Time Series Data


##### Alerting





### Some Examples
##### Controls
A site reliability engineer might create a Bootstrap Policy [^4] that scans application files during deployment.  A traditionally manual process using 3rd party services, our approach uses Bootstrap Policy code to bring the process inline as part of ACP's application deployment pipeline.  The end result is the real-time success or failure (aka control) of an application deployment based on discovered characteristics of the app (with informative messaging back to the developer about why the app was rejected and what they need to fix).

##### Monitoring

##### Taking Action

## Conclusion

The purpose of this paper is to introduce the reader to the concept of site reliability engineering, how it applies to enterprise IT, and how Apprenda's Cloud Platform is an active participant in the SRE function of a PaaS project.


[^1]: https://landing.google.com/sre/interview/ben-treynor.html
[^2]: https://apprenda.com/blog/how-to-staff-paas-excellence-center/
[^3]: http://docs.apprenda.com/swagger/platformops/v1/endpoints.html
[^4]: http://docs.apprenda.com/6-0/bootstrap-policies
[8b1ab4f7]: http://www.[Prometheus](http://prometheus.io).io "[Prometheus](http://prometheus.io).io"
