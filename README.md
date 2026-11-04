# Suricata IDS Lab

I captured some of my own traffic on Kali Linux, ran it through Suricata offline, and wrote a simple custom rule to see how detection rules work.

**Tools:** Suricata (ET Open ruleset), tcpdump, Nmap, Kali Linux in a VM

## 1. Capture the traffic

I used tcpdump to capture a ping and an Nmap scan against `scanme.nmap.org`, Nmap's official test target, and saved it as `mycapture.pcap`.

## 2. Write a custom rule

I added this rule to `/etc/suricata/rules/local.rules`:

```
alert icmp any any -> any any (msg:"ICMP ping detected - custom rule"; sid:1000001; rev:1;)
```

It alerts on any ICMP (ping) traffic from anywhere to anywhere. `sid` values of 1000000 and up are meant for local rules.

## 3. Run Suricata against the capture

```
sudo suricata -r mycapture.pcap -s /etc/suricata/rules/local.rules -l ./logs
cat ./logs/fast.log
```

## 4. Read the alerts

My rule fired on the ping from my VM (`10.0.2.15`) to `scanme.nmap.org` (`45.33.32.156`). Suricata also flagged `UDPv4 invalid checksum` on DNS traffic:

![Suricata alerts in fast.log](screenshot-alerts.png)

The checksum alerts aren't an attack. The checksums get filled in by the network card after tcpdump grabs the packet, so they look wrong in the capture. It's a false positive caused by how the traffic was captured.

## Next

- Send the alerts to a SIEM (Wazuh or Splunk) instead of reading `fast.log`.
- Write rules for something more realistic than ping, like SSH brute-forcing.
