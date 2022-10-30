# ДЗ №06 Templating

Работы выполнялись в managed service for kubernetes by yandex

## nginx-ingress

kubectl create ns nginx-ingress
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -n nginx-ingress

Проверяем, что external ip нарисовался и идем дальше

## cert-manager

Древне говно мамонта из презентации на свежем кубике не взлетело, поэтому установил все более-менее свежее + дополнил созданием cluster issuer

cd kubernetes-templating/cert-manager/
kubectl apply -f cert-manager.crds.yaml 
kubectl apply -f cert-manager.yaml 
kubectl apply -f cluster-issuer.yaml

## chartmuseum

Chartmuseum из методички также уже deprecated, поэтому был взят из другого chart репозитория и также более свежий

cd kubernetes-templating/chartmuseum/
helm repo add chartmuseum https://chartmuseum.github.io/charts
helm install chartmuseum chartmuseum/chartmuseum --wait --namespace=chartmuseum --version 3.1.0 -f values.yaml

## harbor

cd kubernetes-templating/harbor/
kubernetes-templating/chartmuseum/
helm upgrade  --install harbor harbor/harbor --wait -f values.yaml

## Мучаем hipster-shop
В последней итерации хипстеркого магазина весь деплой попилен на 4 блока:
- сам hipster-shop деплой, а точнее то, что от него осталось
- шаблонизированный через хемл frontend деплой подтягивается как dependencies через chart.yaml
- шаблонизированный через kubecfg сервисы paymentservice и shippingservice
- шаблонизированный через kustomize сервис recommendationservice

Кастомизация через kubecfg из методички не взлетела, т.к. библиотеки не рабочие. Пришлось завезти библиотеку локально, поправить её и подключить

cd kubernetes-templating/
kubectl create ns hipster-shop
helm upgrade --install hipster-shop hipster-shop --namespace hipster-shop
kubecfg update jsonnet/services.jsonnet --namespace hipster-shop
kubectl create ns hipster-shop-prod
kubectl apply -k kubernetes-templating/kustomize/overrides/dev