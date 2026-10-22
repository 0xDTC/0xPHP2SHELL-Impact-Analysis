# PHP2SHELL IMPACT ANALYSIS

CVE-2026-109618 and CVE-2026-109619 | Remote code execution in the PHP interpreter | CVSS 4.0: 9.2 (PHP) | CVSS 3.1: 9.8 (NVD)

Data cut-off: 21 October 2026, 22:30 UTC. Cloudflare rule state re-checked on 22 October. Things are still moving, so check CISA KEV, EPSS and your vendor's advisory again before you act on anything here.

---

## EXECUTIVE SUMMARY

### The situation at a glance

- **What:** Two critical bugs in the PHP interpreter, nicknamed PHP2Shell. CVE-2026-109618 is "RCE via OOM corruption". CVE-2026-109619 is "RCE via stale global state".
- **Risk level:** CRITICAL. PHP scores both 9.2 (CVSS 4.0). NVD scores both 9.8 (CVSS 3.1).
- **Impact:** Remote code execution on the web server with no login. On Apache with mod_php the attack lands before your application code even starts. CVE-2026-109619 can also leak server memory.
- **Status:** Fixed on 21 October 2026. Scanning and exploit attempts started hours before the fix went public. Nobody had publicly confirmed a successful break-in at the time of writing, and neither CVE is in CISA's Known Exploited Vulnerabilities catalog yet.
- **Who's safe:** PHP 8.2.35, 8.3.36, 8.4.27, 8.5.12 or later, or a vendor package that backports the fix, with every PHP process restarted afterwards.
- **Who's at risk:** Anything running PHP that answers web requests. WordPress, Drupal, Joomla, Magento, Laravel, Symfony, Nextcloud, Moodle, custom apps, hosting panels, and appliances with a PHP admin page. PHP 8.1 and older are worst off, because there is no official release for them.

*Sources: PHP advisory 2026-10-21-5, GHSA-w8hw-w9fx-j79w, GHSA-6wmf-4rv5-p3rp, NVD, CISA alert of 21 October 2026, VulnCheck and watchTowr telemetry*

### What this means for your organization

The bugs are in PHP itself. Updating WordPress, a plugin or your framework does nothing for this. Palo Alto's Unit 42 counts 22.6 million PHP instances exposed to the internet, and VulnCheck found that 43% of PHP hosts don't even show which version they run.

If an attacker gets this working against one of your servers, they can:

- Run commands as the web server user
- Read database passwords, API keys and session data from config files and process memory
- Drop a web shell and come back whenever they like
- Move from the web server to the databases and internal systems it can reach

The memory leak in CVE-2026-109619 is the quiet part. It can expose secrets without writing a single file, so a clean file system doesn't prove nothing was taken.

**Real-world context:** The CVE IDs leaked on Hacker News on 20 October, about two days before the planned release, and an exploit demo video went out by mistake that same evening. PHP pulled the release forward by a day. VulnCheck's canaries logged the first scanner at 08:37 UTC on 21 October, five hours before the patch was public.

*Sources: PHP Foundation blog (21 October 2026), Unit 42, VulnCheck, Cloudflare*

### Immediate actions required

**HOUR 1:** Find every PHP install

1. List the servers, containers, hosting accounts and appliances that run PHP.
2. Get the version with `php -v`, then check the package changelog. Backported fixes often keep the old version number.
3. Internet-facing PHP 8.1 or older is an emergency. CISA says take it offline until it is patched or upgraded.

**DAY 1:** Patch and restart

1. Update to 8.2.35, 8.3.36, 8.4.27 or 8.5.12, or install your vendor's fixed package.
2. Restart every PHP process. For Apache with mod_php that means a full restart. A reload or graceful restart keeps the old code loaded.
3. Confirm the fixed build is the one actually serving traffic.

**DAY 2:** Look for compromise

1. Search logs for out-of-memory errors that happen before any script runs, and for PHP worker crashes.
2. Look for shells or download tools started by PHP workers, and for new `.php` files in web roots.
3. If anything looks off, rotate every secret the web server can reach.

### Business impact assessment

**Financial risk:**

- Average cost of a data breach: USD 4.44 million worldwide and USD 10.22 million in the US (IBM Cost of a Data Breach Report 2025)
- Patching: usually an hour or two per server. Moving an app off PHP 8.1 or older can take weeks.
- GDPR fines: up to EUR 20 million or 4% of global annual turnover, whichever is higher

**Operational risk:**

- The fix adds a hard memory ceiling. Apps that already run close to `memory_limit` may see workers killed (exit code 137) after the update, so test before you roll out everywhere.
- The usual stopgaps break things. Blocking multipart requests kills file uploads and some checkouts, and turning off upload progress removes upload progress bars.
- Taking a site offline is the official fallback for old PHP. Germany's BSI noted site outages from operators who did exactly that.

**Legal risk:**

- GDPR: 72 hours to tell the regulator after you become aware of a personal data breach
- NIS2 (EU essential and important entities): early warning within 24 hours
- India: CERT-In directions require reporting covered incidents within 6 hours
- US: SEC Form 8-K within four business days of deciding an incident is material, plus state breach laws

---

## RISK DASHBOARD

### Severity metrics

| Metric | Rating | What it means |
| --- | --- | --- |
| **Overall risk** | CRITICAL (9.2) | PHP's CVSS 4.0 score. NVD says 9.8 (CVSS 3.1). Red Hat and SUSE say 8.1 |
| **Attack complexity** | HIGH on paper | CVSS rates it AC:H with attack requirements present. Researchers still built working chains for WordPress, Drupal, Joomla and Apache mod_php |
| **Authentication needed** | NONE | No account, no login |
| **User interaction** | NONE | Nobody has to click anything |
| **Runs before your app** | YES on mod_php | Login pages, CSRF tokens, security plugins and maintenance mode don't stop it |
| **Patch available** | YES | 21 October 2026 for 8.2 to 8.5. Source-only patches for 7.4, 8.0 and 8.1 |
| **Active exploitation** | ATTEMPTS SEEN | Scanning and one failed web shell drop on canaries. No confirmed compromise. Not in CISA KEV |
| **EPSS** | NOT SCORED YET | FIRST had no score for either CVE at the time of writing |

You'll see "CVSS 10.0" quoted in places. That number came from the leak, and the PHP Foundation says it was wrong.

My read: plan around the 9.8, not the 8.1. The lower scores assume the attacker has to get heap layout and timing exactly right. Calif says PHP confirmed its mod_php exploit worked back in August, and Wiz says it demonstrated unauthenticated code execution on WordPress, Drupal and Joomla before the release. Hard for an attacker is not the same as rare.

*Sources: CVE records (PHP CNA), NVD, Red Hat and SUSE CVE pages, CISA SSVC (Exploitation: none, Automatable: no, Technical impact: total), FIRST EPSS API, Calif and Wiz write-ups*

### Affected versions: quick lookup

| PHP branch | Vulnerable | Fixed | Status |
| --- | --- | --- | --- |
| **8.5** | 8.5.0 to 8.5.11 | 8.5.12 | Patch available |
| **8.4** | 8.4.0 to 8.4.26 | 8.4.27 | Patch available |
| **8.3** | 8.3.0 to 8.3.35 | 8.3.36 | Patch available |
| **8.2** | 8.2.0 to 8.2.34 | 8.2.35 | Patch available |
| **8.6 (pre-release)** | up to 8.6.0RC3 | 8.6.0RC4 | Fixed in RC4 |
| **8.1, 8.0, 7.4 (EOL)** | all | No php.net release, only source-level mitigation patches | Vendor backport or upgrade |
| **7.3 and older, 5.x (EOL)** | Reported affected, vendors disagree | Nothing from PHP. Some vendors ship fixes | Upgrade |

There's a trap here for scanners. The official CVE records list only 8.2 to 8.5 and mark everything else "unaffected". PHP's own advisory text, NCSC-NL, SUSE, Zend and Google all say older branches are affected too. Red Hat says the vulnerable code isn't present in the PHP 5.3 and 5.4 builds on RHEL 6 and 7. A scanner that only reads the CVE ranges will call your PHP 7.4 box clean. It isn't.

*Sources: CVE.org records, php.net advisory, NCSC-NL NCSC-2026-0443, Red Hat VEX*

**Quick status check:**

```bash
php -v
php -r 'var_dump(ini_get("memory_hard_limit_percent"));'
```

- `8.5.12`, `8.4.27`, `8.3.36`, `8.2.35` or newer: fixed upstream build.
- An older number is not automatically vulnerable. Debian, Ubuntu, Red Hat, cPanel, Plesk, CloudLinux and Chainguard ship fixed packages that keep the old number. Check the package changelog (commands below).
- The second line prints `bool(false)` on an unpatched 8.2 to 8.5 build. The fix adds the `memory_hard_limit_percent` setting, so a patched build prints a string instead (`string(0) ""` on the builds Mattias Geniar tested). EOL backports may not include the setting, so don't use this test on 7.4, 8.0 or 8.1.

The `php` on your path is the command-line binary. Your web server may run a different binary or package (php-fpm, mod_php, lsphp), so check that one too.

*Sources: php.net advisory, Ubuntu USN-8948-1, Mattias Geniar's tests of patched builds*

### Where the fix comes from

| Platform | Fixed package or version | Notes |
| --- | --- | --- |
| php.net source and Windows builds | 8.2.35, 8.3.36, 8.4.27, 8.5.12 | Windows builds went up between 13:47 and 13:49 UTC on 21 Oct |
| Debian 13 / 12 | php8.4 8.4.27-1~deb13u1 (DSA-6574-1) / php8.2 8.2.35-1~deb12u1 (DLA-4848-1) | Debian 9 to 11 through Freexian ELTS (ELA-1853, ELA-1854, ELA-1855) |
| Ubuntu 22.04 / 24.04 | php8.1 8.1.2-1ubuntu2.28 / php8.3 8.3.6-0ubuntu0.24.04.13 (USN-8948-1) | Old version strings. 14.04 to 20.04 through ESM (USN-8948-2) |
| RHEL 10 / 9 / 8 | RHSA-2026:87070 and :87071 / :87072, :87079 and :87151 / :87080 | RHEL 8 php:8.2 and RHEL 9 php:8.4 streams still open at cut-off. Red Hat says RHEL 6 and 7 are not affected |
| AlmaLinux 8 / 9 / 10 | Fixed builds in production from 21:15 UTC on 21 Oct | |
| Rocky Linux, Oracle Linux | No advisory at cut-off | |
| SUSE / openSUSE | SLE 15 SP7 php8 released | php7 and the 16.x line still affected at cut-off |
| Fedora | F44 and F46 stable | F43 and F45 pending at cut-off |
| Arch / Gentoo | php 8.5.12 and php-legacy 8.3.36 (ASA-202610-1, ASA-202610-2) / GLSA 202610-02 | |
| Alpine | edge and 3.24 fixed | 3.22 and 3.23 had no fix at cut-off |
| Amazon Linux 2023 / 2 | Pending, AWS targeted 22 Oct | No fix planned for AL2023 php8.1 or AL2 core php |
| Docker official `php` images | 8.2.35 to 8.5.12, rebuilt from 15:50 UTC on 21 Oct | `php:8.1` and older were not rebuilt |
| Chainguard / Wolfi | Revision -r4 backports (8.2.34-r4, 8.3.35-r4, 8.4.26-r4, 8.5.11-r4) | The version number didn't change. The revision is what matters |
| Sury (deb.sury.org, ppa:ondrej/php) | 5.6 to 8.5 rebuilt on amd64 | arm64 lagged. If you mix Sury and Debian security, check `apt-cache policy` |
| Remi (EL) | php82 to php85 fixed, php74, php80 and php81 rebuilt | |
| Homebrew | php 8.5.12, php@8.4, php@8.3 and php@8.2 fixed | php@8.1 not fixed |
| cPanel EasyApache 4 | ea-php 7.4.33-19, 8.0.30-12, 8.1.34-5, plus 8.2.35 to 8.5.12 | `rpm -q --changelog ea-php81-php-cli \| grep -m1 CVE-2026-109618` |
| Plesk | PHP 8.2 to 8.5 updated. 7.4, 8.0 and 8.1 rebuilt on 21 Oct with old version numbers | Fixed EOL builds show `built: Oct 21 2026` in `php -v`. Plesk's own sw-engine hotfix was pending at cut-off |
| CloudLinux / TuxCare | ea-php fixed but keeps numbers like 8.5.11-2. alt-php fixes landed in the TuxCare ELS repo | `php -v` can't tell you. Check the package changelog |
| DirectAdmin CustomBuild | 8.2 to 8.5 bumped, patches for 5.6 to 8.1 | |
| Heroku | 8.2.35 to 8.5.12, plus 8.1.34 rebuilt on Heroku-22 | Redeploy required |
| Azure App Service (Linux) | Built-in 8.2.35 to 8.5.12 | Custom containers are yours to rebuild |
| Google Cloud (GCP-2026-070) | App Engine Standard 8.2+ patched automatically | App Engine Flexible and Cloud Run need a rebuild and redeploy |
| AWS Elastic Beanstalk | AL2023 PHP 8.5 and 8.4 platforms v4.13.13 | 8.3 and 8.2 branches pending at cut-off |
| FrankenPHP | v1.13.2 | Also clears last errors between worker requests |
| WordPress.com, VIP, Pressable, WP Cloud | Patched by Automattic, no customer action | Self-hosted WordPress still needs its PHP updated |
| Appliances | Palo Alto PAN-OS and Panorama affected with no fix yet. Cisco ISE, Catalyst Center and SD-WAN Manager under investigation. OPNsense fixed. Sophos products not affected | No statement found from F5, Citrix, Juniper (J-Web), pfSense, Synology or QNAP at cut-off |

*Sources: vendor advisories listed under Additional Resources, from a 97-row vendor matrix compiled for this analysis (snapshot 21 October 2026, 22:16 UTC)*

### Frameworks, CMSs and apps

None of these projects has a bug of its own here. Each one is exposed because it runs on PHP, and on Apache with mod_php the attack lands before the framework's code even starts. Framework updates, security modules and firewall plugins can't fix it. Wordfence itself tells people to update PHP on the server. The fix is the same everywhere: patch PHP, then restart it.

The table shows what each project had said by the cut-off, and whether its official Docker image ships a fixed PHP.

| Framework or app | What the project said | Official Docker image | Notes |
| --- | --- | --- | --- |
| **WordPress** | WordPress.org: no defect in WordPress, no WordPress update planned. Update PHP | `wordpress` on PHP 8.3.36 | Wiz reports an unauthenticated RCE demo. Automattic patched WordPress.com, VIP, Pressable and WP Cloud. WP Engine and Kinsta were mid-rollout (Kinsta had only 8.5 patched at 18:56 UTC). Wordfence and Patchstack: update PHP on the server |
| **Drupal** | PSA-2026-10-20: no defect in Drupal core. Update PHP | `drupal` on PHP 8.5.12 | Wiz reports an unauthenticated RCE demo that used a path only PHP 8.5 has, so on 8.5 the `fatal_error_backtraces = 0` stopgap matters most. Pantheon patched 8.2 to 8.5 on live sites, older branches still rolling out. Upsun images updated for 8.2 to 8.5. Acquia: no statement at cut-off |
| **Joomla** | No statement found | `joomla` on PHP 8.4.27 | Wiz reports an RCE demo against Joomla too |
| **Magento / Adobe Commerce** | No statement or Adobe bulletin found | No official image | Some checkouts post multipart forms, so test before you block multipart as a stopgap |
| **Laravel** | Laravel blog: Laravel Cloud apps patched in place. Forge has an "Update PHP" action for 8.2 to 8.5 | No official image | Forge servers on Ubuntu 20.04 or older can't use it, and Laravel has no patch for 8.0 or 8.1 on Forge or Vapor. Restart queue workers, Horizon and Octane after patching |
| **Symfony, CodeIgniter, CakePHP, Yii, Laminas, Slim** | Framework code isn't involved | No official image | Exposed exactly as much as the PHP underneath. Restart Messenger and other long-running workers after patching |
| **Nextcloud** | Admin wiki: run a fixed PHP | `nextcloud` on PHP 8.5.12, All-in-One v14.4.1 | Snap package updated too |
| **MediaWiki** | Wikimedia paused deployments | `mediawiki` 1.46.2 on PHP 8.3.36 | |
| **Moodle** | Moodle news post. Moodle HQ's own hosting patched | `moodlehq/moodle-php-apache` still on 8.3.35 at check | Rebuild your own images |
| **Matomo** | Blog post with guidance | `matomo` on PHP 8.4.27 | |
| **PrestaShop** | No statement found | Images still on 8.2.34 to 8.5.11 at check | |
| **phpMyAdmin** | No statement found | Official `phpmyadmin` on PHP 8.3.36. The project's own `phpmyadmin/phpmyadmin` image still on 8.3.26 | Keep it off the internet |
| **TYPO3, Shopware, WHMCS, Roundcube** | No statement found | Roundcube `latest` on 8.4.27, other variants lagging | |
| **FrankenPHP, LiteSpeed, Swoole, RoadRunner, Workerman** | FrankenPHP v1.13.2 fixed. LiteSpeed lsphp packages fixed. Swoole images being rebuilt. No statement from RoadRunner or Workerman | | FrankenPHP and LiteSpeed ship their own PHP, so update that package, not just the system PHP |

*Sources: WordPress.org, Drupal PSA-2026-10-20, Laravel blog, Wikimedia, Matomo, Moodle, Automattic, Wiz, hosting status pages and Docker registry data, captured on 21 October 2026*

---

## ATTACK TIMELINE

### How fast this moved

```
2024                  During the OSTIF/Quarkslab audit, a PHP core developer spots the first signs of
                      the out-of-memory problem. No working exploit yet, and the fix stalls for months
30 Jul 2026           Calif starts auditing PHP
11 Aug 2026           Calif and OpenAI independently report the same root cause to PHP
19-24 Aug 2026        Wiz finds and reports four bugs (WordPress and Drupal RCE demos)
25 Sep 2026           Private fix pull request opened
12 Oct 2026           CVE IDs reserved
18 Oct 2026           Final fix committed for 8.2 to 8.5, minimal patches for 7.4, 8.0 and 8.1
20 Oct, 15:20 UTC     LEAK: both CVE IDs posted on Hacker News with a wrong score (10.0) and wrong date
20 Oct, 19:30 UTC     PHP pre-announces and moves the release up to 21 Oct, 14:00 UTC
20 Oct, ~20:00 UTC    A researcher's exploit demo video is published by mistake, deleted, then copied
20 Oct, 23:20 UTC     First Cloudflare emergency WAF rules go live (Free and paid plans)
21 Oct, 07:41 UTC     Chainguard ships patched images early (PHP Foundation's timestamp)
21 Oct, 08:37 UTC     VulnCheck canaries log the first scanner (user agent php2shell-scan/109618)
21 Oct, 13:33 UTC     PUBLIC RELEASE: 8.2.35, 8.3.36, 8.4.27, 8.5.12 and both advisories
21 Oct, 14:04 UTC     CISA KEV update. Neither CVE listed
21 Oct, 14:17 UTC     NVD publishes both CVEs at CVSS 3.1 9.8
21 Oct, 14:19 UTC     watchTowr reports exploitation attempts against its honeypots
21 Oct, 15:50 UTC     Official Docker php images rebuilt
21 Oct, 17:22 UTC     Failed web shell drop against a VulnCheck canary (every request got a 404)
21 Oct, daytime       CISA, CERT-EU, NCSC-UK, NCSC-NL, BSI, CERT-FR, CCCS and about 30 national
                      CSIRTs publish alerts
21 Oct, 21:30 UTC     AWS bulletin. Amazon Linux packages targeted for 22 Oct
22 Oct                This analysis
```

**Critical insight:** scanning started five hours before anyone could patch. The leak took away the head start defenders normally get, so "we'll patch it next sprint" isn't a plan here.

*Sources: PHP Foundation blog, Hacker News API, Cloudflare API and blog, VulnCheck, watchTowr, CISA KEV feed, NVD, docker-library/php*

### Who's attacking

Nobody has attributed any of this activity yet. Here's what the teams with sensors reported on 21 October:

| Source | What they saw | Confidence |
| --- | --- | --- |
| VulnCheck canaries | 5 source IPs. Two scanners hit 188 and 26 canary hosts. One China Unicom IP tried to drop a web shell and failed | Confirmed attempts, raw indicators published |
| watchTowr honeypots | "Active exploitation attempts of both" CVEs, and "nobody has been successful to date" | Credible, attempts only |
| Cloudflare | Early attacks, and attackers changing payloads to slip past the new rules | Credible, no numbers given |
| Patchstack | First "multiple active exploitation incidents", later corrected to "only probing attacks, with no hard evidence of breaches yet" | Self-corrected |
| Cisco Talos | Blog: not seen in the wild as of 14:00 UTC. Snort rule docs: "Attacks/Scans seen in the wild" | Contradicts itself |
| Huntress, Imperva | No successful exploitation seen | Negative |
| CISA, NCSC-NL, PHP Foundation | Not observed, and no working public exploit at release time | Negative at publication |

**Known scanning infrastructure** (VulnCheck canaries unless noted):

| IP | Network | Seen (UTC, 21 Oct) | Activity |
| --- | --- | --- | --- |
| 85.11.167.243 | AS209630 TechTies (NL) | 08:37 to 10:46 | User agent `php2shell-scan/109618`, multipart POSTs to `/index.php`, 188 canary hosts |
| 45.198.224.169 | AS215925 VPSVAULT.HOST | 14:15 to 15:52 | Same user agent and probe, 26 hosts |
| 221.14.159.134 | AS4837 China Unicom (Henan) | 17:22 to 17:24 | User agent `Mozilla/5.0 (php2shell-poc)`, attempted web shell drop, all 404 |
| 91.90.123.25 | AS9009 M247 (Brussels) | 17:23 | User agent `php2shell-verify/1.0` |
| 64.95.12.7 | BL Networks (US) | about 15:01 | Exploit attempts with a public PoC, reported by Vega Threat Research. GreyNoise hadn't seen it scanning |
| 109.71.252.97 | Datalix (DE) | 15:55 | User agent `php2shell-scan/1.3 (eigen-oppervlak controle; ...)`. Reads like a defender scanning its own estate, so don't block it blindly |

Hosting IPs rotate quickly. Use these as hunt leads in your own logs. A blocklist built from them won't keep anyone safe.

*Sources: VulnCheck, watchTowr, Cloudflare, Patchstack, Cisco Talos, Huntress, Imperva, CISA, NCSC-NL, PHP Foundation*

### Global impact statistics

- **22.6 million** PHP instances exposed to the internet (Unit 42, Cortex Xpanse)
- **43%** of PHP hosts hide their version number, so banner-based scanners can't judge them (VulnCheck)
- **14** emergency Cloudflare WAF rules, the first ones live about 14 hours before the patch
- **About 30** national CSIRTs linked from ENISA's CSIRTs Network note, on top of CISA, NCSC-UK, CCCS and others
- **260+** GitHub repositories claiming PoCs or tools in about 30 hours. Of the 85 read for this analysis (statically, nothing was run), 33 claim full RCE and only 13 of those implement the real bug. The other 20 recycle old techniques, target a different bug, or copy unverified code with hard-coded offsets.

---

## AM I AFFECTED? SELF-ASSESSMENT

### Quick 3-minute assessment

**STEP 1: Do you run PHP anywhere? (30 seconds)**

```bash
# PHP binaries and services on this host
command -v php php-fpm php-cgi 2>/dev/null
ls -d /usr/sbin/php-fpm* /opt/cpanel/ea-php*/root/usr/bin/php /opt/plesk/php/*/bin/php /usr/local/lsws/lsphp*/bin/php 2>/dev/null
systemctl list-units --type=service --all | grep -Ei 'php|apache2|httpd|lsws|frankenphp'
```

Also check the places people forget: containers, shared hosting accounts, hosting control panels, webmail and phpMyAdmin installs, and appliances with a PHP web interface (firewalls, NAS boxes, load balancers).

---

**STEP 2: Which version is it, and does it carry the fix? (1 minute)**

```bash
php -v

# Debian / Ubuntu / Sury: look for the CVE in the package changelog
zgrep -lE 'CVE-2026-10961[89]' /usr/share/doc/php*/changelog.Debian.gz

# RHEL, Alma, Rocky, Fedora, SUSE, Remi
rpm -q --changelog php-cli php-fpm 2>/dev/null | grep -E 'CVE-2026-10961[89]'

# cPanel EasyApache 4 (repeat for each ea-php version)
rpm -q --changelog ea-php81-php-cli | grep -m1 CVE-2026-109618

# Plesk: fixed EOL builds keep the old version number
/opt/plesk/php/8.1/bin/php -v      # look for "built: Oct 21 2026"
```

**Interpretation:**

- The changelog names CVE-2026-109618 and CVE-2026-109619: fixed package. Go to Step 4.
- Upstream version 8.2.35, 8.3.36, 8.4.27, 8.5.12 or newer: fixed. Go to Step 4. (Debian packages that moved to the new upstream version may only say "New upstream version". The version number is enough there.)
- Neither: treat it as vulnerable. Go to Step 3.

---

**STEP 3: How exposed is it? (1 minute)**

```bash
# Apache with mod_php (highest risk: researchers confirmed RCE before the application runs)
apachectl -M 2>/dev/null | grep -i php

# PHP-FPM pools
ls /etc/php/*/fpm/pool.d/ /etc/php-fpm.d/ 2>/dev/null

# Settings that matter. Ask the FPM binary, because the CLI config can differ
# (use your version: php-fpm8.3, php-fpm8.4, or plain php-fpm on RHEL)
php-fpm8.4 -i 2>/dev/null | grep -E 'session.upload_progress.enabled|fatal_error_backtraces|memory_limit'
```

| What you found | Priority |
| --- | --- |
| mod_php, internet facing, no fix | CRITICAL. Patch now or take it offline |
| PHP-FPM or LiteSpeed, internet facing, no fix | CRITICAL. Patch today |
| Internal only, no fix | HIGH. Patch this week |
| Command line only (cron, build tools), never serves HTTP | LOWER. Still patch |

---

**STEP 4: Is the fixed code actually running? (30 seconds)**

```bash
# When was the binary or Apache module replaced?
ls -l --time-style=full-iso /usr/sbin/php-fpm* /usr/lib/apache2/modules/libphp*.so /usr/lib64/httpd/modules/libphp*.so 2>/dev/null

# When did the running master processes start?
ps -eo pid,lstart,args | grep -E '[p]hp-fpm: master|[a]pache2|[h]ttpd|[l]sphp'
```

If a master process started before the package was updated, it is still running the vulnerable code. Restart it.

---

### Full vulnerability checklist

**You are VULNERABLE if ALL of these are true:**

- [ ] PHP is older than 8.2.35, 8.3.36, 8.4.27 or 8.5.12, and the package changelog doesn't name both CVEs
- [ ] PHP answers web requests (mod_php, PHP-FPM, CGI, LiteSpeed lsphp, FrankenPHP)
- [ ] Untrusted users can reach it (the internet, a partner network, neighbours on shared hosting)

**Your risk is higher if ANY of these also apply:**

- [ ] Apache with mod_php
- [ ] `session.upload_progress.enabled` is on (it is by default)
- [ ] PHP 8.5 with `fatal_error_backtraces` on (the 8.5 default)
- [ ] PHP 8.1 or older
- [ ] Per-directory INI overrides (`php_value` in `.htaccess`, `fastcgi_param PHP_VALUE` in nginx)
- [ ] The origin server answers direct requests, so your WAF or CDN can be skipped

---

### You are SAFE if:

- [ ] PHP is 8.2.35, 8.3.36, 8.4.27, 8.5.12, 8.6.0RC4 or newer, or a vendor package whose changelog names both CVEs
- [ ] Every PHP process was restarted after the update (Apache fully restarted, not reloaded)
- [ ] Containers were rebuilt from patched base images and redeployed, not just pulled
- [ ] Your managed host has confirmed in writing that your plan's PHP is patched
- [ ] You don't run PHP at all, and you checked panels, webmail and appliances before saying so

**IMPORTANT:** Updating WordPress, Drupal, Laravel or any plugin does not fix this. WordPress.org and Drupal both said no core update is coming because the bug isn't in their code. Cloudflare put it plainly: updating website code or plugins alone does not fix these vulnerabilities.

---

## PROTECTION ROADMAP

### For non-technical executives

#### HOUR 1: Emergency assessment

**Immediate actions checklist:**

- [ ] Get IT, the web team and your hosting provider on one call
- [ ] Ask the question that matters: "Where do we run PHP? Websites, shops, intranets, hosting panels, appliances?"
- [ ] Get version numbers for each one (IT runs the checks above)
- [ ] Ask every hosting provider and MSP, in writing, whether they've patched
- [ ] Flag any internet-facing PHP 8.1 or older
- [ ] Approve emergency maintenance, including taking sites offline if it comes to that
- [ ] Tell legal, compliance and your cyber insurer that you're responding

**Decision points:**

- Everything on fixed versions and restarted: document it and move to the Week 1 checks.
- Unpatched PHP 8.2 or newer: approve patching today.
- Internet-facing PHP 8.1 or older: vendor backport, upgrade, or take it offline. CISA says offline until it's fixed.

---

#### DAY 1: Emergency response

**Executive decision checklist:**

- [ ] Approve maintenance windows. Patching, restarting and checking usually takes under an hour per server.
- [ ] Decide what to do about old PHP:
    - [ ] **Option A:** Take it offline until it's upgraded (safest, and what CISA recommends)
    - [ ] **Option B:** Buy or switch on a vendor backport (TuxCare, Zend, Ubuntu Pro, Debian ELTS, cPanel, Plesk)
    - [ ] **Option C:** Stopgap settings plus a WAF while you upgrade (leaves the most risk behind)
- [ ] Communication plan:
    - [ ] Brief the board if you're publicly traded
    - [ ] Prepare a customer notice in case a site has to go offline
    - [ ] Tell your cyber insurer
    - [ ] Ask legal for a compliance read
- [ ] Resources:
    - [ ] Approve overtime or outside help if the team is stretched
    - [ ] Call in your incident response retainer if you have one

---

#### WEEK 1: Verification and assessment

**Follow-up checklist:**

- [ ] Confirm, with evidence, that every PHP instance is patched and restarted
- [ ] Review logs back to at least 20 October (the leak date) for signs of compromise
- [ ] Re-check vendor and appliance advisories. Several were still "investigating" on 21 October
- [ ] Rotate secrets on any server that showed suspicious activity
- [ ] Hold a post-incident review with the technical teams
- [ ] Compliance:
    - [ ] Check notification duties if you found any sign of compromise
    - [ ] Keep timestamped records of everything you did
    - [ ] Preserve evidence
    - [ ] Get legal advice before you conclude "no breach"

---

#### MONTH 1: Strategic planning

**Longer-term improvements:**

- [ ] Retire PHP 8.1 and older for good. This incident is what EOL software really costs
- [ ] Keep a live inventory of every PHP runtime, including the ones inside containers and appliances
- [ ] Subscribe to php.net announcements and your distro's security feed
- [ ] Lock your origin servers so traffic can't bypass your WAF or CDN
- [ ] Add detection for PHP workers that spawn shells
- [ ] Review hosting contracts for patch timelines and notification duties
- [ ] Run a tabletop exercise based on this incident

---

### For technical teams

#### IMMEDIATE ACTIONS (Hour 1-2)

**Phase 1: Assessment and inventory**

**Checklist:**

- [ ] Find every PHP runtime (hosts, containers, panels, appliances)
- [ ] Record version, package and changelog status for each
- [ ] Record the SAPI (mod_php, FPM, LiteSpeed, FrankenPHP, CGI)
- [ ] Record internet exposure, and whether the origin is locked behind the WAF or CDN
- [ ] Prioritize mod_php and internet-facing servers first

**Commands for assessment:**

```bash
# Packages: Debian / Ubuntu
dpkg -l 'php*' | awk '/^ii/{print $2, $3}'

# Packages: RHEL family, SUSE, cPanel, CloudLinux, LiteSpeed
rpm -qa | grep -Ei '^(php|ea-php|alt-php|lsphp)' | sort

# Packages: Alpine
apk info -vv 2>/dev/null | grep -E '^php'

# Containers on this host
docker ps --format '{{.Names}}' | while read -r c; do
  printf '%s: ' "$c"
  docker exec "$c" php -r 'echo PHP_VERSION, PHP_EOL;' 2>/dev/null || echo "no php"
done

# Which web SAPI is loaded
apachectl -M 2>/dev/null | grep -i php
systemctl list-units --type=service | grep -Ei 'php.*fpm|lsws|frankenphp'
```

For bigger estates, Velociraptor has a ready-made artifact (`Generic.Applications.PHPInventory`) that checks packages, binaries and running processes. The NCSC-NL php2shell repository keeps an inventory of third-party products that bundle PHP.

**Document your findings:**

```
Server / app:
PHP version (php -v):
Package changelog shows the fix: [yes / no]
SAPI: [mod_php / FPM / LiteSpeed / FrankenPHP / CGI]
Internet facing: [yes / no]
Behind WAF/CDN with origin locked down: [yes / no]
session.upload_progress.enabled: [on / off]
Processes restarted after update: [yes / no, time]
Status: [fixed / vulnerable / EOL]
```

---

**Phase 2: Emergency patching**

**Pre-patch checklist:**

- [ ] Snapshot or back up the server or container image
- [ ] Note current package versions (`dpkg -l` or `rpm -qa`) for rollback
- [ ] Write down any stopgap settings you applied (like `memory_limit=-1`) so you can undo them later
- [ ] Tell stakeholders about the maintenance window
- [ ] Have a rollback plan ready

**Patching commands:**

**Option A: Distro or vendor packages (recommended)**

```bash
# Debian / Ubuntu (also deb.sury.org and ppa:ondrej/php)
sudo apt update
sudo apt install --only-upgrade $(dpkg -l 'php*' | awk '/^ii/{print $2}')
apt-cache policy php8.4-fpm     # mixing Sury and Debian security? check which repo wins

# RHEL, AlmaLinux, Rocky, Fedora (module streams and Remi included)
sudo dnf upgrade 'php*'

# SUSE / openSUSE
sudo zypper patch

# Alpine (edge and 3.24 had fixes at cut-off, 3.22 and 3.23 did not)
apk update && apk upgrade

# cPanel / WHM (EasyApache 4)
/scripts/rpmup

# macOS dev machines (Homebrew)
brew update && brew upgrade $(brew list --formula | grep -E '^php(@8\.[2-5])?$')
```

**Option B: Containers**

```bash
# Official php images for 8.2 to 8.5 were rebuilt on 21 Oct. php:8.1 and older were not.
docker build --pull --no-cache -t yourapp:php-fixed .
docker compose up -d --force-recreate
docker compose exec app php -v
```

**Option C: Source and Windows builds**

- Source tarballs: https://www.php.net/downloads
- Windows builds: https://windows.php.net/download/

**EOL branches (8.1, 8.0, 7.4 and older):** php.net ships no release. In order of preference:

1. Upgrade to 8.4 or 8.5.
2. Use a vendor backport (Ubuntu ESM, Debian LTS and ELTS, Red Hat, TuxCare, Zend, HeroDevs, cPanel, Plesk, DirectAdmin, Sury, Remi).
3. Build from PHP's minimal mitigation branches (`PHP-7.4-CVE-2026-109618-109619`, `PHP-8.0-CVE-2026-109618-109619`, `PHP-8.1-CVE-2026-109618-109619`). These are not the full fix. They end the worker on any out-of-memory error and switch session upload progress off whatever php.ini says.

**Restart every PHP process (the step people skip):**

```bash
# PHP-FPM
sudo systemctl restart php8.4-fpm          # RHEL family: php-fpm

# Apache + mod_php: full restart only. "reload", "graceful" and "apachectl -k restart"
# keep the old libphp loaded
sudo systemctl restart apache2             # RHEL family: httpd

# LiteSpeed / OpenLiteSpeed
sudo systemctl restart lsws && sudo pkill lsphp

# Supervisor-managed workers (Laravel queues and Octane, Symfony Messenger)
sudo supervisorctl restart all

# Or have the framework recycle its own workers (Supervisor starts fresh ones)
php artisan queue:restart                  # Laravel queues
php artisan horizon:terminate              # Laravel Horizon
php bin/console messenger:stop-workers     # Symfony Messenger

# FrankenPHP: move to v1.13.2 or later
frankenphp version
```

**Verification:**

```bash
php -v
php -r 'var_dump(ini_get("memory_hard_limit_percent"));'    # a string, not bool(false), on fixed 8.2 to 8.5
php-fpm8.4 -i | grep -E 'session.upload_progress.enabled|fatal_error_backtraces'
ps -eo pid,lstart,args | grep -E '[p]hp-fpm: master|[a]pache2|[h]ttpd|[l]sphp'
```

**Post-patch checklist:**

- [ ] The site works: logins, forms, uploads, checkout
- [ ] Watch the FPM log for `exited with code 137` (the new hard memory limit doing its job)
- [ ] Set `catch_workers_output = yes` in FPM pools so the "Hard memory limit" message reaches the log (Debian ships it commented out)
- [ ] If you set `memory_limit=-1` as a stopgap, put a real limit back
- [ ] Leave `session.upload_progress.enabled=0` if nobody uses upload progress bars. PHP recommends this even after patching
- [ ] Update your asset inventory and record the patch time

---

#### DETECTION & FORENSICS (Day 2-3)

**Check for indicators of compromise (IOCs)**

**Log indicators:**

```bash
# Out of memory before any script ran ("in Unknown on line 0").
# CrowdStrike says this may be the only server-side trace an attack leaves.
zgrep -hE 'Allowed memory size of [0-9]+ bytes exhausted.*in Unknown on line 0' \
  /var/log/php*.log* /var/log/apache2/error.log* /var/log/httpd/error_log* /var/log/nginx/error.log* 2>/dev/null

# Worker crashes (memory corruption). Signal 9 is the kernel OOM killer, not PHP.
zgrep -hE 'exited on signal (11|7)|failed processes threshold' /var/log/php*fpm*.log* 2>/dev/null
zgrep -hE 'exit signal (Segmentation fault|Bus error)' /var/log/apache2/error.log* /var/log/httpd/error_log* 2>/dev/null

# After patching: attempts that ran into the new hard limit
zgrep -hE 'exited with code 137|Hard memory limit of' /var/log/php*fpm*.log* 2>/dev/null

# Bursts of POSTs to .php ending in 500 or 502, counted by source IP (combined log format)
awk '$6 ~ /POST/ && $7 ~ /\.php/ && ($9 == 500 || $9 == 502) {print $1}' /var/log/nginx/access.log \
  | sort | uniq -c | sort -rn | head
```

**Request and network indicators:**

| Indicator | Where to look | Notes |
| --- | --- | --- |
| User agents `php2shell-scan/109618`, `php2shell-verify/1.0`, `Mozilla/5.0 (php2shell-poc)`, `php2shell-109618`, `php2shell-109619` | Access and WAF logs | Seen on canaries and in public PoC code. Some defender tools use similar strings |
| Multipart boundaries `B7kQpoc618` and `B7kQpoc619` | WAF or request-body logs | From a leaked PoC bundle (reported by Precursor Security, unverified). Trivial to change |
| Very long, binary-looking `PHPSESSID` values in `Set-Cookie` response headers | Proxy or WAF response-header logs | Tied to the memory leak (Wiz, php2shell.com). Normal access logs don't record it |
| Bursts of multipart POSTs to any `.php` URL returning 500 or 502 | Access logs | Behavioural. Baseline first |

```bash
zgrep -hiE 'php2shell-(scan|verify|poc|10961)' /var/log/nginx/access.log* /var/log/apache2/access.log* 2>/dev/null
zgrep -hE '85\.11\.167\.243|45\.198\.224\.169|221\.14\.159\.134|91\.90\.123\.25|64\.95\.12\.7' \
  /var/log/nginx/access.log* /var/log/apache2/access.log* 2>/dev/null
```

**File system indicators:**

```bash
# Session files that contain PHP code. Check the FPM pool's session.save_path as well
SESS=$(php -r 'echo session_save_path() ?: sys_get_temp_dir();')
grep -l '<?php' "$SESS"/sess_* 2>/dev/null

# New or changed PHP files in web roots, last 7 days
find /var/www /srv /home/*/public_html -name '*.php' -mtime -7 -ls 2>/dev/null

# Dropped files in temp locations
find /tmp /var/tmp /dev/shm -type f \( -name '*.php' -o -name '*.sh' -o -perm -u+x \) -mtime -7 -ls 2>/dev/null

# Web shell markers in recently changed PHP files (expect noise from legitimate updates)
find /var/www -name '*.php' -mtime -7 -print0 2>/dev/null \
  | xargs -0 grep -lE 'eval\(|assert\(|gzinflate\(|shell_exec\(|passthru\(|base64_decode\(' 2>/dev/null
```

**CMS and framework quick checks** (run each from the application's root directory):

```bash
# WordPress (WP-CLI)
wp core verify-checksums
wp plugin verify-checksums --all
wp user list --role=administrator --fields=ID,user_login,user_registered

# Drupal: PHP files never belong in the public files directory
find sites/*/files -name '*.php' 2>/dev/null
drush sql:query "SELECT uid, name, created FROM users_field_data ORDER BY created DESC LIMIT 20"

# Joomla 4 and 5
php cli/joomla.php user:list
find images tmp -name '*.php' 2>/dev/null

# Magento / Adobe Commerce: no PHP belongs in pub/media or pub/static
find pub/media pub/static -name '*.php' 2>/dev/null

# Laravel and Symfony: only index.php belongs in public/
find public -name '*.php' ! -name index.php 2>/dev/null
find storage -name '*.php' -not -path 'storage/framework/views/*' 2>/dev/null    # Laravel

# Nextcloud
sudo -u www-data php occ integrity:check-core
```

On Magento, also review admin accounts under System > Permissions > All Users.

**Process and persistence indicators:**

```bash
# Shells or download tools whose parent is a PHP worker or the web server
pstree -ap | grep -A5 -E 'php-fpm|apache2|httpd|lsphp' | grep -E '\b(sh|bash|dash|curl|wget|nc|ncat|python3?|perl),'

# Open connections owned by PHP or web server processes
sudo ss -tnp | grep -E 'php-fpm|apache2|httpd|lsphp'

# Persistence through cron
sudo crontab -l -u www-data 2>/dev/null
sudo grep -rE 'curl|wget|base64|/dev/tcp' /etc/cron* /var/spool/cron 2>/dev/null
```

**Compromise indicators checklist:**

- [ ] No startup out-of-memory lines ("in Unknown on line 0") before you patched, or each one explained
- [ ] No unexplained worker crashes (SIGSEGV or SIGBUS) since 20 October
- [ ] No hits from the known user agents or IPs, or each one checked
- [ ] No session files containing PHP code
- [ ] No unexpected new `.php` files in web roots, `/tmp` or `/dev/shm`
- [ ] No shells or download tools spawned by PHP workers
- [ ] No unexplained outbound connections from the web tier
- [ ] Core and plugin checksums clean (WordPress)

**If ANY indicator turns up:** start incident response. Keep in mind that the memory leak leaves nothing on disk. If a server showed the startup out-of-memory pattern or bursts of crashes before you patched, rotate everything it could see: database passwords, API keys, framework secrets and active sessions. Framework secrets means WordPress salts (`wp config shuffle-salts`), Drupal's `hash_salt`, Joomla's `$secret`, Laravel's `APP_KEY` (Laravel 11 and later can keep the old key in `APP_PREVIOUS_KEYS` so encrypted data still decrypts), Symfony's `APP_SECRET` and the Magento encryption key (follow Adobe's documented procedure).

*Sources: CrowdStrike, VulnCheck, php2shell.com, GHSA-6wmf-4rv5-p3rp, PHP and Apache source log formats, CISA*

---

#### DEFENSE-IN-DEPTH MEASURES

Every vendor says the same thing: a WAF buys you time, it doesn't fix PHP. Cloudflare watched attackers mutate their payloads to get around its rules within hours of shipping them.

**Cloudflare**

| Ruleset | Plans | Rules | Notes |
| --- | --- | --- | --- |
| Cloudflare Managed Free Ruleset | All plans, including Free | 7 | Cloudflare says it's deployed automatically |
| Cloudflare Managed Ruleset | Pro, Business, Enterprise | 7 | Only works if the Managed Ruleset is deployed on the zone |
| Cloudflare Managed Response Firewall Default | All plans (per Cloudflare) | 1 | Response-phase rule. It blocks the origin's response but can't undo what already ran at the origin |

All 14 request rules are named "Generic Rules - Anomalous Execution - 1" through "- 7", with Block as the default action. Cloudflare hasn't published what they match.

| # | Managed Ruleset rule ID | Free Ruleset rule ID | Live since (UTC) |
| --- | --- | --- | --- |
| 1 | c1d409ad838f47f59446f21cc438d5dd | 4e315666e8e344cdb64119a3b5ccf32f | 20 Oct 23:20 |
| 2 | f9a45927414245e3a64bb12d0a0c2f64 | 00c10e6cc4da4bd08722adeebcd95984 | 21 Oct 02:33 |
| 3 | 771c654ae1154d4daa72f4e1545d163c | 48ddf592361e4fdb97a8a85982dab691 | 20 Oct 23:20 |
| 4 | 217005865f0145f392300bac212afcc9 | 3556683d512a47f2937899c524b1a4c6 | 21 Oct 02:33 |
| 5 | 38efdb17ea594501bb55a1d71867b12e | da07c85852e34625bb478496302ede46 | 21 Oct 08:43 |
| 6 | 4148ae686fdb4d01be673fab9d9c073d | 19373a83abb840e18ab3ce5a7975ce1b | 21 Oct 08:43 |
| 7 | 1389aa259617437392ce149c48143645 | 07f0b6ddaabf40b1b2042dade8d613fa | 21 Oct 08:43 |

The response-phase rule ID is `afffe22785674c80a31875c848d2242a` ("Default block for High confidence Signatures". Cloudflare says the description will change).

Also worth watching: at 20:13 UTC on 21 October the Managed Ruleset (version 412) gained three new rules in Log mode, with no changelog entry. Their IDs are `77ededc6c429477e94afcaccbe0f6163`, `869cdabd8fa446edaac89894f13b49ab` and `f5b26ba6db3342d8b7ae65e1a8e9d8ae`, and they look at upload size, truncated bodies and serialization markers in request bodies. The timing suggests they're related. Cloudflare hasn't said so. Check Security Events for false positives on large uploads before they move to Block.

**Verify your zones:**

- Check for overrides. A ruleset-level Log override, or an override on the `anomaly` category, silently applies to these new rules too.
- In the dashboard, open Security > Events and filter by the rule IDs above.
- Read the zone's managed phase through the API (read-only):

```bash
curl -s "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/rulesets/phases/http_request_firewall_managed/entrypoint" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  | jq '.result.rules[] | {enabled, ruleset: .action_parameters.id, overrides: .action_parameters.overrides}'
```

The Managed Ruleset's ID is `efb7b8c949ac4650a09736fc376e9aee`. You want an enabled rule that executes it, with no overrides that disable or downgrade the rules above.

- Lock the origin. If the origin answers direct requests, attackers simply go around Cloudflare. Allow only Cloudflare's IP ranges at the origin firewall, or use Authenticated Origin Pulls.
- Body inspection has plan-based size limits, and the origin still receives the whole body.

**Other WAF and security vendors**

Status at the cut-off, taken from vendor pages, rule docs and release notes captured on 21 October.

| Vendor | Product | What shipped | Rule IDs or names | Turn on or verify | Caveats |
| --- | --- | --- | --- | --- | --- |
| Akamai | App & API Protector | Rule shipped | Adaptive Security Engine rule 3000991 | Akamai says customers are protected. Check the rule's action in your security policy | Doesn't say whether it's on by default |
| Bunny | Shield | Rules shipped | WAF-PHP-UPLOAD-PROGRESS-POC, WAF-PHP-MULTIPART-NESTING | Blocks only when Shield is in blocking mode | Pattern-specific. May flag apps that use upload progress |
| Check Point | Quantum IPS | Protection shipped | CPAI-2026-15172 | Update IPS, set the protection to Prevent, install policy | CloudGuard WAF: protection claimed, no rule named |
| Cisco Talos | Snort, Secure Firewall | Rules shipped | SID 1:67359 (both CVEs), SID 1:67360 (CVE-2026-109619) | Enabled, and set to drop in three IPS policies | 67360 uses a rate filter |
| CloudLinux | Imunify360 WAF | Rules shipped | WPT-3968 (ModSecurity rules 8.83 and 8.84) | Applied automatically when the WAF is on. Check the ruleset is 8.83 or later | Logs only on LiteSpeed. CloudLinux's KB and blog disagree on whether it blocks yet |
| Fastly | Next-Gen WAF | Templated rule | CVE-specific templated rule | Opt in from the WAF dashboard | Partial coverage. May block real traffic |
| Imperva | Cloud WAF, Elastic WAF, WAF Gateway | Protection shipped | Elastic WAF 2.2.1. WAF Gateway 14.7 Patch 221 and 15.5 Patch 21, plus ADC content | Cloud WAF: protected out of the box, per Imperva. Elastic WAF: upgrade. Gateway: content update, patch and a manual policy change | Gateway needs every step |
| Palo Alto Networks | NGFW Advanced Threat Prevention | Signature shipped | Threat ID 543004 | Needs an Advanced Threat Prevention subscription | PAN-OS and Panorama are affected themselves, fix pending |
| OPSShield | cPGuard WAF | Claim only | Unnamed rule | Delivered automatically. Toggle the WAF to force an update | No rule ID or version given |
| OWASP CRS | Core Rule Set | Pending | Pull request 4893 (blocks known PoC user agents), still open | Nothing to enable yet | User agents are trivial to change |
| Trend Micro | Deep Security IPS | Unconfirmed | Candidate rule 1012749 (DSRU 26-064) | Check the rule's CVE mapping before you rely on it | No CVE named in the rule notes |

**Detection only (alerts, doesn't block):**

| Vendor | Product | Content |
| --- | --- | --- |
| CrowdStrike | Falcon Next-Gen SIEM | Correlation template "PHP Memory Exhaustion During Request Startup". Needs web server error logs in the SIEM |
| Qualys | VMDR | QIDs 389102, 39033, 5019895 and 5019896 (signatures VULNSIGS-2.6.727-2). QID 45110 finds PHP installs |
| Tenable | Web App Scanning | Version-based plugins 115719 to 115722 |
| Zafran | Snort and Suricata rule | SID 1026109620, alert only |

**Guidance only, no rule:** Patchstack, Sophos (block multipart at your own server or WAF) and Wordfence (update PHP on the server, still investigating at cut-off).

**No PHP2Shell rule found in the captured material:** AWS WAF, Azure WAF (only a community answer saying there's no official signature), Google Cloud Armor, F5, Fortinet FortiWeb (Fortinet's PSIRT entry says "Virtual Patch: No"), Barracuda, Radware, Sucuri, Trustwave, Netlify, Vercel and Atomicorp. Some of those captures predate the disclosure, so ask the vendor before you assume nothing exists. Bear in mind that AWS WAF inspects only the first 8 KB of a request body behind an ALB by default (16 KB behind CloudFront), which limits what any body rule can see.

Vendors also disagree on which CVE a rule covers. Bunny and Talos SID 67360 tie the upload-progress pattern to CVE-2026-109619 only, so check the rule notes before you tell anyone both CVEs are covered.

*Sources: vendor advisories, rule documentation and release notes captured on 21 October 2026*

**Server-side stopgaps (when you can't patch today)**

These come from PHP's own advisories, and none of them is complete. PHP calls updating "the only complete mitigation" for CVE-2026-109618 and says there is "no complete generic configuration mitigation" for CVE-2026-109619.

| Stopgap | Helps against | Leaves open | Side effects |
| --- | --- | --- | --- |
| `session.upload_progress.enabled = 0` | The demonstrated session paths of CVE-2026-109619 | Other variants and the out-of-memory bug | Upload progress bars stop working |
| `fatal_error_backtraces = 0` (PHP 8.5 only) | The strongest confirmed mod_php chain on 8.5, which needs no request body | Other stale-state paths | Fatal errors lose their backtraces |
| `memory_limit = -1`, set by the admin rather than the app | The confirmed out-of-memory exploit of CVE-2026-109618 | Real allocation failures, and all of CVE-2026-109619 | One request can eat all the RAM. Put a real limit back after patching |
| Block `multipart/form-data` where the app doesn't need it | Multipart attack paths and the upload-progress path | On mod_php, ordinary form posts may still reach vulnerable parsing (reported). The PHP 8.5 backtrace path | Breaks uploads and some checkouts |
| Body, part-count and nesting limits at the proxy, including for chunked requests | Some memory-exhaustion tricks | Anything that fits under the limits | Large forms may break |
| Remove per-request INI (`php_value` in `.htaccess`, `fastcgi_param PHP_VALUE`) | The early-diagnostics path of CVE-2026-109619 | Session paths and other state | Config migration work |
| `pm.max_requests = 1` (FPM) | State carried between requests on the same worker | Single-request paths | Big performance hit |
| Read-only mode (allow only GET and HEAD) | Most body-based paths | The PHP 8.5 backtrace path | Logins, forms and checkout stop |
| Take it offline | Everything | Nothing | The site is down |

**Config snippets:**

```ini
; php.ini, or the FPM pool / Apache config. NOT ini_set() or .user.ini, which apply too late.
session.upload_progress.enabled = 0
; PHP 8.5 only
fatal_error_backtraces = 0
```

```ini
; PHP-FPM pool, for example /etc/php/8.4/fpm/pool.d/www.conf
php_admin_flag[session.upload_progress.enabled] = off
php_admin_flag[fatal_error_backtraces] = off
; Optional and slow: one request per worker
; pm.max_requests = 1
```

```apache
# Apache with mod_php (server or vhost config)
php_admin_flag session.upload_progress.enabled off
```

Then restart every worker. These settings only reach new processes.

**Block multipart when the site never takes uploads:**

```nginx
# nginx, inside the server block
if ($content_type ~* "multipart/form-data") {
    return 403;
}
```

```apache
# Apache 2.4
<If "%{CONTENT_TYPE} =~ m#multipart/form-data#i">
    Require all denied
</If>
```

If some pages do need uploads, those pages stay exposed, even behind a login. PHP parses the request body before your code checks who the user is.

Quick test: `curl -s -o /dev/null -w '%{http_code}\n' -F a=b https://example.com/` should now return 403.

**Body limits that also cover chunked requests:**

```nginx
client_max_body_size 8m;
```

```apache
LimitRequestBody 8388608
```

Also lower `max_input_vars`, `max_file_uploads`, `max_multipart_body_parts` and `max_input_nesting_level` to what your app actually needs. `post_max_size` on its own is not enough. PHP's advisory says limits must also cover requests that have no usable Content-Length, and Calif showed that small bodies can still exhaust memory.

**If you use `memory_limit = -1` as a stopgap**, cap memory at the service level instead. Never use RLIMIT_AS or RLIMIT_DATA for this, because PHP warns they trigger the vulnerable path. That includes Apache's `RLimitMEM`.

```ini
# /etc/systemd/system/php8.4-fpm.service.d/memory.conf
[Service]
MemoryMax=4G
```

When the pool hits that cap, the kernel may kill the whole pool rather than one worker, so size it with care.

**Find risky leftovers:**

```bash
grep -rnE '^\s*php_(value|flag)' --include=.htaccess /var/www /home/*/public_html 2>/dev/null
grep -rnE 'PHP_(ADMIN_)?VALUE' /etc/nginx/ 2>/dev/null
grep -rn 'RLimitMEM' /etc/apache2/ /etc/httpd/ 2>/dev/null
```

**What does NOT help:**

- Hiding errors (`display_errors=Off`, `error_reporting`, the `@` operator). PHP says so explicitly.
- `ini_set()` or `.user.ini`. They apply after the attack has already happened.
- Application-level controls such as login checks, CSRF tokens, security plugins and maintenance mode. The startup attacks run before any of them.
- `session.use_strict_mode=1`. No source shows that it helps.
- `file_uploads=Off`. PHP still parses the multipart body and only skips the file parts (Mattias Geniar, tested).
- Disabling the PHP handler in Apache or nginx. The web server then serves your `.php` source code as plain text, config secrets included.
- A WAF in front of an origin that still answers direct requests.

---

**Network isolation options**

**Option 1: VPN-only access**

```bash
# iptables: allow web traffic only from the internal range
iptables -A INPUT -p tcp --dport 443 ! -s 10.0.0.0/8 -j DROP
iptables -A INPUT -p tcp --dport 80 ! -s 10.0.0.0/8 -j DROP
iptables-save > /etc/iptables/rules.v4
```

**Option 2: Read-only mode**

```nginx
# nginx: reject everything except GET and HEAD
if ($request_method !~ ^(GET|HEAD)$) {
    return 405;
}
```

```apache
# Apache: reject everything except GET and HEAD
<Location "/">
    <LimitExcept GET HEAD>
        Require all denied
    </LimitExcept>
</Location>
```

On PHP 8.5, set `fatal_error_backtraces=0` as well, because one known path needs no request body at all.

**Option 3: IP allowlist or reverse proxy authentication**

```nginx
# nginx in front of the PHP site
location / {
    allow 203.0.113.0/24;    # your office or VPN egress
    deny all;
    # ... existing PHP config
}
```

**Contain the blast radius:**

- Run each site's PHP pool as its own user, with no write access to code directories.
- Confine workers with AppArmor or SELinux so a compromised worker can't spawn shells or read other sites. This won't stop the bug or the memory leak, but it limits what comes after.
- `disable_functions` (exec, system, shell_exec, passthru, proc_open, popen) makes life harder for a dropped web shell. It won't stop native code execution from a memory-corruption exploit.

---

**Monitoring and alerting**

**Splunk:**

```spl
index=web (("Allowed memory size of" "in Unknown on line 0") OR "Hard memory limit of" OR "exited on signal 11" OR "exited with code 137")
| stats count min(_time) as first_seen max(_time) as last_seen by host, source
| where count > 3
```

**Grafana Loki alert:**

```logql
sum by (host) (count_over_time({job="php-fpm"} |~ "exited (with code 137|on signal (11|7))" [5m])) > 3
```

**Sigma rule (Linux process creation):**

```yaml
title: Shell or Download Tool Spawned by PHP Worker
id: a3fe0734-719c-4b51-91a7-b47adbaa3d65
status: experimental
description: Detects a PHP worker or web server process spawning a shell or network tool, the usual follow-up after PHP2Shell exploitation (CVE-2026-109618, CVE-2026-109619)
references:
    - https://www.cisa.gov/news-events/alerts/2026/10/21/critical-vulnerabilities-php-require-immediate-patching
author: 0xDTC
date: 2026-10-22
tags:
    - attack.persistence
    - attack.t1505.003
    - attack.execution
    - attack.t1059.004
logsource:
    product: linux
    category: process_creation
detection:
    selection_parent:
        ParentImage|contains:
            - '/php-fpm'
            - '/php-cgi'
            - '/lsphp'
            - '/frankenphp'
            - '/apache2'
            - '/httpd'
    selection_child:
        Image|endswith:
            - '/sh'
            - '/bash'
            - '/dash'
            - '/curl'
            - '/wget'
            - '/nc'
            - '/ncat'
            - '/python3'
            - '/perl'
            - '/base64'
    filter_mail:
        CommandLine|contains: 'sendmail'
    condition: selection_parent and selection_child and not filter_mail
falsepositives:
    - Applications that shell out on purpose (image conversion, git hooks, backups)
level: high
```

The stock SigmaHQ web shell rule (`proc_creation_lnx_webshell_detection`) lists httpd, lighttpd, nginx, apache2, node and caddy as parents, but not php-fpm. A shell spawned by an FPM worker slips past it, which is why the rule above names php-fpm, php-cgi, lsphp and frankenphp.

**Microsoft Defender (KQL):**

```kql
DeviceProcessEvents
| where Timestamp > ago(7d)
| where InitiatingProcessFileName matches regex @"^(php-fpm|php-cgi|lsphp|frankenphp|apache2|httpd)"
| where FileName in~ ("sh", "bash", "dash", "curl", "wget", "nc", "ncat", "python3", "perl", "base64")
| where ProcessCommandLine !contains "sendmail"
| project Timestamp, DeviceName, InitiatingProcessFileName, InitiatingProcessCommandLine, FileName, ProcessCommandLine
```

**CrowdStrike Falcon Next-Gen SIEM:** switch on the correlation rule template "Generic - Web - PHP Memory Exhaustion During Request Startup (CVE-2026-109618 & CVE-2026-109619)".

---

#### ADVANCED FORENSICS

**Crash dumps (if workers crashed before you patched):**

```bash
coredumpctl list | grep -E 'php|apache2|httpd|lsphp'
coredumpctl dump <PID> -o php-worker-<PID>.core
```

**Memory of a live worker (if you suspect compromise):**

```bash
sudo gcore -o /forensics/php-fpm "$(pgrep -f 'php-fpm: pool' | head -1)"
strings /forensics/php-fpm.* | grep -E 'eval\(|base64_decode|/bin/sh|curl |wget '
```

**Docker container forensics:**

```bash
docker commit CONTAINER_ID forensics-$(date +%Y%m%d)
docker save forensics-$(date +%Y%m%d) > container-forensics.tar
docker diff CONTAINER_ID
docker logs CONTAINER_ID > container-logs-$(date +%Y%m%d).log 2>&1
```

**Preserve evidence:**

```bash
# Logs back to at least 20 October, PHP sessions and web roots
sudo tar -czf evidence-$(hostname)-$(date +%Y%m%d).tar.gz \
  /var/log/apache2 /var/log/httpd /var/log/nginx /var/log/php* \
  /var/lib/php/sessions /var/www 2>/dev/null
sha256sum evidence-*.tar.gz > evidence-hashes.txt

# Full disk image if you expect legal action
sudo dd if=/dev/sda of=/mnt/external/forensic-image-$(date +%Y%m%d).dd bs=4M status=progress
sha256sum /mnt/external/forensic-image-*.dd >> evidence-hashes.txt
```

**Fake and booby-trapped PoCs:** if anyone on your team downloaded exploit code to "test", treat that machine as suspect. Among the repositories reviewed for this analysis were a fake PoC built to beacon host details to whoever configures it, a "mass exploit" repo that was only a Telegram lure, and a widely shared bundle that carried about 40 MB of unexplained Linux binaries and had three different hashes on the same day. Hashes worth adding to your EDR blocklist:

| File | SHA-256 |
| --- | --- |
| php-up-2026.tar.gz (one retrieval) | 0c254d547cac650bfbcb2275e30c2083737520a51a0511646f1fbb7ed60a0d63 |
| php-up-2026.tar.gz (hash printed by the publisher, didn't match the download) | a29d640e0481ff64a39541faeeeec0f998ecd18fb692816e75a6bcfa0f683017 |
| php-up-2026.tar.gz (another retrieval) | d1ec08b7a7b837e99e1612078c24dbb2f17a1c94cfe8a4b634f45679aceaead1 |
| php2shell-reup.zip | 74adf2c4eb82776553837d185bae165494288747b43437fd452021a6ff6a93f0 |

---

## BUSINESS IMPACT ANALYSIS

### Financial impact scenarios

#### Scenario 1: Prevention (patch this week)

**Costs:**

- Staff time: roughly 1 to 3 engineer-hours per server for the patch, a full restart and checks
- Testing: about half a day per critical app, more if it runs close to its memory limit
- EOL upgrades: days to weeks per app that has to move off PHP 8.1 or older, or the price of a vendor backport subscription
- Downtime: a short restart window per server

**Benefits:**

- No breach costs, no notifications, no letters from regulators
- An accurate PHP inventory, which you'll want for the next one

---

#### Scenario 2: Breach with web shell and ransomware

**How it plays out:** exploit, web shell, database credentials lifted from config files, lateral movement, then encryption or extortion.

**Costs:**

- Forensics and incident response, legal counsel, breach notification
- Downtime for every site on the affected servers while you rebuild
- Regulatory exposure if personal data was involved
- Higher cyber insurance premiums at renewal
- For scale: IBM puts the average breach at USD 4.44 million worldwide and USD 10.22 million in the US (Cost of a Data Breach Report 2025)

---

#### Scenario 3: Silent theft through the memory leak

**What's different:** the memory leak in CVE-2026-109619 can expose session data, credentials and keys without touching the disk. You may not know anything happened until stolen credentials turn up somewhere else.

**Costs:**

- Rotating every secret the web tier holds and forcing users to log in again
- Investigating every system those secrets open (databases, payment APIs, cloud accounts, email)
- Notification if personal data was in memory at the time
- Time: IBM's 2025 report puts the average time to identify and contain a breach at 241 days. Leaked credentials can sit unused for a long time.

---

### Operational impact timeline

**Hour 0-24: Inventory and patching**

- Web and ops teams pulled off roadmap work
- Restart windows on every PHP server
- Decisions on any EOL PHP that faces the internet

**Day 2-7: Verification**

- Log review back to 20 October
- Chasing hosting providers and appliance vendors for written answers
- Removing stopgaps that broke features, once patches are confirmed

**Week 2-4: Hardening**

- Upgrade projects for PHP 8.1 and older
- Origin lock-down behind the WAF or CDN
- New detection rules and longer log retention

**Month 2-6: Only if you found a compromise**

- Regulator questions, customer communications, possible claims
- Audit follow-ups and insurer reviews

---

### Cost-benefit analysis

| Option | Effort | What's left exposed | Verdict |
| --- | --- | --- | --- |
| **Patch and restart now** | Hours per server | Nothing known | Do this |
| **Stopgaps plus WAF, patch later** | Hours to set up, risk stays | Variants the stopgaps miss, and WAF bypasses (already seen) | A bridge for days, not weeks |
| **Take old PHP offline** | Business downtime | Nothing while it's off | Right call for internet-facing 8.1 and older without a quick backport |
| **Wait** | None today | Everything | Public PoCs exist, and attempts started before the patch did |

---

## COMPLIANCE & LEGAL REQUIREMENTS

### Regulatory notification obligations

If you find signs of compromise, these clocks may start running. This is a summary, not legal advice.

#### GDPR (EU) and UK GDPR

- **Timeline:** 72 hours to notify the supervisory authority after becoming aware of a personal data breach, unless it's unlikely to put people at risk
- **Individuals:** without undue delay if the risk to them is high
- **Fines:** up to EUR 20 million or 4% of worldwide annual turnover (UK: GBP 17.5 million or 4%)
- *Source: GDPR Articles 33, 34 and 83*

#### NIS2 (EU essential and important entities)

- **Early warning:** within 24 hours of becoming aware of a significant incident
- **Incident notification:** within 72 hours
- **Final report:** within one month
- *Source: Directive (EU) 2022/2555, Article 23*

#### India: CERT-In and DPDP

- **CERT-In:** report covered cyber incidents, including unauthorised access and compromise of systems, within 6 hours of noticing them
- **DPDP Act 2023:** breach-reporting duties phase in under the DPDP Rules 2025, so check whether they apply to you yet
- *Source: CERT-In Directions under Section 70B(6) of the IT Act, 28 April 2022*

#### SEC (US-listed companies)

- **Timeline:** Form 8-K Item 1.05 within four business days of determining that a cybersecurity incident is material
- *Source: SEC cybersecurity disclosure rules (2023)*

#### HIPAA (US healthcare data)

- **Timeline:** notify affected individuals without unreasonable delay, and no later than 60 days after discovery. HHS too, and the media if more than 500 residents of a state or jurisdiction are affected
- *Source: 45 CFR 164.400-414*

#### US state breach laws

- All 50 states have breach notification laws. Timelines and triggers vary by state

---

### Documentation requirements checklist

**Create and maintain:**

- [ ] **Incident timeline** with precise timestamps
    - When the CVEs became public (20 October leak, 21 October release)
    - When your organization learned about them
    - When inventory, patching, restarts and verification finished
- [ ] **Technical details:**
    - Inventory of affected systems
    - Versions before and after patching, with restart times
    - Any evidence of compromise
    - Stopgaps applied and removed
- [ ] **Business impact assessment:**
    - Systems and data at risk
    - Customer impact
- [ ] **Decision log:**
    - Who authorized what, and why
    - Risk decisions on EOL PHP
- [ ] **Communications log:**
    - Who was told what, and when

**Retention:** follow your industry's record-keeping rules. Seven years is a common default.

---

### Insurance considerations

- [ ] Check that your cyber policy is active and what it covers (investigation, business interruption, liability, notification costs)
- [ ] Check the notification deadline in the policy. Many require notice within 24 to 72 hours of a suspected incident
- [ ] Check whether you must use the insurer's panel of response firms
- [ ] Keep evidence of your patching speed. Insurers look hard at known, unpatched vulnerabilities

**Call your broker straight away if:**

- You find indicators of compromise
- You're taking systems offline or running on stopgaps for more than a few days
- You're unsure what your policy covers

---

## DECISION MATRIX

### What should we do right now?

| Your situation | Immediate action | Priority | Timeline |
| --- | --- | --- | --- |
| **8.2.35 / 8.3.36 / 8.4.27 / 8.5.12+ and restarted** | Document it, keep monitoring | LOW | This week |
| **Fixed package installed, processes not restarted** | Restart now (Apache: full restart) | HIGH | Today |
| **Unpatched 8.2 to 8.5, internal only** | Patch and restart | HIGH | 48 hours |
| **Unpatched 8.2 to 8.5, internet facing** | **Patch and restart** | CRITICAL | Today |
| **PHP 8.1 or older, internet facing** | **Vendor backport, upgrade, or take it offline** | CRITICAL | Today |
| **Don't know what PHP we run** | **Run the inventory commands** | CRITICAL | 2 hours |
| **Odd crashes, shells or new PHP files** | **Start incident response** | EMERGENCY | Immediately |

---

### Risk decision framework

**For each PHP application:**

```
Step 1: Exposure
- Internet facing?                          HIGH
- Partner or VPN access only?               MEDIUM
- Internal only?                            LOWER

Step 2: Runtime
- Apache with mod_php?                      HIGH
- PHP-FPM, LiteSpeed, FrankenPHP?           HIGH
- Command line only?                        LOWER

Step 3: Version
- 8.1 or older (no official fix)?           HIGH
- 8.2 to 8.5 (fix available)?               MEDIUM, patchable today

Step 4: Data
- Personal, payment or health data,
  or secrets for other systems?             HIGH
- Public content only?                      LOWER

Step 5: Action
Internet facing + any other HIGH            Patch or isolate today
Internal + any other HIGH                   Patch within 48 hours
Everything else                             Patch this week
```

---

### Go/no-go decision checklist

**Before you approve emergency patching:**

- [ ] Backup or snapshot taken
- [ ] Rollback plan written down
- [ ] Stakeholders told about the maintenance window
- [ ] Short test plan agreed (logins, forms, uploads, checkout)
- [ ] Someone on call during and after the change
- [ ] Restart method confirmed (full Apache restart, worker restarts, container recreate)

**If you can't tick every box:**

- Take the application offline for now (safest)
- Apply the stopgaps and check the WAF rules first
- Restrict access to trusted IPs until you can patch

---

## FREQUENTLY ASKED QUESTIONS

### For executives

**Q: How do I know if we're affected?**

**A:** Ask IT one question: "Where do we run PHP, and which version?" Anything that serves web pages with PHP older than 8.2.35, 8.3.36, 8.4.27 or 8.5.12 needs attention, unless the vendor package says it carries the fix. If a hosting company runs your sites, ask them in writing.

---

**Q: What's the worst that could happen?**

**A:** Someone on the internet runs commands on your web server without logging in. From there they can read your database, copy customer data, steal passwords and API keys, plant backdoors, or use your server to attack other people. The memory leak bug can hand over secrets without leaving any files behind.

*Sources: PHP's advisories (GHSA-w8hw-w9fx-j79w, GHSA-6wmf-4rv5-p3rp), CISA alert*

---

**Q: Is anyone actually exploiting this?**

**A:** People are trying. VulnCheck, watchTowr and Cloudflare all saw exploit attempts on 21 October, some of them before the patch was public. As of that evening nobody had publicly confirmed a successful break-in, and CISA hadn't added the CVEs to its exploited list. Expect that to change. Public exploit code exists, and it usually gets better fast.

---

**Q: Why didn't our security tools catch this?**

**A:** Outside the research teams and the PHP project, nobody knew about it until 20 October. What tools can do now is find old PHP (though 43% of PHP hosts hide their version), block known attack patterns at the WAF, and catch what an attacker does after getting in. The patch is still the only real fix.

---

**Q: We use Cloudflare or another WAF. Are we covered?**

**A:** Partly. Cloudflare switched on rules for Free and paid plans, and other vendors shipped rules too. But attackers were already reshaping their requests to get past them, and a WAF does nothing if someone can reach your server directly. Use the WAF to buy time while you patch.

---

**Q: We're on PHP 7.4, 8.0 or 8.1. What now?**

**A:** PHP won't ship a release for these. You can upgrade, get a vendor backport (Ubuntu Pro, Debian LTS, Red Hat, TuxCare, Zend, cPanel, Plesk and others have them), or take the site offline. CISA's advice for 8.1 and older is offline until fixed. And don't trust a scanner that calls these versions "not affected". It's reading the CVE record, which only lists 8.2 to 8.5.

---

**Q: Our sites are on managed WordPress hosting. Do we need to do anything?**

**A:** Get a written answer from your host. Automattic (WordPress.com, VIP, Pressable, WP Cloud) said customers were protected with no action needed. Other hosts were still rolling out patches on 21 October. If you run WordPress yourself, updating WordPress won't help, because PHP is what needs the update.

---

**Q: How long does patching take, and will it break things?**

**A:** Usually under an hour per server, including the restart and checks. Two things can bite. The fix adds a hard memory ceiling, so memory-hungry apps may see workers killed. And stopgaps like blocking multipart uploads break features until you remove them. Patch your busiest app first, watch it, then roll out.

---

**Q: Should we tell our customers?**

**A:**

- **If you patch and find no sign of compromise:** this is routine maintenance, and usually no notification is needed.
- **If you find indicators of compromise:** talk to legal straight away. GDPR, NIS2, CERT-In, HIPAA, SEC and US state laws may all apply, some with deadlines measured in hours.

---

**Q: Should we pay for outside help?**

**A:** Yes if you see any sign of compromise, run a lot of PHP sites, work in a regulated industry, or are stuck on PHP 8.1 or older with no upgrade path. Probably not if you have a handful of servers on 8.2 or newer and a team that can patch and restart today.

---

### For technical teams

**Q: Will WAF rules fully protect us while we patch?**

**A:** No. They block known request shapes. Cloudflare saw payloads mutating within hours, WAFs only inspect bodies up to a size limit, and traffic that hits the origin directly skips the WAF entirely. Treat it as a bridge.

---

**Q: `php -v` shows 8.3.6. Am I vulnerable?**

**A:** Maybe not. Ubuntu 24.04's fixed package is 8.3.6-0ubuntu0.24.04.13, which still reports 8.3.6. The same goes for Debian backports, Red Hat, cPanel ea-php, CloudLinux, Plesk EOL builds and Chainguard revisions. Check the package changelog, and on 8.2 to 8.5 run the `memory_hard_limit_percent` test.

---

**Q: I updated the package. Why might I still be vulnerable?**

**A:** Because the old code can still be in memory. Apache with mod_php keeps the old libphp through a reload or graceful restart, so you need a full restart. LiteSpeed needs `pkill lsphp`. Containers need a rebuild with `--pull` and a recreate. Long-running workers (queues, Octane, FrankenPHP) need restarting too.

---

**Q: Which setups are most at risk?**

**A:** Apache with mod_php is the worst case. Researchers confirmed unauthenticated code execution during request startup, before any application code runs. PHP-FPM is vulnerable as well: a session-based chain was validated there. FPM does end a worker when request startup fails, which blocks the specific mod_php startup exploit, but that's no reason to wait. Command-line-only PHP isn't reachable over HTTP and comes last.

---

**Q: What changes after I patch?**

**A:** Three things:

1. A new setting, `memory_hard_limit_percent`. By default the hard ceiling is 200% of `memory_limit` when the limit is under 1 GB, or the limit plus 1 GB above that. A worker that crosses it prints "Hard memory limit of ... exhausted" to stderr and exits with code 137, without running shutdown functions. Size containers and cgroups for the number of workers times the hard limit.
2. PHP now keeps only the Content-Disposition and Content-Type headers of each multipart part and ignores the rest. Very few apps depend on the others.
3. PHP suggests leaving `session.upload_progress.enabled=0` if you don't use the feature, even after patching.

*Sources: patch diffs for 8.2.35 to 8.5.12, GHSA-6wmf-4rv5-p3rp, Mattias Geniar's hard-limit tests*

---

**Q: Do we need to rebuild Docker images?**

**A:** Yes. The official images for 8.2 to 8.5 were rebuilt on 21 October. `php:8.1` and older were not. Rebuild with `docker build --pull`, push, redeploy, then check `php -v` inside the running container. Anything you update inside a running container disappears on the next restart.

```bash
docker ps --format '{{.Names}}' | while read -r c; do
  printf '%s: ' "$c"
  docker exec "$c" php -r 'echo PHP_VERSION, PHP_EOL;' 2>/dev/null || echo "no php"
done
docker build --pull --no-cache -t yourapp:php-fixed .
```

---

**Q: What about firewalls, NAS boxes and other appliances?**

**A:** Plenty of them run PHP in their web UI. On 21 October Palo Alto said PAN-OS and Panorama are affected with no fixed release yet, Cisco was investigating ISE, Catalyst Center and SD-WAN Manager, OPNsense shipped fixes, and Sophos said its products aren't affected. F5, Citrix, Juniper (J-Web), pfSense, Synology and QNAP had said nothing yet. Keep management interfaces off the internet and follow each vendor.

---

**Q: Are the public PoCs safe to run for testing?**

**A:** Don't. The repositories reviewed for this analysis included a fake PoC built to beacon host details, a Telegram lure with no code, and a widely shared bundle full of unexplained binaries (see the hash table above). Use version checks, vendor scanners and the detection content in this document instead. If you really must test, do it in an isolated lab against your own build.

---

**Q: Our scanner says we're clean. Can we trust it?**

**A:** Find out how it decides. Scanners that read the CVE ranges skip PHP 8.1 and older. Banner-based scanners miss hosts that hide `X-Powered-By` (43% of PHP hosts, per VulnCheck) and misjudge backports. The community Nuclei templates were still unmerged on 21 October, and the active session-based one may also flag patched hosts that have `display_errors` on. Pair package checks on the host with the runtime test above.

---

**Q: We run Drupal, Joomla, Magento or Laravel. Does anything change?**

**A:** The fix doesn't change: patch PHP and restart it. No framework release, module or security plugin fixes this. A few framework-specific points:

- **Drupal on PHP 8.5:** Wiz's Drupal demo used a path that only exists in 8.5. If you can't patch today, set `fatal_error_backtraces = 0`.
- **Magento and other shops:** some checkouts post multipart forms. Test checkout before you block multipart as a stopgap.
- **Laravel and Symfony:** restart queue workers, Horizon, Octane and Messenger after patching, or they keep running the old PHP.
- **Managed platforms:** Pantheon, Upsun, Laravel Cloud and Automattic had patched PHP 8.2 to 8.5 by the cut-off, while WP Engine and Kinsta were still rolling out. Get it in writing for your plan.
- **If you find signs of compromise:** rotate the framework's own secrets too (see the list under Detection & Forensics).

---

**Q: Is the fix complete?**

**A:** PHP says updating is the complete fix for both CVEs. Someone claimed on oss-security that 8.5.12 might be incomplete. The claim was passed to PHP, who called it unrelated and not a real issue.

---

## SUCCESS CRITERIA & VERIFICATION

### You have successfully responded when:

**Technical validation checklist:**

- [ ] **All PHP inventoried:**
    - Hosts, containers, hosting panels and appliances
    - Version, package and SAPI recorded for each
    - Internet exposure documented
- [ ] **All PHP patched:**
    - 8.2.35, 8.3.36, 8.4.27, 8.5.12 or newer, or vendor packages that name both CVEs
    - EOL versions upgraded, backported or offline
- [ ] **All PHP restarted:** process start times are later than package install times
- [ ] **Verification passes:**

    ```bash
    php -v
    php -r 'var_dump(ini_get("memory_hard_limit_percent"));'      # a string on 8.2 to 8.5
    zgrep -lE 'CVE-2026-10961[89]' /usr/share/doc/php*/changelog.Debian.gz   # or rpm -q --changelog
    ps -eo pid,lstart,args | grep -E '[p]hp-fpm: master|[a]pache2|[h]ttpd'
    ```

- [ ] **No indicators of compromise** (see the checklist under Detection & Forensics)
- [ ] **Defense in depth:**
    - [ ] WAF rules active, with no overrides that weaken them
    - [ ] Origin locked to the WAF or CDN
    - [ ] Upload progress off where nobody uses it
    - [ ] Detection rules live (startup OOM, worker crashes, shells from PHP workers)
    - [ ] Log retention extended for forensics

---

**Operational validation checklist:**

- [ ] **Applications work:** logins, forms, uploads and checkout tested
- [ ] **No new 137 exits or 502 spikes** after patching, or the memory limits have been tuned
- [ ] **Stopgaps cleaned up:** `memory_limit=-1` reverted, multipart blocks removed if they're no longer needed
- [ ] **Documentation done:** timeline, versions before and after, restart times
- [ ] **Stakeholders updated:**
    - [ ] Leadership briefed
    - [ ] Legal told whether any compromise was found
    - [ ] Insurer told the remediation is complete

---

**Compliance validation checklist:**

- [ ] **Audit trail kept:** every action timestamped, every decision explained
- [ ] **Evidence preserved** where there was any doubt
- [ ] **Breach decision documented:** legal agrees on whether notification is needed, and the evidence supports that conclusion

---

### Post-incident review checklist

**Hold within a week of finishing:**

- [ ] **What went well:**
    - Time from public release to patch
    - How fast you found every PHP runtime
- [ ] **What needs work:**
    - Gaps in the inventory (containers, appliances, shadow sites)
    - Restart procedures
    - Contacts at hosting providers and appliance vendors
- [ ] **Metrics captured:**
    - Time to inventory: [X hours from the 21 October release]
    - Time to patch: [X hours]
    - Time to verified restart: [X hours]
    - Systems affected: [X]
    - Staff hours: [X]
- [ ] **Actions:**
    - Retire EOL PHP
    - Keep the PHP inventory current
    - Add restart verification to the patch process

---

## ADDITIONAL RESOURCES

### Official security advisories

- **PHP advisory:** https://www.php.net/archive/2026.php#2026-10-21-5
- **GHSA-w8hw-w9fx-j79w (CVE-2026-109618):** https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w
- **GHSA-6wmf-4rv5-p3rp (CVE-2026-109619):** https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp
- **CVE records:** https://www.cve.org/CVERecord?id=CVE-2026-109618 and https://www.cve.org/CVERecord?id=CVE-2026-109619
- **NVD:** https://nvd.nist.gov/vuln/detail/CVE-2026-109618 and https://nvd.nist.gov/vuln/detail/CVE-2026-109619
- **PHP Foundation statement:** https://thephp.foundation/blog/2026/10/21/todays-php-security-release-and-how-were-strengthening-php-security/

### Government and CERT advisories

- **CISA:** https://www.cisa.gov/news-events/alerts/2026/10/21/critical-vulnerabilities-php-require-immediate-patching
- **CERT-EU Security Advisory 2026-018:** https://cert.europa.eu/publications/security-advisories/2026-018/
- **NCSC-UK:** https://www.ncsc.gov.uk/news/critical-security-issue-affecting-php
- **NCSC-NL:** advisory NCSC-2026-0443 (https://advisories.ncsc.nl/) and inventory repository https://github.com/NCSC-NL/php2shell
- **CERT-FR:** https://cert.cyber.gouv.fr/alerte/CERTFR-2026-ALE-014/ and CERTFR-2026-AVI-1359
- **BSI / CERT-Bund:** WID-SEC-W-2026-4055
- **Canadian Centre for Cyber Security:** AL26-026 and AV26-1070
- **ENISA:** EUVD-2026-107260 (CVE-2026-109618) and EUVD-2026-107265 (CVE-2026-109619)
- **Others:** INCIBE-CERT INCIBE-2026-762, CERT Polska 195/2026 and 196/2026, NCSC-IE 2610210276, NCSC-FI 37/2026

### Research and threat intelligence

- **Calif:** https://calif.io/php2shell/
- **Wiz Research:** https://www.wiz.io/blog/php2shell-php-rce
- **php2shell.com:** https://www.php2shell.com/
- **watchTowr:** https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/ and FAQ https://watchtowr.com/intelligence/php-cve-2026-109618-faq/
- **VulnCheck:** https://www.vulncheck.com/blog/php2shell-vulnerabilities
- **Unit 42:** https://unit42.paloaltonetworks.com/cve-2026-109618-cve-2026-109619/
- **CrowdStrike:** https://www.crowdstrike.com/en-us/blog/crowdstrike-protects-customers-from-php-rce-zero-days/
- **Cisco Talos:** https://blog.talosintelligence.com/two-critical-rces-in-php-core/
- **Imperva:** https://www.imperva.com/blog/php2shell-vulnerability-updates/
- **Fastly:** https://www.fastly.com/blog/fastlys-proactive-protection-critical-php-rce-cve-2026-109618
- **Cloudflare:** https://blog.cloudflare.com/php-vulnerabilities-waf-protection/ and https://blog.cloudflare.com/full-transaction-waf/
- **Mattias Geniar (tested mitigations, patch notes, hard limit):** https://ma.ttias.be/php-cve-2026-109618-early-mitigation/, https://ma.ttias.be/php-cve-2026-109618-patch/ and https://ma.ttias.be/php-memory-hard-limit/

### Detection and scanning tools

- **Cloudflare WAF changelog:** https://developers.cloudflare.com/waf/change-log/changelog/
- **Corelight (Zeek and Suricata):** https://github.com/corelight/php2shell-detect
- **Velociraptor:** artifact `Generic.Applications.PHPInventory`
- **Tenable:** Web App Scanning plugins 115719 to 115722 (version-based)
- **Cisco Talos Snort:** SIDs 1:67359 and 1:67360
- **OpenAI version checker:** https://github.com/openai/php2shell-checker
- **CrowdStrike Falcon Next-Gen SIEM:** correlation template "PHP Memory Exhaustion During Request Startup"
- Treat the community Nuclei templates (pull requests 17473 and 17475) and the unmerged Sigma and Sentinel pull requests as unreviewed

### Vendor and distro advisories

- **Debian:** https://www.debian.org/security/2026/dsa-6574 and https://www.debian.org/lts/security/2026/dla-4848
- **Ubuntu:** https://ubuntu.com/security/notices/USN-8948-1 and https://ubuntu.com/security/CVE-2026-109618
- **Red Hat:** https://access.redhat.com/security/cve/CVE-2026-109618 and https://access.redhat.com/security/cve/CVE-2026-109619
- **SUSE:** https://www.suse.com/security/cve/CVE-2026-109618.html
- **AlmaLinux:** https://almalinux.org/blog/2026-10-21-php2shell-cve-2026-109618-cve-2026-109619/
- **Gentoo:** https://security.gentoo.org/glsa/202610-02
- **Arch Linux:** https://security.archlinux.org/ASA-202610-1
- **AWS:** https://aws.amazon.com/security/security-bulletins/2026-143-aws/
- **Google Cloud:** https://cloud.google.com/support/bulletins#gcp-2026-070
- **Azure App Service:** https://azure.github.io/AppService/2026/10/21/php-critical-security-release-app-service-linux.html
- **Heroku:** https://devcenter.heroku.com/changelog-items/3845
- **cPanel:** https://support.cpanel.net/hc/en-us/articles/44397389677335
- **Plesk:** https://support.plesk.com/hc/en-us/articles/44397729587351
- **Docker official images:** https://hub.docker.com/_/php
- **FrankenPHP:** https://github.com/php/frankenphp/releases/tag/v1.13.2
- **Automattic:** https://automattic.com/2026/10/21/your-sites-are-protected-against-the-php2shell-vulnerabilities/
- **WordPress.org:** https://wordpress.org/news/2026/10/critical-php-security-release/
- **Drupal:** https://www.drupal.org/psa-2026-10-20
- **Palo Alto Networks:** https://security.paloaltonetworks.com/CVE-2026-109618
- **Cisco:** https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-php-october26-Jj384Daq
- **Fortinet:** https://fortiguard.fortinet.com/psirt/FG-IR-26-186
- **Sophos:** https://www.sophos.com/en-us/security-advisories/sophos-sa-20261021-php2shell

### Emergency support contacts

- **PHP security team:** security@php.net (new vulnerability reports only)
- **CISA:** report@cisa.gov or +1 888-282-0870
- **FBI IC3:** https://www.ic3.gov
- **CERT-In (India):** incident@cert-in.org.in or 1800-11-4949
- **NCSC-UK:** https://report.ncsc.gov.uk/
- **Your hosting provider's security contact and your incident response retainer**

---

## QUICK REFERENCE CARD

### Print and post this

```
===============================================================
              PHP2SHELL CRITICAL VULNERABILITIES
       CVE-2026-109618 / CVE-2026-109619 | CVSS 4.0 9.2
===============================================================

FIXED VERSIONS:
  PHP 8.5.12   8.4.27   8.3.36   8.2.35   (8.6.0RC4)
  Or a vendor package whose changelog names both CVEs

NO OFFICIAL RELEASE:
  PHP 8.1, 8.0, 7.4 and older
  Vendor backport, upgrade, or take it offline

---------------------------------------------------------------
IMMEDIATE ACTIONS:
---------------------------------------------------------------

1. CHECK (1 minute):
   $ php -v
   $ php -r 'var_dump(ini_get("memory_hard_limit_percent"));'
     bool(false)  = unpatched 8.2 to 8.5 build
     a string     = patched build

2. PATCH:
   Debian/Ubuntu:   apt update && apt upgrade
   RHEL family:     dnf upgrade 'php*'
   Containers:      docker build --pull, then recreate

3. RESTART EVERYTHING:
   PHP-FPM:         systemctl restart php8.x-fpm
   Apache mod_php:  systemctl restart apache2  (NOT reload/graceful)
   LiteSpeed:       systemctl restart lsws && pkill lsphp

4. CAN'T PATCH TODAY?
   session.upload_progress.enabled = 0   (php.ini or pool config)
   fatal_error_backtraces = 0            (PHP 8.5)
   Keep WAF rules on, lock the origin to the WAF/CDN

---------------------------------------------------------------
INDICATORS OF COMPROMISE:
---------------------------------------------------------------

LOGS:
  "Allowed memory size ... exhausted ... in Unknown on line 0"
  "exited on signal 11"  /  "exit signal Segmentation fault"
  After patching: "exited with code 137", "Hard memory limit of"

REQUESTS:
  User agents: php2shell-scan/109618, php2shell-verify/1.0,
               Mozilla/5.0 (php2shell-poc)
  Bursts of multipart POSTs to .php ending in 500/502

HOST:
  sh, bash, curl or wget spawned by php-fpm, apache2, httpd, lsphp
  sess_* files containing <?php
  New .php files in web roots, /tmp, /dev/shm

---------------------------------------------------------------
THREAT LEVEL (21 Oct 2026, 22:30 UTC):
---------------------------------------------------------------

  Exploit attempts seen from 08:37 UTC on 21 Oct, before the patch
  No confirmed compromise published yet
  Not in CISA KEV yet, no EPSS score yet
  Updating your CMS, framework or plugins does NOT fix this

---------------------------------------------------------------
IF COMPROMISED:
---------------------------------------------------------------

  1. Isolate the server (keep it running for memory capture)
  2. Preserve logs back to 20 Oct. Do NOT delete anything
  3. Rotate every secret the web tier could read
  4. Bring in incident response, legal and your insurer
  5. Contact: [Your security team / CISO]

---------------------------------------------------------------
MORE INFORMATION:
---------------------------------------------------------------

  PHP advisory:  php.net/archive/2026.php#2026-10-21-5
  CISA alert:    cisa.gov, 21 Oct 2026, "Critical Vulnerabilities
                 in PHP Require Immediate Patching"
  CERT-EU:       cert.europa.eu, Security Advisory 2026-018

  Emergency Support: [Your IT Security Contact]
  Phone: [Phone Number]
  Email: [Email Address]

===============================================================
     A patched package with old workers still running is
            not patched. Patch, then restart.
===============================================================

Document Version: 1.0
Last Updated: 22 October 2026
Classification: TLP:CLEAR, distribute freely
```

---

## FINAL RECOMMENDATIONS

### The bottom line

**If you're on 8.2.35, 8.3.36, 8.4.27, 8.5.12 or a vendor build with the fix, and every PHP process has restarted:**

- You're fixed for these two CVEs
- Keep the WAF rules on and add the detection rules
- Turn off session upload progress if nobody uses it

**If you're on an unpatched 8.2 to 8.5:**

- Patch and restart today
- Check your logs back to 20 October

**If you're on PHP 8.1 or older:**

- This is the hard case, because there's no official release
- Get a vendor backport, upgrade, or take it offline
- Don't let a scanner talk you into "not affected"

**If you don't know what PHP you run:**

- Find out in the next two hours. Start with the inventory commands above

### Key takeaway for leadership

PHP2Shell takes more work to exploit than React2Shell did. What makes it dangerous is reach. PHP runs under a huge share of the web, much of it on versions that stopped getting updates years ago, and the leak gave attackers material before most teams had a patch.

Use the incident to fix three things:

1. Know where every PHP runtime lives, including inside appliances and containers
2. Retire EOL PHP instead of running it until it breaks
3. Treat restarts as part of patching, not an afterthought

### Remember

- Attack attempts started before the patch did
- A patch only counts once the old workers are gone
- A clean disk doesn't rule out the memory leak, so rotate secrets when in doubt
- Patch first, argue about the CVSS score later

**Patch today. Restart today.**

---

**Document Version:** 1.0
**Last Updated:** 22 October 2026
**Data cut-off:** 21 October 2026, 22:30 UTC
**Classification:** TLP:CLEAR, public distribution
**Prepared by:** 0xDTC (TheSecureLancer)

Facts in this document come from PHP's advisories, the CVE and NVD records, CISA and national CERT alerts, and the vendor and researcher publications listed above. Where those sources disagree, the document says so.
