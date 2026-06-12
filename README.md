```markdown
# ⬇️ File Download & Release Workflow

A high-performance GitHub Actions workflow designed to download large files from any direct link using `aria2c`, perform optional integrity verification, apply smart compression, and automatically handle GitHub's release size limits via splitting.

---

## 🚀 Key Features

* **Accelerated Downloads:** Uses `aria2c` for multi-connection, parallel, and resumable downloading.
* **Smart Content Detection:** Auto-resolves filenames via HTTP headers (`Content-Disposition`) or target URLs.
* **Intelligent Compression:** Detects the file type via MIME checking; skips compression if the file is already a known archive type (ZIP, GZIP, XZ, ZSTD, etc.) to save time and compute power.
* **Automatic Splitting:** Automatically splits artifacts into numbered chunks if they exceed GitHub's single-file limits.
* **Integrity & Security:** Supports on-the-fly SHA256 checksum verification during download and generates a global `SHA256SUMS.txt` for the final release assets.

---

## ⚙️ Workflow Inputs

When triggering the workflow via **Workflow Dispatch**, you can configure the following parameters:

| Input | Description | Default | Required |
| :--- | :--- | :--- | :--- |
| `url` | 🔗 Direct download link of the target file. | *None* | **Yes** |
| `filename` | 📁 Override output filename (leave blank for auto-detection). | *Auto* | No |
| `expected_checksum` | 🔐 Expected SHA256 to verify integrity after download. | *None* | No |
| `connections` | ⚡ Maximum parallel download connections per server. | `8` | No |
| `compress_format` | 🗜️ Target compression format (`zst`, `tar.gz`, `zip`, `none`). | `zst` | No |
| `split_size` | ✂️ Max chunk threshold (e.g., `1900M`). Leave blank to disable. | `1900M` | No |
| `extra_header` | 🔑 Custom HTTP headers for authentication (e.g., `Authorization: Bearer TOKEN`). | *None* | No |
| `prerelease` | 🚧 Mark the generated GitHub release as a pre-release. | `false` | No |

---

## 🛠️ Usage Examples

### Standard Usage (Auto-detect & Max Compress)
Downloads the file, auto-detects its name, verifies nothing is already compressed, and packs it as `.zst`.
```yaml
url: [https://example.com/software-disk-image.img](https://example.com/software-disk-image.img)

```
### Download & Authenticate
Downloads from a protected server or private API endpoint using custom headers.
```yaml
url:            [https://api.example.com/v1/assets/package.tar](https://api.example.com/v1/assets/package.tar)
compress_format: none
extra_header:    Authorization: Bearer ghp_yourSecretTokenHere

```
### Secure Large Download with Splitting
Downloads a huge database backup, verifies its integrity against a known hash, and cuts it into 1.9GB chunks if it exceeds the limit.
```yaml
url:               [https://backups.internal.net/db-dump-2026.sql](https://backups.internal.net/db-dump-2026.sql)
expected_checksum: a1b2c3d4e5f6...your_sha256_hash...7h8i9j0k
split_size:        1900M

```
## 📦 Release Outputs & Restoration
The generated release will be marked automatically as the **Latest Release** and will contain:
 1. The downloaded file (or its compressed equivalent).
 2. Numbered chunks (.part001, .part002, etc.) if splitting was triggered.
 3. SHA256SUMS.txt containing hashes for all uploaded assets.
### 1. Verify Asset Integrity
After downloading the release assets to your local machine, always verify they aren't corrupted:
```bash
sha256sum -c SHA256SUMS.txt

```
### 2. Reassemble Split Parts (If applicable)
If the file was split during the workflow run, merge the chunks back together into the original file before usage:
```bash
# Combine parts
cat filename.zst.part* > filename.zst

# Re-verify the merged file integrity
sha256sum -c SHA256SUMS.txt

```
## 📄 License
This project is licensed under the MIT License — feel free to use and modify.
```

```
