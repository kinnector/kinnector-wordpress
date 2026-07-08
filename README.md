# Kinnector WordPress Helper (wpwarden)

`wpwarden` is the official WordPress integration plugin for the Kinnector Warden server-side EDR security suite. It intercepts malicious HTTP requests and database queries before they execute.

---

## What it protects

WordPress sites are constant targets for arbitrary file uploads, SQL injection (SQLi), and remote code execution (RCE) via themes and plugins. 

`wpwarden` protects the entire WordPress runtime. It intercepts input buffers (`$_GET`, `$_POST`, `$_COOKIE`, and headers) and database queries at the PHP level, offloading validation to the local security daemon to block attacks before WordPress processes them.

---

## Why traditional WordPress plugins are insufficient

Traditional security plugins are written entirely in PHP. If an attacker successfully uploads a malicious shell or bypasses the PHP entrypoint, the plugin's hooks are bypassed completely. Additionally, parsing large HTTP payloads in PHP introduces heavy CPU overhead, bottlenecking high-traffic sites.

`wpwarden` solves this. In standalone mode, it provides baseline vetting. When integrated with the host EDR daemon, it offloads heavy payload validation to the local Warden socket, reducing response overhead to less than 50 microseconds. If an exploit runs, the server daemon blocks it at the OS kernel level, terminating unauthorized subprocesses.

---

## Core Capabilities

* **Early-Stage Lifecycle Interception**: Hooks early in the boot sequence (`plugins_loaded`) to filter raw input values before themes or core plugins load.
* **Database Query Wrapper**: Wraps the core `wpdb` class, analyzing raw SQL statements prior to driver execution to block SQLi attempts.
* **Automated Daemon Bootstrapping**: Detects the local Unix domain socket and helps administrators install the server-level daemon if missing.

---

## Technical Integration Flow

```
[ HTTP Request ] ──> [ wpwarden PHP Hook ] ──(Unix Socket)──> [ wardend Daemon ]
                                                                   │
                                                      ALLOWED? ────┼─── YES ──> [ Run WordPress / DB ]
                                                                   └─── NO  ──> [ Abort (403 Forbidden) ]
```

---

## Warden Daemon Bootstrapping

`wpwarden` checks for the active daemon socket at `/var/run/kinnector/warden.sock`. If unavailable, it falls back to standalone PHP-only protection and presents setup options:

### 1. Automatic Installation
If the PHP process has root/sudo write permissions (e.g. inside Docker containers), the plugin will install the pre-compiled `wardend` binary to `/usr/local/bin/wardend` and configure the systemd service automatically.

### 2. Manual Installation
For environments with restricted PHP permissions, the plugin displays a dashboard notice with host installation instructions:

```bash
curl -sSL https://raw.githubusercontent.com/kinnector/kinnector-installer/main/install-warden.sh | sudo bash
```

---

## File Structure

* `kinnector-wp-helper.php`: Core plugin loader and hook registrations.
* `src/class-wpwarden-db.php`: Custom `wpdb` database wrapper class for query vetting.
* `src/class-wpwarden-client.php`: Socket and HTTP communicator for `/var/run/kinnector/warden.sock`.
* `src/class-wpwarden-admin.php`: Administrative dashboard settings and setup alerts.