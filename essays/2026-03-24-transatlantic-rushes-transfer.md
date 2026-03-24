# Transferring 10TB of Film Rushes Across the Atlantic with Claude Code

*by Ismael Joffroy Chandoutis -- March 2026*

---

## The Problem

You are a filmmaker. You are shooting in Jacksonville, Florida. Your NAS is in Paris, behind an institutional network. Between the two: 116ms of latency, a shared connection, and aggressive UDP throttling.

The upload link measures 376 Mbps. You should be able to push roughly 47 MiB/s. Instead, you are getting 1 MiB/s. That is 0.3% utilization. At that rate, transferring 10 TB of rushes takes 115 days. The shoot ends in three weeks.

This is the kind of problem Claude Code solves well: not creative, not conceptual, but infrastructural. A multi-variable optimization across protocols, network conditions, and cost constraints. The kind of thing where a filmmaker should not have to become a network engineer, but the network engineer needs to understand the filmmaker's constraints.

---

## What We Tried

Every test ran on the same link: MacBook Air M3 in Jacksonville to Mac Mini M4 in Paris, connected via Tailscale mesh VPN. The test file was a 1.2 GB ProRes 4444 clip, representative of the actual rushes.

### SMB over Tailscale VPN

```
Protocol: SMB3 (macOS native Finder transfer)
Result:   1.26 MiB/s
```

SMB is a chatty protocol. Every file operation requires multiple round trips: negotiate, authenticate, open, read, acknowledge, close. At 116ms RTT, each round trip is a quarter-second tax. For large sequential files this is bad. For a folder of thousands of small sidecar files (.xmp, .json, .srt), it is catastrophic.

SMB was designed for local networks with sub-millisecond latency. Over a transatlantic link, it collapses.

### SFTP Single Stream

```
Protocol: SFTP (OpenSSH, default settings)
Result:   4.19 MiB/s
```

Better. SFTP has less protocol overhead than SMB. But a single TCP stream cannot fill a high-bandwidth, high-latency pipe. The bandwidth-delay product (BDP) explains why:

```
BDP = bandwidth x RTT
    = 376 Mbps x 0.116s
    = 43.6 Mbit
    = 5.45 MB
```

A single TCP connection needs a window size of at least 5.45 MB to fully utilize the link. Default SSH buffer sizes are far smaller. The pipe is never full.

### SFTP Multi-Stream (8 threads)

```
Protocol: SFTP via lftp (8 parallel streams, 1MB buffer)
Command:  lftp -e "mirror --parallel=8 -R /source sftp://user@host/dest"
Result:   16.4 MiB/s
```

The breakthrough. Eight parallel streams, each filling its own TCP window. Combined throughput: 16.4 MiB/s, a 13x improvement over single-stream SFTP and a 23x improvement over SMB.

At this speed, 87 GB of rushes transfers in about 90 minutes. Viable for a daily dump. Not viable for 10 TB.

### rsync with Optimized Cipher

```
Protocol: rsync over SSH (aes128-gcm cipher, no compression)
Command:  rsync -avz --progress -e "ssh -c aes128-gcm@openssh.com" /source user@host:/dest
Result:   7.85 MiB/s
```

rsync's delta-transfer algorithm does not help with fresh rushes (no existing version on the destination). The cipher optimization matters: default chacha20-poly1305 is slower than AES-GCM on Apple Silicon with hardware AES. But rsync is still single-stream at its core.

### Multiple Parallel SMB Transfers

```
Protocol: SMB3 (4 Finder windows copying simultaneously)
Result:   ~7 MiB/s combined
```

A brute-force attempt. Four simultaneous Finder copies to the same SMB share. The combined throughput improved but remained bottlenecked by SMB's per-operation latency. Each stream individually degraded under contention.

### WireGuard Tunnel via Relay

```
Protocol: WireGuard tunnel through a VPS relay (Paris)
Result:   ~10 MiB/s
```

A relay server in Paris running WireGuard, forwarding traffic to the NAS. This reduced effective latency for the final hop (VPS to NAS: <5ms) but required server-side setup and introduced a new single point of failure. The VPS upstream bandwidth became the bottleneck.

---

## The Solution: Cloudflare R2 as a CDN Relay

The insight: stop trying to push data directly from Florida to Paris. Instead, use a cloud storage bucket as a relay point. Upload from Jacksonville to the nearest Cloudflare edge (Miami, ~15ms). Download from Paris to the nearest edge (~10ms). The transatlantic hop happens inside Cloudflare's backbone, not over your consumer link.

```
Jacksonville ──(15ms)──> Cloudflare R2 (Miami edge)
                              │
                    [Cloudflare backbone]
                              │
Paris <──(10ms)──── Cloudflare R2 (Paris edge)
```

### Why R2

The critical advantage of Cloudflare R2 over AWS S3 or Google Cloud Storage: **zero egress fees**. S3 charges $0.09/GB for data leaving the bucket. For 10 TB, that is $900 just to download your own files. R2 charges nothing.

| | Upload | Storage | Egress | 10 TB total cost |
|---|---|---|---|---|
| Cloudflare R2 | Free | $0.015/GB/mo | Free | ~$150/mo storage |
| AWS S3 | Free | $0.023/GB/mo | $0.09/GB | ~$1,130 |
| Google Cloud | Free | $0.020/GB/mo | $0.12/GB | ~$1,400 |

For a filmmaker transferring large files once (not serving them repeatedly), R2's pricing model is transformative.

### Measured Performance

```
Upload (Jacksonville → R2):  ~50 MiB/s (saturating the upload link)
Download (R2 → Paris):        91 MiB/s (near line speed)
```

An 87 GB batch of rushes: 30 minutes up, 16 minutes down. Under an hour total, versus 90 minutes point-to-point with the best direct method. For 1 TB: roughly 6 hours. For 10 TB: roughly 2.5 days.

---

## Setup

### 1. Create a Cloudflare Account and Enable R2

Sign up at [cloudflare.com](https://dash.cloudflare.com). Navigate to R2 Object Storage in the sidebar. Enable it (requires a payment method, but the free tier includes 10 GB storage and 10 million requests/month).

### 2. Create a Bucket

```
Bucket name: rushes-2026
Location hint: North America (automatic, but the hint helps)
```

### 3. Create an API Token

In R2 settings, create an API token with read/write access to your bucket. You will get:

- An **Access Key ID**
- A **Secret Access Key**
- An **endpoint URL** in the form `https://<ACCOUNT_ID>.r2.cloudflarestorage.com`

### 4. Install and Configure rclone

[rclone](https://rclone.org) is the tool. It speaks S3-compatible protocols, handles retries, parallel transfers, and chunked uploads natively.

```bash
# macOS
brew install rclone

# Linux
curl https://rclone.org/install.sh | sudo bash
```

Configure the R2 remote:

```bash
rclone config
```

Or edit `~/.config/rclone/rclone.conf` directly:

```ini
[r2-rushes]
type = s3
provider = Cloudflare
access_key_id = <YOUR_ACCESS_KEY_ID>
secret_access_key = <YOUR_SECRET_ACCESS_KEY>
endpoint = https://<ACCOUNT_ID>.r2.cloudflarestorage.com
acl = private
```

### 5. Upload from Source

```bash
# Upload a directory of rushes
rclone copy /Volumes/NavTGV1/rushes_2026/ r2-rushes:rushes-2026/batch-01/ \
  --transfers 16 \
  --checkers 8 \
  --s3-chunk-size 64M \
  --s3-upload-concurrency 8 \
  --progress \
  --log-file /tmp/rclone-upload.log \
  --log-level INFO
```

Key flags:

- `--transfers 16`: 16 parallel file transfers. Each opens its own connection to the CDN edge.
- `--s3-chunk-size 64M`: large chunks reduce per-request overhead. For ProRes files (1-4 GB each), this means fewer round trips.
- `--s3-upload-concurrency 8`: multipart upload parallelism within each file. A 2 GB file splits into 32 chunks uploaded 8 at a time.
- `--progress`: live speed and ETA in the terminal.

For resuming interrupted transfers (power outage, network drop):

```bash
# Same command. rclone checks existing files by size + hash and skips them.
rclone copy /Volumes/NavTGV1/rushes_2026/ r2-rushes:rushes-2026/batch-01/ \
  --transfers 16 \
  --checkers 8 \
  --s3-chunk-size 64M \
  --s3-upload-concurrency 8 \
  --progress
```

### 6. Download from Destination

On the Paris machine:

```bash
rclone copy r2-rushes:rushes-2026/batch-01/ /Volumes/NAS/rushes_2026/ \
  --transfers 16 \
  --checkers 8 \
  --s3-chunk-size 64M \
  --progress \
  --log-file /tmp/rclone-download.log \
  --log-level INFO
```

### 7. Verify Integrity

```bash
# Compare source and destination by hash
rclone check /Volumes/NavTGV1/rushes_2026/ r2-rushes:rushes-2026/batch-01/ \
  --one-way \
  --log-file /tmp/rclone-check.log
```

### 8. Clean Up the Bucket

Once verified on both ends:

```bash
rclone delete r2-rushes:rushes-2026/batch-01/
```

Do not leave 10 TB sitting in R2. Storage is $0.015/GB/month. That is $150/month if you forget.

---

## Optional: Generate a Public Download Link

If your Paris-side operator needs to download from a browser (no rclone installed), you can enable public access on the bucket and generate a static index.

In the Cloudflare dashboard:

1. Go to R2 > your bucket > Settings
2. Enable **R2.dev subdomain** (gives you a `https://pub-<hash>.r2.dev` URL)
3. Or connect a custom domain under your Cloudflare zone

The URL is public but unguessable (long hash). For sensitive rushes, use pre-signed URLs instead:

```bash
# Generate a pre-signed URL valid for 24 hours
rclone link r2-rushes:rushes-2026/batch-01/A001_C003.mov --expire 24h
```

---

## Key Insights

### SMB Is Terrible Over WAN

This is well-known in IT but not in film production, where Finder-to-NAS is the default workflow. SMB was designed for offices where the server is in the next room. At 116ms latency, every protocol handshake becomes visible as delay. If you are using SMB over a VPN for remote rushes transfer, you are leaving 95%+ of your bandwidth unused.

### The Bandwidth-Delay Product Explains Everything

A network link has two properties: bandwidth (how wide the pipe is) and latency (how long it takes a packet to traverse it). The product of these two numbers tells you how much data must be "in flight" simultaneously to fill the pipe. For a transatlantic link, that number is large. A single TCP stream with default buffer sizes cannot fill it. You need parallelism: multiple streams, multiple connections, or a protocol designed for high-BDP links.

### SFTP Multi-Stream Is the Best Point-to-Point Solution

If you need direct machine-to-machine transfer with no cloud intermediary, SFTP with 8+ parallel streams (via `lftp`, `parallel-ssh`, or rclone's SFTP backend) is the best option tested. It is simple, encrypted, resumable, and requires no server-side setup beyond standard SSH.

### Cloud Relay Beats Direct Transfer When Upload Is Fast

The relay approach (upload to CDN, download from CDN) wins when your upload bandwidth to the nearest CDN edge exceeds your effective throughput to the final destination. In our case: 50 MiB/s to R2 vs 16.4 MiB/s direct. The CDN's internal backbone handles the transatlantic hop far more efficiently than a consumer internet path.

This generalizes. Anytime you are transferring large files between two consumer networks separated by high latency, a cloud relay in between will likely outperform direct transfer.

### For Film Production: R2 + rclone vs Dedicated Services

Professional film transfer services exist. Here is how they compare:

| Service | Pricing | Speed | Setup |
|---|---|---|---|
| **Cloudflare R2 + rclone** | ~$0.015/GB storage, zero egress | ~50 MiB/s up, ~91 MiB/s down | 15 min config |
| **MASV** | $0.25/GB | ~50 MiB/s (UDP accelerated) | Instant (desktop app) |
| **Aspera (IBM)** | Enterprise pricing (~$0.10/GB) | Up to 100 MiB/s (FASP protocol) | IT department required |
| **Mega** | $30/mo for 16TB | Variable, often throttled | Browser or MEGAcmd |
| **Google Drive** | $10/mo for 2TB | ~20 MiB/s | Browser or rclone |
| **BitTorrent / Resilio Sync** | Free | Depends on seeder bandwidth | Port forwarding required |
| **Physical drive (FedEx)** | ~$50 shipping | Infinite bandwidth, 24-48h latency | Sneakernet |

For a one-time 87 GB transfer, MASV costs $21.75 and works immediately. For 1 TB, MASV costs $250. For 10 TB, MASV costs $2,500. R2 costs $150 in storage (deleted after download: $0 ongoing) and zero in egress.

The crossover point is around 100 GB. Below that, MASV's convenience wins. Above that, R2 + rclone pays for itself immediately.

For recurring large transfers (daily rushes over a multi-week shoot), R2 is not even close to a fair comparison. It is an order of magnitude cheaper.

---

## The Claude Code Part

Everything described above was researched, benchmarked, and configured within a single Claude Code session. The workflow:

1. Described the problem in natural language: "I'm in Florida, my NAS is in Paris, SMB is giving me 1 MiB/s, I need to transfer 87 GB of rushes."
2. Claude Code ran the network diagnostics: `iperf3`, `speedtest-cli`, latency measurements via `mtr`.
3. Claude Code benchmarked each protocol sequentially, measuring throughput with `pv` and `time`.
4. After identifying R2 as the solution, Claude Code configured rclone, created the bucket via the Cloudflare API, and ran the first transfer.
5. On the Paris side (Mac Mini via SSH over Tailscale), Claude Code ran the download and verified integrity.

Total time from "I have a problem" to "files verified in Paris": under two hours. Most of that was the actual transfer. The research, benchmarking, and configuration took about twenty minutes.

This is what Claude Code does for a filmmaker. Not the creative work. The infrastructure that makes the creative work possible. The kind of problem that would otherwise require hiring a systems administrator or spending a full day reading Stack Overflow threads about TCP window scaling.

The rushes are in Paris. The edit can begin.

---

*Ismael Joffroy Chandoutis is a Paris-based artist and filmmaker working at the intersection of cinema, contemporary art, and artificial intelligence.*
