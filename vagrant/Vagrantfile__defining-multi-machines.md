bento/centos-7.2 imageを使って複数のVMを立ち上げるVagrantfile例

```

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"

  config.vm.define :server do | server |
    server.vm.network "private_network", ip: "192.168.33.10"
  end
  
  config.vm.define :client do | client |
    client.vm.network "private_network", ip: "192.168.33.11"
  end
end

```

今のままの設定だと nmcli で見たときに新たに作成された
device(enp0s8)が登録されていないので、それを登録する方法を調べる。
また、同じdeviceに IPv6設定する方法も調べる。
