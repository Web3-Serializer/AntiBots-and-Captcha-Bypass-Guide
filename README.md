# Comprehensive Guide: AntiBot, CAPTCHA & CSRF Token Bypass Techniques

This repository collects practical techniques, guides, and code samples to bypass various web protections such as AntiBot systems, CAPTCHAs, and CSRF tokens. It's designed for developers, researchers, and pentesters aiming to automate web access or analyze security mechanisms ethically.

---

## Table of Contents

- [Introduction](#introduction)  
- [Common AntiBot & CAPTCHA Protections](#common-antibot--captcha-protections)  
- [Best Practices](#best-practices)  
  - [Baleen Bypass](#baleen-bypass)  
  - [Cloudflare, Akamai, Amazon WAF Bypass](#cloudflare-akamai-amazon-waf-bypass)  
  - [CSRF Token Bypass](#csrf-token-bypass)  
- [Recommended Libraries](#recommended-libraries)  
- [Disclaimer](#disclaimer)  

---

## Introduction

Anti-bot protections and CAPTCHAs are widely used to block malicious automated access on websites. This guide explains how to analyze and ethically bypass these protections, **only on systems where you have permission**.

---


## Common AntiBot & CAPTCHA Protections

Protection               | Description                                    | Official Website                             
------------------------|------------------------------------------------|----------------------------------------------
Baleen                  | Advanced cookie and session management anti-bot | https://baleen.cloud/          
Cloudflare              | Web security platform with challenge pages       | https://www.cloudflare.com/                   
Akamai                  | CDN and web security with bot detection           | https://www.akamai.com/                        
Amazon WAF              | Web application firewall on AWS                     | https://aws.amazon.com/waf/                    
hCaptcha                | Popular CAPTCHA alternative                         | https://www.hcaptcha.com/                      
reCAPTCHA               | Google's CAPTCHA service                            | https://developers.google.com/recaptcha      

---

### Best Practices

- **Use Mobile or Alternative Browser User-Agents**  
  Many anti-bot systems have lighter protection for mobile browsers or less common desktop browsers. Using user-agent strings from Android, iOS (Safari), Firefox, or Opera can help avoid default detection rules that target Chrome automation.

- **Avoid VPN and Datacenter IPs — Use Residential Proxies**  
  Most bot mitigation platforms use threat intelligence databases to detect IP reputation. VPNs and datacenter proxies are often flagged as high-risk. Instead, opt for residential proxies with low detection scores. These proxies rotate real ISP-assigned IP addresses that mimic human users more reliably.

- **Use `tls-client` for Requests Instead of `requests` or `httpx`**  
  Libraries like [`tls-client`](https://github.com/FlorianREGAZ/Python-Tls-Client) allow your requests to imitate real browsers by customizing JA3 fingerprints and using modern TLS versions. Traditional libraries like `requests` often expose your automation via outdated TLS or HTTP/1.1 fingerprints.

- **Bypass Cloudflare UAM by Finding the Backend IP Address**  
  Cloudflare serves as a reverse proxy that filters traffic before it reaches the origin server. If you can discover the origin server’s real IP (e.g., via DNS leaks or services like [Censys](https://search.censys.io/)), you may be able to bypass Cloudflare entirely and send requests directly to the backend.

- **Use Mobile API Endpoints When Available**  
  Many modern applications use separate infrastructure for mobile apps. These mobile APIs (`/api/v2/mobile/...`, etc.) may lack the full protection stack (e.g., no JavaScript challenges or CAPTCHA), making them easier to automate. Always inspect app traffic using tools like `mitmproxy` or `charlesproxy`.

### Baleen Bypass

Baleen requires you to fetch the initial page, extract a specific cookie token embedded in the HTML, and add it to your session cookies. Example in Python with `tls_client`:

```python
import re
import tls_client

session = tls_client.Session()
base_url = "https://www.cdiscount.com"
response = session.get(base_url)
html = response.text

match = re.search(r'"name":"(visit_baleen_ACM-[^"]+)",.*?"value":"([^"]+)"', html)
if match:
    name, value = match.groups()
    session.cookies.set(name, value, domain=".cdiscount.com", path="/")
    print(f"Cookie set: {name} = {value}")
else:
    print("Baleen token not found.")
```

---

### Cloudflare, Akamai, Amazon WAF Bypass

- **Cloudflare (UAM disabled) and Akamai Passive**: Can often be bypassed with a full TLS handshake client emulation like [tls-client](https://github.com/FlorianREGAZ/Python-Tls-Client).
- **Amazon WAF**: Typically bypassed using browser automation with [undetected-chromedriver](https://github.com/ultrafunkamsterdam/undetected-chromedriver) to interact with challenge elements.

Using forged TLS fingerprints and mobile UserAgents often improves bypass success rates.

---

### CSRF Token Bypass

Many websites protect forms using CSRF tokens. To bypass this protection during automation:

1. **Request the page containing the form**, usually something like `https://example.com/login`.
2. **Parse the page HTML** to extract the CSRF token, usually found in an `<input>` element with attributes like `id="csrf-token"` or `name` containing "csrf".
3. **Send the token back with your POST request** to authenticate or submit the form.

Example in Python using `requests` and `BeautifulSoup`:

```python
import requests
from bs4 import BeautifulSoup

session = requests.Session()
login_url = "https://example.com/login"

# Step 1: Get login page to retrieve CSRF token
response = session.get(login_url)
soup = BeautifulSoup(response.text, 'html.parser')

# Step 2: Extract CSRF token from input element
csrf_input = soup.find('input', attrs={'id': 'csrf-token'}) or              soup.find('input', attrs={'name': lambda x: x and 'csrf' in x.lower()})

csrf_token = csrf_input['value'] if csrf_input else None
print(f"CSRF token found: {csrf_token}")

# Step 3: Use the token in your form submission
payload = {
    'username': 'myusername',
    'password': 'mypassword',
    'csrf-token': csrf_token  # or the correct field name
}

post_response = session.post(login_url, data=payload)
print(post_response.status_code)
```
---

## Recommended Libraries

Protection Library      | Description                           | GitHub Link                                               
------------------------|-------------------------------------|-----------------------------------------------------------
tls-client              | Full TLS client with fingerprinting | https://github.com/FlorianREGAZ/Python-Tls-Client                  
undetected-chromedriver | Stealthy ChromeDriver for Selenium  | https://github.com/ultrafunkamsterdam/undetected-chromedriver      
anticaptchaofficial *[PAID]*     | CAPTCHA solving API client           | https://github.com/Anti-Captcha/anticaptcha-python        

---

## Disclaimer

This repository is for **educational purposes only**. Misusing these techniques may violate laws or terms of service. Always ensure you have explicit permission before testing or bypassing protections on any website.

---

*Contributions welcome! Feel free to add more techniques, libraries, or examples.*
