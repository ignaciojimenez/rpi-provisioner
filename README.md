# Raspberry Pi Provisioner

Flash and configure Raspberry Pi SD cards in one command. Download the latest OS, set up WiFi, install SSH keys, and configure system settings automatically.

## Usage

```bash
./provision_pi <image> <hostname> [options]

# Create your config file (optional - for defaults and presets)
cp hosts.yml.sample hosts.yml
# Edit with your WiFi credentials, GitHub username, etc.
```

**Common Options:**
- `--wifi-ssid <ssid>` and `--wifi-pass <password>` - WiFi credentials
- `--github <username>` - Install SSH keys from GitHub
- `--user <name>` - Primary user (default: `choco`)
- `--timezone <tz>` - System timezone (default: `Europe/Madrid`)
- `--no-wifi` - Disable WiFi/Bluetooth (for wired-only Pis)
- `--skip-flash` - Only configure an already-flashed SD card

**Image Options:** `raspbian` (64-bit), `raspbian32` (32-bit), `raspbian-full` (desktop), or custom URL

**Headless Optimizations:** Lite images automatically get performance optimizations (reduced GPU memory, disabled HDMI/audio, faster boot). Desktop images keep default settings.

**Note:** Config file is gitignored. The main dependency is `wpa_passphrase` which is typically in `wpa_supplicant` or `wpasupplicant` package. On macOS, you may need: `sudo port install wpa_passphrase`

### Examples

```bash
# Basic setup with WiFi and SSH keys
./provision_pi raspbian mypi --wifi-ssid "Home" --wifi-pass "secret" --github myusername

# Wired-only Pi (no WiFi)
./provision_pi raspbian mypi --no-wifi --github myusername

# Use a preset from hosts.yml (automatically applies settings)
./provision_pi raspbian mypi
```

## Configuration: two files

`hosts.yml` is **committed** — it holds the presets, which are worth version
history and are not sensitive. `hosts.secrets.yml` is **gitignored** and holds
the values that would be published if they lived in the tracked file:

```bash
cp hosts.secrets.yml.sample hosts.secrets.yml   # then fill it in
```

Any key in `hosts.secrets.yml` overrides the same key in `hosts.yml`'s
`defaults:` block. The secrets file is optional — without it the tool still runs,
presets and all; you just get no WiFi credentials or default password.

⚠️ **Back up `hosts.secrets.yml` out of band.** It is the one file a fresh laptop
cannot rebuild from the repo.

## Host Presets

Set up default configurations in `hosts.yml`:

```yaml
defaults:
  wifi_ssid: "MyNetwork"
  wifi_pass: "MyPassword"
  github: myusername
  disable_wifi: false

presets:
  pihole:
    disable_wifi: true  # Wired only
```

When you provision with `./provision_pi raspbian mypi`, it automatically uses your defaults plus the preset settings. CLI options always override.

## Troubleshooting

**SD card not detected?** Run `diskutil list` (macOS) or `lsblk` (Linux) to see available disks.

**Can't SSH after boot?** Wait 3-5 minutes for first boot. Check hostname: `ssh user@hostname.local`

**WiFi not working?** Verify SSID/password are correct. Check if Pi has WiFi hardware (some models don't).

**Need help?** Run with `--help` to see all options or check `provision_pi` script comments.
