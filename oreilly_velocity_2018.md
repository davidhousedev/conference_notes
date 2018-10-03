# Kubernetes 101
## Training resources
* [Excellent github repo with container training resources](https://github.com/jpetazzo/container.training)
	* Contains slides and tutorials for learning containers and container deployments
	* Slides for my training, and the more expensive one, are both hosted at the HTML host of the slides: https://container.training/
* [Slides from my talk](https://velny-k8s101-2018.container.training)
* [Play with Kubernetes](https://training.play-with-kubernetes.com/)

## Major Takeaways
* Benefits of using k8s
	* Robust tooling for zero-downtime deployments
	* Handles significant distributed system complexity
	* Immense, active, open-source community
* k8s-as-a-service is an excellent first step into the platform (e.g. EKS, AKS)
* k8s can be run with plugins to simplify maitenance and configuration

## K8s Features
* Start 5 containers using image atseashop/api:v1.3
* Place an internal load balancer in front of these containers
* Start 10 containers using image atseashop/webfront:v1.3
* Place a public load balancer in front of these containers
* It's Black Friday (or Christmas), traffic spikes, grow our cluster and add containers
* New release! Replace my containers with the new image atseashop/webfront:v1.4
* Keep processing requests during the upgrade; update my containers one at a time

### Additional features
* Basic autoscaling
* Blue/green deployment, canary deployment
* Long running services, but also batch (one-off) jobs
* Overcommit our cluster and evict low-priority jobs
* Run services with stateful data (databases etc.)
* Fine-grained access control defining what can be done by whom on which resources
* Integrating third party services (service catalog)
* Automating complex tasks (operators)

## Additional resources
* [Rancher](https://rancher.com/), a SaaS control for k8s

# Intro to AWS Lambda and API Gateway
Slides: https://docs.google.com/presentation/d/17n5maGxIxH2JLaL6rs6fudaW6rUpJ2ljusN0FqOGQGA/edit
## Major Takeaways
* AWS Lambda works well for many solutions, but there are a couple circumstances that are not well-suited:
	* Ultra-low latency requests (e.g. `<100ms required`)
	* Long running requests (lambda cost = memory req'd X execution time)
		* a lambda function is cut off at 5m, always
* Benefits
	* No servers to maintain
	* Autoscaling out of the box
	* Very inexpensive, especially for an architecture well-suited to lambda (e.g. full implementation of API gateway, not `zappa`)
* Drawbacks
	* Code reuse can be challening
	* Dependency versioning becomes difficult
	* API Gateway is a pain to configure
* [`serverless`](https://serverless.com/) framework makes development easier, speaker has been using it for years. Similar to Terraform, for serverless specifically
	* deploys are easier
	* invocation is easier

## Lambda
* Execution environment
* AWS provisions your environment using containers
* Containers must spin up, some requests take longer
* Supports many different languages, python included

## API Gateway
* Very complicated abstraction of API logic
* `serverless` and Terraform might help with complexity?
* Built to be the API platform for AWS, but mostly handles Lambda
* Can be used to ease gradual migration to serverless
	1. Place API Gateway atop current load balancer
	2. Migrate a part of your app to serverless
	3. Migrate traffic using API Gateway to your lambda function, away from your app
	4. Repeat 2-4
	5. ...profit!

# Tuesday Keynote
## Anil from Glitch.com
* At the beginning of the internet, everyone could create technology
* With the increase in sophistication, the number of people who can build has decreased
* Barriers fall along privilege/marginalizaiton lines
* Development teams can communicate their values by enabling individuals to contribute

## Changing people systems
* Learning how to program teaches system-level thinking
* It is challenging to learn how to bring those skills out of technology systems, into people systems
* Find a people system to change, using your systems thinking skills

## Risk management with Kris Beevers, NS1
* Invest in risk mitigation where `cost of failure x change velocity` is highest

## Site Reliability Engineering with Dave Rensin at Google
* SRE has the same principles as DevOps
* Site Reliability Engineering is an opinionated implementation of DevOps
* "Reliability is the single-most important feature of your system"
* Reliability => User trust, only from the user's perspective

## Kavya, applying performance theory to practice
* A `web server` can be modeled as a `queuing sytem`, an open system
* However, performance/load testing tools send traffic like a closed system, with a fixed number of clients
* For scaling systems, avoid contention (serialization) and crosstalk (synchronization), prefer just contention over contention and crosstalk


# Terraform with Healthcare.gov
* Manage all infrastructure with one tool
* Terraform benefits
	* Can incorporate manual changes
	* Much ~10x faster than AWS CloudFormation
* Use shared modules sparingly
	* Modules within a single app can be reused easily
	* Modules shouldn't be shared between discrete applications
	* e.g. DART shouldn't reuse a terraform module for a Canvas LTI
	* Use an application template to reuse configuration between applications, not shared modules
* Reuse constants, not infrastructure
	* e.g. Share VPN configurations, not a load balancer configuration
* `terraform import` CLI is limited, consider use `dtan4/terraforming` instead
* Lock your terraform version in the configuration!
* Terraform should be applied in a CI/CD tool
	* Single source of truth for updates
	* Risk losing internet connection mid-apply
	* Consider using Hashicorp `atlas`
* Use precommit hook `terraform fmt` to style your terraform files
* CI/CD Flow utilized
	1. CI would wait for input
	2. Developer input required to apply plan in CI
	* We should probably do this by committing the plan file
* It's unclear how to test a terraform configuration
* Blue/green deployments are managed using Terraform, in ELB settings; new EC2s are brought up and traffic is shifted when they pass health checks

# Microreleases with Grubhub
_This talk had some very helpful slides but they were not made available immediately._
* Service discovery is essential for microservices deployments
* Synchronous requests
	* When releasing services, specify what percentage of traffic the new version should receive
		* e.g. version 2.0 should receive 1% of traffic, 1.9 should receive 99%
	* New services can be released with 0% traffic
		* CD platform allows you to run the app against those 0% traffic containers
* Async actions
	* Message router sits above message queues
	* Message router examines service discovery percentages
	* Sends messages to queues according to service discovery percentages
	* CD platform can target 0% message queues for the request
* State changes
	* What happens when a new version of a microservice edits state?
	* New version of service writes to second table
	* Validate data in new table compared to old one
	* Steadily ramp up reads to new table
	* Once reads are going to new table, and writes are going to both, decommission old table
* OSS Projects
	* [Eurkea](https://github.com/Netflix/eureka) - Netflix project for configuring traffic percentages
	* [Envoy](https://www.envoyproxy.io/) or [Itsio](https://istio.io/) for routing (GrubHub is currently investigating using these instead of home-built projects)
	* Deployments with [Spinnaker](https://www.spinnaker.io/)


# Debugging microservices with Idit Levine at [solo.io](https://www.solo.io/)
_The speaker spoke very quickly and was accented. It was difficult to follow-along with the big picture. TL;DR: Squash and OpenTracing look cool._
## Problem
Too many microservices, no complete picture for debugging

## Solution 1: OpenTracing
* Each request gets a unique ID at the edge
* Logs across services can be joined by request ID
* One engine: [Jaeger](https://www.jaegertracing.io/)

### Uses
* Logging, easy logging joining among services
* Metrics/alerting, measure based on tags, timing

## Solution 2: [Squash](https://github.com/solo-io/squash)
* Provides live debugging across multiple microservices
* Requires a server to maintain state, unless you're running with k8s

## Solution 3: Service Mesh
* Potential solutions
	* Istio
	* Consul
	* Linkerd

# Sponsor booth
_I talked to a lot of companies that would have been very interested to talk to someone from ITS DevOps. They were too big/expensive for us._
* HAProxy
	* 6x faster than Traefik; "Golang will not match C code for performance"
	* For k8s applications, HAProxy is plug-and-play
	* HAProxy has an office in Boston, sales staff were happy to come for a meeting and provide support


# Second day Keynotes
## Janna B Dougan at Google
### Critical Path Driven Development (ccpd)
* Availailability of underlying services is not the goal
* Availability of critical paths is the goal

#### Principles:
* Discover the critical paths
* Make them reliable and fast
* Make them debuggable

#### Distributed tracing
* Allows you to identify points of performance optimization
* [`github.com/w3c/distrubted-tracing`](github.com/w3c/distrubted-tracing), proposal for distributed tracing spec consolidation

## Roger Magoulas at O'Reilly Media roger@oreilly.com, O'Reilly Learning Platform
### Bullish Trends
* Python
* Java
* Kotlin
* Rust
* Go
* k8s, +200%
	* "People don't go into k8s casually"
	* "people are implementing it"
* Blockchain, +450%
* AWS
* New ML tools (e.g. pytorch)
* GCP (though, still way behind AWS)

### Bearish Trends
* Javascript, though it's still _very_ popular

### Takeaways
* Microservices/k8s will be the "next architecture"
* Blockchain is very hyped, but we don't know what's going to happen

## "Why Marketing Matters" with Michael Bernstein at Reify
_"Marketing is more important than code in the adoption of software"_
* "More and better marketing leads to more and better adoption"
* Consider investing in marketing at the beginning

### Understanding marketing
"If you build it, they won't come. Unless you tell them about it."
* Marketing is everything customer-facing that you do except for coding
* "Developers should appreciate that work more, and participate in it"

### Appreciating marketing
* Developers don't show interest in marketing
* They don't participate in marketing
* So they aren't encouraged to participate

### Improving marketing
* Work on the marketing from the beginning of the project
* Suggestions, before you start writing code
	* Talk to potential users
	* Be confident about the value that you will provide
	* Be clear on the customer base you hope to serve
* __Start now__:
	1. Define your audence
	2. Design your message
	3. Refine your product

__"If it's worth building, it's worth marketing"__


# Technical Interviewing with Moishe Lettvin at MailChimp
"Every photographer brings themselves to the portrait."
"All photographs are accurate, none of them is the truth."
"Every interview is as much about the interviewer as it is about the candidate."

## Foundation of good interview(er)s
* Morality
	* Reflect the diversity of the world
	* Hire individuals who contribute to, and compliment the company culture
	* Hire individuals who will forward the company's goals
* Kindness
	* "Don't make candidates feel bad."
	* Create conditions where a candidate is able to succeed
	* Avoid brainteasers
	* Be vulnerable
* Iteration
	* Try new interviewing techniques in a small scale, learn from them
	* e.g. Give candidates the option to choose how they want to be interviewed
	* Easy to experiment in a small scale
	* Interview frequently! You'll be a better interviewer if you do it often.
* Observability
	* Collect qualitative and quantitative data on the interview process
	* Record the conditions of the interview, your mood, energy, etc.
	* Have a shadow in every interview, whose responsibility is to assess the interview
* Redundancy
	* Add more redundancy for more important positions
	* Bring newer interviewers up to speed by pairing them with experienced interviewers

## Craft
* Interviewing is a craft
* Embrace the humanity of it
* As an interviewer, you are asking your interviewee to be vulnerable
	* To conduct a sucessful interview, you must also be vulnerable
	* Brainteaser questions, on the other hand, are coorelated with sadism and narcissism of the interviewer
* Mock interviews can be very helpful in an organization
* "Go interview at other companies, to see what they do"
* __"Interviews will never be perfect"__, recognize the limits

__"The interviewer asks in questions what cannot be asked in questions"__



# Modern security best practices for microservices and distributed systems with Seth Vargo from Google
* Security is everyone's responsibility, not just the security team
* Security algorithms require PhDs to understand, and shouldn't be the responsibility of developers
* Security requires a centralized, language-agnostic API. enter: [Vault](https://www.vaultproject.io/)
* Principles:
	* Encrypt everything
	* Credentials
* Hashicorp Vault handles many best practices
	* Encrypted key/val store
	* Data encrypted in transit and rest

__"We've been conditioned to believe that security is hard."__

## Vault features
* Very, very secure
* Versioned key/val store
* Encrypt database data to store in postgres
	* Can rotate keys, write data with version
* Vault can store database credentials
	* You provide Vault with a root-enabled DB credential, then it creates service accounts that it cycles through, and decommissions the root account
* Vault also provides `totp` implementation, necessary for 2FA

__"You are unlikely to have completed a financial transaction in the year that hasn't gone through Vault."__



# Canary Deployments with Kubernetes with Jason Yee at Data Dog
## Rolling deployments
* K8s default
* Rapid, gradual rollout of new versions across all replicas (i.e. pods, containers)
* Rollback only occurs automatically if healthchecks fail
* Always a 100% rollout, can cause problems if all of your users suddenly start writing state with bad code

## Canary deployments
* Migrate small number of replicas, e.g. only one of 10 replicas
* Manually monitor activity of new version, check for errors
* Can diff the usage patterns between old and new code

### Strategy
* Divert trafic based on representative sets
* Who will use this code?
* Ensure that your intended user base will use the canary-deployed code

### Monitoring
* Metadata tags have to be on your log events
* Use ML on logs to determine anomalies
* Signals to watch, before and after canary (LETS)
	* Latency - Increase or decrease in response time?
	* Errors - More/less? New errors?
	* Traffic - Any changes in how users are engaging with your app?
	* Saturation - how much of your resources are you using?

## Service mesh with k8s (e.g. [Istio](https://istio.io/docs/concepts/what-is-istio/#why-use-istio))
* Traditional load balancer only diverts traffic to routes
* __Service mesh allows highly-granular routing to specific versions of services__
* Routing and load balancing
* Service discovery
* Timeouts & retries
* Policy enforcement
* Monitoring and tracing

### Implementing a [service mesh](https://istio.io/docs/concepts/what-is-istio/#what-is-a-service-mesh)
1. Add a data plane with an Envoy sidecar
2. Add a control plane, Istio itself
3. Create k8s manifest, run `istioctl kube-inject` against it, deploy result
	* Alternatively, use an isito webhook to modify the service definition automatically
_Now, you can inspect requests to route them to specific versions of your service_
