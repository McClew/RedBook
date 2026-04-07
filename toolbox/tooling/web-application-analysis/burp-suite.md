# Burp Suite



{% hint style="info" %}
#### Download & Install

[https://portswigger.net/burp/communitydownload](https://portswigger.net/burp/communitydownload)
{% endhint %}

***

## Tools

### Target

This tab displays the site map of all applications passed through the proxy.

* **Site Map:** Organises content hierarchically. It builds a tree structure of the web application (directories, APIs, assets) using a spider/crawler as you browse.
* **Drill-down:** Clicking a resource in the site map allows you to view the specific **Request** (sent by the client) and **Response** (returned by the server) without needing to re-send traffic.
* **Use Case:** Highly useful for discovering "hidden" backend requests or API calls that occurred in the background while you were browsing the UI.
* **Sub-tabs:**
  * **Scope:** Used to define which domains you want to test.
  * **Issues (Pro Only):** Displays vulnerabilities detected by the automated scanner.

### Proxy

The core of Burp Suite, allowing you to intercept and modify traffic.

* **Intercept:** Stops requests before they go to the server.
* **HTTP History:**
  * **Function:** specific log of every HTTP request proxied through Burp.
  * **Workflow:** We can right-click any request in history and select **"Send to Repeater"** to manipulate it, even if we didn't intercept it originally.
* **WebSockets History:** Similar to HTTP history but for WebSocket connections.
* **Match and Replace:**
  * **Function:** Automatically finds a string in a request/response and replaces it with another.
  * **Example:** Automatically changing the `User-Agent` header from "Firefox" to "iPhone" for every request.

### Intruder

* **Purpose:** An automation tool used to send a large number of customised requests.
* **Use Case:** Brute-forcing login credentials, fuzzing API endpoints, or testing for rate limiting.

#### Attack Types

| **Sniper** (Default) | Takes a single list of payloads. It iterates through the list, testing **one position at a time**. It finishes position 1, then moves to position 2, etc. | Fuzzing multiple fields individually to see if any specific field breaks.                                          |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Battering Ram**    | Takes a single list of payloads. It places the **same payload into all positions simultaneously**.                                                        | Testing if a specific value needs to be identical across multiple fields (e.g., user ID repeated in URL and Body). |
| **Pitchfork**        | Uses multiple payload lists (one for each position). It iterates through them **in parallel** (Payload A1 with Payload B1, then A2 with B2).              | Testing known pairs, like a specific username with its specific password.                                          |
| **Cluster Bomb**     | Uses multiple payload lists. It tests **every possible combination** of all payloads (A1+B1, A1+B2, A2+B1...).                                            | Brute-forcing credentials (trying every password against every username).                                          |

### Repeater

* **Purpose:** A manual testing interface.
* **Function:** Allows us to modify a single request and resend it repeatedly to analyse how the server response changes.

### Collaborator

{% hint style="warning" %}
#### Professional Edition

This tool is only available on **Burp Suite Professional**.
{% endhint %}

* **Purpose:** A tool for detecting out-of-band vulnerabilities (e.g., Server-Side Request Forgery - SSRF).
* **How it works:** It generates a unique, public-facing URL (hosted by PortSwigger). We inject this URL into the target application. If the target server connects to that URL, Collaborator notifies you.
* **Usage:** Useful for detecting when a server is triggered to reach out to external systems.

### Sequencer

* **Purpose:** Analyses the quality of randomness in data tokens.
* **Use Case:** If an application issues a Session ID or Cookie, Sequencer captures thousands of them and performs statistical analysis to see if they are random or predictable. If predictable, an attacker could hijack user sessions.

### Decoder

* **Purpose:** A built-in tool for encoding and decoding data.
* **Functionality:** Similar to **CyberChef**.
* **Supported Formats:** Base64, URL Encoding, HTML Encoding, Hex, etc.
* **Usage:** Quickly decoding a Base64 string found in a cookie or encoding a payload to bypass a filter.

### Comparer

* **Purpose:** Visually compares two HTTP requests or responses.
* **Use Case:**
  * **Access Control:** Compare a response received as an "Admin" vs. a "User" to see exactly what data is different.
  * **Modes:** Can compare by **Words** or **Bytes**.

### Logger

* **Purpose:** Records a complete log of all activity and traffic within Burp.
* **Use Case:**
  * **Debugging:** Tracing what happened if a test fails.
  * **Liability:** Proving that our specific traffic did - or did not - cause a server crash.

### Organizer

* **Organizer:** A tool to store, annotate, and manage specific request messages for later review.
* **Learn:** Provides direct links to the PortSwigger Web Security Academy for tutorials.

***

## Useful Extensions

_Extensions that only work with Burp Suite Professional are marked with_ (Pro)_._

<table data-column-title-hidden data-view="cards"><thead><tr><th>Name</th><th></th><th data-type="content-ref"></th></tr></thead><tbody><tr><td>Param Miner</td><td>This extension identifies hidden, unlinked parameters. It's particularly useful for finding web cache poisoning vulnerabilities.</td><td><a href="https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943">https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943</a></td></tr><tr><td>JS Miner (Pro)</td><td>Scans static JavaScript files for hardcoded secrets, such as AWS keys, API credentials, or PII.</td><td></td></tr><tr><td>JSON Web Tokens</td><td>Tools for decoding, editing, and fuzzing JSON Web Tokens (JWTs), which are often used for session management.</td><td></td></tr><tr><td>Autorize</td><td>Automates Broken Access Control testing. It captures requests as a high-privilege user and automatically replays them as a low-privilege user to see if the server accepts them.</td><td></td></tr><tr><td>Active Scan++ (Pro)</td><td>Extends the default active scanner with additional checks for advanced vulnerabilities.</td><td></td></tr></tbody></table>
