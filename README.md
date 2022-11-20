```bash
eric@xps15:~/Documents/github/charts/mychart$ helm install solid-vulture . --dry-run --debug --set favoriteDrink=slurm
install.go:192: [debug] Original chart version: ""
install.go:209: [debug] CHART PATH: /home/eric/Documents/github/charts/mychart

NAME: solid-vulture
LAST DEPLOYED: Sat Nov 19 22:35:19 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
favoriteDrink: slurm

COMPUTED VALUES:
favoriteDrink: slurm
vault:
  entryPoint: python3 -c ./start
  envVar:
    passWord: data
    userName: test
  inject: true
  role: web
  secretConfigFileName: secret-config
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
            export passWord="data"
            export userName="test"
            export api_key="{{ .Data.data.payments_api_key }}"
          {{- end }}
    spec:
      serviceAccountName: web
      containers:
        - name: web
          image: alpine:latest
          command:
            ['sh', '-c']
          args:
            ['source /vault/secrets/config && python3 -c ./start']
          ports:
            - containerPort: 9090
```