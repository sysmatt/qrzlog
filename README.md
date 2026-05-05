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

## Default log file

qrzlog automatically looks for a WSJT-X log at startup:

```
~/.local/share/WSJT-X/wsjtx_log.adi
~/.local/share/WSJT-X/wsjtx_log.adif
```

If found, `--upload` can be used without specifying a file. To set a different default:

```
qrzlog --save-log-file /path/to/wsjtx_log.adi
```

This saves `log_file` to the `[qrzlog]` section of your config file. Run `--help` to see whether a default log file was detected.

## Usage

### Upload

```
qrzlog --upload                    # uses configured or auto-detected log file
qrzlog --upload contacts.adif     # explicit file
```

Inserts QSOs from an ADIF file into your QRZ logbook. Each QSO is sent as an individual INSERT request (the QRZ API processes one record per call).

Before uploading, a summary of the ADIF file is printed:

```
File:      /home/user/.local/share/WSJT-X/wsjtx_log.adi
Size:      142.3 KB  (145,718 bytes)
Modified:  2026-05-05 14:32:18
Records:   1,234
Program:   WSJT-X  2.7.1
ADIF Ver:  3.1.4
Created:   20260505 143218
```

Header fields (Program, ADIF Ver, Created) are only shown when present in the file. When `--tail` or `--tail-days` filters are active, the Records line shows both the total and the filtered count:

```
Records:   1,234 total  ->  50 after filters
```

Each QSO is displayed in a formatted table as it is uploaded, with a status column appended once the API responds:

```
CALL        DATE        TIME   BAND   MODE    SENT   RCVD   GRID    STATUS
----------  ----------  -----  -----  ------  -----  -----  ------  ---------
W1AW        2026-05-05  14:32  20m    FT8     -10    -12    FN31    ADDED
K1TTT       2026-05-04  22:15  40m    FT8     -05    -08    FN32    Duplicate
VK2XYZ      2026-05-03  08:47  15m    FT8     +02    -03    QF56    FAILED
  -> Logbook full
```

Status values:

| Status | Meaning |
|--------|---------|
| `ADDED` | QSO accepted and inserted |
| `Duplicate` | QRZ recognised it as already logged; skipped |
| `FAILED` | API rejected the record; reason printed on the next line |
| `ERROR` | Unexpected API response; raw response snippet printed on the next line |

The upload always continues past individual failures. A final summary breaks down uploaded, duplicate, and error counts.

To upload only a recent subset of a large log file:

```
qrzlog --upload contacts.adif --tail 100         # last 100 QSOs
qrzlog --upload contacts.adif --tail-days 7      # QSOs from the last 7 days
```

`--tail` and `--tail-days` are mutually exclusive. Records with no parseable `QSO_DATE` field are always kept when `--tail-days` is used (a warning is printed).

Use `--dry-run` to preview the file summary and QSO table without making any API requests:

```
qrzlog --upload contacts.adif --tail-days 30 --dry-run
qrzlog --upload contacts.adif --tail-days 30 --dry-run --trace   # also dumps raw ADIF
```

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

Shows what would happen without making any API requests. For uploads, the full file summary and QSO table are still printed so you can verify the records before committing. The STATUS column shows `-` since no API calls are made.

```
CALL        DATE        TIME   BAND   MODE    SENT   RCVD   GRID    STATUS
----------  ----------  -----  -----  ------  -----  -----  ------  ---------
W1AW        2026-05-05  14:32  20m    FT8     -10    -12    FN31    -
K1TTT       2026-05-04  22:15  40m    FT8     -05    -08    FN32    -
```

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
| `--upload [FILE]` | Upload an ADIF file to QRZ Logbook (FILE optional if default configured/detected) |
| `--download FILE` | Download your complete QRZ log to FILE |
| `--save-key API_KEY` | Save API key to config file and exit |
| `--save-log-file FILE` | Save default ADIF log path to config file and exit |
| `--key API_KEY` | API key (overrides config and env var) |
| `--config FILE` | Config file path (default: `~/.config/qrzlog/qrzlog.ini`) |
| `--dry-run` | Show what would happen without making any requests |
| `--verbose` | Show request details and pagination info |
| `--debug` | Show raw API responses (implies --verbose) |
| `--trace` | Show full untruncated responses (implies --debug) |
| `--logfile FILE` | Also write log output to FILE |
| `--syslog` | Also send log output to syslog |
| `--tail N` | Upload only the last N QSOs (upload only) |
| `--tail-days N` | Upload only QSOs from the last N days (upload only) |

## Config file format

```ini
[qrzlog]
api_key  = YOUR_KEY_HERE
log_file = /path/to/wsjtx_log.adi

[logging]
level   = info      # trace | debug | verbose | info (default: info)
logfile = /path/to/qrzlog.log
syslog  = false
```

The `[logging]` section is optional. `level` sets the default verbosity without needing a command-line flag. Command-line flags (`--verbose`, `--debug`, `--trace`) always take precedence over the config file.

`log_file` in `[qrzlog]` sets the default ADIF file used by `--upload` when no FILE argument is given. It takes precedence over the WSJT-X auto-detection paths.
