# -*- mode: ruby -*-
# vi: set ft=ruby :

# Launch a Fedora minimal desktop pre-configured to connect into Dell iDRACs
Vagrant.configure('2') do |config|
  config.vm.define :dell_idrac_client do |d|
    # base on fedora
    d.vm.box = 'fedora/27-cloud-base'
    d.vm.box_url = 'https://download.fedoraproject.org/pub/fedora/linux/releases/27/CloudImages/x86_64/images/Fedora-Cloud-Base-Vagrant-27-1.6.x86_64.vagrant-virtualbox.box'

    # use a gui
    d.vm.provider :virtualbox do |vb|
       vb.gui = true
       vb.customize ['modifyvm', :id, '--memory', 1024]
    end
    d.vm.provider :libvirt do |libvirt|
       libvirt.graphics_type = 'vnc'
       libvirt.memory = 1024
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

    # configure system for use with the java idrac client and install racadm tools
    d.vm.provision 'shell', inline: <<-SHELL
       dnf -y --setopt=deltarpm=false install wget perl virt-manager
       wget -q -O - https://linux.dell.com/repo/hardware/dsu/bootstrap.cgi | bash
       sed 's@f$releasever@el7@' /etc/yum.repos.d/dell-system-update.repo| sed 's@dell-system-update_@dsu-dell-system-update_@' > /etc/yum.repos.d/dell-system-update-dsu.repo
       wget -q -O - https://linux.dell.com/repo/hardware/latest/bootstrap.cgi | bash
       sed -i 's@f$releasever@el6@' /etc/yum.repos.d/dell-system-update.repo
       dnf -y --setopt=deltarpm=false install @lxde-desktop-environment firefox onboard java-\*-openjdk.i\*86 srvadmin-idrac
       dnf -y --setopt=deltarpm=false install https://jsteffan.fedorapeople.org/drac_rpms/icedtea-web-1.7-0.4.pre06.fc27.x86_64.rpm https://jsteffan.fedorapeople.org/drac_rpms/firefox-49.0.2-1.fc27.x86_64.rpm
       grep -q 'excludepkgs=icedtea-web' /etc/dnf/dnf.conf 2>&1 || echo 'excludepkgs=icedtea-web' >> /etc/dnf/dnf.conf
       ln -fs /usr/lib64/libssl.so.10 /usr/lib64/libssl.so
       sed -i 's@:/opt/dell/srvadmin/bin@:/opt/dell/srvadmin/bin:/opt/dell/srvadmin/sbin@' /etc/profile.d/srvadmin-path.sh
       sed -i 's@JAVA=.*@'"JAVA=$(rpm -qa java\*|grep i686|xargs rpm -ql|grep bin/java)"'@' /usr/bin/javaws.itweb
       rm -vf $(rpm -qa java\*|grep i686|xargs rpm -ql|grep security/java.security)
       sed -i 's@# autologin=.*@autologin=vagrant@' /etc/lxdm/lxdm.conf
       mkdir -p /home/vagrant/.config/{autostart,clipit}
       echo -e '[Desktop Entry]\nType=Application\nExec=firefox https://linux.dell.com/repo/hardware/dsu/' > /home/vagrant/.config/autostart/firefox.desktop
       echo -e '[Desktop Entry]\nType=Application\nExec=onboard' > /home/vagrant/.config/autostart/onboard.desktop
       echo -e '[rc]' > /home/vagrant/.config/clipit/clipitrc
       echo -e 'lock: False\nmode: blank\ntimeout: 0:10:00' > /home/vagrant/.xscreensaver
       chown -R vagrant /home/vagrant
       systemctl set-default runlevel5.target
       reboot
    SHELL
  end
end
