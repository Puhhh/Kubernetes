### LDAP Federation

* General options

| Option           | Value              |
| ---------------- | ----------------- |
| UI display name  | kubernetes.local   |
| Vendor           | Active Directory   |

* Connection and authentication settings

| Option              | Value              |
| ------------------- | ------------------ |
| Connection URL      | ldaps://...        |
| Connection pooling  | On                 |
| Bind DN             | User               |
| Bind credentials    | Password           |

* LDAP searching and updating

| Option                  | Value                             |
| ----------------------- | --------------------------------- |
| Edit mode               | READ_ONLY                         |
| Users DN                | OU=Example,DC=kubernetes,DC=local |
| Username LDAP attribute | sAMAccountName                    |
| User object classes     | user                              |
| Search scope            | Subtree                           |

* Synchronization settings

| Option                      | Value   |
| --------------------------- | ------- |
| Sync Registrations          | Off     |
| Periodic full sync          | On      |
| Full sync period            | 604800  |
| Periodic changed users sync | On      |
| Changed users sync period   | 86400   |

* Advanced settings

| Option      | Value |
| ----------- | ----- |
| Trust Email | On    |

![ldap_1](/images/keycloak/ldap/ldap_1.png)
![ldap_2](/images/keycloak/ldap/ldap_2.png)
![ldap_3](/images/keycloak/ldap/ldap_3.png)
![ldap_4](/images/keycloak/ldap/ldap_4.png)

### Kube API Auth
![client_1](/images/keycloak/kube-api/client_1.png)
![client_2](/images/keycloak/kube-api/client_2.png)
![client_3](/images/keycloak/kube-api/client_3.png)
![client_4](/images/keycloak/kube-api/client_4.png)
![group](/images/keycloak/kube-api/group.png)

<details>
<summary>patch.yaml</summary>

```yaml
---
machine:
  files:
    - content: |
        -----BEGIN CERTIFICATE-----

        -----END CERTIFICATE-----
      permissions: 0644
      path: /var/lib/certs/ca-certs
      op: create
cluster:
  apiServer:
    extraArgs:
      oidc-client-id: kube-api
      oidc-groups-claim: groups
      oidc-issuer-url: https://keycloak.kubernetes.local/realms/kubernetes
      oidc-username-claim: email
      oidc-ca-file: /etc/kubernetes/certs/ca-certs
    extraVolumes:
      - hostPath: /var/lib/certs
        mountPath: /etc/kubernetes/certs
        readonly: true
---
```
</details>

```bash
talosctl patch mc --patch @patch.yaml -n 172.168.101.100 -e 172.168.101.100
```
```bash
kubectl oidc-login setup \
--oidc-issuer-url=https://keycloak.kubernetes.local/realms/kubernetes \
--oidc-client-id=kube-api \
--oidc-client-secret=...
```
```bash
kubectl config set-credentials oidc \
--exec-api-version=client.authentication.k8s.io/v1beta1 \
--exec-command=kubectl \
--exec-arg=oidc-login \
--exec-arg=get-token \
--exec-arg=--oidc-issuer-url=https://keycloak.kubernetes.local/realms/kubernetes \
--exec-arg=--oidc-client-id=kube-api \
--exec-arg=--oidc-client-secret=... 
```
<details>
<summary>clusterrolebinding.yaml</summary>

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: oidc-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: Group
  name: kube-adm
```
</details>

```bash
kubectl config set-context oidc --namespace=default --user=oidc --cluster=k8s
```
```bash
kubectl config use-context oidc
```

### ArgoCD Auth
![scopes_1](/images/keycloak/argocd/scopes_1.png)
![scopes_2](/images/keycloak/argocd/scopes_2.png)
![client_1](/images/keycloak/argocd/client_1.png)
![client_2](/images/keycloak/argocd/client_2.png)
![client_3](/images/keycloak/argocd/client_3.png)
![client_4](/images/keycloak/argocd/client_4.png)

```bash
echo -n '${secret}' | base64
```
```bash
kubectl edit secret argocd-secret -n argocd
```
<details>
<summary>argocd-secret</summary>

```yaml
data:
  ...
  oidc.keycloak.clientSecret: U1Vh...
  ...
```
</details>

```bash
kubectl edit configmap argocd-cm -n argocd
```
<details>
<summary>argocd-cm</summary>

```yaml
...
  oidc.config: |
    name: Keycloak
    issuer: https://keycloak.kubernetes.local/realms/kubernetes
    clientID: argocd
    clientSecret: $oidc.keycloak.clientSecret
    requestedScopes: ["openid", "profile", "email", "groups"]
...
```
</details>

```bash
kubectl edit configmap argocd-rbac-cm -n argocd
```
<details>
<summary>argocd-rbac-cm</summary>

```yaml
...
  policy.csv: |
    g, kube-adm, role:admin
...
```
</details>

### Kiali Auth
![client_1](/images/keycloak/kiali/client_1.png)
![client_2](/images/keycloak/kiali/client_2.png)
![client_3](/images/keycloak/kiali/client_3.png)
![client_4](/images/keycloak/kiali/client_4.png)

### HeadLamp
![headlamp](/images/keycloak/kube-api/headlamp.PNG)

### OAuth2 Proxy
![oauth2-proxy_1](/images/keycloak/oauth2-proxy/oauth2-proxy_1.png)
![oauth2-proxy_2](/images/keycloak/oauth2-proxy/oauth2-proxy_2.png)
![oauth2-proxy_3](/images/keycloak/oauth2-proxy/oauth2-proxy_3.png)
![oauth2-proxy_4](/images/keycloak/oauth2-proxy/oauth2-proxy_4.png)
![oauth2-proxy_5](/images/keycloak/oauth2-proxy/oauth2-proxy_5.png)
![oauth2-proxy_6](/images/keycloak/oauth2-proxy/oauth2-proxy_6.png)

### Grafana
![client_1](/images/keycloak/grafana/client_1.png)
![client_2](/images/keycloak/grafana/client_2.png)
![client_3](/images/keycloak/grafana/client_3.png)
![scopes](/images/keycloak/grafana/scopes.png)
![roles](/images/keycloak/grafana/roles.png)
![mapper](/images/keycloak/grafana/mapper.png)