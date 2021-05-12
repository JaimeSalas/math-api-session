# Staging Tests and Canary Deployment

## Prerequisites

1. Setup jenkins CI/CD server running
2. AWS IAM valid account with enough priviliges to create a cluster
3. eksctl
4. kubectl
5. helm

## Branch Strategy E2E tests

devlop --> staging --> master

## Branch Strategy Canary Deployment

devlop --> staging --> master --> Production

## Grafana

Once Grafana is deployed into the Kubernetes cluster run the following command to find it:

```bash
export ELB=$(kubectl get svc grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

echo "http://$ELB"
```
## Istio Ingress

```bash
kubectl -n istio-system get svc
```

`http://ae594cd55f2a94d6a921b6916692872a-1620865600.eu-west-3.elb.amazonaws.com/api/sum?a=2&b=2`
