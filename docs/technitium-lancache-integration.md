## Architectural Log: Decoupling LanCache & Technitium DNS (Resolving the API Log Storm)

### 🔴 The Incident & Problem Statement
Our automated gaming domain orchestration script (`push_cache_domains.py`) running on the LanCache VM (`192.168.1.132`) was task-scheduled to inject massive batches of CDN domains into our core Technitium DNS Server (`192.168.1.251`). 

Following upstream application updates, a strict schema mismatch occurred. The script was pushing malformed requests to Technitium's `/api/zones/records/add` endpoint missing mandatory JSON parameters (specifically key string `type` and `ipAddress`), throwing continuous `.NET` stack traces: `DnsServerCore.DnsWebServiceException: Parameter 'type' missing.`

**Impact:**
- **API Bombing:** Unthrottled request loops bombarded the web engine dozens of times per second.
- **Log Explosions:** Flat-text application logs rapidly ballooned to hundreds of megabytes/gigabytes.
- **UI Denial of Service:** When attempting to render system logs via the Web UI, memory exhaustion occurred, triggering a hard browser crash and generic `HTTP 500 Internal Server Error` exceptions.

---

### 🛠️ The Architecture Resolution: Local Flat-File Synchronization Engine
Rather than merely patching the Python dict structure—which preserves a fragile, high-overhead API dependency that risks future throttling or timeouts—we cleanly decoupled the network fabric mapping entirely.
