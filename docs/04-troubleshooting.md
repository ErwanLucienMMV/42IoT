# When something does not work

[Start page](../README.md) · [Evaluation practice](05-evaluation.md)

Start with the earliest broken layer. If a VM cannot reach the internet, changing an Ingress will not fix package downloads. Write down the command, machine, directory, and actual error before changing anything.

```text
Physical computer / outer VM
  ↓ virtualization and available RAM
Vagrant VM or Docker / K3d node
  ↓ network and service running
Kubernetes API and Ready nodes
  ↓ application scheduled and ready
Service with matching ready endpoints
  ↓ routing or port mappings
Your HTTP request and response
```

## Check where you are

**Any Linux shell**:

```bash
hostname
pwd
whoami
ip -br address
```

**Where kubectl is configured**:

```bash
kubectl config current-context
kubectl config get-contexts
```

Part 1/2 commands inside a K3s guest use `sudo kubectl` in these guides. Part 3 commands in LAB use your user's K3d kubeconfig without sudo. If a command unexpectedly connects to `localhost:8080`, kubectl may have no usable kubeconfig. That port is unrelated to our Argo UI choice.

## Vagrant and Linux guests

| Symptom | Meaning and next action |
| --- | --- |
| CPU virtualization unavailable | Check nesting support and outer VM CPU settings; prove a basic guest can boot |
| Missing `IOT_...` environment variable | Export the four verified values in the same LAB shell that runs Vagrant |
| Box/provider/architecture mismatch | Choose a box published for your provider and CPU architecture |
| SSH timeout | Run `vagrant status` and `vagrant ssh-config MACHINE`; inspect guest boot/provider console and forwarded port |
| Permission denied using raw SSH | Use `vagrant ssh MACHINE` first; raw SSH needs the correct user, key, host, and port |
| Dedicated IP not reachable | Check actual `ip -br address`, provider host-only adapter, subnet conflicts, guest firewall, and your caller's network |
| Unreliable responses from `.110` | Make sure Part 1 and Part 2 are not both attached with that address |
| Script says `bash\r` or `$'\r': command not found` | File has Windows CRLF endings; save as LF and re-provision |
| Files under `/vagrant` missing | The sample disables the shared folder; explicitly provision each required file |

Use **LAB**, from the affected part's directory:

```bash
vagrant validate
vagrant status
vagrant ssh-config
```

Provisioning is not automatically rerun just because you edit a script. Use `vagrant provision MACHINE` once you understand the failure and have changed its source. Keep edits in the repository, not only inside the guest.

## K3s node problems

**SERVER**:

```bash
sudo systemctl status k3s --no-pager
sudo journalctl -u k3s -n 100 --no-pager
sudo kubectl get nodes -o wide
sudo kubectl describe node ACTUAL_NODE_NAME
```

**AGENT**:

```bash
sudo systemctl status k3s-agent --no-pager
sudo journalctl -u k3s-agent -n 100 --no-pager
ip -br address
ip route
```

For an absent agent, inspect its server URL, join-token file, route to `.110`, firewall access to TCP 6443, clock, and version agreement. Use `curl -k https://192.168.56.110:6443/cacerts` only as a local connectivity diagnostic: returning CA data shows an HTTPS endpoint responds, not that the agent is authenticated or healthy. `-k` skips TLS verification and is not a fix for cluster authentication.

For a node that exists but is NotReady, inspect `describe node` and networking/runtime logs. A wrong NAT address can register the node with an unusable internal IP. Inspect `node-ip` and `flannel-iface`; discover the actual interface rather than guessing its name.

## Pod status tells you what to inspect

For **Part 3, LAB**; use `sudo kubectl` inside the Part 2 guest and omit/change `-n dev` there:

```bash
kubectl -n dev get pods -o wide
kubectl -n dev describe pod ACTUAL_POD_NAME
kubectl -n dev logs ACTUAL_POD_NAME
kubectl -n dev logs ACTUAL_POD_NAME --previous
kubectl -n dev get events --sort-by=.metadata.creationTimestamp
```

`--previous` works only if that container has a previous terminated instance. A Pod that never started may have no logs; its events are then more useful.

| Status or event | Look for |
| --- | --- |
| Pending / FailedScheduling | Insufficient RAM/CPU, node NotReady, taints, unbound storage |
| ErrImagePull / ImagePullBackOff | Typo, unpublished tag, wrong architecture, registry authentication, rate limits, DNS |
| CrashLoopBackOff | App exits repeatedly; inspect logs, command, config, and dependencies |
| Running but `0/1` Ready | Readiness probe wrong path/port or app not yet responding |
| OOMKilled | Container or node memory pressure; inspect limits and actual usage |
| ContainerCreating | Image download, volume mount, CNI, or runtime events |

`Running` alone does not mean ready to serve HTTP. `ImagePullBackOff` describes Kubernetes waiting between failed pulls; deleting the Pod repeatedly does not repair an invalid tag.

## HTTP routing and Services

For **Part 2, SERVER**:

```bash
sudo kubectl get ingress -o yaml
sudo kubectl get ingressclass
sudo kubectl get services
sudo kubectl get pods --show-labels
sudo kubectl get endpointslices
sudo kubectl -n kube-system logs deployment/traefik --tail=80
```

| Result | First interpretation |
| --- | --- |
| Timeout | Route, VM state, firewall, or address issue |
| Connection refused | Nothing listening at that address/port yet |
| Controller's 404 | Request reached a controller but no matching usable route |
| 502 / 503 | Matching route may have unavailable backends or wrong target ports |
| Wrong app | Host header, catch-all priority, or Service selector points elsewhere |

To isolate a Service from Ingress, **SERVER**, terminal A:

```bash
sudo kubectl port-forward service/app1 8081:80
```

In another SSH session to **the same SERVER**, `curl http://127.0.0.1:8081` should return app1. It will not appear on LAB's own localhost without another forwarding step. If this works but the external Host test fails, focus on Traefik, Ingress, and VM port 80.

## Docker and K3d

**LAB**:

```bash
docker info
docker ps -a
k3d cluster list
kubectl config current-context
kubectl get nodes -o wide
ss -lnt
```

Docker permission errors need correct user access and a fresh login, not `chmod 777` on its socket. Confirm the service with `sudo systemctl status docker --no-pager`.

For a Part 3 app that is healthy but unreachable, inspect the port mapping and Service:

```bash
docker port k3d-iot-server-0
kubectl -n dev get service playground -o yaml
kubectl -n dev get endpointslices -l kubernetes.io/service-name=playground
```

Check `127.0.0.1:8888 → node 30888 → Service target 8888`. If Docker reports a port already allocated, find its owner before changing or killing anything. An old port-forward may still be open. The [K3d exposure guide](https://k3d.io/stable/usage/exposing_services/) explains the separate Docker and Kubernetes layers.

## Argo CD is not deploying the new version

**LAB**:

```bash
kubectl -n argocd get application playground -o yaml
kubectl -n argocd describe application playground
kubectl -n argocd logs deployment/argocd-repo-server --tail=100
kubectl -n argocd logs statefulset/argocd-application-controller --tail=100
```

| Symptom | Check |
| --- | --- |
| No Application object | Apply the Application bootstrap definition in `argocd`; ensure its CRD is installed |
| Repository connection error | Public visibility, clone URL, DNS/TLS, network, credentials if private |
| Manifest generation error | `source.path` exists on the watched branch; YAML is valid and only deployable files are included |
| Synced but old version | You may have edited the wrong repo/branch/path, not pushed, or changed an unwatched file |
| OutOfSync | Auto-sync enabled? Sync failure/conditions? Repo-server can fetch latest commit? |
| Synced but not Healthy | Git matches live configuration, but the application itself may be failing |
| Manual image edit keeps reverting | `selfHeal` is restoring the version still declared in Git |
| Version remains old briefly | Wait for Git detection and rollout, then compare commit SHA, Deployment image, and HTTP response |

**Synced** means Argo's desired and live manifests agree. **Healthy** reports resource health. Both are useful; HTTP is still the final behavior check. Use [Argo CD's sync documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/) for policy details.

If a port-forward stops when its selected Pod is replaced, restart that command. Stopping your UI port-forward does not stop Argo CD's controllers.

## Before asking someone else for help

Collect: which part, machine, OS/tool versions, exact command, expected result, actual error, relevant events, and the smallest changed file. Remove tokens and credentials. State the checks you already performed. This makes a useful question and often reveals the fix as you write it down.
