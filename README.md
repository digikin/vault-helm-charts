# Helm Template for Vault Annotations

Got tired trying to chase down templates and stack overflow links to solve this.  So, just putting this out there for other people hitting the same problem.  

I will try to come back and finish the other examples from here:  
- https://developer.hashicorp.com/vault/docs/platform/k8s/injector/examples  

For now this is just one chart example for the following Vault Agent Injenctor environment variable reference.  
- https://developer.hashicorp.com/vault/docs/platform/k8s/injector/examples#environment-variable-example

```bash
eric@xps15:~/Documents/github/charts/mychart$ helm install solid-vulture . --dry-run --debug
install.go:192: [debug] Original chart version: ""
install.go:209: [debug] CHART PATH: /home/eric/Documents/github/charts/mychart

NAME: solid-vulture
LAST DEPLOYED: Wed Nov 23 19:45:37 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
vault:
  entryPoint: python3 -c ./start
  envVar:
    passWord: VaultKVsecret2
    userName: VaultKVsecret1
  inject: true
  role: web
  secretConfigFileName: config
  secretConfigPath: secret/data/web

HOOKS:
MANIFEST:
---
# Source: mychart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "web"
        vault.hashicorp.com/agent-inject-secret-config: "secret/data/web"
        # Environment variable export template
        vault.hashicorp.com/agent-inject-template-config: |
          {{- with secret "secret/data/web" -}}
            export passWord='{{ .Data.data.VaultKVsecret2 }}'
            export userName='{{ .Data.data.VaultKVsecret1 }}'
            # Example used in the Vault documentation for reference
            # export api_key="{{ .Data.data.payments_api_key }}"
          {{- end }}
    spec:
      serviceAccountName: web
      containers:
        - name: web
          image: alpine:latest
          command:
            ['sh', '-c']
          args:
            ['source /vault/secret/config && python3 -c ./start']
          ports:
            - containerPort: 9090
```
