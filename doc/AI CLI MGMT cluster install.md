# AI CLI Management Cluster install (Connected to Internet)
## *Red Hat does not provide commercial support for the content of these repos*

```bash
#############################################################################
DISCLAIMER: The "aicli" TOOL IS AN UNSUPPORTED COMMUNITY TOOL.

THESE REFERENCES ARE PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
#############################################################################
```

## References
- https://github.com/karmab/assisted-installer-cli
- https://generator.swagger.io/?url=https://raw.githubusercontent.com/openshift/assisted-service/master/swagger.yaml

## Assisted Installer CLI
### Install
```bash
sudo pip3 install -U aicli
sudo pip3 install -U assisted-service-client
```
### Set token and alias
 - Get temporary token from 
     - https://console.redhat.com/openshift/token

```bash
export TOKEN="eyJhbGciOiJIUzI1NiIsIn......."
alias aicli='aicli  --offlinetoken $TOKEN --url https://api.openshift.com '
```

## Create and Deploy Cluster
**NOTE**: ~/.ssh/id_rsa.pub or dsa are injected automatically on cluster create

### Create cluster on AI
- Get your pull-secret from the following URL and place it in the local folder naming it "pull-secret.json"
    - https://console.redhat.com/openshift/downloads
```bash
aicli create cluster ocptest \
  -P pull_secret=pull-secret.json
```
### Customise cluster
```bash
aicli update cluster ocptest \
  -P base_dns_domain=lab.diktio.net \
  -P vip_dhcp_allocation=false \
  -P cluster_network_cidr=10.128.0.0/14 \
  -P cluster_network_host_prefix=23 \
  -P service_network_cidr=172.30.0.0/16 \
  -P network_type=OVNKubernetes 
```
#### Proxy (optional)

If a proxy is used then the following options will also be needed:

```bash
  -P http_proxy="http://<username>:<pswd>@<ip>:<port>"
  -P https_proxy="http://<username>:<pswd>@<ip>:<port>"
  -P no_proxy=""
```
An "*" or a comma-separated list of destination domain names, domains, IP addresses, or other network CIDRs to exclude from proxying.


### Cusomise networking and create ISO
Refer to "nmstate_ocptest.yml" below for network customisation example

```bash
aicli create iso ocptest \
  --paramfile ~/nmstate_ocptest.yml \
  -P image_type=full-iso
```

### Download ISO and boot machines
```bash
aicli download iso ocptest
```

### Finalise the configuration

#### Configure static VIPs to match your DNS entries
Refer to the example below on how to prepare the DNS server.

```bash
aicli update cluster ocptest \
  -P api_vip=192.19.129.21 \
  -P ingress_vip=192.19.129.22
```

#### Rename hosts 
The nodes can automatically be named via a DHCP options or reverse DNS lookup in the static IP case. If neither DHCP or DNS are available in the given environment then the nodes will automatically name themselves "localhost" which means they must be renamed before proceeding to deploy the cluster.

To rename the hosts via the "aicli" tool you need to perform the following steps:
 - Obtain the UUID assigned to your nodes.
 ```bash
 aicli list host
 ```
 - Determine which node is which based on the discovered inventory. For example, the static IP address assigned to the node.
 ```bash
 aicli info host <host_uuid> --full | egrep "^inventory\:" | awk -F "^inventory:" '{print $2}' | jq .
 ```
 - Rename each node accordingly.
 ```bash
 aicli update host <host_uuid> -P name=<new_name>
 ```


### Deploy the cluster
Wait for AI to be in deploy ready state before trying to start the deployment else it will fail with error.
```bash
aicli start cluster ocptest
```

---
# nmstate_ocptest.yml 
```bash

static_network_config:
  - dns-resolver:
      config:
        server:
          - 192.19.128.2
    interfaces:
    - name: bond0
      ipv4:
        address:
          - ip: 192.19.129.23
            prefix-length: 24
        dhcp: false
        enabled: true
      ipv6:
        enabled: false
      link-aggregation:
        mode: active-backup
        options:
          miimon: '150'
          primary: enp1s0
        slaves:
          - enp1s0
      mtu: 8996
      state: up
      type: bond
    - name: enp1s0
      mac-address: '52:54:00:e9:0e:fd'
      ipv4:
        enabled: false
      ipv6:
        enabled: false
      mtu: 8996
      state: ip
      type: ethernet
    - name: enp2s0
      mac-address: '52:54:01:e9:0e:fd'
      ipv4:
        enabled: false
      ipv6:
        enabled: false
      mtu: 8996
      state: ip
      type: ethernet
    - name: enp3s0
      mac-address: '52:54:02:e9:0e:fd'
      ipv4:
        enabled: false
      ipv6:
        enabled: false
      mtu: 8996
      state: up
      type: ethernet
    routes:
      config:
        - destination: 0.0.0.0/0
          next-hop-address: 192.19.129.254
          next-hop-interface: bond0

  - dns-resolver:
      config:
        server:
          - 192.19.128.2
    interfaces:
      - name: bond0
        ipv4:
          address:
            - ip: 192.19.129.24
              prefix-length: 24
          dhcp: false
          enabled: true
        ipv6:
          enabled: false
        link-aggregation:
          mode: active-backup
          options:
            miimon: '150'
            primary: enp1s0
          slaves:
            - enp1s0
        mtu: 8996
        state: up
        type: bond
      - name: enp1s0
        ipv4:
          enabled: false
        ipv6:
          enabled: false
        mtu: 8996
        mac-address: 52:54:00:96:aa:61
        state: ip
        type: ethernet
      - name: enp2s0
        ipv4:
          enabled: false
        ipv6:
          enabled: false
        mtu: 8996
        mac-address: 52:54:01:96:aa:61
        state: ip
        type: ethernet
      - name: enp3s0
        ipv4:
          enabled: false
        ipv6:
          enabled: false
        mtu: 8996
        mac-address: 52:54:02:96:aa:61
        state: up
        type: ethernet
    routes:
      config:
        - destination: 0.0.0.0/0
          next-hop-address: 192.19.129.254
          next-hop-interface: bond0
  - dns-resolver:
      config:
        server:
          - 192.19.128.2
    interfaces:
      - name: bond0
        ipv4:
          address:
            - ip: 192.19.129.25
              prefix-length: 24
          dhcp: false
          enabled: true
        ipv6:
          enabled: false
        link-aggregation:
          mode: active-backup
          options:
            miimon: '150'
            primary: enp1s0
          slaves:
            - enp1s0
        mtu: 8996
        state: up
        type: bond
      - name: enp1s0
        ipv4:
          enabled: false
        ipv6:
          enabled: false
        mtu: 8996
        mac-address: 52:54:00:65:65:56
        state: ip
        type: ethernet
      - name: enp2s0
        ipv4:
          enabled: false
        ipv6:
          enabled: false
        mtu: 8996
        mac-address: 52:54:01:65:65:56
        state: ip
        type: ethernet
      - name: enp3s0
        ipv4:
          enabled: false
        ipv6:
          enabled: false
        mtu: 8996
        mac-address: 52:54:02:65:65:56
        state: up
        type: ethernet
    routes:
      config:
        - destination: 0.0.0.0/0
          next-hop-address: 192.19.129.254
          next-hop-interface: bond0
```
---
# Example DNS setup using IDM/IPA
```bash
# On the IDM server

kinit admin

# Zone
ipa dnszone-add ocptest.lab.diktio.net

# API
ipa host-add --ip-address 192.19.129.21 api.ocptest.lab.diktio.net
ipa service-add HTTP/api.ocptest.lab.diktio.net
ipa dnsrecord-add ocptest.lab.diktio.net api-int  --cname-hostname api

# Nodes
ipa host-add --ip-address 192.19.129.23 master1.ocptest.lab.diktio.net
ipa host-add --ip-address 192.19.129.24 master2.ocptest.lab.diktio.net
ipa host-add --ip-address 192.19.129.25 master3.ocptest.lab.diktio.net

# Ingress Controller/Router
ipa host-add --ip-address 192.19.129.22 ingress.ocptest.lab.diktio.net
ipa dnsrecord-add ocptest.lab.diktio.net "*.apps"  --cname-hostname ingress
ipa service-add HTTP/ingress.ocptest.lab.diktio.net
ipa service-add-principal HTTP/ingress.ocptest.lab.diktio.net@LAB.DIKTIO.NET HTTP/*.apps.ocptest.lab.diktio.net@LAB.DIKTIO.NET

```