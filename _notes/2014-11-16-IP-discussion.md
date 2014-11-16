
IP Discussion
=============

#2 IP Address Structure

IPv4 - 4 tuples of two hex digits written in decimal

eg: 192.168.1.12

Subnet mask specifies which digits are `network` and which are `address`

eg: a subnet mask for network: 192.168.1.* and host *.*.*.0-255 would be `255.255.255.0`

another example: network: 192.168.*.*; host: *.*.1-2.0-255 is: `255.255.254.0`

IPv4 addresses are too limited, so now we have IPv6 (128-bit) to replace it.

ipconfig (ifconfig on unix/linux) will show your IPv4, IPv6, Subnet mask, and default gateway.


#3 Static IP vs. Dynamic IP

Why is static IP useful

- You can set up firewall rules specific to IP addresses
- You can set up an ip for a web server you have deployed

*DHCP server* - issues IP addresses to clients from a database of available addresses.
- IP addresses are leased, and leased addresses cannot be reissued until the lease expires
- DHCP server prevents IP conflicts by issuing unique addresses
- Can be a server or a network device
- Clients can renew leases by alerting the DHCP that they are still online.

#4 DHCP commands

- `ipconfig /release` -- Done with this IP; cancels lease
- `ipconfig /renew` -- Requests a new IP address. May be the same address if the lease still exists.

#4 Exercise
# Check your IP address using ipconfig
# Try `ipconfig /all` too
# Try to ping the IP address of someone sitting next to you
# Try to trace the route to someone else's computer using `tracert`
# Try to trace the route to [www.google.com](www.google.com)

#3 DNS (Domain Name Server)

~Resolution between IP address and Host name (Name resolution)~

names are easily remembered than IP addresses

#4 DNS mini-exercise
- ping google.com
- ping www.google.com

*Local name resolution*
- ping localhost
- C:\Windows\System32\drivers\etc\hosts

When you type a URL the computer checks first in the hosts file for the URL. If it's not found then it goes to the next level for the DNS (usually the default gateway)

