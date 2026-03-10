# ACM + AAP


Run the playbook

```sh
ansible-navigator -m stdout \
  --eei registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9 \
  --extra-vars="@tmp/vars.yml" \
  run playbook.yml 
```

vars file

```sh
cat tmp/vars.yml

hub_url: https://cluster-proxy-user.xxxxx
hub_token:XXX
```
