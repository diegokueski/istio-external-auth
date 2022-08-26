#  Istio External Authorizer
This repo is just for save my notes about, how setup an external authorizer with istio

### References
* https://medium.com/@lucario/istio-external-oidc-authentication-with-oauth2-proxy-5de7cd00ef04
* https://istio.io/latest/docs/tasks/security/authorization/authz-custom/

## Create the services
```
kubectl create namespace dummy-service
kubectl apply -f .k8s/staging/sleep.yaml -n dummy-service
kubectl apply -f .k8s/staging/httpbin.yaml -n dummy-service
```

## Create the auth service
kubectl apply -n dummy-service -f  .k8s/staging/ext-authz.yaml

## Add istio extension provider
Add the code ***.k8s/staging/update-istio-extension-provider.yaml*
```
kubectl edit configmap istio -n istio-system
```

**Apply the new istio config**
#kubectl rollout restart deployment/istiod -n istio-system

## Create Auth policy
kubectl apply -n dummy-service -f  .k8s/staging/auth-policy-ext-authz.yml.yml

### Test
**Using the slep pod, execute the next curls**

+ Not work
curl http://httpbin.dummy-service:8000/headers -H "x-ext-authz: deny" -s 

+ Work
curl http://httpbin.dummy-service:8000/headers -H "x-ext-authz: allow" -s
curl http://httpbin.dummy-service:8000/ip -s

# Okta - External auth provier
Hay recursos de los cuales no estan versionados los manifiestos, pero en esencia es, levantar un nginx:latest en el path /app y crear un hostname que utilice el isitio-gateway (con su respectivo SSL)

## Install oauth2-proxy
helm install oauth2-proxy oauth2-proxy/oauth2-proxy -f oauth2-okta.yaml -n dummy-service

helm uninstall oauth2-proxy -n dummy-service

## Create Auth policy
kubectl apply -n dummy-service -f  .k8s/okta/auth-policy-ext-auth.yml

****
En el provider que se registra en Istio le cambie el puerto del servicio oauth2-proxy al 80 (ya que en la definición este utiliza)