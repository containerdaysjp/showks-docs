# TODO

* helm templateで生成したのをapplyする

# helm clientのインストール

```
curl -L https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-linux-amd64.tar.gz -o helm.tar.gz
tar -zxvf helm.tar.gz
mkdir ~/bin
mv linux-amd64/helm ~/bin/
```

# kubectl clientのインストール

```
curl -O https://storage.googleapis.com/kubernetes-release/release/v1.10.5/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl ~/bin/
```

```
gcloud container clusters get-credentials showks-dev --zone asia-northeast1-b --project ipc-containerdays
```

# Helmの初期化

```
kubectl -n kube-system create serviceaccount tiller

kubectl create --save-config clusterrolebinding tiller \
    --clusterrole=cluster-admin \
    --user="system:serviceaccount:kube-system:tiller"

helm init --service-account tiller
```

# 証明書の作成

```
sudo apt update
sudo apt install certbot

sudo certbot certonly --manual -d *.showks.containerdays.jp -m jkd-showk@googlegroups.com --agree-tos --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory

# KeyとCert
# /etc/letsencrypt/live/showks.containerdays.jp/privkey.pem
# /etc/letsencrypt/live/showks.containerdays.jp/fullchain.pem

sudo cp /etc/letsencrypt/live/showks.containerdays.jp/fullchain.pem /etc/letsencrypt/live/showks.containerdays.jp/privkey.pem ~/.ssh
sudo chown `whoami`: ~/.ssh/privkey.pem ~/.ssh/fullchain.pem
```

# Concourseのインストール

```
kubectl create secret tls concourse-web-tls --key ~/.ssh/privkey.pem  --cert ~/.ssh/fullchain.pem

helm install stable/concourse --name my-cci --version 2.0.2 --values values.yaml

cat << _EOF_ > ./values.yaml
concourse:
  web:
    externalUrl: https://concourse.showks.containerdays.jp
  auth:
    github:
      enabled: true
    mainTeam:
      github:
        user: MasayaAoyama
        org: containerdaysjp
web:
  ingress:
    enabled: true
    hosts:
      - concourse.showks.containerdays.jp
    annotations:
      kubernetes.io/ingress.global-static-ip-name: "showks-concourse-ip"
    tls:
      - secretName: concourse-web-tls
        hosts:
          - concourse.showks.containerdays.jp
_EOF_
```
