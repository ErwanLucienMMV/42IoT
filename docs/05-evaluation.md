# Practice the evaluation before evaluation day

[Start page](../README.md) · [Requirement map](01-subject-map.md) · [Troubleshooting](04-troubleshooting.md)

Treat this as a rehearsal, not a claimed result. Check boxes only after you personally perform the checks on your actual environment. The subject says evaluation happens on your group's computer and only repository work is assessed.

## Before the demonstration

- [ ] All three mandatory parts are present in `p1/`, `p2/`, and `p3/`.
- [ ] Scripts are in `scripts/`; separate configuration files are in `confs/`.
- [ ] You can rebuild without private files or manual changes hidden inside an old VM.
- [ ] All placeholders are replaced in your actual implementation files.
- [ ] Required versions, resource choices, and machine details are recorded in [the notebook](07-lab-notebook.md).
- [ ] Git has the files you intend to submit; inspect `git status` and `git ls-files`.
- [ ] No credentials, private keys, node tokens, kubeconfig, or `.vagrant` files are tracked.
- [ ] Your public Part 3 GitHub repo has the login in its **repository name**.
- [ ] Your internet connection can fetch boxes, packages, Git repositories, and images required for a rebuild.
- [ ] The public GitHub clone and image pulls do not depend on credentials only your current cache has.

The documents in this repository explain how to build; they do not count as proof that you built or tested the cluster.

## Part 1: show the machines, then the cluster

**LAB, `p1/`**, with `IOT_LOGIN` exported:

```bash
vagrant validate
vagrant status
vagrant ssh "${IOT_LOGIN}S" -c 'hostname; ip -br address'
vagrant ssh "${IOT_LOGIN}SW" -c 'hostname; ip -br address'
vagrant ssh "${IOT_LOGIN}S" -c 'sudo kubectl get nodes -o wide'
```

Show exactly two guests, passwordless SSH, expected hostname suffixes, dedicated IPs, and two Ready Kubernetes nodes. Explain why the worker can show `<none>` in the roles column. Show the `Vagrantfile` and provisioning scripts; explain how they create this state again.

Practice answering:

- What does Vagrant do, and what does its provider do?
- Which address does the worker contact to join? What authenticates it?
- Why does each VM have a dedicated project address?
- What is the difference between the server and agent?
- Why select the K3s node IP and Flannel interface explicitly?
- Which decisions differ from the subject's minimal-resource advice, and why?

## Part 2: prove behavior, replicas, and the Ingress

Transition deliberately: shut down/remove Part 1's guests before assigning `.110` to Part 2. Start Part 2 from its own directory.

**LAB**:

```bash
curl -H 'Host: app1.com' http://192.168.56.110
curl -H 'Host: app2.com' http://192.168.56.110
curl http://192.168.56.110
curl -H 'Host: a-host-you-did-not-configure.example' http://192.168.56.110
```

The results must identify app1, app2, app3, app3, respectively. Then show, **SERVER**:

```bash
sudo kubectl get deployments,pods,services,ingresses
sudo kubectl get pods -l app=app2 -o wide
sudo kubectl describe ingress named-apps
sudo kubectl describe ingress fallback-app
```

Walk through one HTTP request: Host header → Traefik → matching Ingress rule → Service selector → ready Pod. Explain the hostless fallback and its lower priority. The subject specifically asks you to show the Ingress during defense.

Practice answering:

- Why does setting `Host` work without buying a domain?
- What happens if the requested host is not app1 or app2?
- What maintains app2's three replicas?
- What connects a Service to the correct Pods?
- Why does a Deployment not appear as a network hop?
- What is the difference between an Ingress object and its controller?

## Part 3: show the automatic version change live

**LAB**, use the K3d context:

```bash
kubectl config use-context k3d-iot
k3d cluster list
kubectl get nodes -o wide
kubectl get namespaces
kubectl -n argocd get application playground
kubectl -n dev get deployments,pods,services
curl http://127.0.0.1:8888
```

Start with v1. Open the public GitHub repository and show the watched branch and path. Show the Application source settings and automatic synchronization policy.

In the deployment repository, change `v1` to `v2`, commit, and push. Wait for Argo to detect it. Show the new synced commit SHA, Healthy/Synced status, updated Deployment image, ready Pod, and HTTP response identifying v2. Do this during evaluation; a screenshot from yesterday is not the same demonstration.

Show your dependency installation script and cluster bootstrap script. Explain how a clean VM gets from no tools to this working state.

Practice answering:

- How are K3s and K3d related? Why is Docker involved here?
- What does the `argocd` namespace contain, and what belongs in `dev`?
- What lives in GitHub, and what lives in Docker Hub?
- What exactly makes Argo CD synchronize after your push?
- Why is pushing only a new Docker image insufficient in this setup?
- What do Synced and Healthy each tell you?
- How would you change back to v1 through Git?
- What does localhost mean in your Windows browser, lab VM, and a Pod?

## Optional bonus demonstration

Show namespace `gitlab`, the local GitLab instance, its recorded app/chart versions, and the local repository. Show that Argo's repo-server can fetch it. Change v1 to v2 through a **push to local GitLab** and repeat the same commit/image/HTTP evidence. Explain how persistent GitLab data and credentials are stored.

## Last check before committing implementation work

Stage exact intended files, inspect `git diff --cached`, and verify executable bits with `git ls-files --stage`. Commit and push to the repository expected by your school. This GitHub repository and a separate public deployment repository may serve different purposes; make sure all required implementation files are in the actual submission.

If a check fails, keep its evidence and use [troubleshooting](04-troubleshooting.md). A clear explanation of how you diagnosed a problem is more useful preparation than memorizing commands without knowing where they run.
