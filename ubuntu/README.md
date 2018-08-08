## Server setup
```sh
apt install docker.io -y
cat > /etc/docker/daemon.json <<EOF
{
  "insecure-registries" : ["172.30.0.0/16"]
}
EOF

systemctl restart docker
docker ps
```

```sh
wget https://github.com/openshift/origin/releases/download/v3.10.0/openshift-origin-client-tools-v3.10.0-dd10d17-linux-64bit.tar.gz
tar -xvf openshift-origin-client-tools-*-linux-64bit.tar.gz
mv openshift-origin-client-tools-*-linux-64bit/oc /usr/bin/
```

```sh
oc cluster up --public-hostname='lamit.win' --routing-suffix='lamit.win'
apt-get install software-properties-common
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install certbot
mkdir -p /var/www/lamit.win
oc cluster down
certbot certonly --standalone -w /var/www/lamit.win -d lamit.win  --email binhbat@live.com
/etc/letsencrypt/live/lamit.win/
```

```sh
cat > patch.yaml <<EOF
servingInfo:
namedCertificates:
- certFile: fullchain.pem
  keyFile: privkey.pem
  names:
  - "lamit.win"
EOF
```

```sh
wget https://github.com/mikefarah/yq/releases/download/2.1.1/yq_linux_amd64
chmod + yq_linux_amd64 && mv yq_linux_amd64 /usr/local/bin/yq

cp /var/lib/origin/openshift.local.config/master/master-config.yaml config.yaml
yq merge patch.yaml config.yaml > /var/lib/origin/openshift.local.config/master/master-config.yaml

cp /etc/letsencrypt/live/lamit.win/fullchain.pem var/lib/origin/openshift.local.config/master
cp /etc/letsencrypt/live/lamit.win/privkey.pem /var/lib/origin/openshift.local.config/master

oc cluster up --public-hostname='lamit.win' --routing-suffix='lamit.win' --use-existing-config=true
```
