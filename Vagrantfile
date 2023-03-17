# -*- mode: ruby -*-
# vi: set ft=ruby :

# Launch a Fedora minimal desktop pre-configured to connect into Dell iDRACs
Vagrant.configure('2') do |config|
  config.vm.define :dell_idrac_client do |d|
    # base on fedora
    d.vm.box = 'fedora/36-cloud-base'

    # use a gui
    d.vm.provider :virtualbox do |vb|
       vb.gui = true
       vb.customize ['modifyvm', :id, '--memory', 2048]
    end
    d.vm.provider :libvirt do |libvirt|
       libvirt.graphics_type = 'vnc'
       libvirt.memory = 2048
    end
    d.ssh.forward_x11 = true

    # use all available cpu cores
    host = RbConfig::CONFIG['host_os']
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
    elsif host =~ /linux/
      cpus = `nproc`.to_i
    else
      cpus = 1
    end
    d.vm.provider :virtualbox do |vb|
      vb.customize ['modifyvm', :id, '--cpus', cpus]
    end
    d.vm.provider :libvirt do |libvirt|
      libvirt.cpus = cpus
    end

    # disable default synced folder
    d.vm.synced_folder ".", "/vagrant", disabled: true

    # configure system for use with the java idrac client
    d.vm.provision 'shell', inline: <<-SHELL
       dnf -y install @lxde-desktop-environment onboard virt-manager firefox https://kojipkgs.fedoraproject.org//packages/icedtea-web/2.0.0/pre.0.3.alpha16.patched1.1.fc36.2/x86_64/icedtea-web-2.0.0-pre.0.3.alpha16.patched1.1.fc36.2.x86_64.rpm
       dnf -y install https://kojipkgs.fedoraproject.org/packages/java-1.8.0-openjdk/1.8.0.362.b09/1.fc36/i686/java-1.8.0-openjdk-1.8.0.362.b09-1.fc36.i686.rpm https://kojipkgs.fedoraproject.org/packages/java-1.8.0-openjdk/1.8.0.362.b09/1.fc36/i686/java-1.8.0-openjdk-headless-1.8.0.362.b09-1.fc36.i686.rpm
       sed -i 's@JAVA=.*@'"JAVA=$(rpm -qa java\*|grep i686|xargs rpm -ql|grep bin/java)"'@' /usr/bin/javaws
       rm -vf $(rpm -qa java\*|grep i686|xargs rpm -ql|grep security/java.security)
       sed -i 's@# autologin=.*@autologin=vagrant@' /etc/lxdm/lxdm.conf
       mkdir -p /home/vagrant/.config/{autostart,clipit}
       echo -e '[Desktop Entry]\nType=Application\nExec=firefox https://www.java.com/en/download/help/testvm.xml' > /home/vagrant/.config/autostart/firefox.desktop
       echo -e '[Desktop Entry]\nType=Application\nExec=onboard' > /home/vagrant/.config/autostart/onboard.desktop
       echo -e '[rc]' > /home/vagrant/.config/clipit/clipitrc
       echo -e 'lock: False\nmode: blank\ntimeout: 0:10:00' > /home/vagrant/.xscreensaver
       chown -R vagrant /home/vagrant
       systemctl set-default runlevel5.target
       reboot
    SHELL
  end
end
