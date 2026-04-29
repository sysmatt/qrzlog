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

### Options

| Flag | Description |
|------|-------------|
| `--upload FILE` | Upload an ADIF file to QRZ Logbook |
| `--download FILE` | Download your complete QRZ log to FILE |
| `--save-key API_KEY` | Save API key to config file and exit |
| `--key API_KEY` | API key (overrides config and env var) |
| `--config FILE` | Config file path (default: `~/.config/qrzlog/qrzlog.ini`) |
| `--dry-run` | Show what would happen without making any requests |

## Config file format

```ini
[qrzlog]
api_key = YOUR_KEY_HERE
```
