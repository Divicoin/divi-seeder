XSN Seeder
====================

xsn-seeder is a crawler for the XSN Core network, which
exposes a list of reliable nodes via a built-in DNS server, or instead
just generates that list to push to a remote CloudFlare server.

Features:
* CloudFlare DNS integration
* regularly revisits known nodes to check their availability
* bans nodes after enough failures, or bad behaviour
* accepts nodes down to v0.3.19 to request new IP addresses from,
  but only reports good post-v0.3.24 nodes.
* keeps statistics over (exponential) windows of 2 hours, 8 hours,
  1 day and 1 week, to base decisions on.
* very low memory (a few tens of megabytes) and cpu requirements.
* crawlers run in parallel (by default 24 threads simultaneously).


REQUIREMENTS
------------

$ sudo apt-get install build-essential libboost-all-dev libssl-dev


USAGE: LOCAL DNS SERVER MODE
----------------------------

Assuming you want to run a dns seed on dnsseed.example.com, you will
need an authorative NS record in example.com's domain record, pointing
to for example vps.example.com:

$ dig -t NS dnsseed.example.com

;; ANSWER SECTION
dnsseed.example.com.   86400    IN      NS     vps.example.com.

On the system vps.example.com, you can now run dnsseed:

./dnsseed -h dnsseed.example.com -n vps.example.com

If you want the DNS server to report SOA records, please provide an
e-mail address (with the @ part replaced by .) using -m.


USAGE: CLOUDFLARE API MODE
--------------------------

Have the seeder above running all the time, but with no NS record:

```bash
./dnsseed -h {host_ip}
```

Fill in CloudFlare API config (see ./cf-uploader/README.md) and have the CloudFlare upload run on a
cron job:

#### Create a python virtual env:

```bash
virtualenv ./venv
```

#### Set up a crontab entry to call `cf-uploader` every 30 minutes:

```$ crontab -e```

In the crontab editor, add the lines below:

```bash
* * * * * cd {xsn_seeder_path}/cf-uploader && ./venv/bin/python seeder.py >/dev/null 2>&1
```

COMPILING
---------
Compiling will require boost and ssl.  On debian systems, these are provided
by `libboost-dev` and `libssl-dev` respectively.

$ make

This will produce the `dnsseed` binary.


RUNNING AS NON-ROOT
-------------------

Typically, you'll need root privileges to listen to port 53 (name service).

One solution is using an iptables rule (Linux only) to redirect it to
a non-privileged port:

$ iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353

If properly configured, this will allow you to run dnsseed in userspace, using
the -p 5353 option.
