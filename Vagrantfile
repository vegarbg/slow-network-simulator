# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.require_version ">= 1.6.0"

$script1 = <<SCRIPT
((
echo "Creating desktop shortcut to network shaped Firefox"
mkdir -p /home/vagrant/Desktop/
chown vagrant:vagrant /home/vagrant/Desktop/
) 2>&1) | tee -a $LOGFILE
SCRIPT

$script2 = <<SCRIPT
LOGFILE=/vagrant/log/setup.log
export DEBIAN_FRONTEND=noninteractive

((
echo "Updating apt cache"
) 2>&1) | tee -a $LOGFILE
apt-get update -qq

#echo "Installing xfce4"
#apt-get install -y -qq xfce4 >>$LOGFILE 2>&1
#echo "allowed_users=anybody" >/etc/X11/Xwrapper.config

((
echo "Installing MATE"
echo "Adding repositories"
) 2>&1) | tee -a $LOGFILE
apt-add-repository ppa:ubuntu-mate-dev/ppa >>$LOGFILE 2>&1
apt-add-repository ppa:ubuntu-mate-dev/trusty-mate >>$LOGFILE 2>&1
((
echo "Updating apt cache"
) 2>&1) | tee -a $LOGFILE
apt-get update -qq >>$LOGFILE 2>&1
((
echo "Upgrading packages"
) 2>&1) | tee -a $LOGFILE
apt-get upgrade -y -qq >>$LOGFILE 2>&1
((
echo "Installing X11 and MATE packages (500 MB; this will take a while)"
) 2>&1) | tee -a $LOGFILE
apt-get install -y -qq --no-install-recommends ubuntu-mate-core ubuntu-mate-desktop >>$LOGFILE 2>&1
echo "allowed_users=anybody" >/etc/X11/Xwrapper.config

((
echo "Installing VirtualBox guest tools"
) 2>&1) | tee -a $LOGFILE
apt-get install -y -qq virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11 >>$LOGFILE 2>&1

((
#echo "Starting VBoxClient"
#VBoxClient-all

echo "Installing firefox"
) 2>&1) | tee -a $LOGFILE
apt-get install -y -qq firefox >>$LOGFILE 2>&1
((
chmod a+x /home/vagrant/Desktop/slow_firefox.desktop

echo "Downloading Firebug add-on to Firefox"
wget -qO /home/vagrant/Desktop/firebug.xpi https://addons.mozilla.org/firefox/downloads/latest/1843/addon-1843-latest.xpi?src=search >>$LOGFILE 2>&1

echo "Setting default home page in Firefox"
) 2>&1) | tee -a $LOGFILE
echo 'pref("browser.startup.homepage", "about:blank")' >>/etc/xul-ext/ubufox.js

((
echo "Installing trickle"
) 2>&1) | tee -a $LOGFILE
apt-get install -y -qq trickle >>$LOGFILE 2>&1

((
echo "Setting up network shaping"
tc qdisc add dev eth0 root handle 1:0 netem delay 300ms 100ms 25% loss 1% 25%
tc qdisc add dev eth0 parent 1:1 handle 10: tbf rate 70kbit buffer 1600 limit 3000
modprobe ifb
ip link set dev ifb0 up
tc qdisc add dev eth0 ingress
tc filter add dev eth0 parent ffff: protocol ip u32 match u32 0 0 flowid 1:1 action mirred egress redirect dev ifb0
tc qdisc add dev ifb0 root netem delay 300ms 100ms 25% loss 1% 25%

echo "Starting trickled"
trickled -d 120 -u 70

echo "To start the GUI, log in as user 'vagrant' (password: vagrant), and run"
#echo "startxfce4 &"
echo "startx &"
) 2>&1) | tee -a $LOGFILE
SCRIPT

Vagrant.configure(2) do |config|
    config.vm.hostname = "slownetsim"
    config.ssh.insert_key = false
    config.vm.box = "ubuntu/trusty32"
    config.vm.define "slow-net-sim" do |v|
    end
    config.vm.provider :virtualbox do |vb|
        vb.name = "Slow network simulator"
        vb.gui = true
    end
    config.vm.provision "shell", inline:
        "echo \"Clearing log file\"; echo \"\" >/vagrant/log/setup.log"
    config.vm.provision "shell", inline: $script1
    config.vm.provision "file", source: "resources/slow_firefox.desktop", destination: "/home/vagrant/Desktop/slow_firefox.desktop"
    config.vm.provision "shell", inline: $script2
end
