# Your lab notebook

[Start page](../README.md)

Fill this in as you build. These are intentionally unanswered fields, not claimed test results. Keep credentials out of this file.

## Decisions

| Item | Your actual value |
| --- | --- |
| Team member's 42 login | To record |
| Physical OS and CPU architecture | To record |
| Outer VM tool, version, Linux OS | To record |
| Nested virtualization smoke test | To record |
| Vagrant version / provider version | To record |
| Guest box name / pinned box version | To record |
| Guest OS release / date checked as latest stable | To record |
| K3s pinned release for p1/p2 | To record |
| CPU/RAM allocation and reason | To record |
| Docker Engine version | To record |
| K3d release / selected K3s node image | To record |
| kubectl version / server version | To record |
| Argo CD release / downloaded manifest checksum | To record |
| Public deployment repository URL / watched branch / path | To record |
| Required login appears in repository name | To verify |
| Application image tags / digests / architecture | To record |
| Optional GitLab app release / Helm chart version | To record only if implemented |

## Evidence you collect

| Check | Date and actual result |
| --- | --- |
| p1 clean `vagrant up`, passwordless SSH to both machines | Not run |
| p1 two nodes Ready with .110 / .111 internal IPs | Not run |
| p2 app1 Host / app2 Host / IP / unknown Host | Not run |
| p2 app2 Deployment and ready endpoints show three replicas | Not run |
| p2 Ingress rules shown and explained | Not run |
| p3 clean dependency installation | Not run |
| p3 namespaces and Argo application Healthy/Synced | Not run |
| p3 Git commit SHA for v1 and observed HTTP response | Not run |
| p3 Git commit SHA for v2 and observed HTTP response | Not run |
| p3 version rollback through Git | Not run |
| Rebuild on evaluation computer | Not run |
| Bonus local GitLab version-change demonstration | Not run |

## One entry per problem

```text
Date:
Which machine and directory:
Command:
Expected result:
Actual result (remove credentials):
Cause:
Change that fixed it:
Command that proved the fix:
```

Keep useful evidence concise. Never paste `kubectl config view --raw`, a node token, or Secret contents into a committed notebook. A cluster reporting Ready today is evidence for today's environment, not a promise that a future release behaves identically.
