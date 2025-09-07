ocp-ingress-chard

Helm chart para:
- Criar um **MachineSet vSphere** dedicado para hospedar os pods do Ingress.
- Criar um **IngressController** "shard" com `routeSelector` para selecionar apenas as rotas com o label `ingress=xpto`.
- Definir `replicas: 2` no IngressController para alta disponibilidade e fazer *pinning* em nós dedicados via `nodePlacement`.

> **Importante**: `routeSelector` seleciona rotas pelo **label**, não pelo host. Para que apenas rotas da *xpto.gov.br* sejam atendidas por este shard, rotule as rotas correspondentes:
>
> ```bash
> oc label route minha-rota -n meu-namespace ingress=xpto --overwrite
> ```
>
> Opcionalmente, defina `spec.domain` no IngressController como `xpto.gov.br` e use hosts que terminem com esse domínio nas suas rotas.

## Como usar

1. Edite `values.yaml` preenchendo os parâmetros do vSphere (vCenter, Datacenter, Datastore, Rede, Template, etc.) e o `clusterID` do seu cluster.
2. Instale o chart:
   ```bash
   helm install ingress-xpto ./ocp-ingress-chard -n openshift-machine-api --create-namespace
   ```
3. Verifique:
   ```bash
   oc get machineset -n openshift-machine-api
   oc -n openshift-ingress-operator get ingresscontroller
   oc -n openshift-ingress get pods -o wide
   ```

### Notas vSphere

Este chart assume que você já possui os Secrets:
- `worker-user-data` (renderizado pelo Machine API em clusters IPI)
- `vsphere-cloud-credentials`

Os nós criados recebem os rótulos:
- `node-role.kubernetes.io/worker`
- `node-role.kubernetes.io/ingress-xpto`

E um *taint* opcional `node-role.kubernetes.io/ingress-xpto=true:NoSchedule` (pode ser desativado em `values.yaml`).

### Seleção por rótulo

O IngressController inclui:
```yaml
routeSelector:
  matchLabels:
    ingress: "xpto"
```
Logo, **somente** as rotas com esse rótulo serão admitidas neste shard.

### Dica

Se você usa *LoadBalancerService*, ajuste `endpointPublishingStrategy` conforme sua infra (Escopo Externo ou Interno). Para *baremetal* ou ambientes sem LB, considere `HostNetwork` ou `NodePortService`.

---

> Compatível com OpenShift 4.x.# ingress-chards
