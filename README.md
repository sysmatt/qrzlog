# qrzlog

A command-line tool to upload and download ADIF logs via the [QRZ Logbook API](https://www.qrz.com/docs/logbook/QRZLogbookAPI.html).

## Requirements

- Python 3.10+
- No third-party dependencies

## Installation

```
chmod +x qrzlog
cp qrzlog ~/.local/bin/   # or anywhere on your PATH
```

## Configuration

Get your API key from **QRZ.com → Logbook → Settings → API Key**, then save it:

```
qrzlog --save-key YOUR_KEY_HERE
```

This writes `~/.config/qrzlog/qrzlog.ini` (mode 0600). You can also supply the key at runtime:

```
export QRZ_API_KEY=YOUR_KEY_HERE
# or
qrzlog --key YOUR_KEY_HERE --upload contacts.adif
```

Precedence: `--key` flag > `QRZ_API_KEY` environment variable > config file.

## Usage

### Upload

```
qrzlog --upload contacts.adif
```

Inserts all QSOs from the ADIF file into your QRZ logbook.

### Download

```
qrzlog --download mylog.adif
```

Downloads your complete QRZ logbook to an ADIF file. Large logs are fetched in pages of 250 records and merged into a single well-formed ADIF file.

### Dry run

```
qrzlog --upload contacts.adif --dry-run
qrzlog --download mylog.adif --dry-run
```

Shows what would happen without making any API requests.

### Verbosity levels

Three mutually exclusive flags control how much detail is printed to stderr:

```
qrzlog --download mylog.adif --verbose   # page fetches, counts, logid ranges
qrzlog --download mylog.adif --debug     # adds truncated raw API responses
qrzlog --download mylog.adif --trace     # adds full untruncated responses
```

Verbosity can also be set persistently in the config file (see [logging] below).
Command-line flags take precedence over the config file.

### Log file and syslog

Output can be mirrored to a log file or syslog in addition to the console.
Log file and syslog entries include the program name, timestamp, and level prefix.

```
qrzlog --download mylog.adif --verbose --logfile qrzlog.log
qrzlog --download mylog.adif --debug --syslog
```

### Options

| Flag | Description |
|------|-------------|
| `--upload FILE` | Upload an ADIF file to QRZ Logbook |
| `--download FILE` | Download your complete QRZ log to FILE |
| `--save-key API_KEY` | Save API key to config file and exit |
| `--key API_KEY` | API key (overrides config and env var) |
| `--config FILE` | Config file path (default: `~/.config/qrzlog/qrzlog.ini`) |
| `--dry-run` | Show what would happen without making any requests |
| `--verbose` | Show request details and pagination info |
| `--debug` | Show raw API responses (implies --verbose) |
| `--trace` | Show full untruncated responses (implies --debug) |
| `--logfile FILE` | Also write log output to FILE |
| `--syslog` | Also send log output to syslog |

## Config file format

```ini
[qrzlog]
api_key = YOUR_KEY_HERE

[logging]
level   = info      # trace | debug | verbose | info (default: info)
logfile = /path/to/qrzlog.log
syslog  = false
```

The `[logging]` section is optional. `level` sets the default verbosity without needing a command-line flag. Command-line flags (`--verbose`, `--debug`, `--trace`) always take precedence over the config file.
