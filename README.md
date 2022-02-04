# ELK

## Requirements

- vagrant

```Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.define "debelk" do |debelk|
    debelk.vm.box_download_insecure = true
    debelk.vm.box = "debian/buster64"
    debelk.vm.network "forwarded_port", guest: 9300, host: 9300
    debelk.vm.network "private_network", ip: "100.0.0.20"
    debelk.vm.hostname = "debelk"
    debelk.vm.provider "virtualbox" do |v|
      v.name = "debelk"
      v.memory = 3072
      v.cpus = 2
    end
  end

end
```

```sh
cd vagrant
$ vagrant up
```
## Installation of the stack

1. Install Elasticsearch

```sh
sudo wget --directory-prefix=/opt/ https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.1-amd64.deb
sudo dpkg -i /opt/elasticsearch*.deb
# edit /etc/elasticsearch/jvm.options
#   -Xms512m
#   -Xmx512m
# edit /etc/elasticsearch/elasticsearch.yml
# node.name: node-1
# network.host: 0.0.0.0    
# host.name + master
#network.host: 0.0.0.0
#discovery.seed_hosts: ["127.0.0.1", "100.0.0.20"]
systemctl enable elasticsearch
systemctl start elasticsearch

```

2. Install Kibana

```sh
sudo wget --directory-prefix=/opt/ https://artifacts.elastic.co/downloads/kibana/kibana-7.6.1-amd64.deb
sudo dpkg -i /opt/kibana*.deb
# edit /etc/kibana/kibana.yml
#server.host: "0.0.0.0"
sudo systemctl start kibana
sudo systemctl enable kibana

```

3. Install Logstash

```sh
sudo apt install openjdk-11-jdk-headless
sudo wget --directory-prefix=/opt/ https://artifacts.elastic.co/downloads/logstash/logstash-7.6.1.deb
sudo dpkg -i /opt/logstash*.deb
sudo systemctl enable logstash
sudo systemctl start logstash
```