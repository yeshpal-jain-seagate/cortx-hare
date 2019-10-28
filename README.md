## What is Hare?

Hare
1. configures components of the distributed Mero object store;
2. makes arrangements to ensure that the system remains available even
   if some of its components fail;
3. provides CLI for starting/stopping Mero system.

Hare implementation leverages the key-value store and health-checking
mechanisms of [Consul](https://www.consul.io) service networking
solution.

## Prerequisites

### :ballot_box_with_check: Checklist

Before starting the cluster as \<user\> at \<origin\> machine,
ensure that

\# | Check | Where
--- | --- | ---
1 | passwordless `sudo` works for \<user\> | all machines
2 | \<user\> can `ssh` from \<origin\> to other machines | \<origin\>
3 | `lustre-client` rpm is installed | all machines
4 | `mero` and `hare` rpms are installed | all machines
5 | `/opt/seagate/hare/bin` is in \<user\>'s PATH | all machines
6 | CDF exists and corresponds to cluster setup | \<origin\>

### LNet

Mero object store uses Lustre networking (LNet).  Run these commands
on every machine of the cluster to ensure that LNet is installed:

```bash
(set -eu

if ! rpm -q lustre-client --quiet lustre-client; then
    if ! sudo yum install -y lustre-client; then
        repo=downloads.whamcloud.com/public/lustre/lustre-2.10.4/el7/client
        sudo yum-config-manager --add-repo=https://$repo
        sudo tee -a /etc/yum.repos.d/${repo//\//_}.repo <<< gpgcheck=0
        unset repo

        sudo yum install -y lustre-client
    fi
fi

# `lnetctl import /etc/lnet.conf` will fail unless /etc/lnet.conf
# exists and is a valid YAML document.  YAML document cannot be empty;
# document separator ('---') is the shortest valid YAML document.
if [[ ! -s /etc/lnet.conf ]]; then
    sudo tee /etc/lnet.conf <<<'---' >/dev/null
fi
)
```

### Install Mero and Hare

* Download `mero` and `hare`
  [rpm packages](http://ci-storage.mero.colo.seagate.com/releases/master/BCURRENT/).

* Install them on every machine of the cluster.

* Add `/opt/seagate/hare/bin` to PATH.
  ```sh
  export PATH="/opt/seagate/hare/bin:$PATH"
  ```

### Prepare a CDF

To start the cluster for the first time you will need a cluster
description file (CDF).

Make a copy of
`/opt/seagate/hare/share/cfgen/examples/ees-cluster.yaml` (or
`singlenode.yaml` in case of single-node setup) and adapt it to match
your cluster.  `host`, `data_iface`, and `io_disks` fields may require
modifications.

```sh
cp /opt/seagate/hare/share/cfgen/examples/ees-cluster.yaml ~/CDF.yaml
vi ~/CDF.yaml
```

See `cfgen --help-schema` for the description of CDF format.

## Hare we go

* Start the cluster

  ```sh
  hctl bootstrap --mkfs ~/CDF.yaml
  ```

* Write some data

  XXX _Wish I knew how._

* Stop it

  ```sh
  hctl shutdown
  ```
