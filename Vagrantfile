
Vagrant.configure("2") do |config|
    # Disbled synced folder
    #config.vm.synced_folder "C:/Users/david.martinez/Documents/vagrantshare", "/vagrant_data"

    # Enable hostnamager plugins to use hostname in Ansible inventory
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = true
    config.hostmanager.include_offline = true

    # Centos 7 vm Couchbase
    config.vm.define "centos7_couchbase", autostart: false do |centos7_couchbase|
        centos7_couchbase.vm.box = "centos/7"
        centos7_couchbase.vm.box_check_update = true
        centos7_couchbase.vm.hostname = "couchbase"
        centos7_couchbase.vm.network :private_network, ip: "10.0.3.15"
        #centos7_couchbase.vm.network "public_network", auto_config: true
        
        centos7_couchbase.vm.provider "virtualbox" do |v|
            v.name = "centos7-couchbase"
            v.memory = "3512"
            v.cpus = 2
        end
        centos7_couchbase.vm.network "forwarded_port", guest: 8091, host: 8091       
        centos7_couchbase.hostmanager.ip_resolver = proc do |vm, resolving_vm|
            if hostname = (vm.ssh_info && vm.ssh_info[:host])
                `vagrant ssh centos7-couchbase -c "hostname -I"`.split()[1]
            end
        end
       
        centos7_couchbase.vm.provision "shell", inline: "sudo yum -y update"
        centos7_couchbase.vm.provision "shell", inline: "sudo yum -y install net-tools"
        centos7_couchbase.vm.provision "shell", inline: "sudo curl -O https://packages.couchbase.com/releases/5.5.5/couchbase-server-enterprise-5.5.5-centos7.x86_64.rpm"
        centos7_couchbase.vm.provision "shell", inline: "sudo rpm -i couchbase-server-enterprise-5.5.5-centos7.x86_64.rpm"
        centos7_couchbase.vm.provision "shell", inline: "sudo yum -y update"
        centos7_couchbase.vm.provision "shell", inline: "sudo yum -y install git"
        centos7_couchbase.vm.provision "shell", inline: "sudo yum -y install wget"
        centos7_couchbase.vm.provision "shell", inline: "sudo yum -y install couchbase-server"
        centos7_couchbase.vm.provision "shell", inline: "wget https://github.com/totvslabs/couchbase-exporter/releases/download/v1.5.1/couchbase-exporter_1.5.1_linux_amd64.tar.gz"
        centos7_couchbase.vm.provision "shell", inline: "tar xzvf couchbase-exporter_1.5.1_linux_amd64.tar.gz"        
        centos7_couchbase.vm.provision "shell", run: "always",  inline: "sudo nohup /home/vagrant/./couchbase-exporter --couchbase.username Administrator --couchbase.password Administrator --web.listen-address=\":9420\" --couchbase.url=\"http://127.0.0.1:8091\" 0<&- &>/dev/null &"
    end

    # Centos 7 vm Prometheus
    config.vm.define "centos7_prometheus", autostart: false do |centos7_prometheus|
        centos7_prometheus.vm.box = "centos/7"
        centos7_prometheus.vm.box_check_update = true
        centos7_prometheus.vm.hostname = "prometheus"
        centos7_prometheus.vm.network :private_network, ip: "10.0.3.17"        
        centos7_prometheus.vm.provider "virtualbox" do |v|
            v.name = "centos7-prometheus"
            v.memory = "1024"
            v.cpus = 2
        end
                
        centos7_prometheus.vm.network "forwarded_port", guest: 9090, host: 9090        
        centos7_prometheus.vm.network "forwarded_port", guest: 3000, host: 3000
        centos7_prometheus.vm.network "forwarded_port", guest: 9093, host: 9093
        centos7_prometheus.vm.network "forwarded_port", guest: 9094, host: 9094

        centos7_prometheus.hostmanager.ip_resolver = proc do |vm, resolving_vm|
            if hostname = (vm.ssh_info && vm.ssh_info[:host])
                `vagrant ssh centos7_prometheus -c "hostname -I"`.split()[1]
            end
        end
       
        centos7_prometheus.vm.provision "shell", inline: "sudo yum -y update"
        centos7_prometheus.vm.provision "shell", inline: "sudo yum -y install net-tools"        
        centos7_prometheus.vm.provision "shell", inline: "sudo yum -y install wget"
        centos7_prometheus.vm.provision "shell", inline: "sudo yum -y install nano"
        centos7_prometheus.vm.provision "shell", inline: "sudo yum -y install git"
        centos7_prometheus.vm.provision "shell", inline: "wget https://github.com/prometheus/prometheus/releases/download/v2.14.0/prometheus-2.14.0.linux-amd64.tar.gz"
        centos7_prometheus.vm.provision "shell", inline: "tar xzvf prometheus-2.14.0.linux-amd64.tar.gz"        
        centos7_prometheus.vm.provision "shell", inline: "sudo rm -rf /home/vagrant/prometheus-2.14.0.linux-amd64/prometheus.yml"
        config.vm.provision "file", source: "./provision/prometheus_couchbase.yml", destination: "/home/vagrant/prometheus.yml"
        centos7_prometheus.vm.provision "shell", inline: "sudo cp /home/vagrant/prometheus.yml /home/vagrant/prometheus-2.14.0.linux-amd64/prometheus.yml"
        config.vm.provision "file", source: "./provision/couchbase.rules.yml", destination: "/home/vagrant/couchbase.rules.yml"
        centos7_prometheus.vm.provision "shell", inline: "sudo cp /home/vagrant/couchbase.rules.yml /home/vagrant/prometheus-2.14.0.linux-amd64/couchbase.rules.yml"
        centos7_prometheus.vm.provision "shell", inline: "nohup sudo ./prometheus-2.14.0.linux-amd64/prometheus --config.file /home/vagrant/prometheus-2.14.0.linux-amd64/prometheus.yml 0<&- &>/dev/null &"        
        config.vm.provision "file", source: "./provision/grafana.repo", destination: "/home/vagrant/grafana.repo"
        centos7_prometheus.vm.provision "shell", inline: "sudo mv /home/vagrant/grafana.repo /etc/yum.repos.d/"
        centos7_prometheus.vm.provision "shell", inline: "sudo yum -y install grafana"
        centos7_prometheus.vm.provision "shell", inline: "sudo systemctl start grafana-server"
        centos7_prometheus.vm.provision "shell", inline: "sudo systemctl enable grafana-server.service"
        config.vm.provision "file", source: "./provision/prometheus_ds.json", destination: "/home/vagrant/prometheus_ds.json"
        centos7_prometheus.vm.provision "shell", inline: "curl -u admin:admin -H 'Content-Type: application/json;charset=UTF-8' 'http://localhost:3000/api/datasources/' -X POST --data-binary @/home/vagrant/prometheus_ds.json"        


        #Alert manager
        centos7_prometheus.vm.provision "shell", inline: "wget https://github.com/prometheus/alertmanager/releases/download/v0.19.0/alertmanager-0.19.0.linux-amd64.tar.gz"
        centos7_prometheus.vm.provision "shell", inline: "tar xzvf alertmanager-0.19.0.linux-amd64.tar.gz"

        config.vm.provision "file", source: "./provision/alertmanager.yml", destination: "/home/vagrant/alertmanager.yml"
        centos7_prometheus.vm.provision "shell", inline: "sudo cp /home/vagrant/alertmanager.yml /home/vagrant/alertmanager-0.19.0.linux-amd64/alertmanager.yml"
        centos7_prometheus.vm.provision "shell", inline: "nohup sudo ./alertmanager-0.19.0.linux-amd64/alertmanager --config.file /home/vagrant/alertmanager-0.19.0.linux-amd64/alertmanager.yml 0<&- &>/dev/null &"
    end    
end
