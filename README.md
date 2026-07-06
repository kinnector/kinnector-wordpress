# Kinnector WordPress Helper (wpwarden)

`wpwarden` is a WordPress security plugin designed to protect sites at both the application level and the host level. It can operate as a standalone security plugin or fully integrated with the host's server-side EDR daemon, `kinnector-warden` (Warden).

While the plugin provides baseline SQLi and CMD-i vetting in standalone mode, connecting it to the Warden daemon enables complete host-level protection:

* **System Call Interception**: Hooks system calls to detect and prevent Remote Code Execution (RCE) in real-time.
* **Process & Subprocess Monitoring**: Automatically tracks and stops unauthorized child processes spawned from the web server context.
* **Low-Latency Vetting**: Offloads heavy HTTP request parsing to Warden's zero-allocation C++ engine (vetting takes less than 50 microseconds), preventing PHP bottlenecking.

## Features

* **Early-Stage Interception**: Intercepts and parses incoming parameters (`$_GET`, `$_POST`, `$_COOKIE`, and headers) early in the WordPress lifecycle (`plugins_loaded`).
* **SQL Injection Prevention**: Wraps the core `wpdb` database class to inspect and vet raw queries prior to execution.
* **Automated Bootstrapping**: Automatically detects the local Warden socket or assists in installing it.

## Warden Daemon Bootstrapping

On activation, `wpwarden` checks for the Unix socket at `/var/run/kinnector/warden.sock`. If unavailable, it falls back to standalone mode and attempts recovery:

### Automatic Installation
If the PHP process has write access to system bin directories (e.g., in developer/Docker environments), the plugin will install the pre-compiled `wardend` binary to `/usr/local/bin/wardend` and configure systemd.

### Manual Installation
If system-level permissions are restricted, the plugin displays an administrative dashboard notice with instructions to run the host installer:

```bash
curl -sSL https://raw.githubusercontent.com/kinnector/kinnector-installer/main/install-warden.sh | sudo bash
```

## Directory Structure

* `kinnector-wp-helper.php`: Main plugin entry point and hook registrations.
* `src/class-wpwarden-admin.php`: Admin settings panels and system setup alerts.
* `src/class-wpwarden-client.php`: Socket and HTTP client communication layer for `/var/run/kinnector/warden.sock`.
* `src/class-wpwarden-db.php`: Custom `wpdb` wrapper class for query analysis.