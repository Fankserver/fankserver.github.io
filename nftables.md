NFTables
========

Example
-------

Default firewall configuration, eth0 (External, Internet) and eth1 (Internal, Lan):
```conf
table ip filter {
	chain input {
		type filter hook input priority 0;
		ct state established accept
		ct state related accept
		iif lo accept
		iif eth1 tcp dport ssh ct state new tcp flags & (syn | ack) == syn counter packets 0 bytes 0 accept
		icmp type echo-request accept
		input tcp dport 0-65535 reject
		input udp dport 0-65535 counter drop
		counter log drop
	}
	chain forward {
		type filter hook forward priority 0;
		iif eth0 oif eth1 ct state established accept
		iif eth0 oif eth1 ct state related accept
		iif eth1 oif eth0 counter accept
		iif eth0 oif eth1 counter log drop
	}
	chain output {
		type filter hook output priority 0;
		ct state established accept
		ct state related accept
		oif lo accept
		ct state new counter accept
	}
}
table ip6 filter {
	chain input {
		type filter hook input priority 0;
		ct state established accept
		ct state related accept
		iif lo accept
		iif eth1 tcp dport ssh ct state new tcp flags & (syn | ack) == syn counter packets 0 bytes 0 accept
		icmpv6 type echo-request accept
		ip6 hoplimit 1 icmpv6 type nd-neighbor-advert accept
		ip6 hoplimit 1 icmpv6 type nd-neighbor-solicit accept
		ip6 hoplimit 1 icmpv6 type nd-router-advert accept
		ip6 hoplimit 255 icmpv6 type nd-neighbor-advert counter accept
		ip6 hoplimit 255 icmpv6 type nd-neighbor-solicit counter accept
		ip6 hoplimit 255 icmpv6 type nd-router-advert counter accept
		input tcp dport 0-65535 reject
		input udp dport 0-65535 counter drop
		counter log drop
	}
	chain forward {
		type filter hook forward priority 0;
		iif eth0 oif eth1 ct state established accept
		iif eth0 oif eth1 ct state related accept
		iif eth1 oif eth0 counter accept
		iif eth0 oif eth1 counter log drop
	}
	chain output {
		type filter hook output priority 0;
		ct state established accept
		ct state related accept
		oif lo accept
		ct state new counter accept
	}
}
```
Here the command list with description:
```conf
# Allow icmp echo requests (ping)
nft add rule ip filter input icmp type echo-request accept
nft add rule ip6 filter input icmpv6 type echo-request accept

# Allow icmp ipv6 advertisement
nft add rule ip6 filter input ip6 hoplimit 1 icmpv6 type nd-neighbor-advert accept
nft add rule ip6 filter input ip6 hoplimit 1 icmpv6 type nd-neighbor-solicit accept
nft add rule ip6 filter input ip6 hoplimit 1 icmpv6 type nd-router-advert accept
nft add rule ip6 filter input ip6 hoplimit 255 icmpv6 type nd-neighbor-advert counter accept
nft add rule ip6 filter input ip6 hoplimit 255 icmpv6 type nd-neighbor-solicit counter accept
nft add rule ip6 filter input ip6 hoplimit 255 icmpv6 type nd-router-advert counter accept

# Allow established and related connections, in and out (but not from External -> Internal)
nft add rule ip filter input ct state established accept
nft add rule ip filter input ct state related accept
nft add rule ip filter forward iif eth0 oif eth1 ct state established accept
nft add rule ip filter forward iif eth0 oif eth1 ct state related accept
nft add rule ip filter output ct state established accept
nft add rule ip filter output ct state related accept
nft add rule ip6 filter input ct state established accept
nft add rule ip6 filter input ct state related accept
nft add rule ip6 filter forward iif eth0 oif eth1 ct state established accept
nft add rule ip6 filter forward iif eth0 oif eth1 ct state related accept
nft add rule ip6 filter output ct state established accept
nft add rule ip6 filter output ct state related accept

# Allow loopback
nft add rule ip filter input iif lo accept
nft add rule ip filter output oif lo accept
nft add rule ip6 filter input iif lo accept
nft add rule ip6 filter output oif lo accept

# Allow new tcp connection on SSH from Internal
nft add rule ip filter input iif eth1 tcp dport ssh ct state new tcp flags \& \(syn \| ack\) == syn counter packets 0 bytes 0 accept
nft add rule ip6 filter input iif eth1 tcp dport ssh ct state new tcp flags \& \(syn \| ack\) == syn counter packets 0 bytes 0 accept

# Reject everything else to firewall
nft add rule ip filter input tcp dport 0-65535 reject
nft add rule ip filter input udp dport 0-65535 counter drop
nft add rule ip filter input counter log drop
nft add rule ip6 filter input tcp dport 0-65535 reject
nft add rule ip6 filter input udp dport 0-65535 counter drop
nft add rule ip6 filter input counter log drop

# Allow Internal -> External connection
nft add rule ip filter input iif eth1 oif eth0 counter accept
nft add rule ip6 filter input iif eth1 oif eth0 counter accept

# Drop External -> Internal connection and log it
nft add rule ip filter input iif eth0 oif eth1 counter log drop
nft add rule ip6 filter input iif eth0 oif eth1 counter log drop
```