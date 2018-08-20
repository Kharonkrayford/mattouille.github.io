---
layout: post
title: "Kubernetes through iteration at a startup"
excerpt: "At Capstone Metering we've been building our new application on top of Kubernetes like many other businesses. For those that are unfamiliar, Capstone is a software company that provides specialized services for automation and we're also your average device tech startup. Architecting the stack in our case was equally important as architecting the software itself. We've been pretty hell bent on this continuous iteration approach where we used MVP's to learn and ascertain direction for our engineering organization."
categories: Tutorials
tags: [Kubernetes, Distributed Systems]
comments: true
---

At Capstone Metering we've been building our new application on top of Kubernetes like many other businesses. For those that are unfamiliar, Capstone is a software company that provides specialized services for automation and we're also your average device tech startup. Architecting the stack in our case was equally important as architecting the software itself. We've been pretty hell bent on this continuous iteration approach where we used MVP's to learn and ascertain direction for our engineering organization.

I felt that telling the story of our iteration through different forms of Kubernetes was important. There are probably things that we overlooked, however, I think we built our requirements as our knowledge grew and showing that iteration may help someone in the future. If you're just wanting to know our production workflow for Kubernetes, feel free to jump ahead to the next section, though I think you'll find that the article lacks value if that's all you want.

#### Managed or Unmanaged? That is the question.

**Iteration 1**

During our initial MVP we setup minikube and deployed some services to it. We had already written some MVP's for what microservices would look like but they were nowhere near finalized. They did contain a lot of the mechanics of a general microservice so I think it provided a fair benchmark to assess orchestrators on. I could go into why Kubernetes was the right choice for us, but it's rather arbitrary. Here's the things that I noted as "cool" (very engineer of me, right?)

* The Container Network Interface (CNI) and Container Runtime Interface (CRI) are easily pluggable
* CRD's allow you to extend the functionality of Kubernetes fairly simply (Check out KubeBuilder)
* Exposing resources is carefully controlled through services/ingresses and easy to visualize when needed (I heavily use CLI's, but a dashboard can be nice for groking)
* Concept of ingresses (This is an interesting abstraction around how we use proxies)
* RBAC is pretty simple, albeit at times vague
* Secrets management is supported out of the box and even extendable
* Namespace separation

I did figure out pretty quickly that you can't just go load a lot of plugins up on Minikube. It _is_ actually substantially different from a production Kubernetes deployment. This leaves out a local development option for Kubernetes but everyone has fast internet these days so I figured we could work around that with namespaces.

**Conclusion**: It's worth further exploring.
{: .alert .alert-info }

**Iteration 2**

For our next trick, we attempted to setup Kubernetes in our server farm. To give some context to that, and to highlight how much of a ~~joke~~ learning opportunity it was, our server farm consists of four Dell R700's. For this we used Rancher and we also attempted to run just a vanilla stack. I will note that both of these attempts succeeded and I want to give a shoutout to Rancher Labs for building an awesome product (I'm not affiliated with Rancher at all, but if you're stressed about being late to the container game Rancher has a great product for you.)

There are some beta storage providers that solve the problem of Ceph or Gluster running on your kubelet, for this check out [Rook](https://github.com/rook/rook) (Thanks to [coderanger](https://coderanger.net/) for that). I had attempted to run Gluster in our environment and I was quickly reminded of how operationally expensive it is to run Gluster. We're a startup so this obviously didn't fit the mold.

We never really intended to run production workloads on this setup, it was more so we could easily test and play with Kubernetes, however, we quickly ran into the problem with not owning a proper SAN to feed PV's with.

**Conclusion**: Running Kubernetes on baremetal has some significant cost implications.
{: .alert .alert-info }

**Iteration 3**

Our next shot was to run Kubernetes on our existing infrastructure at AWS setup via `kubeadm`. If you haven't checked out [`kubeadm`](https://kubernetes.io/docs/setup/independent/install-kubeadm/) then definitely give it a shot. My personal notes on it were that while it seemed very convenient I also felt like this was not the way you deploy or manage production infrastructure.

I already can tell statements like this about `kubeadm` or `kops` will likely incur the wrath of mentions on my Twitter. I personally wanted to be able to create some bash to setup Kubernetes as a `systemd` service with whatever supporting things I needed.
{: .alert .alert-info }

Spoiler alert: That's how GKE does it.
{: .alert .alert-danger}

This iteration quickly became about just deploying a cluster that would work for more than a day. I tried Chef, `kubeadm`, and Rancher. `kubeadm` lacked support at the time for multi-master setups, in Chef I was inundated with being a newb at running `etcd`, and with Rancher I felt far too far away from my infrastructure.

As a result, I started to dig deeper into Kelsey Hightower's famed [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way). KTHW doesn't get you into the nitty-gritty running and maintenance of a Kubernetes cluster and all of it's components, instead I think it provides a nice clean view of the basic tools that need to be running for Kubernetes to minimally function.

**Conclusion**: Kubernetes didn't just have cost when associated with on-prem deployments. Kubernetes had an operational overhead that was quite enormous. If I had a team of experienced systems-engineers who could code I think that this might've been where research-iteration stopped and development-iteration began.
{: .alert .alert-info }

**Iteration 4**

This is where our Kubernetes deployment checks in with real-time. My boss and I had a back forth discussion about moving to Google Cloud which was somewhat contentious.

His argument: _Google Cloud is cheaper and has managed Kubernetes_

My argument: _I will never migrate a cloud provider and build a new app at the same time ever again._

**Spoiler Alert**: I migrated to a new cloud provider and am building a new app at the same time.
{: .alert .alert-danger }

This all came about because AWS had a series of outages that affected us really hard. The outages that eventually made me change my mind were global and regional resource outages. While I think my concerns are still valid, to my bosses credit we do have a pretty small footprint and we weren't doing a lift and shift. Instead, we were building an entirely new app and this time it would be on top of Kubernetes, not an actual cloud providers API's.

After wiring up Terraform to our account and doing some scaffolding I spun up GKE. When I originally wrote this piece I listed some limitations of GKE. After much further iteration I've actually found that rather there's just some things GKE doesn't seek to do that are actually part of incubator project to further integrate cloud providers with Kubernetes in a sustainable fashion. For this I point to [`cert-manager`](https://github.com/jetstack/cert-manager) and [`external-dns`](https://github.com/kubernetes-incubator/external-dns).

**Conclusion**: GKE satisfied our needs, as well as we got monitoring, logging, and tracing for free.
{: .alert .alert-info }

#### Fourth time's the charm!

You might say we chose Kubernetes the Easy Way™. Here's some highlights of GKE so far:

* IAM integration is provided out of the box
* `gcloud` sdk makes it quite painless to obtain credentials
* Node autoscaling nodes is quite painless and even works in conjunction with upgrades and repairs
* Projects like `cert-manager` and `external-dns` allow for integrating with cloud providers and interacting with them via annotations. Both of them support service accounts, which is how I prefer to deliver access.
* Options for public and private clusters as well as integrated VPC networking
* Multi-zone master and `etcd`
* Regional persistent disk support (two zone limit)
* Nodes are only semi-managed
* Logs, tracing, and monitoring were easily wired up with Stackdriver and Prometheus
* Auto-upgrades and auto-repair (Yes, there are problems with this)

I had previously noted here that node-pool and master upgrades suck and resulted in downtime. This was prior to me discovering regional clusters, dual-zone persistent disks, and autoscaling. **Pro-tip**: If you run a regional cluster then make sure you set regional persistent disks (or what I call dual-zone, because that's what it is) to the default `StorageClass` otherwise pods will get superglued to one zone and you'll suffer from zonal outages anyway.

Regional Persistent Disks did not work under Kubernetes 1.9.Y. I had to upgrade to 1.10.6 in order to make things work. This is not (currently) annotated in the docs, but I have made Google aware of it.
{: .alert .alert-warning }

That said, all of those things for free out of the box make the challenging concept of running a production grade Kubernetes cluster multitudes more simple. However, Kubernetes is really only one piece.

We set out on actually _building_ a production grade Kubernetes cluster for our internal applications. This includes a chatops bot, some business intelligence and analytics apps, our CI/CD agents, etc... The internal cluster would be the model for how we'd run our production app clusters in the future, which is great because our employees tolerate failure a lot more than our customers do :)

#### Bootstrapping the environment

The first thing I did was bootstrapping everything with Terraform. Terraform does not handle day to day infrastructure automation for us. It merely handles major changes and initial provisioning. There are even some explicit `lifecycle` `ignore_changes` I have set where we manage things with different tooling/services. These tools came about because Terraform lacks some ability to reason about infrastructure, which is fine - it's perfectly good doing what it's doing.

One major lesson to be learned here was managing `node_pool`s separately from the cluster itself. Changing `node_pool`s that are described within a cluster causes resource recreation. This became immediately obvious during some of my load-testing game days.

#### A persistent mistake

One mistake I made early on was conflating my on-prem Kubernetes experience and GKE. In GKE I continued to use PV's and I was even spinning up block storage to associate with PV's. 

`gce-pd` has dynamic support (what I was trying to achieve by setting up Gluster) for PVC's. There are some usecases where you may want to pre-populate a persistent disk with information and have that PV copy it. That's fine, but that wasn't what I was trying to do.

Now my applications stay persistent using dynamic and regionally available disks. I will again note, if you're using a regional cluster please save yourself the headache and potential wonder by also using regional disks as the default.

#### A story of two mono-repos

I'm a big fan of mono-repos. It's easy to find and connect things. Even moreso, we're a Go and Terraform shop so out of the gate many of our tools are biased to mono-repos. We have an infrastructure repo and an application repo.

A great side effect of this was that our wikis became much more consolidated as the repository solidified. We now have documentation living in the code as well as higher level documentation living in a wiki tied to the code. While this isn't necessarily Kubernetes related you will find just how chaotic a large list of repositories can become with a distributed system. What helps reign that in is by using a very strong single source of truth.

In the infrastructure repository our _actual_ source of truth is the Terraform statefile. You'll see later how we interact directly with state to make sure the repository reflects the source of truth.

#### Manifests and secrets, oh my

After a while of deploying containers on our cluster through manifests I began to look around at some ways of storing and managing these manifests. I didn't have anything like Terraform to execute my manifests, which was disappointing. Yes, I realize Helm is a thing, but I am staunchly against using Tiller. Helm 3 will supposedly fix a lot of my gripes.

Aside from that I noticed I had two separate different categories of manifests:
* Cluster support
* Application

**Cluster Support**

These manifests were `cert-manager`, `SealedSecrets`, `external-dns`, `storage-classes`, `prometheus`, etc... All the things that boost the functionality of my cluster and often _have_ to be provisioned first and _sometimes_ have a dependent load order.

**Applications**

These are kind of obvious. It's our actual apps that we run on Kubernetes that our employees or customers might see.

As a more important note, due to having my manifests organized like this I can rebuild a cluster by hand in less than five minutes.

Being that I had just stood up Kubernetes I didn't have a ton of time to go sort out Hashicorp Vault's integration into my workflow (It's on the list, okay?) I opted for BitNami's `SealedSecrets` (thanks to _@whereismyjetpack_). This encrypts your secrets with a 2048bit RSA key generated and stored on the cluster (that you can back up.)

The biggest pain of SealedSecrets is if you screw a secret manifest up it takes a while to redo but this kind of reinforces the idea that you should back up your secrets multiple times. Additionally, storing secrets in your repository potentially opens you up for side-channel attacks so long term this will likely be undone.

In the end I have not completely solved templating or variable substitution (though I've been building a cli/service to do just that) but `kubectl apply -f <dir>` works fine for now. Hopefully Helm v3 will be more focused to my usecase. While looking for a website I wanted to give credit to in this article I also found [this](https://medium.com/devopstricks/ci-cd-with-gitlab-kubernetes-399f81ac91ae), which is interesting.

#### Managing cloud-provider infrastructure

I mentioned earlier that I use Terraform. In the time that I started writing this article and now I have actually cut out half of my Terraform code just by using applications that integrate with a provider. This is, however, a good time to mention that even though I may have a sweet managed Kubernetes, statefile separation is still really important.

```
project
|_ 00-project-global
  |_ hostedzone.tf
|_ prod
  |_ internal-gke.tf
|_ staging
  |_ apps-gke.tf
```

`00-project-global` carries very few things, but its main purpose to manage the project and project access. `prod` and `stage` are quite obvious. Each cluster gets its own file for clarity. Mainly the thing that gets stood up here is managed zones, service accounts, and project access. This isn't unique to GCP, this is a lesson I learned in AWS as well.

The only thing that's not obvious here is that I spun up a project solely for managing the entire GCP account and that's where Terraform state is stored and executed from.

#### Executing on provider infrastructure

Something without a way to collaborate around it is essentially nothing. I introduce to you, [Atlantis](https://runatlantis.io). Atlantis' purpose is to move the Terraform command line to PR's and create a layer of locking for statefiles. It's actually easier for me to give you an idea of what this looks like:

![Atlantis Screenshot](/assets/posts/2018-08-kubernetes-through-iteration-with-a-startup/Screenshot_20180812_180522.png)

Atlantis is basically a proxy for Terraform but ships with a nice UI for discarding locks.

From the PR I can issue commands like `atlantis plan -d gcp/internal/production/kube` and provide some variations with `atlantis apply`. Atlantis also has a pretty nifty config that can live at the root of your repo whereby it can be instructed to do auto-planning (where it shows you everything you've changed in each directory) and you can inject some commands into `plan` and `apply`.

That happens to lead to my next commentary: I am super paranoid and although I don't store secrets in `.tfvars` it provides a lot of context for how my account is laid out. For this I used [git-crypt](https://github.com/AGWA/git-crypt) and I was able to drop git-crypt pretty plainly into Atlantis. Git-crypt actually encrypts the entire file so it was a new challenge, but made our infrastructure repository all that much more truthful.

Since everyone is building infrastructure from this repo in merge requests it keeps the code in the repository as close to near-state as possible while allowing further evolution.

#### Challenges with `ingress-gce`

When I setup Atlantis I setup [`ingress-gce`](https://github.com/kubernetes/ingress-gce). I really didn't know a lot about it other than that it was Google's Layer 7 Load Balancer as an ingress. I'll list some pro's and con's:

**Pros**

* Allows access to GCE resources via annotations
* Very reliable and easy to monitor via Stackdriver
* Mostly hands off
* Syntax is close to nginx-ingress

**Cons**

* Liveness and readiness probes take a while to bubble up
* Missing HTTP-to-HTTPS redirection functionality
* Blackbox D:

I have since moved away from `ingress-gce` and mostly just use services in combination with `cert-manager` and `external-dns` to do what `ingress-gce` was doing.

Our internal applications cluster is run entirely on preemptible nodes, that means our nodes go down with fair frequency (once a day or more) and then come back up. The first time I ever noticed the backend healthcheck latency was during one of these times. The endpoint came back online fairly quickly but not in every region.

It's also fair to note that `ingress-gce` is a work in progress and relies on some beta features of Kubernetes but supports great things like annotations.
{: .alert .alert-info }

#### `cert-manager`

[`cert-manger`](https://github.com/jetstack/cert-manager) is a real life saver. I have mine setup to do domain verification for our public services with LetsEncrypt. In the future I also intend to loop Vault in for some activities. Overall Cert-Manager is quite painless to setup. You create an Issuer and a Certificate and it makes the keypair available to you in a co-named secret.

A word to the wise, `Issuer` and `Certificate` must be in the same `namespace`. There's no obvious reason for this from what I can tell, but it's easy to make that mistake.
{: .alert .alert-info }

Cert-Manager is not billed as production ready, however, for what I've been using it for it's been quite nice.

#### `external-dns`

[`external-dns`](https://github.com/kubernetes-incubator/external-dns) allows you to correlate a LoadBalancer or Ingress IP to a DNS record and works as a job in Kubernetes. It definitely makes life quite a bit easier and plays well with other tools that do similar things.

#### GitLab CI and dind

If you aren't familiar with the term `dind` then get ready. `dind` stands for "Docker in Docker" and is one of the primary ways to build docker images in the cloud much less inside a container orchestrator. What `dind` also tends to imply is some form of privileged containering.

I'm sure folks will swap stories about not only the security implications of privileged containers but also how things can go functionally awry. There is a methodology that works without privileged containers but I have not moved to it yet. Instead, I have namespace isolated the containers. That said, I will continue to play with escaping the container and determining whether there are implications worth further considering. The reason I am satisfied with this setup is that both of my Kubernetes executors are locked to that namespace and they don't talk to anything but GitLab. Additionally, you'd have to break our GitLab quite deeply in order to access these containers. Nonetheless, they are fully isolated from our production application clusters.

Running `dind` as a service with GitLab was actually a bit challenging at first and it's because it spins up an executor. One note I will make is that GitLab CI does not currently supply a working manifest for GKE. I will share mine here:

**namespace.yml**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: gitlab
  labels:
    name: gitlab
```

**role-binding.yml**

These aren't the actual resources I've alotted, so replace that `*` with what you'd like to give the role-binding access to. Also note this is not a CRB.
{: .alert .alert-info }

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: gitlab-role
  namespace: gitlab
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]

---

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: gitlab-role-binding
  namespace: gitlab
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: gitlab-role
subjects:
  - name: default
    namespace: gitlab
    kind: ServiceAccount
```

**config-map.yml**

> Pay close attention to the mounting method here for the secret.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gitlab-runner
  namespace: gitlab
data:
  config.toml: |
    concurrent = 4
    check_interval = 30

  run.sh: |
    #!/bin/bash

    unregister() {
      kill %1
      echo "Unregistering runner ${RUNNER_NAME}..."
      /usr/bin/gitlab-runner unregister \
        -t "$(/usr/bin/gitlab-runner list 2>&1 | tail -n1 | awk '{print $4}' | cut -d '=' -f2)" \
        -n ${RUNNER_NAME}
      exit $?
    }
    trap 'unregister' EXIT HUP INT QUIT PIPE TERM

    set -xe

    cp /scripts/config.toml /etc/gitlab-runner/

    # Register the runner
    /usr/bin/gitlab-runner register --non-interactive \
      --url $GITLAB_URL \
      --executor kubernetes \
      --kubernetes-namespace gitlab

    echo "    [[runners.kubernetes.volumes.secret]]" >> /etc/gitlab-runner/config.toml
    echo "      name = \"gitlab-executor\"" >> /etc/gitlab-runner/config.toml
    echo "      mount_path = \"/secrets/\"" >> /etc/gitlab-runner/config.toml
    echo "      read_only = true" >> /etc/gitlab-runner/config.toml

    echo "Starting runner ${RUNNER_NAME}..."

    # Start the runner
    /usr/bin/gitlab-runner run --user=gitlab-runner \
      --working-directory=/home/gitlab-runner \
      -n ${RUNNER_NAME} &
    wait
```

**secret.yml**

The secret that belongs here is your registration token from the GitLab UI. The runner self registers and applies its own token.
{: .alert .alert-info }

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gitlab-runner-secret
  namespace: gitlab
data:
  runner-registration-token: <runner registration token>
type: Opaque
```

**executor-secret.yml**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gitlab-executor
  namespace: gitlab
data:
  gitlab-service-account.json: <base64 of docker config using a service account>
type: Opaque
```

**deployment.yml**

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: gitlab-runner
  namespace: gitlab
spec:
  replicas: 1
  selector:
    matchLabels:
      name: gitlab-runner
  template:
    metadata:
      labels:
        name: gitlab-runner
    spec:
      containers:
      - name: gitlab-runner
        image: gitlab/gitlab-runner:latest
        imagePullPolicy: Always
        command:
        - /bin/bash
        - /scripts/run.sh
        env:
        - name: RUNNER_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: GITLAB_URL
          value: "https://gitlab.com/"
        - name: REGISTRATION_TOKEN
          valueFrom:
            secretKeyRef:
              name: gitlab-runner-secret
              key: runner-registration-token
        - name: KUBERNETES_PRIVILEGED
          value: "true"
        volumeMounts:
        - mountPath: /scripts
          name: config
        - mountPath: /etc/gitlab-runner/certs
          name: cacerts
          readOnly: true
      restartPolicy: Always
      volumes:
      - configMap:
          name: gitlab-runner
        name: config
      - hostPath:
          path: /var/mozilla
        name: cacerts
      - hostPath:
          path: /var/run/docker.sock
        name: var-run-docker-sock
```

In my GitLab CI yaml I then have a `before_script` that any time I want to push or pull from a private repo I used `docker login -u _json_key --password-stdin https://us.gcr.io < /secrets/gitlab-service-account.json`. 

The reason you have to use this methodology is we don't have mounting ability on the service pod (`svc-0`) which is actually running Docker. When you replace the Docker config, that's telling the Docker _daemon_ what to do and that mount occurs on the `builder` pod where the daemon isn't running.

I can't claim total credit here. I took a lot of inspiration from [this person](https://tortuemat.gitlab.io/blog/2017/07/26/gitlab-runner-kubernetes/) and [this person](https://edenmal.moe/post/2017/GitLab-Kubernetes-Running-CI-Runners-in-Kubernetes/). Both of those websites include some really great code and tutorials even beyond the ones I've linked.

#### Database storage

I am still in the infancy stages of supporting databases on Kubernetes. I will note if you're looking to start then Postgres, Redis, and InfluxDB all play well within Kubernetes on a `StatefulSet`. Don't try to support persistence on day one though, you will lose. You will need to develop strategies for backing these databases up, scaling them, and restoring them.

#### Cost

Our internal cluster currently runs roughly 30 containers on four nodes across 2 zones. So far we've noticed that the cluster costs us about $50 per month. If we had run each of these apps on virtual machines we would be easily looking at double that or more, though the added operational cost may even it out in the near term.

If anyone is seriously interested in a cost analysis I can do a more thorough one rather than just evaluating our monthly bill.

#### Conclusion

I have an obvious to-do list:

* Performing actions on many clusters at once (I have been writing a tool/service to do this)
* Manifest execution and variable substitution (Looking at you Helm 3)
* Disaster recovery with [Heptio Ark](https://github.com/heptio/ark)
* Container image scanning strategy with [Anchore](https://anchore.com/)

This article was about our experience as a startup with limited resources, time, and money while developing our next generation solution to deliver greater reliability to our customers. Ultimately, I think it's flexing the power of iteration and our strategy of using MVP's as tools of iteration and continuous learning. Each iteration was about two weeks (or a sprint in many Agile worlds) and was done alongside maintaining our current production application.

**Post Script**

* One thing I think that's really of value is just because Kubernetes exists doesn't mean that Nomad, Mesos, ECS, and the like are suddenly irrelevant. Use the right tool for the job. We chose Kubernetes because we had a multitenancy model that was easy to wrap around the Kubernetes ecosystem.
* This article is not meant to compare and contrast AWS and GCP. Personally, I believe both cloud providers have value and generally they are both good to get the job done. It is up to you which strengths and merits you value.
* Just because you run GKE or EKS doesn't mean that you now have the opportunity to forget how Kubernetes is built or managed behind the scenes. If anything, these things become even more important so that you don't waste time or pull double duty.
* If you find yourself coming to the end of an iteration and saying, "This is nice" then try and do a game day around your product. Introduce some failure modes and see how things go. All that to say, that's why we started out using preemptible nodes; it was a cheap substitution for something like [Gremlin](https://www.gremlin.com/) in the near term.
* Although it may seem like we did a lot, we have a long way to go and this really only a part 1. Our plan for full adoption is about a year from now.

**Special Thanks**

* To #kubernetes on HangOps for some peer review
* Capstone Metering for providing me a place to work that takes pride in my writing and open source
* To the people who send me feedback on LinkedIn, Twitter, Slack, and IRC