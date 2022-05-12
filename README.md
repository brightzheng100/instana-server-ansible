|Activity|Time required|
|---|---|
|[Watch screen recording on how to use this playbook](https://ibm.box.com/s/ugn866sudhv0edm0mbhdgr5mymi6yoeb)|8m|
|Instana using this playbook|40m|


Install Instana on Baremetal or Virtual Machine
===============================================

> The machine you are installing Instana onto, must have 16 CPU and 64 GB available memory.
> Check [Instana docs](https://www.instana.com/docs/self_hosted_instana/installation/) for more information.

1. Check out the repo:

```bash
# Please create your TOKEN under your account -> Settings -> Developer settings -> Personal access tokens
$ git clone https://<YOUR IBM GITHUB TOKEN>@github.ibm.com/Bhavesh-Patel/instana.git instana-ansible

$ cd instana-ansible
```

2. Generate SSH key and copy it to the server:

```bash
# Let's generate the SSK key pair and save them to current folder as `id_rsa` and `id_rsa.pub`
# Don't worry, these files will be ignore by this repo's .gitignore
$ ssh-keygen

# Copy the generated public key to the server and trust it
$ ssh-copy-id -i id_rsa.pub <USER>@<INSTANA_SERVER_IP>
```

When prompted, key in the password and you should see something like this:

```log
Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'root@<INSTANA_SERVER_IP>'"
and check to make sure that only the key(s) you wanted were added.
```

3. Create a `hosts` file by refering to the [hosts.sample](https://github.ibm.com/Bhavesh-Patel/instana/blob/master/hosts.sample)  with the IP address of the Instana VM:

> NOTE: DE USE THE RIGHT IP FOR BELOW COMMAND!

```bash
$ cat > hosts <<EOF
[instana]
# The IP/FQDN of your Instana Server
xxx.xxx.xxx.xxx
EOF
```

4. Create a `settings.json` with required variables.

There are a few required variables:
- `instana_server_fqdn`: the FQDN of Instana Server. Using IP is also fine
- `instana_tenant`: the tenant code, which should be your company name, for example `ibm`
- `instana_unit`: the unit code, which could be the owner department, for example `apac`
- `instana_agent_key`: the agent key that you got from Instana license request
- `instana_download_key`: the download key that you got from Instana license request, typically same as agent key
- `instana_sales_key`: the sales key that you got from Instana license request

> NOTE: DE USE THE RIGHT VALUES FOR BELOW COMMAND!

```bash
$ cat > settings.json <<EOF
{
  "instana_server_fqdn":    "<The FQDN, or IP of Instana Server>",
  "instana_tenant":         "<The tenant code, e.g. ibm>",
  "instana_unit":           "<The unit code, e.g. apac>",
  "instana_agent_key":      "<The agent key from the Instana license request>",
  "instana_download_key":   "<The download key from the Instana license request>",
  "instana_sales_key":      "<The sales key from the Instana license request>",
  "instana_metrics_mount":  "/mnt/metrics",
  "instana_traces_mount":   "/mnt/traces",
  "instana_data_mount":     "/mnt/data"
}
EOF
```

> IMPORTANT NOTE: For production installation, Instana recommendeds to have each mount on a separate SSD.

5. Run Ansible script to install Instana on the VM along with required dependencies. [Use this document](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) to install Ansible on MacOS/Windows/Linux. Follow the troubleshooting steps if errros were discovered during install.

```bash
$ ansible-playbook main.yml --extra-vars "@settings.json"
```

Troubleshoot
==================================

|**Problem**|**Solution**|
|---|---|
|Installation fails within 2 minutes with an error `fatal: [xxx.xxx.xxx.xxx]: FAILED! => {"changed": false, "msg": "No package matching 'docker-ce' is available"}`|The installation process updated packages and the Linux host requires a reboot. Login to the node and type `reboot`. Wait for upto 5 minutes and re-run the installation command `ansible-playbook main.yml --extra-vars "@settings.json"`.|
|Installation fails at end during licence import step.|Unknown issue. License file could be invalid? Login to the host and check `/var/log/instana/console.log` file for credentials. If the file contains login credentials then login to the host and Instana login screen will be displayed.|


Install Instana Agents
==================================

1. Install Agent with Infrastructure mode for self-monitoring

It's a best practice to let Instana monitors Instana itself.

To do that, we can generate the script through Instana UI with another variable `-m infra` to indicate that `infra` mode is good enough for self monitoring.
The script may look like this:

```sh
curl -o setup_agent.sh https://setup.instana.io/agent && chmod 700 ./setup_agent.sh && sudo ./setup_agent.sh -a xxxxxxxxxxxxx -t dynamic -e xxx.xxx.xxx.xxx:1444 -y -m infra
```

We may customize the zone a bit too and then start it up:

```bash
# Customize it with zone name
cat <<EOF | sudo tee /etc/systemd/system/instana-agent.service.d/custom-environment.conf
[Service]
Environment=INSTANA_ZONE="Instana Server"
EOF

# Enable it so it can auto start even after OS reboot
$ sudo systemctl enable instana-agent

# Start it up
$ sudo systemctl daemon-reload
$ sudo systemctl restart instana-agent
```

2. Install Agent for OpenShift

If you want to monitor OpenShift, you may generate the script through Instana UI and install it like this:

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

Uninstall Instana agent from OpenShift
======================================

```bash
helm uninstall instana-agent
```
