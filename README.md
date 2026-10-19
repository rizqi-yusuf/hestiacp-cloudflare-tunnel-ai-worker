<div align="center">

# 🚀 HestiaCP on Proxmox + Cloudflare Tunnel + AI Worker (Gemini)

**Mini project documentation:** From installing a hosting panel in a Proxmox VM, publishing via a domain without port forwarding, to building a website with a secure AI chat.

![Proxmox](https://img.shields.io/badge/Proxmox-VE-E57000?logo=proxmox&logoColor=white)
![HestiaCP](https://img.shields.io/badge/HestiaCP-Control%20Panel-D6336C)
![Cloudflare](https://img.shields.io/badge/Cloudflare-Tunnel%20%26%20Workers-F38020?logo=cloudflare&logoColor=white)
![Gemini](https://img.shields.io/badge/Google-Gemini%20API-4285F4?logo=googlegemini&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES%20Module-F7DF1E?logo=javascript&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-green)

🇮🇩 [Baca dalam Bahasa Indonesia](README.md)

</div>

---

## 📌 Summary

This repository documents my learning project (as a Computer Network and Telecommunication Engineering student at SMKN 3 Kota Tangerang) while building an end-to-end mini hosting system:

1. **Proxmox VE** runs a Linux VM (IP `192.168.1.233`).
2. **HestiaCP** is installed on the VM as the hosting control panel.
3. **Cloudflare Tunnel** publishes the panel and website via a domain without opening ports on the router.
4. The **Qyv Hermes Agent OS** website is uploaded to `public_html` using **WinSCP (SFTP)**.
5. The **Qyv Agent** AI chat feature calls a **Cloudflare Worker** acting as a proxy to the **Gemini API**, ensuring the API key is never exposed in the browser.

| Domain | Function |
| --- | --- |
| `qyvv.site` | Qyv Hermes Agent OS website |
| `panel.qyvv.site` | HestiaCP Control Panel |
| `api.qyvv.site` | `/chat` endpoint (Cloudflare Worker `qyvv-proxy`) |

## 🏗️ Architecture
```
flowchart LR
    U[Visitor] -->|HTTPS| CF[Cloudflare]
    CF -->|qyvv.site| T[Cloudflare Tunnel Hestia Panel]
    CF -->|panel.qyvv.site| T
    T --> VM[HestiaCP VM 192.168.1.233]
    U -->|POST /chat| W[Worker qyvv-proxy api.qyvv.site]
    W -->|API key from secret| G[Gemini API]
```

Single chat message flow:

1. A visitor types a message into Qyv Agent on `qyvv.site`.
2. The browser sends a `POST https://api.qyvv.site/chat` request containing the conversation history.
3. The Worker checks the origin, validates the message, and then calls Gemini using the API key from secrets.
4. Gemini replies, the Worker forwards `{ "reply": "..." }`, and the browser displays it.

## 🧰 Technologies

| Component | Role |
| --- | --- |
| Proxmox VE | Virtualization, running the server VM |
| HestiaCP | Hosting control panel (web domains, users, files) |
| Cloudflare Zero Trust Tunnel | Service publishing without port forwarding |
| Cloudflare Workers | Serverless AI proxy (JavaScript) |
| Gemini API | Chat AI model (`gemini-3.5-flash-lite`) |
| WinSCP | File upload via SFTP/SSH |
| Tailwind CSS, Phosphor Icons, Google Fonts | Website styling (loaded via CDN) |

## 📁 Repository Structure

```text
.
├── README.md              # Bahasa Indonesia
├── README.en.md           # English
├── LICENSE
├── .gitignore
├── website/
│   └── index.html         # Qyv Hermes Agent OS website (single file)
├── worker/
│   └── index.js           # Cloudflare Worker code (AI proxy to Gemini)
└── docs/
    └── screenshots/       # Practice result evidence
```

## 🛠️ Implementation Steps

### 1. Setting up the VM in Proxmox VE

Create a Linux VM, configure a static local IP (`192.168.1.233`), and ensure the VM has internet access and SSH enabled from the client machine.

### 2. Installing HestiaCP

Log into the VM as root via SSH, then download and run the official installation script:

```bash
wget [https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh](https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh)
bash hst-install.sh
```

Once complete, the panel can be accessed on port `8083`. Accounts are separated between administrator and regular users for web hosting.

## Hestia Dashboard <img width="1050" height="402" alt="image" src="https://github.com/user-attachments/assets/a4130332-0086-49f5-98ca-5fb4efad9d2d" />


### 3. Cloudflare Zero Trust Tunnel

1. Go to **Zero Trust → Networks → Tunnels**, and create a tunnel named `Hestia Panel`.
2. Install the `cloudflared` connector on the VM using the command provided by Cloudflare (keep the tunnel token secret, do not commit it).
3. Add two *published application* routes on the same tunnel:

| Hostname | Target (example) |
| --- | --- |
| `panel.qyvv.site` | `https://localhost:8083` |
| `qyvv.site` | `http://localhost:80` |

4. Ensure the tunnel status is **Healthy**, then test both domains in your browser.

## Tunnel List <img width="1050" height="384" alt="image" src="https://github.com/user-attachments/assets/a20c8b03-049a-4b02-9112-2f78d68fb329" />


## Tunnel Routes <img width="1050" height="520" alt="image" src="https://github.com/user-attachments/assets/9d010707-b128-4d08-885f-e32ae4b86c9f" />


## Hestia Login via Domain <img width="1366" height="715" alt="{726E21B5-C40C-48BB-9425-FF393C196842}" src="https://github.com/user-attachments/assets/a64f5118-77e4-4574-9a48-c89df99dbc05" />


### 4. Web Domain and File Upload via WinSCP

1. In Hestia, open the **Web** menu and add the domain `qyvv.site`.
2. In WinSCP, create an **SFTP** connection to `192.168.1.233` (port 22).
3. Open the `public_html` directory in the right panel, then drag files from `website/` into it.
4. Open `https://qyvv.site` to verify the website displays correctly.

## WinSCP <img width="834" height="266" alt="image" src="https://github.com/user-attachments/assets/aa16b4d2-e0ed-4c53-aa29-8b29d330f34f" />


## Website <img width="1050" height="564" alt="image" src="https://github.com/user-attachments/assets/03782b38-b5de-4522-8331-a8b85b927f22" />


### 5. Cloudflare Worker as AI Proxy

1. In Cloudflare, go to **Workers & Pages**, and create a Worker named `qyvv-proxy`.
2. Paste the contents of [`worker/index.js`](worker/index.js), then deploy.
3. Add the custom domain `api.qyvv.site`.
4. Fill in the following variables and secrets:

| Name | Type | Function |
| --- | --- | --- |
| `ALLOWED_ORIGIN` | Variable | Comma-separated list of allowed origins permitted to call the Worker (e.g., `https://qyvv.site`) |
| `GEMINI_API_KEY` | Secret | Gemini API key, never sent to the browser |
| `GEMINI_MODEL` | Variable | Gemini model name (here `gemini-3.5-flash-lite`) |
| `RATE_LIMITER` | Binding (optional) | Per-IP rate limiter; bypassed if not installed |

## Worker Overview <img width="1050" height="616" alt="image" src="https://github.com/user-attachments/assets/c308203f-9b4d-400c-a71d-62e16bfe30cd" />


## Worker Variables <img width="1050" height="445" alt="image" src="https://github.com/user-attachments/assets/cd449406-7016-46ca-bcd7-d422c1e88ade" />


### 6. Testing

Send a test message through Qyv Agent. The Worker indicator turns green and the AI replies according to its persona.

## Qyv Agent Chat <img width="1366" height="729" alt="{63228ED9-554D-4D66-ADE8-276F8450AA2A}" src="https://github.com/user-attachments/assets/bd95f085-2f97-4a80-be47-cee5a2370b67" />

## 🔌 Worker API

`POST https://api.qyvv.site/chat`

```json
{ "messages": [ { "role": "user", "content": "test" } ] }
```

| Condition | HTTP | Response |
| --- | --- | --- |
| Success | 200 | `{ "reply": "..." }` |
| Not `POST /chat` | 404 | `not_found` |
| Disallowed Origin | 403 | `origin_blocked` |
| Too many requests | 429 | `rate_limited` + `retryAfter` |
| Invalid body or message | 400 | `bad_request` |
| API key or model missing | 500 | `server_misconfigured` |
| Gemini timeout (>20s) | 504 | `upstream_timeout` |
| Gemini failure or empty reply | 502 | `upstream_error` |

Limits used by the Worker: maximum **10 messages** per request, **500 characters** per message, **512 output tokens**, **20 seconds** timeout, temperature **0.9**. Client-provided `system` roles are rejected.

On the website side, history is trimmed to the last 9 messages and 450 characters per message to always stay below Worker limits. If the Worker becomes unreachable, Qyv Agent enters offline mode and answers using built-in fallback material.

## 🔐 Security Notes

- The Gemini API key is stored only as a **secret** in the Worker, not in the HTML file.
- The Worker rejects origins not present in `ALLOWED_ORIGIN`. Since Origin headers can be spoofed by non-browser clients, this should be complemented with rate limiting.
- In this deployment, the `RATE_LIMITER` binding is not yet installed, so per-IP rate limiting is inactive.
- `ALLOWED_ORIGIN` in this deployment also includes `null` (origins from local files or sandboxed iframes). Remove this if unnecessary.
- The Hestia panel is exposed to the internet via `panel.qyvv.site`. It is recommended to enable 2FA in Hestia and consider using Cloudflare Access.
- Do not commit tunnel tokens, API keys, or passwords. Mask emails, tunnel IDs, and IP addresses in screenshots before publishing.

## 📚 Troubleshooting & Lessons Learned

| Issue | Solution / Lesson |
| --- | --- |
| Browser-based upload felt laggy | Switched to WinSCP (SFTP over SSH) |
| Qyv Agent indicator temporarily showed `worker 502` | Code 502 means `upstream_error`: call to Gemini failed or returned an empty reply; it normalized on subsequent tests |
| Text `let _lastAct=Date.now();` appeared outside the page | The line was located after `</html>`; removed from `website/index.html` (the variable is already declared in the main script) |

Main takeaway: A single tunnel can handle multiple hostnames, secrets must stay on the server side (Worker), and restrictions should be applied on both ends (website and Worker).

## 🗺️ Improvement Plan

- [ ] Install rate limiting binding (`RATE_LIMITER`) on the Worker
- [ ] Remove `null` from `ALLOWED_ORIGIN` if unnecessary
- [ ] Finalize `SYSTEM_PROMPT` (remove placeholder lines)
- [ ] Enable Hestia 2FA and consider Cloudflare Access for the panel

## 📄 License

Released under the [MIT](LICENSE) license.

## 👤 Author

* **Name:** [Rizqi Maulana Yusuf]
* **GitHub:** [@rizqi-yusuf](https://github.com/rizqi-yusuf)
