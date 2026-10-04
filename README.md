# mac socks5 proxy: how to set one up on macOS, route the apps you care about, and pick a residential provider that actually supports it

Setting a SOCKS5 proxy on a Mac takes about forty seconds. Getting it to route exactly what you want — and to leave everything else alone — takes a bit longer, and that's usually where people get stuck.

The other half of the problem is the endpoint itself. macOS has a SOCKS proxy field built in, but it doesn't come with a proxy. Most people searching for a mac socks5 proxy are trying to solve two things at once: how to configure it system-wide, and where to get a SOCKS5 endpoint that isn't a dead free list from 2019. This covers both, including the parts of the setup that quietly fail.

## What macOS actually does with a SOCKS5 proxy

SOCKS5 is a transport-level protocol. It forwards raw TCP connections to an upstream server without understanding what's inside them. That's the practical difference from an HTTP proxy: an HTTP proxy speaks HTTP, rewrites headers, and can be picky about protocol versions. SOCKS5 doesn't care whether you're sending HTTP, TLS, or something custom — it just opens a socket.

Two consequences matter on a Mac:

- **Authentication is optional but supported.** macOS has a "Proxy server requires password" checkbox in the SOCKS section. If your provider gives you username/password credentials, this works — but see the command-line section below, where it doesn't.

- **DNS handling depends on the client.** Whether your hostnames get resolved at the proxy or locally varies by app. That's why leak testing isn't paranoia; it's part of the setup.

Also worth knowing: the system-wide setting only applies to applications that read system proxy settings. Safari, Mail, and most App Store apps do. A large share of CLI tools, and plenty of desktop apps, ignore it entirely.

## Three ways to run SOCKS5 on a Mac

They're not mutually exclusive, and the right one depends on whether you need one app or the whole machine.

### 1. System-wide via System Settings

Apple's own path: **Apple menu → System Settings → Network → select your active service (Wi-Fi or Ethernet) → Details → Proxies**. Check **SOCKS proxy**, enter host and port, click OK. On older macOS builds the same panel lives under System Preferences → Network → Advanced → Proxies.

This is the lowest-effort option and the one most guides show. It's also the bluntest: everything that respects system proxy settings goes through the proxy, including your online banking session, unless you use the bypass list.

### 2. Per-app

- **Browsers**: Firefox has native SOCKS5 support under Settings → Network Settings, including a "Proxy DNS when using SOCKS v5" toggle. Chrome and Safari rely on the system setting, so separate profiles or extensions are the usual workaround.

- **Telegram**: built-in. Settings → Data and Storage → Proxy Settings → Add proxy → SOCKS5, with address, port, username, password.

- **Everything else**: Proxifier routes individual applications through a proxy even when they have no proxy support. You point it at a SOCKS5 host and port, tag the apps you want, and it handles the rest.

### 3. Command line

For scripting, headless work, and anything running in a terminal, the system setting usually does nothing. Use per-command or per-environment configuration instead:

bash

# DNS resolved at the proxy (recommended)

curl --socks5-hostname user:pass@proxy-host:1080 https://example.com

# SOCKS5 proxy through environment variables, for tools that read them

export ALL_PROXY=socks5h://user:pass@proxy-host:1080



The `h` in `socks5h` is the part people miss. `socks5://` resolves DNS locally; `socks5h://` hands hostnames to the proxy. If a site is supposedly blocked and the connection still fails, that's often the reason.

## System-wide SOCKS5 on macOS: the exact steps

1. Confirm you have a SOCKS5 host and port from your provider. With most residential proxy services this means the endpoint is either an IP with a forwarded port, or a gateway hostname plus a generated port.

2. Open **System Settings** and search for "Proxies" — on recent macOS versions this jumps straight to the right panel. Otherwise navigate Network → your service → Details → Proxies.

3. Tick **SOCKS proxy** and fill in the server field with the hostname or IP, and the port in the small box after the colon.

4. If your credentials are username/password, tick **Proxy server requires password** and enter them. If your provider supports IP whitelisting instead, you can often skip this step.

5. Click OK, then Apply.

6. Verify before you trust it.

If you prefer to script it, `networksetup` does most of the work:

bash

networksetup -setsocksfirewallproxy "Wi-Fi" 203.0.113.10 1080

networksetup -setsocksfirewallproxystate "Wi-Fi" on

networksetup -getsocksfirewallproxy "Wi-Fi"



The service name has to match exactly — "Wi-Fi", "Ethernet", or whatever your Mac calls it. One limitation: `networksetup` sets host and port but has no argument for SOCKS credentials, so authenticated setups need the GUI or per-app config. To inspect what's currently active, `scutil --proxy` prints the live proxy state, which is a faster diagnostic than clicking through panels.

To turn it off again:

bash

networksetup -setsocksfirewallproxystate "Wi-Fi" off



## Where the SOCKS5 endpoint comes from

Free SOCKS5 lists exist. They're also the reason people conclude SOCKS5 is unreliable — the endpoints are shared, often already blacklisted, and frequently just offline. For anything that needs to look like a normal user in a specific location, residential IPs are what's being sold, and most providers in this space support SOCKS5 as an output protocol alongside HTTP.

One thing to check before buying: **not every provider treats SOCKS5 as a first-class option on every plan.** Some offer it only on certain products, some want it configured through the dashboard, some require a desktop client.

9Proxy is a residential proxy provider with 20M+ residential IPs across 90+ countries, and it supports both HTTP/HTTPS and SOCKS5 across its product lines. On a Mac there's one structural detail worth understanding before you buy, because it changes the setup steps:

- **GB-based plans** work entirely from the dashboard. You generate an endpoint, authenticate with username/password or IP whitelisting, and paste it straight into macOS, Proxifier, or an antidetect browser. No local software involved.

- **IP-based plans** use per-IP port forwarding, which historically required the desktop app. 9Proxy also offers a browser-based access panel for these, so you can retrieve and manage IP-based proxies without installing anything — useful on locked-down Macs.

Both modes support country, state, city, ZIP, and ISP-level targeting, with sticky or rotating sessions.

👉 [Check 9Proxy's current IP-based and GB-based plans](https://bit.ly/9-Proxy)

## 9Proxy plans compared

The pricing structure below reflects the adjustment 9Proxy announced for IP-based and bundle packages taking effect June 1, 2026. Bandwidth (GB) pricing was explicitly left unchanged. Prices are listed as published; confirm current figures on the live page before purchasing.

**IP-based residential packages** — pay per IP, unlimited bandwidth per IP, unused IPs don't expire.

| Package | What you get | Price | Per-IP | Purchase |
| --- | --- | --- | --- | --- |
| 100 IPs | 100 residential IPs, unlimited traffic | $24 | $0.24 | [Get the 100 IP package](https://bit.ly/9-Proxy) |
| 500 IPs | 500 residential IPs, unlimited traffic | $72 | $0.144 | [Get the 500 IP package](https://bit.ly/9-Proxy) |
| 1,000 + 500 IPs | 1,500 residential IPs, unlimited traffic | $126 | $0.084 | [Get the 1,500 IP package](https://bit.ly/9-Proxy) |
| 2,500 IPs | 2,500 residential IPs, unlimited traffic | $210 | $0.084 | [Get the 2,500 IP package](https://bit.ly/9-Proxy) |
| 5,000 IPs | 5,000 residential IPs, unlimited traffic | $360 | $0.072 | [Get the 5,000 IP package](https://bit.ly/9-Proxy) |
| 15,000 IPs | 15,000 residential IPs, unlimited traffic | $720 | $0.048 | [Get the 15,000 IP package](https://bit.ly/9-Proxy) |
| 25,000 IPs | 25,000 residential IPs, unlimited traffic | $863 | $0.035 | [Get the 25,000 IP package](https://bit.ly/9-Proxy) |
| 50,000 IPs | 50,000 residential IPs, unlimited traffic | $1,438 | $0.029 | [Get the 50,000 IP package](https://bit.ly/9-Proxy) |
| Business 100,000 IPs | High-volume package | $2,300 | $0.023 | [Get the 100,000 IP package](https://bit.ly/9-Proxy) |
| Business 200,000 IPs | High-volume package | $4,140 | $0.021 | [Get the 200,000 IP package](https://bit.ly/9-Proxy) |
| Business 500,000 IPs | High-volume package | $8,625 | $0.018 | [Get the 500,000 IP package](https://bit.ly/9-Proxy) |

**GB-based residential packages** — pay per GB, unlimited endpoints, 180-day validity (unlimited on Enterprise).

| Package | What you get | Price | Effective rate | Purchase |
| --- | --- | --- | --- | --- |
| 5 GB | 5 GB traffic, unlimited endpoints | $15 | $3.00/GB | [Get the 5 GB package](https://bit.ly/9-Proxy) |
| 50 GB + 5 GB | 55 GB total traffic | $105 | $2.10/GB | [Get the 55 GB package](https://bit.ly/9-Proxy) |
| 100 GB | 100 GB traffic | $150 | $1.50/GB | [Get the 100 GB package](https://bit.ly/9-Proxy) |
| 200 GB | 200 GB traffic | $200 | $1.00/GB | [Get the 200 GB package](https://bit.ly/9-Proxy) |
| 1,000 GB | 1,000 GB traffic | $800 | $0.80/GB | [Get the 1,000 GB package](https://bit.ly/9-Proxy) |
| 2,000 GB | 2,000 GB traffic | $1,500 | $0.75/GB | [Get the 2,000 GB package](https://bit.ly/9-Proxy) |

**Bundle packages** — IPs plus traffic in one purchase, 180-day traffic validity.

| Bundle | What you get | Price | Purchase |
| --- | --- | --- | --- |
| Starter | 100 IPs + 5 GB | $30 | [Get the Starter bundle](https://bit.ly/9-Proxy) |
| Popular | 1,500 IPs + 50 GB | $180 | [Get the Popular bundle](https://bit.ly/9-Proxy) |
| Pro | 5,000 IPs + 500 GB | $720 | [Get the Pro bundle](https://bit.ly/9-Proxy) |

**Enterprise** is quoted separately rather than sold self-serve. It includes unlimited data validity, team mode with one owner and up to five members, per-member traffic controls, shared bandwidth with no expiration inside the team, activity logs, and VIP pricing.

👉 [Compare bundles and Enterprise options on 9Proxy](https://bit.ly/9-Proxy)

## Which plan fits a Mac-based workflow

The choice comes down to one question: is your bottleneck IP count or bandwidth?

**One Mac, one browser, a handful of profiles.** The 100 IP package at $24 is the sensible starting point. Unused IPs don't expire until you activate them, so nothing rots while you figure out your workflow. If your work is more burst-oriented — short scraping runs, a few browser profiles, occasional geo-checks — 5 GB for $15 gets you the same SOCKS5 protocol support for less money.

**Sticky sessions where the identity has to persist.** IP-based is the right model. Each IP stays live somewhere between a few hours and roughly 24 hours, with unlimited traffic while it's active, and 9Proxy also supports auto-rotation on selected ports if you'd rather it switched on a timer.

**High-rotation automation.** GB-based. You generate unlimited endpoints and only the traffic is metered, which means rotating every request costs you nothing extra. Country, state, city, ZIP and ISP targeting are available for either mode, so the location-level selection isn't the differentiator — the billing model is.

**Mixed workloads.** That's what the bundles are for. The Starter bundle at $30 beats buying 100 IPs and 5 GB separately by exactly nothing on list price, so treat bundles as a convenience rather than a discount unless you're at the higher tiers, where the gap widens considerably (the Pro bundle combines 5,000 IPs and 500 GB).

Worth flagging: since the June 2026 adjustment raised IP and bundle prices while leaving bandwidth pricing untouched, the GB-based line is now the better value of the two if your workload can tolerate rotating IPs.

## Verify before you trust it

Configuration silently failing is normal on macOS. Two checks catch most of it.

**Check the IP.** Load any IP-check page before enabling the proxy and after. If the reported IP and location haven't changed, the traffic isn't going through SOCKS5 — usually a wrong port, or an app that ignores system proxy settings.

**Check for DNS leaks.** Whether DNS queries travel through SOCKS5 depends on the client. Run a DNS leak test and confirm you see resolvers associated with the proxy location, not your local ISP's. Firefox has a built-in "Proxy DNS when using SOCKS v5" option; command-line tools need `socks5h://`. Disabling IPv6 helps if the proxy endpoint only handles IPv4, since IPv6 traffic can bypass a SOCKS5 route entirely.

If you want to confirm the system-level state without opening panels, `scutil --proxy` shows exactly which proxy types macOS considers active.

## Troubleshooting macOS SOCKS5 problems

**Connection refused.** The endpoint is offline, the port is wrong, or the IP you forwarded has expired. IP-based residential IPs do not last forever. Verify the host and port first, then check whether the IP is still active in your dashboard.

**Authentication failures.** Credentials are case-sensitive, and structured usernames — where the username itself encodes targeting like country, session ID, or ISP — break if you truncate any segment. If you're using `networksetup`, remember it can't set SOCKS credentials at all; that's a limitation of the command, not of your provider.

**Some apps ignore the proxy.** Expected, not a bug. Use Proxifier for those, or configure the app directly.

**Slow or unstable.** Load and physical distance both matter. A residential IP in another country will be slower than one nearby, and residential IPs naturally churn as real users' connections drop. If you're testing against a specific target, pick an endpoint in the same region as the audience you're trying to look like.

**Works in one browser, not another.** Safari and Chrome follow the system SOCKS setting; Firefox has its own proxy configuration and will ignore it. Check Firefox's network settings separately.

## Things to know before you pay

A few items that reviewers consistently raise, and that are worth reading before checkout rather than after:

- **Refund terms are narrow.** Third-party reviews report that credit refunds essentially cover IPs that die within about 60 seconds, not a general change-of-mind window.

- **No guaranteed free trial.** 9Proxy has offered limited trials to new users depending on availability, typically granted through support rather than self-serve. Don't plan around it.

- **Streaming got more restrictive.** Reviews note a policy shift away from media streaming on IP-based plans. If streaming is the goal, confirm current terms first.

- **Performance figures are vendor-published.** 9Proxy advertises roughly 99.5% success rate, around 0.6s average response time and 99.95% uptime. Independent testers have reported latency in the 0.8–1.4s range on US IPs — in the ballpark, but treat the headline numbers as marketing rather than an SLA.

- **Payments are broad.** Cards, Apple Pay, Google Pay, Alipay, and crypto including USDT, BTC, ETH, LTC, and DOGE. Crypto payments reportedly carry a bonus on IP purchases.

👉 [See 9Proxy's current pricing and payment options](https://bit.ly/9-Proxy)

## FAQ

**Does macOS support SOCKS5 natively?**

Yes. The SOCKS proxy field in Network → Details → Proxies handles SOCKS5 including authentication. What it doesn't do is force uncooperative apps to use it.

**Can I use a SOCKS5 proxy in Safari and Chrome?**

Both follow the system proxy setting, so yes — but they'll route everything through it unless you use a bypass list. Firefox manages its own proxy settings independently, with native SOCKS v5 support.

**Do I need to install software to use 9Proxy on a Mac?**

For GB-based plans, no — you generate an endpoint in the dashboard and enter it in macOS or an app. IP-based plans involve per-IP port forwarding, which the desktop app handles, and 9Proxy also provides browser-based access for those proxies.

**How do I use 9Proxy with Python or Scrapy on a Mac?**

Point your HTTP client at the SOCKS5 endpoint using a SOCKS proxy agent, or test it first with `curl --socks5-hostname user:pass@host:port`. Both the IP-based and GB-based lines support SOCKS5, so either works for scripted traffic.

**How long does an IP-based proxy last?**

Anywhere from a few hours to about 24 hours, depending on the IP. Unused IPs in your balance don't expire until you activate them.

---

The macOS side is a five-minute job once you know which panel to open and which apps refuse to cooperate. The endpoint side is where the actual decision lives: pick a provider that treats SOCKS5 as a real protocol rather than a checkbox, and match the billing model to what your workflow burns through. If your work is sticky and identity-based, count IPs. If it rotates, count gigabytes.

👉 [Start with 9Proxy and test SOCKS5 on your Mac](https://bit.ly/9-Proxy)
