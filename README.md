Install Instana on VM
=====================

1. Update [hosts](https://github.ibm.com/Bhavesh-Patel/instana/blob/master/hosts) file with the IP address of Instana VM

```bash
[instana]
xxx.xxx.xxx.xxx
```

2. Update [instana configuration](https://github.ibm.com/Bhavesh-Patel/instana/blob/master/roles/instana/templates/settings.hcl.j2) file based on your requirements

```bash
type                    = "single"
profile                 = "normal"
tenant                  = "xxxxxxxxxx"
unit                    = "xxxxxxxxxx"
agent_key               = "xxxxxxxxxx"
download_key            = "xxxxxxxxxx"
sales_key               = "xxxxxxxxxx"
host_name               = "xxxxxxxxxx.xxxxxxxxxx.xxx"
token_secret            = "xxxxxxxxxx"
```

> Recommended to have each mount on a separate SSD.

```bash
dir {
  metrics = "/mnt/metrics"
  traces  = "/mnt/traces"
  data    = "/mnt/data"
  logs    = "/var/log/instana"
}
```

3. Run Ansible script to install Instana on the VM along with required dependencies

```bash
ansible main.yml
```

Install Instana agent on OpenShift
==================================

1. If not already installed, Install Instana Operator on OpenShift

```bash
helm install instana-agent \
   --repo https://agents.instana.io/helm \
   --namespace instana-agent \
   --create-namespace \
   --set openshift=true \
   --set agent.key=xxxxxxxxxx \
   --set agent.endpointHost=xxx.xxx.xxx.xxx \
   --set agent.endpointPort=1444 \
   --set cluster.name='xxxxxxxx' \
   --set zone.name='xxxxxxxx' \
   instana-agent
```
