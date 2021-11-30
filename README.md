Instana Install Playbook
========================

1. Update hosts file with the IP address of Instana VM

```bash
[instana]
xxx.xxx.xxx.xxx
```

2. Update instana/templates/settings.hcl file based on your requirements

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