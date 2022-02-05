# ELK

## Requirements

- vagrant

```Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.define "debelk" do |debelk|
    debelk.vm.box_download_insecure = true
    debelk.vm.box = "debian/buster64"
    debelk.vm.network "forwarded_port", guest: 9200, host: 9200
    debelk.vm.network "forwarded_port", guest: 80, host: 8080
    debelk.vm.network "forwarded_port", guest: 5601, host: 5601
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
Launch on your browser http://100.0.0.20:5601/

3. Install Logstash

```sh
sudo apt install openjdk-11-jdk-headless
sudo wget --directory-prefix=/opt/ https://artifacts.elastic.co/downloads/logstash/logstash-7.6.1.deb
sudo dpkg -i /opt/logstash*.deb
sudo systemctl enable logstash
sudo systemctl start logstash
```

4. Logstash : Nginx local

- install nginx for logstash
```sh
sudo apt install nginx
sudo usermod -aG adm logstash
# => 100.0.0.20 - - [04/Feb/2022:23:58:57 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.64.0"
```

```sh
curl http://100.0.0.20
sudo less /var/log/nginx/access.log
```

- fichier de patterns

```sh
sudo mkdir /etc/logstash/pattern
sudo chmod 755 -R /etc/logstash/pattern
cat /etc/logstash/pattern/nginx
NGUSERNAME [a-zA-Z\.\@\-\+_%]+
NGUSER %{NGUSERNAME}

sudo nano /etc/logstash/conf.d/nginx.conf

```

```json
input {
    file {
        path => "/var/log/nginx/access.log"
        start_posistion => "beginning"
        sincedb_path => "/dev/null"
    }
}
filter {
    grok {
        patterns_dir => ["/etc/logstash/pattern"]
        match => {
            "message" => "%{IPORHOST:clientip} %{NGUSER:ident} %{NGUSER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{URIPATHPARAM:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response} "
        }
    }
}

output {
    elasticsearch {
        hosts => ["127.0.0.1:9200"]
        index => "nginx-%{+YYYY.MM.dd}"
    }
}
```
Rq: https://grokdebug.herokuapp.com/