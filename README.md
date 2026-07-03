# Kinnector WordPress Helper (wpwarden)

`wpwarden` is a WordPress security plugin that can operate fully standalone or integrated with the host's server-side daemon, `kinnector-warden` (aka **Warden**).

While `wpwarden` runs standalone to provide baseline, application-level SQLi/CMD-i vetting, the **Warden daemon is mandatory** to enable advanced protections:
- **Process & Subprocess Monitoring**: Tracking and blocking unauthorized child processes spawned from the web server context.
- **RCE Prevention**: Hooking system calls to intercept and block Remote Code Execution attempts in real-time.
- **Low-Latency Request Vetting**: Offloading HTTP request parsing to Warden's zero-allocation C++ engine ($<50\,\mu\text{s}$) to avoid PHP block delays.

## Features
- **Early Hook Interception**: Intercepts and parses HTTP parameters (`$_GET`, `$_POST`, `$_COOKIE`, and headers) at the `plugins_loaded` hook.
- **SQLi Vetting**: Extends the `wpdb` class to vet query strings prior to execution.
- **Auto-Bootstrapping**: Attempts to automatically discover, install, or suggest the installation of Warden on the server.

## Warden Daemon Bootstrapping

On initialization, the plugin checks for an active Warden socket at `/var/run/kinnector/warden.sock`. If Warden is not found, the plugin falls back to **Standalone Mode** with reduced capabilities.

### Auto-Installation
If the PHP runtime has write access to system directories (e.g. in privileged Docker containers), `wpwarden` will download the `wardend` binary to `/usr/local/bin/wardend` and configure the systemd unit automatically.

### Dashboard Notice (Manual Installation)
If permissions are insufficient, `wpwarden` displays an admin dashboard notice requesting that the host administrator run the bootstrap script:
```bash
curl -sSL https://raw.githubusercontent.com/kinnector/kinnector-installer/main/install-warden.sh | sudo bash
```

## Directory Structure
- `kinnector-wp-helper.php` - Plugin entry point & hook registrations.
- `src/class-wpwarden-admin.php` - Dashboard settings and manual setup notices.
- `src/class-wpwarden-client.php` - Socket socket/HTTP client for `/var/run/kinnector/warden.sock`.
- `src/class-wpwarden-db.php` - Custom `wpdb` wrapper class.