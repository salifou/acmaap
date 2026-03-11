# AAP + ACM Demo

Demo of Ansible Automation Platform (AAP) job execution across ACM managed clusters. This implementation leverages several key ACM addons and features, including:

- **ManagedServiceAccount addon** & **ManifestWork**: Used for the dynamic generation of managed cluster authentication credentials and RBAC.
- **Cluster Proxy addon**: Utilized for proxying job execution requests across managed clusters via the Hub cluster.

This work is inspired by the article [Multicluster authentication with Ansible Automation Platform] (https://developers.redhat.com/articles/2025/09/08/multicluster-authentication-ansible-automation-platform#).


## Playbook Summary

1. Connect to the Hub cluster (local-cluster) using predefined credentials.
2. Retrieve the list of managed clusters.
3. Generate a token to connect to each cluster using **ManagedServiceAccount**.
4. Set the desired RBAC on each token using **ManifestWork**.
5. Use **Cluster Proxy** to connect and execute jobs/tasks across managed clusters.
6. Perform cleanup.

## Setup

1. Create K8S resources:
    - A namespace on the Hub cluster.
    - A service account to be used by AAP for connecting to the Hub cluster.
    - The appropriate RBAC for the service account.
2. Retrieve the service account token.
3. Retrieve the proxy base URL for connection to the managed clusters.


1. Create K8S resources

```sh
oc apply -f manifest.yml
```

2. Retrieve the service account token
```sh
# Long-lived 1 year
HUB_TOKEN=$(oc create token aap-sa -n aap-integration --duration=8760h)
```

3. Retrieve the proxy base URL for connection to managed clusters

```sh
HUB_PROXY_URL=$(oc get route -n multicluster-engine cluster-proxy-addon-user -o jsonpath='{.spec.host}')
```

## Running the playbook

1. Create the vars file

```sh
cat <<EOF > vars.yml
hub_url: https://$HUB_PROXY_URL
hub_token: $HUB_TOKEN
EOF
```

2. Run the playbook

```sh
ansible-navigator -m stdout \
  --eei registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9 \
  --extra-vars="@vars.yml" \
  run playbook.yml 
```

## Links

- [Multicluster authentication with Ansible Automation Platform](https://developers.redhat.com/articles/2025/09/08/multicluster-authentication-ansible-automation-platform#)
- [ManagedServiceAccount addon](https://open-cluster-management.io/docs/getting-started/integration/managed-serviceaccount/)
- [ManifestWork](https://open-cluster-management.io/docs/concepts/work-distribution/manifestwork/)
- [Cluster Proxy addon](https://open-cluster-management.io/docs/getting-started/integration/cluster-proxy/)
