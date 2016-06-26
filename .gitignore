rpm -ivh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum -y  update
yum -y install strongswan
yum -y install xl2tpd
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/g' /etc/sysctl.conf
sysctl -p
iptables -A INPUT -p udp --dport 500 -j ACCEPT # IKE
iptables -A INPUT -p udp --dport 4500 -j ACCEPT # NAT-T
iptables -A INPUT -p udp -m policy --dir in --pol ipsec -m udp --dport 1701 -j ACCEPT # 強制l2tp透過ipsec存取
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE
ip_addr=` ifconfig  | grep 'inet addr:'| grep -v '127.0.0.1' |cut -d: -f2 | awk '{ print $1}'`
echo "conn %default
        ikelifetime=24h
        keylife=24h
        rekeymargin=30m 
        keyingtries=1
        rekey=no
 
 
conn l2tp
        keyexchange=ikev1 # IKE版本
        left=$ip_addr
        leftsubnet=0.0.0.0/0
        leftprotoport=17/1701 # l2tp udp流量
        authby=secret # PSK驗證
        leftfirewall=no # 不要讓strongswan更改防火牆
        right=%any # 任意IP
        rightprotoport=17/%any # 任意port udp流量
        type=transport # ipsec transport mode 
        auto=add">>/etc/strongswan/ipsec.conf
echo ': PSK "fuck_great_firewall"'>>/etc/strongswan/ipsec.secrets
sed 's/\[global\].*$/&\nlisten-addr ='$ip_addr'/g' /etc/xl2tpd/xl2tpd.conf
sed -i 's/^mtu.*/mtu 1200/g' /etc/ppp/options.xl2tpd
sed -i 's/^mru.*/mru 1200/g' /etc/ppp/options.xl2tpd
echo 'login'>>/etc/ppp/options.xl2tpd
echo 'test * test123 *'>>/etc/ppp/chap-secrets
service strongswan start
service xl2tpd start
chkconfig strongswan on
chkconfig xl2tpd on
