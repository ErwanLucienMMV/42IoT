# Official references and when to open them

[Start page](../README.md)

The supplied **IoT.pdf**, Inception-of-Things v4.0, is the source for assignment requirements. These upstream references explain the software; they cannot waive the subject's requirements.

Technical references were reviewed on **13 November 2026**. A URL containing `stable` or `latest` can change later. Record exact versions and recheck compatibility before rebuilding. The examples are instructional; no Vagrant, Docker, Kubernetes, or GitLab deployment was performed while creating these documents.

## Machines and Linux

| Reference | Open it when |
| --- | --- |
| [Debian releases](https://www.debian.org/releases/) | Checking the latest stable Debian release |
| [Vagrant installation](https://developer.hashicorp.com/vagrant/install) | Installing Vagrant on your lab OS |
| [Vagrant box catalog](https://portal.cloud.hashicorp.com/vagrant/discover) | Selecting a box for your provider, architecture, and stable Linux release |
| [VirtualBox installation](https://www.virtualbox.org/manual/ch02.html) | Checking host requirements, kernel setup, and supported platforms |
| [Vagrant private networks](https://developer.hashicorp.com/vagrant/docs/networking/private_network) | Assigning dedicated guest addresses |
| [Vagrant VirtualBox networking](https://developer.hashicorp.com/vagrant/docs/providers/virtualbox/networking) | Understanding host-only versus internal networks |

## K3s and Kubernetes

| Reference | Open it when |
| --- | --- |
| [K3s quick start](https://docs.k3s.io/quick-start) | Understanding the server/agent installation flow |
| [K3s requirements](https://docs.k3s.io/installation/requirements) | Sizing nodes and allowing required network traffic |
| [K3s releases](https://github.com/k3s-io/k3s/releases) | Choosing and recording a real stable release |
| [K3s configuration](https://docs.k3s.io/installation/configuration) | Moving flags into `config.yaml` and understanding installer behavior |
| [K3s server options](https://docs.k3s.io/cli/server) | Selecting API advertisement and server behavior |
| [K3s agent options](https://docs.k3s.io/cli/agent) | Selecting node IP, token, server, and Flannel interface |
| [K3s networking services](https://docs.k3s.io/networking/networking-services) | Understanding bundled Traefik and ServiceLB |
| [Kubernetes concepts](https://kubernetes.io/docs/concepts/) | Learning Pods, Deployments, labels, Services, and namespaces in more depth |
| [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) | Checking host/path routing semantics and controller requirements |
| [Traefik Kubernetes Ingress](https://doc.traefik.io/traefik/reference/routing-configuration/kubernetes/ingress/) | Checking controller-specific route priorities |
| [Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) | Selecting a compatible client and verifying its downloaded checksum |

## Docker and K3d

| Reference | Open it when |
| --- | --- |
| [Docker Engine on Debian](https://docs.docker.com/engine/install/debian/) | Installing from Docker's apt repository and handling package conflicts |
| [K3d overview and installation](https://k3d.io/stable/) | Understanding containerized K3s nodes and installing K3d |
| [K3d releases](https://github.com/k3d-io/k3d/releases) | Selecting a release and reviewing compatibility |
| [K3d cluster create](https://k3d.io/stable/usage/commands/k3d_cluster_create/) | Selecting the node image, API port, and published ports |
| [K3d service exposure](https://k3d.io/stable/usage/exposing_services/) | Connecting a host port to an Ingress or NodePort |
| [K3d with Podman](https://k3d.io/stable/usage/advanced/podman/) | Understanding the experimental alternative; the subject path here uses Docker |
| [Subject's playground image tags](https://hub.docker.com/r/wil42/playground/tags) | Checking that v1/v2 and your CPU architecture are available |

## Argo CD and optional GitLab

| Reference | Open it when |
| --- | --- |
| [Argo CD getting started](https://argo-cd.readthedocs.io/en/stable/getting_started/) | Installing the controller, accessing the UI, and managing the initial password |
| [Argo CD releases](https://github.com/argoproj/argo-cd/releases) | Pinning a supported installation release |
| [Argo Application specification](https://argo-cd.readthedocs.io/en/stable/user-guide/application-specification/) | Configuring repository, branch, path, destination, and sync options |
| [Argo automatic synchronization](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/) | Understanding auto-sync, prune, and self-heal |
| [Argo private repositories and TLS](https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/) | Connecting to local GitLab with credentials or a local certificate authority |
| [Helm installation](https://helm.sh/docs/intro/install/) | Installing the optional chart manager |
| [GitLab requirements](https://docs.gitlab.com/install/requirements/) | Planning resources and supported dependencies |
| [GitLab Helm installation](https://docs.gitlab.com/charts/installation/) | Reviewing current chart prerequisites, including external services |
| [GitLab chart deployment](https://docs.gitlab.com/charts/installation/deployment/) | Installing the chosen official chart release |
| [GitLab chart values](https://docs.gitlab.com/charts/installation/command-line-options/) | Configuring DNS, TLS, storage, and prerequisite connections |

When a command fails, prefer the reference for the version you installed. Do not mix an old tutorial's YAML with a new chart's values and assume they still mean the same thing.
