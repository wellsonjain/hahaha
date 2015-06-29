---
title: Setup Vagrant for Rails Development
date: 2015-05-27 14:07 UTC
tags: devops, rails
layout: post
---
> The price of greatness is responsibility. - Winston Churchill

[Vagrant](http://www.vagrantup.com/) 一直是我很想好好搞懂的東西，因為搞懂了這個才能接著繼續了解 [Chef](https://www.chef.io/chef/)，進一步了解 [Opsworks](http://aws.amazon.com/opsworks/)，這是個一連串的學習鏈，首先今天就來記錄一下我架設 Vagrant 的過程。

首先先來介紹一下 Vagrant 是什麼，Vagrant 是一個可以在本機的 Virtual Mechine 架設另一個開發環境，簡單來說就是一個平行宇宙的精神時光屋，所以只要 OS 版本以及其他相關 package 的版本一致，那麼就算一起工作的夥伴，每個人的電腦版本內容不同，只要在 Virtual Machine 上都可以模擬出相同的環境。

><strong>講個秘訣</strong>：在開發的過程當中要力求環境版本相同以減少溝通成本，不然環境十個人十個不同你以為在測相容性測試嗎?

至於 Chef 則是在我們架設精神時光屋的時候，可能會需要的套件，因為一個 Vagrant 架起來可能裡面什麼套件都沒有相當陽春，所以 Chef 在這時候就起了一個相當牛逼的功效。

#### Setting Up Vagrant
<em>確保電腦有 2G 的 "Free" RAM</em>，因為雖說是 run 在 Virtual Mechine 上，但是依然是會 run 一整個 OS，所以無論如何都是需要一些基本的硬體規格。

首先要做的事情有兩個，

1. [Install Vagrant](http://www.vagrantup.com/downloads.html)
2. [Install VirtualBox](https://www.virtualbox.org/wiki/Downloads)

VirtualBox 就是我們 run VM 的地方，安裝好上面的兩個軟體之後，接著我們要在 terminal 裡面做一些事情，

```
  $ vagrant plugin install vagrant-vbguest
  $ vagrant plugin install vagrant-librarian-chef-nochef
```
`vagrant-vbguest` 會自動的安裝 VirtualBox Guest Additions 在我們的 guest system。
`vagrant-librarian-chef-nochef` 則是讓我們在啟動 vagrant 的時候可以也自動啟動 chef。

上面兩件事情都做完之後，我們就在 terminal 中移動到我們的 rails 資料夾下，執行 `vagrant init` 來初始化 vagrant，然後再做 `touch Cheffile`，這就跟 Gemfile 一樣，我們在裡面註明我們需要哪些 package，然後我們要做的就是在 Cheffile 裡面放入下面的設定，

```sql
site "http://community.opscode.com/api/v1"

cookbook 'apt'
cookbook 'build-essential'
cookbook 'mysql', '5.5.3'
cookbook 'ruby_build'
cookbook 'nodejs'
cookbook 'rbenv', git: 'https://github.com/aminin/chef-rbenv'
cookbook 'vim'
```
接著就是設定我們在初始化 vagrant 之後所生出來的 `Vagrantfile`，把下面的設定貼上去就好，不過如果你想了解細節的話，初始化的內容裡註解都寫得相當詳細，

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Use Ubuntu 14.04 Trusty Tahr 64-bit as our operating system
  config.vm.box = "ubuntu/trusty64"

  # Configurate the virtual machine to use 2GB of RAM
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
  end

  # Forward the Rails server default port to the host
  config.vm.network :forwarded_port, guest: 3000, host: 3000

  # Use Chef Solo to provision our virtual machine
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = ["cookbooks", "site-cookbooks"]

    chef.add_recipe "apt"
    chef.add_recipe "nodejs"
    chef.add_recipe "ruby_build"
    chef.add_recipe "rbenv::user"
    chef.add_recipe "rbenv::vagrant"
    chef.add_recipe "vim"
    chef.add_recipe "mysql::server"
    chef.add_recipe "mysql::client"

    # Install Ruby 2.2.1 and Bundler
    # Set an empty root password for MySQL to make things simple
    chef.json = {
      rbenv: {
        user_installs: [{
          user: 'vagrant',
          rubies: ["2.2.1"],
          global: "2.2.1",
          gems: {
            "2.2.1" => [
              { name: "bundler" }
            ]
          }
        }]
      },
      mysql: {
        server_root_password: ''
      }
    }
  end
end
```
接著就是啟動 vagrant，第一次的初始化因為要下載必要套件，所以可能會花上一段時間，結束之後我們只要用 vagrant ssh 就能進到我們的精神時光屋了！

```shell
  $ vagrant up

  # 經過漫長的等待...

  $ vagrant ssh
```
裏頭的狀況就跟我們剛把一個專案從 remote repo 拉下來一樣，還是得要做 bundle install 以及 migration，然後一個在獨立的環境的獨立 rails app 就這樣完成了。
