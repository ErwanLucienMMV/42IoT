# What you are being asked to build

[Start page](../README.md) · [Next: the building blocks](02-building-blocks.md)

Source: the supplied **IoT.pdf**, Inception-of-Things v4.0, 18 PDF pages. Page references below use the **printed page numbers** at the bottom of the document; PDF viewer page numbers are one higher. This is a paraphrased checklist, not a replacement for the subject.

**Requirement** means the subject asks for it. **Guide choice** means this documentation proposes a way to do it. **Example** means an illustration you can adapt.

## Rules that apply everywhere

| Requirement | Where in subject | How to satisfy it |
| --- | --- | --- |
| Do the project in a virtual machine | p. 4 | Use a Linux lab VM; understand the nesting described in [workspace setup](03-workspace.md) |
| Put mandatory work in root folders `p1`, `p2`, `p3` | pp. 4, 17 | Keep each part's own files together |
| Put optional work in root folder `bonus` | pp. 4, 17 | Add bonus configuration there if you implement it |
| Put scripts in `scripts`, configuration in `confs` | p. 17 | Use `p1/scripts`, `p1/confs`, and equivalent folders in other parts |
| Complete mandatory parts in order | p. 5 | Check Part 1 before moving to Part 2, then Part 3 |
| Submit work in Git; evaluation uses the evaluated group's computer | p. 17 | Rebuild from the repository on that computer and rehearse there |

You may choose the host VM tools and the Vagrant provider. This guide uses VirtualBox examples; VirtualBox itself is **not** required by the subject.

## Part 1: K3s and Vagrant

Printed pp. 6–8:

- [ ] Two machines are created and run with Vagrant, using the latest stable release of your chosen Linux distribution.
- [ ] Names use one team member's login: `LOGINS` for the server and `LOGINSW` for the worker. `S` and `SW` are uppercase suffixes.
- [ ] Server has dedicated address `192.168.56.110`; agent has `192.168.56.111` on the intended primary project network interface.
- [ ] SSH works to both without typing a password; use SSH keys.
- [ ] First machine runs K3s in server/controller mode; second runs K3s in agent mode.
- [ ] `kubectl` is available and used to inspect the cluster.
- [ ] Both nodes are `Ready` and their Kubernetes internal addresses match the dedicated addresses.

The subject **strongly advises** minimal resources: 1 CPU and 512 MB or 1024 MB RAM. This is advice, not proof that current K3s will fit those limits. Current upstream sizing can be higher; see [workspace setup](03-workspace.md). Record resource choices and explain them.

The subject's old screenshots are examples. Do not copy their old K3s release, Linux distribution, or `eth1` name. Discover actual interface names with `ip -br address`.

## Part 2: K3s and three web applications

Printed pp. 9–11:

- [ ] One Vagrant VM, latest stable chosen distribution, hostname `LOGINS`, address `192.168.56.110`.
- [ ] K3s runs in server mode.
- [ ] Three distinguishable web applications run in the cluster.
- [ ] A request to the VM using HTTP `Host: app1.com` displays app1.
- [ ] A request using `Host: app2.com` displays app2.
- [ ] Other hostnames, including direct requests to the IP address, display app3 by default.
- [ ] App2 has **three replicas**.
- [ ] You can show and explain your Ingress configuration to the evaluators.

The sample web page design is an example; your apps may be simple. Three Services on different ports without hostname routing do not demonstrate the required behavior.

## Part 3: K3d and Argo CD

Printed pp. 12–15:

- [ ] Install K3d in your VM, without Vagrant for this part.
- [ ] Use Docker for K3d, and provide a script that installs the required packages and tools for the defense.
- [ ] Explain how K3s differs from K3d.
- [ ] Create an Argo CD namespace; `argocd` is the guide's choice and the subject's example.
- [ ] Create a namespace named exactly `dev` for the application.
- [ ] Argo CD automatically deploys the application from a **public GitHub repository**.
- [ ] That repository's **name contains one group member's login**. The GitHub account name alone does not satisfy this wording.
- [ ] The app has two visibly different versions, `v1` and `v2`.
- [ ] Change the image version in GitHub, then demonstrate the automatically updated running app during evaluation.

Two application choices are allowed: use `wil42/playground` (`v1`, `v2`, application port `8888`), or publish your own application with both tags to a **public Docker Hub repository**. Using a local-only image or a different registry for your own app would miss the subject's Docker Hub requirement.

The current repository is named `42IoT`. That satisfies the login-in-name rule **only if** `42IoT` actually contains a group member's school login. Otherwise create an additional public deployment repository such as `YOUR_LOGIN-iot-app`, or deliberately rename this one later and update every reference. This documentation does not rename repositories or change their visibility for you.

The subject calls this continuous integration. The specific demonstrated behavior is GitOps continuous delivery: Git changes cause deployment changes. A build pipeline is a separate concept; you do not need to invent one to use the supplied image.

## Optional bonus: local GitLab

Printed p. 16:

- [ ] Finish all mandatory requirements first; the subject says bonus is assessed only if mandatory work is flawless.
- [ ] Run a local GitLab instance using the latest release available from the official source when you implement it.
- [ ] Create namespace `gitlab`.
- [ ] Configure GitLab to work with the cluster.
- [ ] Repeat Part 3's Git-driven version-change workflow using local GitLab.
- [ ] Put the additional scripts and configuration under `bonus/`.

Helm is suggested, not mandatory. Merely opening GitLab's login page does not demonstrate the required integration.

## Your eventual submission layout

```text
42IoT/
├── README.md
├── docs/                     explanations and your lab notes
├── p1/
│   ├── README.md
│   ├── Vagrantfile           you create this from the guide
│   ├── scripts/              provisioning scripts you create
│   └── confs/                any separate configuration files
├── p2/
│   ├── README.md
│   ├── Vagrantfile
│   ├── scripts/
│   └── confs/                application and Ingress YAML
├── p3/
│   ├── README.md
│   ├── scripts/              dependency installation and cluster bootstrap
│   └── confs/                Argo Application and optionally app manifests
└── bonus/
    ├── README.md
    ├── scripts/              if you implement the bonus
    └── confs/
```

Only documentation and repository hygiene files are supplied in this initial commit. The illustrated implementation files are learning tasks, not files that already exist.
