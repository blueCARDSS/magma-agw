# magma-agw

### Prerequisites

[VirtualBox](https://www.virtualbox.org) [Vagrant](https://vagrantup.com)

---

### Setup AGW

download magma repo:
```bash
git clone https://github.com/magma/magma.git --depth 1
```

move to gateway folder: 
```bash
cd magma/lte/gateway
```

download vagrant box:
```bash
vagrant box add magmacore/magma_dev \
  --box-version=1.1.20210618 \
  --provider=virtualbox
```

start vagrant box:
```bash
vagrant up magma
```

ssh inside vagrant box:
```bash
vagrant ssh magma
```
---

### Build AGW

build AGW
```bash
cd magma/lte/gateway
make run
```
---

### Configure AGW

Install scp plugin for Vagrant and copy the rootCA.pem file to AGW
```
HOST$ vagrant plugin install vagrant-scp
HOST$ vagrant scp /tmp/rootCA.pem magma:~

HOST$ vagrant ssh magma
```

First, copy the root CA for your Orchestrator deployment into your AGW:
```
AGW$ sudo mkdir -p /var/opt/magma/tmp/certs/
AGW$ sudo mv rootCA.pem /var/opt/magma/tmp/certs/rootCA.pem
```

Then, point your AGW to your Orchestrator:
```
AGW$ sudo mkdir -p /var/opt/magma/configs
AGW$ cd /var/opt/magma/configs
AGW$ sudo vim control_proxy.yml
```

Put the following contents into the file:
```
cloud_address: controller.magma.shubhamtatvamasi.com
cloud_port: 443
bootstrap_address: bootstrapper-controller.magma.shubhamtatvamasi.com
bootstrap_port: 443
fluentd_address: fluentd.magma.shubhamtatvamasi.com
fluentd_port: 24224

rootca_cert: /var/opt/magma/tmp/certs/rootCA.pem
```

Then restart your services to pick up the config changes:
```
AGW$ sudo service magma@* stop
AGW$ sudo service magma@magmad restart
AGW$ sudo service magma@magmad status

# check status of magma services
AGW$ sudo systemctl status magma@*
```

check logs:
```
AGW$ sudo tail -f /var/log/syslog
AGW$ sudo journalctl -u magma@magmad -f
```

grab the hardware secrets off your AGW:
```
AGW$ sudo pip3 install snowflake
AGW$ export AGW_SCRIPTS=/home/vagrant/magma/orc8r/gateway/python
AGW$ ln -s ${AGW_SCRIPTS}/magma ${AGW_SCRIPTS}/scripts/
AGW$ ${AGW_SCRIPTS}/scripts/show_gateway_info.py
```

test network:
```bash
checkin_cli.py
```

---

### Extras

Generate new Challenge key:
```bash
cd /var/opt/magma/certs
sudo openssl ecparam -name secp384r1 -genkey -noout -out gw_challenge.key
sudo chmod 644 gw_challenge.key
```

Generate new Hardware ID:
```bash
sudo snowflake --force-new-key
```

remove gateway keys to reset AGW:
```bash
cd /var/opt/magma/certs
sudo rm gateway.crt gateway.key
```
---

check certificate details:
```bash
openssl x509 -text -noout -in rootCA.pem

openssl x509 -text -noout -in /var/opt/magma/tmp/certs/rootCA.pem
```


