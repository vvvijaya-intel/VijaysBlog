---
layout: post
title: "How To: Overcome 403 - Public Access is Disabled Error"
date: 2026-06-24
tags: [azure, ai-foundry, networking, private-endpoint]
---

## The Setup

You just spun up an **Azure AI Foundry** workspace, deployed a model (say, `gpt-4.1-mini`), grabbed your endpoint URI and API key from the portal, and wrote a quick test:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://<your-resource>.services.ai.azure.com/openai/v1",
    api_key="<your-key>",
)

response = client.responses.create(
    model="gpt-4.1-mini",
    input="What is the capital of France?",
)
print(response.output[0])
```

You run it — and immediately hit this:

```
PermissionDeniedError: Error code: 403 - {'error': {'code': '403', 'message': 'Public access is disabled. Please configure private endpoint.'}}
```

---

## What's Going On

Your org has an **Azure Policy** enforcing "no public network access" on AI services. The Foundry resource only accepts connections through a **private endpoint** — a private IP inside your corporate network. There's no toggle to turn public access on in the portal because the policy blocks it at the org level.

---

## Step 1 — Find the Private IP

In the Azure portal:

1. Go to your **Resource Group**
2. Click the **Private Endpoint** resource associated with your Foundry instance
3. On the left, click **DNS configuration** or **Network Interface** (under Settings)
4. Note the **Private IP address** (e.g. `10.x.x.x`)

> **Note:** The portal has an option to download a `hostfile.txt`. It may be empty — that's a known gap when the private DNS zone isn't fully wired up. Get the IP manually as above.

---

## Step 2 — Update Your Hosts File

You need to tell your machine to resolve the Azure hostname to the private IP instead of the public one.

1. Open **Notepad as Administrator** (right-click → Run as administrator)
2. Open: `C:\Windows\System32\drivers\etc\hosts`
3. Add a line at the bottom (no `#` — that makes it a comment):

```
10.x.x.x    <your-resource>.services.ai.azure.com
```

4. Save the file, then flush DNS:

```powershell
ipconfig /flushdns
```

5. Verify it worked:

```powershell
Resolve-DnsName <your-resource>.services.ai.azure.com
```

You should see your `10.x.x.x` private IP in the output — not a public Azure IP.

---

## Step 3 — Check for a Corporate Proxy

Still getting 403 after the hosts file update? Your Python environment may be sending requests through a **corporate proxy**, which uses its own DNS (ignoring your local hosts file) and hits the public endpoint.

Check in Python:

```python
import os
print('HTTPS_PROXY:', os.environ.get('HTTPS_PROXY') or os.environ.get('https_proxy', 'not set'))
```

If it returns something like `http://proxy-corp.example.com:912` — that's your problem.

**Fix:** add `NO_PROXY` for the Azure hostname at the top of your script/notebook cell, before the `openai` client is created:

```python
import os

# Bypass corporate proxy for Azure private endpoint
os.environ["NO_PROXY"] = "<your-resource>.services.ai.azure.com"
os.environ["no_proxy"] = "<your-resource>.services.ai.azure.com"

from openai import OpenAI

endpoint = "https://<your-resource>.services.ai.azure.com/openai/v1"
api_key = "<your-key>"

client = OpenAI(base_url=endpoint, api_key=api_key)

response = client.responses.create(
    model="gpt-4.1-mini",
    input="What is the capital of France?",
)
print(response.output[0])
```

Setting `NO_PROXY` tells the `openai` library (and the underlying `httpx`/`requests`) to connect directly to that hostname, bypassing the proxy entirely.

---

## Summary

| Problem | Fix |
|---|---|
| 403 — Public access disabled | Expected. Resource uses private endpoint only. |
| `hostfile.txt` is empty | Get the private IP manually from the portal's Network Interface page. |
| DNS not resolving to private IP | Add entry to `C:\Windows\System32\drivers\etc\hosts`, flush DNS. |
| Still 403 after hosts file update | Corporate proxy is bypassing your hosts file — set `NO_PROXY` in your script. |

Once DNS resolves correctly **and** the proxy is bypassed, the `openai` SDK connects straight to the private endpoint and your call goes through.
