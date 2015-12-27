#flash openwrt 15.05
#set the router ip to 192.168.10.1
#change the luci port to 8080 by change /etc/config/uhttpd

#copy the basehosts to /etc
#dnsmasq is a all in 1 of dhcp and dns. by default dnsmasq will assign itself (192.168.10.1) as dns server to its dhcp client, and it will use this same address as dns for itself,
what we want is, its client will still use 192.168.10.1 as their dns, but the router itself will use its upstream dns, or 8.8.8.8,
change /etc/init.d/dnsmasq, add the following 3 lines to the bottom of start_service

rm -f /tmp/resolv.conf
ln -s /tmp/resolv.conf.auto /tmp/resolv.conf
cp /etc/basehosts /tmp/hosts/

these 3 lines basicly did: it will send 192.168.10.11 to its clients as dns, but will have /etc/resolve.conf have 192.168.2.1 (its upstream dns)
dnsmasq have dns function which will lookup /tmp/hosts/, that is why we copy basehosts into /tmp/hosts/. remember edit the basehosts so all these servers point to the router

so now, when the clients need access api.youku.com, the router will tell the client the ip of api.youku.com is 192.168.10.1. and then the client will send request
to the router, the privoxy will pick up the request and send then request to rule cnforward.action, which will send the request to 192.241.222.103, a server in china

#/etc/init.d/dnsmasq reload
#so now, if you ping api.163.com from client, it will return 192.168.10.1, if you ping api.163.com from the router, it will return the real ip address

#opkg install privoxy
#copy cnfilter.filter, cnforward.action, cnfilter.action into /etc/privoxy.
#change /etc/config/privoxy, add these 5 lines

list    actionsfile     'cnforward.action'
list    actionsfile     'cnfilter.action'
list    filterfile      'cnfilter.filter'

list    listen_address  '192.168.10.1:80'
option  accept_intercepted_requests     '1'

#the privoxy has to use port 80, and accept_intercepted_requests should be 1 (true)
#chnage /etc/init.d/privoxy, becuase only root can bind to port 80

#/usr/sbin/privoxy --pidfile $PIDFILE --user privoxy.privoxy $CFGFILE
/usr/sbin/privoxy --pidfile $PIDFILE --user root.root $CFGFILE
