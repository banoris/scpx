# scpx - parallel file transfer over SSH

Sometimes I need to work on private servers (`10.x` network) that are only reachable through a jump host.
Every so often I need to push a large file, e.g. ~500 MB archive. I can't (or don't want to) install
extra stuff, but since I can already `ssh` into it, `scp` is the natural tool.
Alas, with multiple hops in between, 500 MB transfer can take over 11 minutes!

What if the file were split into parts and sent over multiple SSH sessions at once?

```console
# Basic scp
$ scp testfile-500MB.tar.gz root@10.23.135.2:/tmp/
testfile-500MB.tar.gz        2%   11MB 725.3KB/s   11:30 ETA
Execution time: 11m 13.07s

# 8 parallel transfer (default)
$ scpx testfile-500MB.tar.gz root@10.23.135.2:/tmp/
...
==> splitting testfile-500MB.tar.gz into 8 chunks
Execution time: 3m 24.15s

# 16 parallel transfer
$ scpx testfile-500MB.tar.gz root@10.23.135.2:/tmp/ 16
==> splitting testfile-500MB.tar.gz into 16 chunks
...
Execution time: 1m 41.19s
```

## Installation

Drop the script into a directory on your `PATH` and make it executable:

```bash
curl -fsSL https://raw.githubusercontent.com/banoris/scpx/main/scpx -o ~/.local/bin/scpx
chmod +x ~/.local/bin/scpx

scpx --help
```

## Usage

```text
scpx <local-file> <user@host:/remote/dir/> [streams] [stagger-secs]
```

| Argument       | Description                                  | Default |
| -------------- | -------------------------------------------- | ------- |
| `local-file`   | Local file to transfer.                      | -       |
| `user@host:..` | Remote destination directory, in `scp` form. | -       |
| `streams`      | Number of parallel SSH streams.              | `8`     |
| `stagger-secs` | Delay between launching each stream.         | `0.5`   |

```console
# Default: 8 streams
$ scpx bigfile.tar.gz root@10.23.135.2:/tmp/

# 16 streams
$ scpx bigfile.tar.gz root@10.23.135.2:/tmp/ 16
```

## How it works

`split` divides the file into chunks locally. Each chunk is streamed in parallel to a temporary
directory on the remote, one SSH connection per chunk. The remote reassembles them in order, and a
`sha256sum` on both ends confirms the copy is identical.

Works for both Linux and Windows (via Cygwin Bash) remote host.
