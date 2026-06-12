# Home Network Threat Hunting Lab: Pi-hole + Suricata

I set this up to get some actual hands-on time with the stuff SOC analysts deal with day to day — DNS visibility, intrusion alerts, and digging through logs to figure out what happened. It's all running on Ubuntu, with Pi-hole watching DNS and Suricata watching the wire.

## Setup

Internet → Router → Pi-hole (DNS) → Suricata (traffic inspection) → my devices

Pi-hole sits in front as the local resolver, so every DNS query from anything on the network goes through it first. Suricata then looks at the traffic itself and checks it against signature rules.

## Tools

Ubuntu for the platform, Pi-hole for DNS logging and filtering, Suricata for IDS, the Linux CLI for pretty much everything else, and testmyids.com to check the IDS was actually catching things.

## Pi-hole

Once it was set up as the network's resolver, the dashboard gave me total queries, blocked queries, active clients, blocked domains, and a full query history. The query log ended up being the most useful part — I could see exactly which domains every device on the network was hitting.

## Suricata

Suricata ran as the IDS — packet inspection, signature matching, logging everything it flagged. Checked it was running with:

```
sudo systemctl status suricata
```

It stayed active and kept monitoring continuously.

## Testing it actually works

To make sure Suricata wasn't just sitting there doing nothing, I ran:

```
curl http://testmyids.com
```

This site exists specifically to trip IDS signatures. It returns a fake response that looks like output from a compromised system:

```
uid=0(root) gid=0(root) groups=0(root)
```

That's the kind of string you'd see if someone got command execution on a box and ran `id` to check what privileges they landed with — which is exactly why it's used to test IDS rules.

## Checking the alerts

After running the test, I grepped Suricata's log:

```
grep '"event_type":"alert"' /var/log/suricata/eve.json
```

And it showed up:

```
GPL ATTACK_RESPONSE id check returned root
```

Severity 2, flagged as potentially bad traffic. The alert fired because the response matched a known attack-response signature. The traffic itself was harmless (just the test site), but it confirmed Suricata was actually looking at the right things and reacting correctly.

## What I learned

**DNS** — how queries get logged and resolved, and how Pi-hole's blocking works under the hood.

**Traffic analysis** — basic TCP/IP, HTTP/HTTPS, TLS, and what packet inspection actually looks like once you're staring at it.

**IDS** — how signature-based detection works, and what separates a real alert from noise.

**Log analysis** — reading through eve.json, pulling out source/destination IPs, signatures, severity, flow data.

**Threat hunting** — not just waiting for alerts to fire, but actually going and checking: DNS activity, correlating events across logs, confirming a detection means what it says.

**Linux admin** — service management, troubleshooting, and generally living in the terminal.

## The annoying parts

Getting Pi-hole's DNS forwarding configured right took a few tries — nothing major, just the usual fiddling. Figuring out whether I was actually seeing all the traffic I expected to was its own small puzzle. And honestly, the first time I looked at raw Suricata output, none of it meant much. Telling a real concern apart from background noise took a bit of getting used to.

## Next steps

I'd like to add Wazuh or ELK for proper dashboards at some point, bring in a threat intel feed or two, write some custom Suricata rules instead of relying on the defaults, and maybe get Sysmon in there for endpoint visibility.

## Wrap-up

Small setup, but it covered a decent chunk of the basics — DNS monitoring, IDS alerting, log digging, poking around a network to see what's actually on it. Solid starting point if I go further into SIEM/SOC stuff later.
