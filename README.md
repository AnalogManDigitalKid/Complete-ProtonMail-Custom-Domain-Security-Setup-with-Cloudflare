# Complete ProtonMail Custom Domain Security Setup with Cloudflare

A comprehensive guide to setting up all essential mail security features for ProtonMail using a free Cloudflare plan.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Security Features Covered](#security-features-covered)
- [Setup Instructions](#setup-instructions)
  - [1. SPF (Sender Policy Framework)](#1-spf-sender-policy-framework)
  - [2. DKIM (DomainKeys Identified Mail)](#2-dkim-domainkeys-identified-mail)
  - [3. DMARC (Domain-based Message Authentication, Reporting & Conformance)](#3-dmarc-domain-based-message-authentication-reporting--conformance)
  - [4. DNSSEC and DANE](#4-dnssec-and-dane)
  - [5. CAA (Certification Authority Authorization)](#5-caa-certification-authority-authorization)
  - [6. MTA-STS and TLS-RPT](#6-mta-sts-and-tls-rpt)
  - [7. WKD (Web Key Directory)](#7-wkd-web-key-directory)
- [Testing Your Setup](#testing-your-setup)
- [Troubleshooting](#troubleshooting)
- [Credits](#credits)

## Overview

This tutorial will help you configure advanced email security features for your ProtonMail custom domain using Cloudflare's free plan. These features significantly improve email security, deliverability, and provide detailed reporting on potential abuse attempts.

## Prerequisites

- Domain configured with Cloudflare nameservers (registrar can be elsewhere)
- ProtonMail custom domain already set up and working
- Basic familiarity with DNS record management
- Cloudflare account (free plan sufficient)

## Security Features Covered

| Feature | Purpose | Difficulty |
|---------|---------|------------|
| **SPF** | Prevents email spoofing by defining authorized senders | Easy |
| **DKIM** | Cryptographically signs emails to verify authenticity | Easy |
| **DMARC** | Policy framework that uses SPF and DKIM for email authentication | Easy |
| **DNSSEC** | Secures DNS queries against tampering | Easy |
| **DANE** | Binds TLS certificates to domain names via DNS | Easy |
| **CAA** | Controls which Certificate Authorities can issue certificates | Easy |
| **MTA-STS** | Enforces TLS encryption for email transport | Advanced |
| **TLS-RPT** | Provides reports on TLS connection failures and MTA-STS policy enforcement | Advanced |
| **WKD** | Enables automatic PGP key discovery | Advanced |

## Setup Instructions

### 1. SPF (Sender Policy Framework)

SPF prevents other people from sending emails on behalf of your domain by defining which servers are authorized to send email.

**DNS Record to Create:**

| Type | Host | Value                                    |
| ---- | ---- | ---------------------------------------- |
| TXT  | @    | `v=spf1 include:_spf.protonmail.ch ~all` |

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture.png)

**For multiple email services**, include additional domains:
```
v=spf1 include:_spf.protonmail.ch include:spf.example.com ~all
```

### 2. DKIM (DomainKeys Identified Mail)

DKIM adds a digital signature to your emails that receiving servers can verify.

**DNS Records to Create:**

| Type | Host | Value | Proxy |
|------|------|-------|--------|
| CNAME | protonmail._domainkey | `<proton domain key 1>` | Disabled |
| CNAME | protonmail2._domainkey | `<proton domain key 2>` | Disabled |
| CNAME | protonmail3._domainkey | `<proton domain key 3>` | Disabled |

> **Note**: Get your specific ProtonMail DKIM values from your ProtonMail domain settings.

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture2.png)

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture3.png)

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture4.png)

### 3. DMARC (Domain-based Message Authentication, Reporting & Conformance)

DMARC uses SPF and DKIM to determine what happens to emails that fail authentication.

**DNS Record to Create:**

| Type | Host | Value |
|------|------|-------|
| TXT | _dmarc | `v=DMARC1; p=quarantine` |

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture5.png)

**Enable DMARC Management in Cloudflare:**

1. Navigate to: **Your domain > Email > DMARC Management**
2. Enable DMARC management

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture6.png)

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture7.png)

3. Click **Next** and allow the updates

This enables you to receive reports when emails are quarantined by DMARC policies.

### 4. DNSSEC and DANE

DNSSEC secures DNS queries, and DANE binds TLS certificates to domain names. To enable DANE on ProtonMail, simply enable DNSSEC.

**Steps:**
1. Navigate to: **Account Home > Domain Registration > Manage Domains > Your Domain > Manage > Configuration**
2. Enable DNSSEC

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture8.png)

### 5. CAA (Certification Authority Authorization)

CAA records specify which Certificate Authorities are authorized to issue certificates for your domain.

**DNS Records to Create:**

| Type | Host | Value |
|------|------|-------|
| CAA | @ | `0 issue google.com` |
| CAA | @ | `0 issue letsencrypt.org` |
| CAA | @ | `0 issue sectigo.com` |

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture9.png)

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture10.png)

> **Note**: Add CAA records for each Certificate Authority you want to authorize. The above covers most Cloudflare use cases.

### 6. MTA-STS and TLS-RPT

MTA-STS enforces TLS encryption for email transport, while TLS-RPT provides reporting on TLS connection issues.

**Goal:** Create a Cloudflare worker to handle and serve MTA-STS.txt policy files for multiple domains.

#### Step 1: Create the Worker

1. Navigate to **Account Home > Compute (Workers) > Workers & Pages**
2. Create a new worker
3. Choose "Start with Hello World!" and name it "MTA-STS"
4. Deploy the worker

#### Step 2: Configure the Worker Code

Click **Edit code** and use the following:

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture11.png)

```javascript
// This worker is designed to handle MTA-STS policies for multiple domains.
// Make a new worker with this script and add your domains to the stsPolicies dict like the example.
// Add a DNS AAAA record for mta-sts.yourdomain.com pointing to 100:: and set to proxied,
// then add a workers route for mta-sts.yourdomain.com/* pointing to this worker.
// You'll still need to manually add the appropriate _mta-sts.yourdomain.com TXT record to enable the policy, 
// and the _smtp._tls.yourdomain.com TXT record for reporting.

const stsPolicies = {
  "example1.com":
`version: STSv1
mode: enforce
mx: mail.protonmail.ch
mx: mailsec.protonmail.ch
max_age: 86400`,
  "example2.com":
`version: STSv1
mode: enforce
mx: mail.protonmail.ch
mx: mailsec.protonmail.ch
max_age: 86400`,
}

const respHeaders = {
  "Content-Type": "text/plain;charset=UTF-8"
}

addEventListener("fetch", event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const reqUrl = new URL(request.url)
  
  if (!reqUrl.hostname.startsWith("mta-sts.")) {
    return new Response(`Incorrect worker route. mta-sts policies must be served on the mta-sts subdomain\n`, {status: 500, headers: respHeaders})
  }
  
  const policyHost = reqUrl.hostname.slice(8)
  
  if (!stsPolicies.hasOwnProperty(policyHost)) {
    return new Response(`${policyHost} is not defined in the mta-sts worker\n`, {status: 500, headers: respHeaders})
  }
  
  if (reqUrl.protocol !== "https:" || reqUrl.pathname !== "/.well-known/mta-sts.txt") {
    reqUrl.protocol = "https:"
    reqUrl.pathname = "/.well-known/mta-sts.txt"
    return Response.redirect(reqUrl, 301)
  }
  
  return new Response(stsPolicies[policyHost] + "\n", {status: 200, headers: respHeaders})
}
```

**Configuration Notes:**
- Replace `example1.com` and `example2.com` with your actual domains
- Add more domains by copying the policy block
- Set `mode: enforce` (production) or `mode: testing` (for initial setup)

**For SimpleLogin domains:**
```javascript
  "example3.com":
`version: STSv1
mode: enforce
mx: mx1.simplelogin.co
mx: mx2.simplelogin.co
max_age: 86400`,
```

Deploy the worker after making changes.

#### Step 3: Configure Worker Routes

In the worker, navigate to **Settings > Domains & Routes**

For each domain, add a route:
- Zone: `example.com`
- Route: `mta-sts.example.com/*`
- Failure mode: fail closed

#### Step 4: Configure WAF Rules

On each domain, navigate to **Security > WAF > Custom rules**

1. Create rule named "Disabling Browser Integrity Check for MTA-STS"
2. Click "edit expression" and paste:
```
(http.request.full_uri wildcard "https://mta-sts.example.com/*") or (http.host eq "mta-sts.example.com") or (http.request.full_uri eq "https://mta-sts.example.com/")
```
3. Replace `example.com` with your domain
4. Choose action: **Skip**
5. Click "More components to skip" and select **Browser Integrity Check**
6. Keep order as **Last**
7. Deploy

#### Step 5: Configure Rules

On each domain, navigate to **Rules > Overview**

1. Click **Create rule > Configuration Rule** named "Disabling Browser Integrity Check for MTA-STS"
2. Click "edit expression" and paste the same expression as Step 4
3. Replace `example.com` with your domain
4. Click **Add** next to "Browser Integrity Check" and turn the slider to **disabled**
5. Keep order as **Last**
6. Deploy

#### Step 6: Create DNS Records

For each domain, create these DNS records:

| Type | Host | Value | Proxy |
|------|------|-------|--------|
| AAAA | mta-sts | `100::` | Enabled |
| TXT | _mta-sts | `v=STSv1; id=xxxxxxxxxx` | N/A |
| TXT | _smtp._tls | `v=TLSRPTv1; rua=mailto:postmaster@example.com` | N/A |

**Important Notes:**
- Replace `xxxxxxxxxx` with current epoch timestamp from https://www.epochconverter.com/
- Replace `postmaster@example.com` with your email for TLS-RPT reports
- The `id` must be changed whenever you modify the MTA-STS policy in the worker

### 7. WKD (Web Key Directory)

WKD enables automatic discovery of PGP public keys, making encrypted email easier to use.

#### Step 1: Create the Worker

1. Navigate to **Account Home > Compute (Workers) > Workers & Pages**
2. Create a new worker
3. Choose "Start with Hello World!" and name it "Proton-WKD"
4. Deploy the worker

#### Step 2: Configure the Worker Code

Click **Edit code** and use the following:

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture11.png)

```javascript
// 1. Create a worker in the dashboard and paste the following code into it
// 2. Replace all references to example.com to your domain, add more domains if needed
// 3. Deploy the worker
// 4. Setup routes in the dashboard for:
//    - example.com/.well-known/openpgpkey/*
//    - openpgpkey.example.com/.well-known/openpgpkey/*
//    - (apply above for additional domains too)
// 5. For each domain create these two DNS record
//    -     Type    Name         Value       Proxy
//    -     A       @            192.0.2.1   proxied  - this IP can be anything including your normal A record, it just needs to be proxied to work
//    -     CNAME   openpgpkey   @           proxied
// 6. If a domain does not host a website at the root A record (aka you set the IP to 192.0.2.1), then add a route inside the block-homepage-traffic worker.
// 7. Verify that it is working with https://metacode.biz/openpgp/web-key-directory
//    - Use these instead: https://wkd.chimbosonic.com/ (open-source) or webkeydirectory.com (proprietary)

export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    switch (url.hostname) {
      case 'example1.com':
//    case 'example2.org':
        return await handleRoot(request, url);
      case 'openpgpkey.example1.com':
//    case 'openpgpkey.example2.org':
        return await handleSubdomain(request, url);
      default:
        return await fetch(request);
    }
  },
};

async function handleRoot(request, url) {
  const lastSegment = url.pathname.split("/").pop();
  let newUrl;
  if (lastSegment === "policy") {
    newUrl = "https://api.protonmail.ch/.well-known/openpgpkey/" + url.hostname + "/policy";
  } else {
    newUrl = "https://api.protonmail.ch/.well-known/openpgpkey/" + url.hostname + "/hu/" + lastSegment + url.search;
  }
  const protonResponse = await fetch(newUrl, request);
  const alteredResponse = new Response(protonResponse.body, protonResponse);
  alteredResponse.headers.set("Access-Control-Allow-Origin", "*");
  return alteredResponse;
}

async function handleSubdomain(request, url) {
  const newUrl = "https://api.protonmail.ch" + url.pathname + url.search;
  const protonResponse = await fetch(newUrl, request);
  const alteredResponse = new Response(protonResponse.body, protonResponse);
  alteredResponse.headers.set("Access-Control-Allow-Origin", "*");
  return alteredResponse;
}
```

Replace all references to `example1.com` with your domain and add more domains if needed.

Deploy the worker.

#### Step 3: Configure Worker Routes

In the worker, navigate to **Settings > Domains & Routes**

For each domain, add these routes:

**Route 1:**
- Zone: `example.com`
- Route: `example.com/.well-known/openpgpkey/*`
- Failure mode: fail closed

**Route 2:**
- Zone: `example.com`  
- Route: `openpgpkey.example.com/.well-known/openpgpkey/*`
- Failure mode: fail closed

#### Step 4: Create DNS Records

For each domain, create these DNS records:

| Type | Host | Value | Proxy |
|------|------|-------|--------|
| A | @ | `192.0.2.1` | Enabled |
| CNAME | openpgpkey | @ | Enabled |

**Notes:**
- The A record value can be any IP address, including your actual web host
- `192.0.2.1` is a non-routable IP if you don't host a website
- The proxy setting is required for Cloudflare to intercept and route traffic

#### Step 5: Block Homepage Traffic (Optional)

If you don't host a website at your domain root, create a worker to prevent timeout errors:

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture12.png)

1. Navigate to **Account Home > Compute (Workers) > Workers & Pages**
2. Create a new worker named "Block-homepage-traffic"
3. Deploy and edit with this code:

```javascript
export default {
  async fetch(request) {
    const url = new URL(request.url);
    // Block homepage/root "/"
    if (url.pathname === "/" || url.pathname === "") {
      return new Response('Not Found', { status: 404 });
    }
    // Let everything else proceed as normal
    return await fetch(request);
  }
}
```

4. Add routes for domains without websites:
   - Zone: `example.com`
   - Route: `example.com/*`
   - Failure mode: fail closed

## Testing Your Setup

### Comprehensive Security Analysis
Visit https://www.hardenize.com/ and search your domain. You should see all security features properly configured:

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture13.png)

### WKD Testing
Test Web Key Directory functionality:
- **Open Source**: https://wkd.chimbosonic.com/
- **Proprietary**: https://www.webkeydirectory.com/

Enter an email address with your custom domain:

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture14.png)

![](https://raw.githubusercontent.com/AnalogManDigitalKid/Complete-ProtonMail-Custom-Domain-Security-Setup-with-Cloudflare/refs/heads/main/images/Untitled%20picture15.png)

### DMARC Reports
Monitor your Cloudflare DMARC management dashboard for authentication reports and any potential abuse attempts.

## Troubleshooting

### Common Issues

**MTA-STS not working:**
- Verify the worker route matches exactly: `mta-sts.yourdomain.com/*`
- Check that the AAAA record has the proxy enabled
- Test that `https://mta-sts.yourdomain.com/.well-known/mta-sts.txt` returns content

**WKD not working:**
- Confirm both routes are configured correctly
- Verify DNS records have proxy enabled

**Worker errors:**
- Check browser developer tools for JavaScript errors
- Verify domain names in worker code match your actual domains
- Ensure WAF and Configuration rules are properly set

**DNS propagation:**
- Allow up to 24-48 hours for DNS changes to propagate globally
- Use online DNS checkers to verify records are visible

### Getting Help

If you encounter issues:
1. Check the original sources credited below for community discussions
2. Verify each step was completed exactly as described
3. Test individual components before moving to the next feature
4. Use the testing tools to identify which specific feature isn't working

## Credits

This guide compiles various community contributions and best practices:

- **MTA-STS Worker**: [Tugzrida's Cloudflare Worker script](https://gist.github.com/Tugzrida/61235545dfc122262c69b0ab50265582)
- **WKD Worker**: [Yrlish's ProtonMail WKD implementation](https://www.reddit.com/r/ProtonMail/comments/18r7dey/how_to_use_cloudflare_workers_to_proxy_wkd/) and [accompanying Gist](https://gist.github.com/Yrlish/e69765f5ad245ec0d858158e79e4becf)
- **Configuration**: Best practices from ProtonMail documentation and email security standards

The goal is to provide a comprehensive, single-source tutorial for the ProtonMail community.

---

**Disclaimer**: This tutorial is provided as-is for educational purposes. Always test configurations in a non-production environment first, and ensure you understand the security implications of each feature before deployment.
