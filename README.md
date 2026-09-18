# fortiautoblock
Automatic permanent blocking for failed SSL VPN login attempts on fortigate devices

Fortinet SSL VPNs exposed to the internet are constantly scanned and logins are attempted with credential stuffing attacks.  Even though the SSL VPN is being discontinued by fortinet, I thought I'd post our solution here if it helps anyone out.

Our solution requires an internal linux VM/PC running a LAMP stack and it works like this

2 major parts, individual ip blocking and subnet blocking

Individual IP blocking

1. failed SSL logins have an automation in the fortigate that use a webhook to submit the source IP and username of the login to a PHP (I know PHP) API that inserts this into a MYSQL/MariaDB on the server.
2. there is a list of allowed vpn usernames in the DB that this new login failure is compared to, if the username matches the login is logged, but not flagged to be blocked
3. if the username does not match an allowed user the db is flagged to be blocked
4. apache webserver uses this table in the DB and generates a list of blocked IP addresses and the fortigate subscribes to this list on an internal URL and that is used as a block list in the rules before traffic is allowed to the SSL VPN endpoint.
5. finally a cron job runs and submits the ip addresses to ipinfo.io to look up the host or company that owns the ip addresses and populates the DB with this info.  This if very helpful for the 2nd part where if the IP belongs to a hosting company, we will ban the entire ip block since no remote user is coming from a server farm or hosting company.

Entire subnet blocking

1. another cron job runs against the logins table periodically and if the ISP is a webhost or server farm, something like Amazon etc, where no user would ever be logging in from, then that record is switched to a /24 subnet and moved to another table of subnets to block
2. there are a number of command line tools that can search and move records to this subnet table as /24 also, do a whois lookup and pick larger subnet masks to cover more ip ranges with a single entry
3. this table is also used to generate another list the fortigate can subscribe to for blocking subnets more than individual IP addresses

currently from the fortigates we manage we there are about 3700 entries in the individual ip blocking list and about 3200 in the subnet blocking list.  This applies mostly to IP addresses in the US as that is where the endpoints are so we use country blocking as well to limit connections from other locations.  This would work from anywhere though.
the alerts from failed logins and credential stuffing attacks are down to single digits per day with occasional flare ups when new ip blocks are bought or sold. 


08-18-2026 this is the initial repo creation, code will be posted in the coming days for the full solution
