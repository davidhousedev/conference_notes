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
