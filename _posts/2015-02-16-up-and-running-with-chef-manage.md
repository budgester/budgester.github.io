---
title: "Up and Running with Chef Manage - Web UI Console"
permalink: /2015/02/so-to-get-full-chef-stack-running-on.html
---

So to get the full Chef stack running on Vagrant, you have to follow the steps below.

The base `precise64` image is Ubuntu 12.04, so make sure you download the correct Chef installer.

Increase the memory of the Vagrant machine. I made it 2 GB and this made it work fairly quickly:

```ruby
config.vm.provider "virtualbox" do |vb|
  vb.customize ["modifyvm", :id, "--memory", "2048"]
end
```

When adding Opscode Manage, make sure you allow a relevant port to be forwarded. I added a port forward of 4443 to 4443.

Edit `/etc/opscode-manage/manage.rb` and add the line:

```
public_port 4443
```

Log into the interface on <https://127.0.0.1:4443>.
