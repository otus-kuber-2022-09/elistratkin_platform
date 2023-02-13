# elistratkin_platform

## ДЗ №12 Production cluster

Создадим тестовые машины и установим туда kubernetes версии 1.24.0

![Тестовые машины](./kubernetes-production-cluster/test-machines.png)

Проверяем, что кластер поднялся

![Старый кластер](./kubernetes-production-cluster/old-cluster.png)

Запускаем нагрузку на кластере

![Запускаем нагрузку](./kubernetes-production-cluster/deployment-nginx.png)

Обновляем мастер

![Обновляем мастер](./kubernetes-production-cluster/update-master.png)

Проверяем, что все компоненты мастера обновились

![Обновляем мастер](./kubernetes-production-cluster/update-components.png)

Обновляем воркеры, вешаем заразу на одну из воркер нод

![Обновляем воркеры](./kubernetes-production-cluster/drain-nodes.png)

Проверяем, что весь кластер обновился

![Кластер готов](./kubernetes-production-cluster/new-cluster.png)

## ДЗ №11 Vault

```bash
helm status vault
------------------
NAME: vault
LAST DEPLOYED: Sat Jan 28 20:26:14 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault
```

```bash
kubectl exec -it vault-0 -- vault operator init -key-shares=1 -key-threshold=1

Unseal Key 1: wyNh5oLbR49SeriBQy3pReQvvBxnBIdeEQeYMbpgsDA=

Initial Root Token: hvs.RZg0HPs1uA5q0R9ZDznEJe9b
```

```bash
kubectl exec -it vault-0 -- vault status

Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.12.1
Build Date      2022-10-27T12:32:05Z
Storage Type    consul
Cluster Name    vault-cluster-336c39e0
Cluster ID      b1b97dc2-26d7-9cd1-24c1-4c5b95b321be
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
Active Since    2023-01-28T17:41:27.766768321Z
```

```bash
kubectl exec -it vault-0 -- vault login

Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.RZg0HPs1uA5q0R9ZDznEJe9b
token_accessor       386SfDNrSr8zQ2IjKBCJ4uaz
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

```bash
kubectl exec -it vault-0 -- vault login

Path      Type     Accessor               Description                Version
----      ----     --------               -----------                -------
token/    token    auth_token_be5b4bb1    token based credentials    n/a
```

```bash
kubectl exec -it vault-0 -- vault read otus/otus-ro/config

Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus
```

```bash
kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config

====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```

```bash
kubectl exec -it vault-0 -- vault auth list

Path           Type          Accessor                    Description                Version
----           ----          --------                    -----------                -------
kubernetes/    kubernetes    auth_kubernetes_1feb11b1    n/a                        n/a
token/         token         auth_token_be5b4bb1         token based credentials    n/a
```

```bash
Почему мы смогли записать otus-rw/config1 но не смогли otusrw/config?

Не хватало update capability в политике для пути otus/otus-rw/*"
```

```bash
kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="gitlab.example.ru" ttl="24h"

Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUXp3CoY5nlbkjQjRDI+IwvPoegKQwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMzAxMjkwOTE1MzJaFw0yODAx
MjgwOTE2MDJaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALB70+HRzpuu
GxFhjjUnJLe3taPM0bDVHIUADTpDTeAzJt4MvZuAWOhOspEyN/PFT7n+FqK8prAL
aWz6FEPFwlGLwwuWvQ2BUE0ifHua7i99V0MRYFuaUeTkFTmuZ0tUKpJOy/cmwph1
QT4yX8IfaYsKerF2hVBbUBQ722BF2GT0pwTVpsWKUfJuPrYb5otZCApUXSqbluyb
rvybz20OxO63y8f6Q0kHf+WVja9P2ajNM89yWZyE5pb16cFLEdf1ko4MkFbpp6r1
/l0Y64uEUmiCJ6PWDvBk+dZPmJT3Hc736J4ePjTgCN93LTnuw03dd00N5w+Xz9sO
Bg2rLt6XK2cCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUpJNYVNV9zodgvBQA48S2Bn3eq+EwHwYDVR0jBBgwFoAU
VGXsXCKg1wYZ0XyPWpx0UVXb10YwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
v1spk7E42bd1PvJfxr3uKjFXwAAVZLvv45hGyrQ7HZEuNXrFBad8LL8sZQzJ5Zvf
viHgiHScs0DXSOtaVH14wVarsJanpSe0aGTSvvgxCHHL3CvUI6VS97Oh2PatLuQP
+0gl4eDZJhnZpBRjfFpEzYhthRGkDyyzGNtu4GUNTjlU3V1NjxpCSmei+Q8RlOjJ
NGfVgqNbSuk8m3e0/XU6TAlyY2rSch3Tm/Yjy6yjAT77Qt024CIgIGh75qDtgxCL
/AKyp8lnMSURcbyvbq59T322wKOCOBf1X8MLTO3BzV7ZlvS0gOWCDBf0UUi9ywIG
76NIhHYDkqVSssgT0qpYgw==
-----END CERTIFICATE----- -----BEGIN CERTIFICATE-----
MIIDMjCCAhqgAwIBAgIUP1bFxcwV8n5v0uAVCz+S3zz9xXkwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMzAxMjkwOTA3MjlaFw0zMzAx
MjYwOTA3NTlaMBUxEzARBgNVBAMTCmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQC/36BYvfuObpLZtNoEHSr1SEnKpw8wflC/0DPsPjz9
tqBmZnBHHhaJF7YjuOURq0j98WzdBWYJsUT6oeHysoe8RXEYgB0IrafUyEfqDAHH
Gh+sC15vHF5rth9O20xQw8hzz6G/JsnRblFdmTJUFgvEofKyAOUgtsBoLy6Tfn5v
rUKN2YstF+kSsisGji9c0h6srqulkXI1Z7jiVYrwrc4MCxpYxQHVgICfg6yBLsn0
xO0LIPyIqm+eRKigzgEMpmPThglBuUdHDZGkRTal5/BDTY/hPXhgL6Qk5miEcv/9
e1kvCzA5Gt33D7CfIps8qQIce+7W23uwcuHEsINlEnuRAgMBAAGjejB4MA4GA1Ud
DwEB/wQEAwIBBjAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBRUZexcIqDXBhnR
fI9anHRRVdvXRjAfBgNVHSMEGDAWgBRUZexcIqDXBhnRfI9anHRRVdvXRjAVBgNV
HREEDjAMggpleGFtcGxlLnJ1MA0GCSqGSIb3DQEBCwUAA4IBAQA5/bCXTCHgXEbL
sEYMU0Sttm6MeoG+TfCJMPASROmIMVjwmx4+iGwPa9l15K9yTBur6KmLdvUTQ5TE
oJjrA5qXk+/gdAjz3XhJfNcgKVgjQxgZ9WjPHmn78784w0lkbO6BWsSibnBOggMK
B4ERIMuwvoVk7ns4mblfIU1tf+fZWeQFnf8Ec9BQJh/tBQlwgTD0JrhunXyadrUA
/8b1CifIojC2tJw2ar5LXGNZdZaCiBFyYSz9W64ybAicCofrLWLfnktVUm6YZwEa
wV8kIosGg8ykAXoKCY0wiq6BbRFRpDwuU9LIqJxvmJf8YsLtzA2msRXcsD73ChZQ
MwewrLK9
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZzCCAk+gAwIBAgIUBeNSOl2Ye8TSdJfF7de/WARQJ4MwDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIzMDEyOTA5MjMwNVoXDTIzMDEzMDA5MjMzNVowHDEaMBgGA1UEAxMRZ2l0
bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDK
qqzTlCog+T58KkXP2+hqq6cQxpQQZcxMXgAxFew3/zRPCXRH2q9T+UmhIHmsuEq9
MDUySddB3duYeC75DnOFnXchjJ228Bmkuko/dlwEM64wbkh5077e0/rXx2uDcqsW
O0WO2WndSi9FbTEU+MuPqP4MzDzxW5TpJpkkGOzSPKyXvOv+JNDDYWpotScmIsOK
EMq6YEEx5SKT3WA07cNOfrnBradg9RUcon5ksOjRqMTi7r6l9PuymkHmmD2Zwn5w
dR7PjFKNj+iJWRk0014/Dr2VjmhYpooaWhzh6UtiiHBa4flznOanKOrLM+j9gCT5
p6Dl8XKRdNGbju6gzys3AgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQUYuAWYV0mDl4RsesM
0oJa6B/q/g8wHwYDVR0jBBgwFoAUpJNYVNV9zodgvBQA48S2Bn3eq+EwHAYDVR0R
BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBABPfVxZc
OBmEINLX2uvQjJeMZfkwaKQhcvW95kbP1DgRTPX4P2JUMzpNOMCi4ZvoWjGxRsBL
Pe9nIujCnFqMrTG3m+54tCtAZ4uCHifMJHQSKGsFP+3Do0Chlo76ndt2GGI1Kmw0
qJXZCk77b+VQItLzwYpDF6WFsVKAxkj8bQk0gRM/rQ/Ze4lm37dMblbeTOiieIQD
O3pKL9uazRHXlA9VZcnRe4wGAu1cH2NuC+KZ4FgDApKNxzrmLprMxbrS851vi/g6
jhnYHsSo/4pa68jvYRW/y/7TQ/z3Ecu5cQSulWX2RRcbFQpb+B7cZfTo055uzY0F
casYPFnx+zR/IBw=
-----END CERTIFICATE-----
expiration          1675070615
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUXp3CoY5nlbkjQjRDI+IwvPoegKQwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMzAxMjkwOTE1MzJaFw0yODAx
MjgwOTE2MDJaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALB70+HRzpuu
GxFhjjUnJLe3taPM0bDVHIUADTpDTeAzJt4MvZuAWOhOspEyN/PFT7n+FqK8prAL
aWz6FEPFwlGLwwuWvQ2BUE0ifHua7i99V0MRYFuaUeTkFTmuZ0tUKpJOy/cmwph1
QT4yX8IfaYsKerF2hVBbUBQ722BF2GT0pwTVpsWKUfJuPrYb5otZCApUXSqbluyb
rvybz20OxO63y8f6Q0kHf+WVja9P2ajNM89yWZyE5pb16cFLEdf1ko4MkFbpp6r1
/l0Y64uEUmiCJ6PWDvBk+dZPmJT3Hc736J4ePjTgCN93LTnuw03dd00N5w+Xz9sO
Bg2rLt6XK2cCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUpJNYVNV9zodgvBQA48S2Bn3eq+EwHwYDVR0jBBgwFoAU
VGXsXCKg1wYZ0XyPWpx0UVXb10YwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
v1spk7E42bd1PvJfxr3uKjFXwAAVZLvv45hGyrQ7HZEuNXrFBad8LL8sZQzJ5Zvf
viHgiHScs0DXSOtaVH14wVarsJanpSe0aGTSvvgxCHHL3CvUI6VS97Oh2PatLuQP
+0gl4eDZJhnZpBRjfFpEzYhthRGkDyyzGNtu4GUNTjlU3V1NjxpCSmei+Q8RlOjJ
NGfVgqNbSuk8m3e0/XU6TAlyY2rSch3Tm/Yjy6yjAT77Qt024CIgIGh75qDtgxCL
/AKyp8lnMSURcbyvbq59T322wKOCOBf1X8MLTO3BzV7ZlvS0gOWCDBf0UUi9ywIG
76NIhHYDkqVSssgT0qpYgw==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAyqqs05QqIPk+fCpFz9voaqunEMaUEGXMTF4AMRXsN/80Twl0
R9qvU/lJoSB5rLhKvTA1MknXQd3bmHgu+Q5zhZ13IYydtvAZpLpKP3ZcBDOuMG5I
edO+3tP618drg3KrFjtFjtlp3UovRW0xFPjLj6j+DMw88VuU6SaZJBjs0jysl7zr
/iTQw2FqaLUnJiLDihDKumBBMeUik91gNO3DTn65wa2nYPUVHKJ+ZLDo0ajE4u6+
pfT7sppB5pg9mcJ+cHUez4xSjY/oiVkZNNNePw69lY5oWKaKGloc4elLYohwWuH5
c5zmpyjqyzPo/YAk+aeg5fFykXTRm47uoM8rNwIDAQABAoIBAHmx/O9bg+Si5dN3
Ks6C1U/gXQRkyZoDFqcwttrCuEJzuzxOiAVxmmBJS941+vos+6wY24ij8F770pX+
bO5esB2mjVeW8UldaXiR6G2tK9nWEp3AS0LkVDPHhFO3yqZ236cM1lrN0JTxq+r7
g0J9lvZf2pViuqGPhZ5u7/xN/z7J4Ak6uPUT1Vs0qsja94jmA240gsbH7/8k+wBy
z3wgtykD8vOf/P03wJ/nOhXUIUYDC7+MdrHQBXRgZxQ0PXirKBAjmQZXcSE8FHEt
e07pazs0Gn8tgIUH8S1PIB+MrFChRF5iM8itDnhZfxmImOmltRFAhr5hPYJP82M8
uHRND9ECgYEA86jCQNW9ldmJw2bw9ElugYsj+tokKb6VkdIfgQ3sJTvQlVaVEs+n
IQWnTfwTZ6FHi3VUMvL4yr8YGDOZOYD/5CTdFSHWJBhSXq7dbLG8wENUABuGzxNN
VdVAvIbyVJyY860zP6njNgRj0zkQOu5O73Mr4OJZnDipykQ75xFmfJkCgYEA1O5q
L7dOCFdy35/870e+dlyFxqL4bPywL8La9Pv6vv9l5uhhZv2ck5zxt7KIXlbvJNDT
k01NyNN285C30zDo8+cdz2RlDuw53Xn1iHUi7I/2NnwoKnNyrWHr+uo+WL1nQtzA
JvJHql5fy9zmPDW5ocdr0Zx8Rn/x7OPAebNfeE8CgYEA7R6UPEhFgaj8Ol7ZqsIz
enoP/7V+b+l64TfdVNhZsTpumIZz+KSAftBjqq7fq3/2gi8iS5rMp3IkXpkISj/R
xxJg3InbPbE1p9MAxSCLEYf0HU6seKTOI/OuVkn1VoY3CxzIvPH1ncHT6+CifNch
lkE8uFNO5Yr3IVIaAZqbZDECgYEAmyQSzi1ad41vryucWKMHTKng9ndrVc1OeZ4Z
doHZkiv3jSQnQ6jo2kxxeYxk+4ePul1EqoCaegZJFvGMXh22vW7ZM3BMiXDoQExJ
72CZT48cr1oK0dNH7Lf5/dmKD5IAmKDKiBXizWjxl1GdlqT2Wb84+YJwsdUq6ZSP
epyMpNUCgYBge/cn3PPPAJMXCaz9QVBnJqrhuMyQn5Bb5fxgWV96Xv5nSqLz8Vi7
hYaOoNB1UCHb3MLDUa/whk8RhaGYnlR47fAS3C1wSvelGo/EesFOs9Xw41b6W5J0
xsVt1LNc6JzVqkuJvltMMKTRwuyudYFywLG2NHIZ5gn8D4ElYXx+XQ==
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       05:e3:52:3a:5d:98:7b:c4:d2:74:97:c5:ed:d7:bf:58:04:50:27:83
```

```bash
kubectl exec -it vault-0 -- vault write pki_int/revoke serial_number="05:e3:52:3a:5d:98:7b:c4:d2:74:97:c5:ed:d7:bf:58:04:50:27:83"

Key                        Value
---                        -----
revocation_time            1674984309
revocation_time_rfc3339    2023-01-29T09:25:09.202364005Z
```

## ДЗ №07 Monitoring

```bash
cd kubernetes-monitoring/
helm install test-app test-app/
helm repo add nexclipper https://nexclipper.github.io/helm-charts

for i in {0..2}; do helm install nginx-exporter$i nexclipper/nginx-exporter --namespace front --set nginxServer="http://front-app.front.svc.cluster.local:8$i/basic_status"; sleep 10; done

```

## ДЗ №06 Templating

Работы выполнялись в managed service for kubernetes 1.22 by yandex

### nginx-ingress

```bash
kubectl create ns nginx-ingress
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -n nginx-ingress
```

Проверяем, что external ip нарисовался и идем дальше

### cert-manager

Древне говно мамонта из презентации на свежем кубике не взлетело, поэтому установил все более-менее свежее + дополнил созданием cluster issuer

```bash
cd kubernetes-templating/cert-manager/
kubectl apply -f cert-manager.crds.yaml 
kubectl apply -f cert-manager.yaml 
kubectl apply -f cluster-issuer.yaml
```

### chartmuseum

Chartmuseum из методички также уже deprecated, поэтому был взят из другого chart репозитория и также более свежий

```bash
cd kubernetes-templating/chartmuseum/
helm repo add chartmuseum https://chartmuseum.github.io/charts
helm install chartmuseum chartmuseum/chartmuseum --wait --namespace=chartmuseum --version 3.1.0 -f values.yaml
```

### harbor

```bash
cd kubernetes-templating/harbor/
kubernetes-templating/chartmuseum/
helm upgrade  --install harbor harbor/harbor --wait -f values.yaml
```

### Мучаем hipster-shop

В последней итерации хипстеркого магазина весь деплой попилен на 4 блока:

- сам hipster-shop деплой, а точнее то, что от него осталось
- шаблонизированный через хемл frontend деплой подтягивается как dependencies через chart.yaml
- шаблонизированный через kubecfg сервисы paymentservice и shippingservice
- шаблонизированный через kustomize сервис recommendationservice

Кастомизация через kubecfg из методички не взлетела, т.к. библиотеки не рабочие. Пришлось завезти библиотеку локально, поправить её и подключить

```bash
cd kubernetes-templating/
kubectl create ns hipster-shop
helm upgrade --install hipster-shop hipster-shop --namespace hipster-shop
kubecfg update jsonnet/services.jsonnet --namespace hipster-shop
kubectl create ns hipster-shop-prod
kubectl apply -k kubernetes-templating/kustomize/overrides/dev
```
