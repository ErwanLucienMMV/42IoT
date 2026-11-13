# Part 1: make two machines join one cluster

[Start page](../README.md) · [Requirements](../docs/01-subject-map.md) · [Next: Part 2](../p2/README.md)

Goal: Vagrant creates two Linux VMs. One starts the K3s control plane. The other joins it as an agent. You can SSH to each, and Kubernetes sees both as Ready.

```text
Linux lab VM: run vagrant and keep the repository here
  │
  └─ dedicated project network: 192.168.56.0/24
       ├─ YOUR_LOGINS     192.168.56.110   K3s server
       └─ YOUR_LOGINSW    192.168.56.111   K3s agent
                              │
                              └── joins https://192.168.56.110:6443
```

Read [workspace setup](../docs/03-workspace.md) first. The examples below are file contents to create yourself; they have not been exercised on a VM in this documentation task.

## 1. Set your choices

**LAB**, from the repository root: export the four variables from [workspace setup](../docs/03-workspace.md#select-a-real-vagrant-box), using real values. Then:

```bash
cd p1
mkdir -p scripts confs
```

Use your school's actual login for `IOT_LOGIN`, not this repository owner's GitHub handle by assumption. The examples give Vagrant machines and guest hostnames the requested uppercase suffixes. Kubernetes node names use lowercase because they must be valid DNS-style names.

## 2. Describe the two VMs

Create **`p1/Vagrantfile`** with this content. This is a VirtualBox example; changing provider needs corresponding provider configuration.

```ruby
require "fileutils"
require "securerandom"

login = ENV.fetch("IOT_LOGIN")
box = ENV.fetch("IOT_BOX")
box_version = ENV.fetch("IOT_BOX_VERSION")
k3s_version = ENV.fetch("IOT_K3S_VERSION")
abort "Replace IOT_LOGIN with your real login" if login == "YOUR_LOGIN"

runtime_dir = File.join(__dir__, ".runtime")
FileUtils.mkdir_p(runtime_dir, mode: 0700)
token_path = File.join(runtime_dir, "join-token")
unless File.exist?(token_path)
  File.write(token_path, SecureRandom.hex(32))
end
File.chmod(0600, token_path)

Vagrant.configure("2") do |config|
  config.vm.box = box
  config.vm.box_version = box_version
  config.ssh.insert_key = true
  config.vm.synced_folder ".", "/vagrant", disabled: true

  [
    ["#{login}S", "server", "192.168.56.110", 2048, 2],
    ["#{login}SW", "agent", "192.168.56.111", 1024, 1]
  ].each do |name, mode, address, memory, cpus|
    config.vm.define name do |machine|
      machine.vm.hostname = name
      machine.vm.network "private_network", ip: address
      machine.vm.provider "virtualbox" do |provider|
        provider.name = "iot-p1-#{name}"
        provider.memory = memory
        provider.cpus = cpus
      end
      machine.vm.provision "file", source: token_path,
        destination: "/tmp/iot-join-token"
      machine.vm.provision "shell", path: "scripts/install-k3s.sh",
        args: [mode, address, k3s_version]
    end
  end
end
```

What each important piece does:

- `Vagrant.configure("2")` selects Vagrant's configuration format, not Kubernetes version 2.
- `define` creates two separately addressable Vagrant machines.
- `hostname` sets the name inside each guest.
- `private_network` adds the static project network; Vagrant's NAT adapter normally remains for management and internet access.
- `insert_key` asks Vagrant to replace the box's default SSH key. `vagrant ssh` manages the resulting key for you.
- The `file` provisioner copies a randomly generated cluster join token into each guest; the shell provisioner moves it into root-only storage.
- The shared folder is disabled so scripts are not accidentally dependent on VirtualBox Guest Additions or a shared checkout.

The `.runtime` directory is ignored by Git and holds a local lab secret. Keep the same token while these VMs exist. Regenerating it without reconfiguring both nodes breaks their agreement. Do not use a published constant such as `password` for all clusters. This pre-shared short-token approach is suitable for this isolated exercise; production bootstrapping needs stronger trust handling.

## 3. Tell a guest how to install K3s

Create **`p1/scripts/install-k3s.sh`**:

```bash
#!/usr/bin/env bash
set -euo pipefail

mode=${1:?Expected server or agent}
address=${2:?Expected node IP address}
version=${3:?Expected pinned K3s release}
case "$mode" in
  server|agent) ;;
  *) echo 'Mode must be server or agent' >&2; exit 1 ;;
esac

apt-get update
apt-get install -y ca-certificates curl iproute2
install -d -m 0755 /etc/rancher/k3s
install -m 0600 /tmp/iot-join-token /etc/rancher/k3s/join-token
rm -f /tmp/iot-join-token

project_iface=$(ip -o -4 address show | awk -v wanted="$address" \
  '{split($4, a, "/"); if (a[1] == wanted) print $2}')
if [[ -z "$project_iface" || "$project_iface" == *$'\n'* ]]; then
  echo 'Expected exactly one interface with the dedicated project IP' >&2
  exit 1
fi
node_name=$(hostname | tr '[:upper:]' '[:lower:]')

cat > /etc/rancher/k3s/config.yaml <<EOF
node-name: "$node_name"
node-ip: "$address"
flannel-iface: "$project_iface"
token-file: /etc/rancher/k3s/join-token
EOF

if [[ "$mode" == server ]]; then
  cat >> /etc/rancher/k3s/config.yaml <<EOF
advertise-address: "$address"
tls-san:
  - "$address"
EOF
else
  cat >> /etc/rancher/k3s/config.yaml <<EOF
server: https://192.168.56.110:6443
EOF
fi

curl -fsSL https://get.k3s.io -o /tmp/iot-install-k3s.sh
INSTALL_K3S_VERSION="$version" INSTALL_K3S_EXEC="$mode" \
  sh /tmp/iot-install-k3s.sh

if [[ "$mode" == server ]]; then
  for attempt in $(seq 1 120); do
    if k3s kubectl get --raw=/readyz >/dev/null 2>&1; then
      exit 0
    fi
    sleep 2
  done
  echo 'K3s API did not become ready; inspect journalctl -u k3s' >&2
  exit 1
fi
```

Vagrant runs its shell provisioner as root by default, so this script does not put `sudo` on every command. It selects the interface by **address**, not by guessing `eth1`. It writes node settings before installation. K3s reads `/etc/rancher/k3s/config.yaml` on startup. Review the [official installer](https://get.k3s.io) and [K3s configuration options](https://docs.k3s.io/installation/configuration) before trusting it in an automated script.

The server installer includes `kubectl` (also available through `k3s kubectl`). [K3s documents the server and agent installation flow](https://docs.k3s.io/quick-start). The relevant [agent settings](https://docs.k3s.io/cli/agent) explicitly select the node IP and Flannel interface.

The script's successful exit on the agent means installation completed; it does **not** prove that Kubernetes considers the agent Ready. The checks below establish that separately.

## 4. Start the server, then the agent

**LAB**, still in `p1/`:

```bash
chmod +x scripts/install-k3s.sh
vagrant validate
vagrant up "${IOT_LOGIN}S"
vagrant up "${IOT_LOGIN}SW"
vagrant status
```

The first launch downloads a box and software, so it needs internet access. Start the server first so the agent has an API to join. If provisioning fails, keep its error and inspect the machine; do not repeatedly destroy it without learning the cause.

Check SSH without a password prompt:

```bash
vagrant ssh "${IOT_LOGIN}S" -c 'hostname; ip -br address'
vagrant ssh "${IOT_LOGIN}SW" -c 'hostname; ip -br address'
```

These are two separate checks: a VM that accepts SSH is alive, but its Kubernetes service may still be broken.

## 5. Inspect the cluster from the server

**LAB**:

```bash
vagrant ssh "${IOT_LOGIN}S"
```

Now you are **SERVER**:

```bash
sudo systemctl status k3s --no-pager
sudo kubectl get nodes -o wide
sudo kubectl wait --for=condition=Ready nodes --all --timeout=180s
sudo kubectl get nodes -o wide
sudo kubectl get pods -A
```

Expected: **two** nodes, both `Ready`, server internal IP `.110`, agent internal IP `.111`. The server's roles normally include `control-plane`; the worker may show `<none>`. `<none>` in the role column is not an error.

Important: `wait --all` waits for nodes that already exist. If the agent never registered, it can succeed for the server alone. Always count the nodes with `get nodes` afterward.

Use `exit` to return to the lab terminal. To inspect the worker:

```bash
vagrant ssh "${IOT_LOGIN}SW"
```

**AGENT**:

```bash
sudo systemctl status k3s-agent --no-pager
sudo journalctl -u k3s-agent -n 60 --no-pager
ip -br address
exit
```

The agent's unit is **`k3s-agent`**, not `k3s`. Keep logs containing tokens out of Git.

## 6. Understand the two network interfaces

VirtualBox guests commonly have NAT on the first adapter and the dedicated address on a second adapter. NAT is useful for downloading packages and Vagrant's management connection. The subject's examples also illustrate checking a dedicated interface rather than blindly using the first address.

The example makes the dedicated interface primary **for K3s traffic** through `node-ip`, `advertise-address`, and `flannel-iface`. Those settings do not reorder Linux adapters or change the default route. If your evaluation interprets "primary network interface" as adapter numbering or the OS default route, adapt the provider/guest network configuration explicitly and retest SSH and internet access. Do not claim that setting `node-ip` alone changes the guest's primary adapter.

Use these **SERVER/AGENT** checks instead of memorizing an interface name:

```bash
ip -br address
ip route
sudo cat /etc/rancher/k3s/config.yaml
```

## 7. Repeatability and a stopping point

`vagrant provision MACHINE` reruns provisioning. `vagrant halt` shuts down without deleting disks. `vagrant up` starts the guests again.

When you deliberately want a clean rebuild, **from `p1/`**, `vagrant destroy` deletes these guests and their disks after confirmation. It does not delete committed configuration. Keep the ignored token for a consistent rebuild, or regenerate it only when both old guests are gone.

For evaluation, automate through Vagrant rather than relying on manual SSH edits. An edit made only inside a VM disappears on a fresh build. If you move inline configuration out of the script, store the source template under `p1/confs/` and provision it explicitly.

You have finished this stage when:

- [ ] `vagrant validate` succeeds and `vagrant up` can recreate both machines.
- [ ] Both SSH connections work without passwords.
- [ ] Guest hostnames have the correct login and suffixes.
- [ ] Kubernetes shows exactly two Ready nodes with the intended internal addresses.
- [ ] You can explain the join token, server URL, and dedicated network selection.

Record your actual results in [the notebook](../docs/07-lab-notebook.md). **Destroy or otherwise fully disconnect Part 1's VMs before starting Part 2**, because both labs use `.110` and a shared host-only network can produce an IP conflict even when the VMs have different names.
