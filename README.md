# Arker Archive (April 8, 2026)
>[!CAUTION]
>
> **TECHNICAL AUDIT NOTICE**
>
>This repository is a static archive preserved for forensic auditing and youth safeguarding oversight. The source code contained herein implements automated snapshotting, permanent archival of student repositories, and the "forced persistence" of digital footprints belonging to minors.
>
>All technical liability, privacy inquiries, or complaints regarding the original source code, its intended functions, or its deployment by the Hack Foundation must be directed exclusively to the original project maintainers via the [official Hack Club GitHub organization](https://github.com/orgs/hackclub/people) or their public contact channels on the [Hack Club website](https://hackclub.com). The owner of this forensic mirror provides no contact relay for the original authors.

>[!IMPORTANT]
>
> **LEGAL NOTICE & ARCHIVE PROTOCOL**
>
>* No Support of Content: The archiver does not support, endorse, or condone any viewpoints, ideologies, or organizational practices expressed, implied, or implemented within this repository.
>
>* No Warranties: This archive is provided "as-is" for transparency and forensic purposes; no warranties of any kind are provided regarding the functionality, safety, or legality of the contained code.
>
>* Non-Production Warning: The archiver strictly does not condone the use of this software in a production environment and/or with real-world user data. This code implements data persistence mechanisms that may bypass a user's "Right to be Forgotten" and violate international privacy standards.
>
>* Purpose of Archive: This repository is maintained strictly for security and privacy audits, legal documentation, historical archival, and forensic research using simulated/synthetic data. It serves as a public interest case study in the degradation of nonprofit transparency and as a technical reference for the infrastructure used in centralized youth surveillance.
>
>* Non-Affiliation: The archiver (the owner of this fork) is not a contributor to this project, has no affiliation with the original authors (including but not limited to Hack Club or the Hack Foundation), and assumes no responsibility for the function or deployment of this code.
>
>* Trademark Notice: All third-party trademarks mentioned (including but not limited to Hack Club, The Hack Foundation, GitHub, Google, itch.io, YouTube, and AWS) are the property of their respective owners. The archiver claims no ownership rights over any third-party trademarks mentioned herein.

## Audit Summary

This fork preserves the repository state as of April 8, 2026. This snapshot was captured to document the storage and archival backend utilized by the Orpheus Engine data pipeline. This archive ensures a permanent public record of the technical mechanisms used to maintain permanent "shadow copies" of student projects.

Documented Functional Constraints
The following operations are hard-coded into this suite and are utilized by the Orpheus Engine to maintain data persistence:

* Capable of creating "Chrome snapshots" and "git clones" of student projects. When triggered by the Orpheus Engine, this allows the organization to maintain a permanent copy of a student's work even if the student later deletes the original or makes it private.

* Operates as a "forced memory" layer, effectively removing a minor's ability to manage their digital footprint or exercise a "right to be forgotten" once a project has been submitted for review.

* Utilizes zstd compression and S3-compatible cloud storage to facilitate the long-term, high-volume retention of student-generated content.

---

## Original Project Documentation (Unchanged)

A self-hostable minimalist version of <https://archive.org>.

- Creates Chrome snapshots of URLs and serves them at nice short URLs like <https://archive.hackclub.com/p9OGi>
- Also supports git clones, YouTube videos, itch.io games, and website screenshots
- Comprehensive API

- Stores everything compressed using [zstd](https://github.com/facebook/zstd) level 6 (seekable format for random access)
- Flexible storage: local filesystem or S3-compatible cloud storage

Try out the demo instance at <https://arker-demo.hackclub.com>.

## Configuration

### Using .env Files

Arker supports loading configuration from a `.env` file for easier local development and deployment. 

1. **Copy the example file:**
   ```bash
   cp .env.example .env
   ```

2. **Edit the `.env` file** with your specific configuration values

3. **Run the server** - it will automatically load the `.env` file if present

The `.env` file is optional. If it doesn't exist, Arker will use environment variables or default values. Environment variables always take precedence over `.env` file values.

### Environment Variables

- `DB_URL` - PostgreSQL connection string (default: `host=localhost user=user password=pass dbname=arker port=5432 sslmode=disable`)
- `STORAGE_PATH` - Archive storage directory (default: `./storage`) - *only used when `STORAGE_TYPE=filesystem`*
- `CACHE_PATH` - Git clone cache directory (default: `./cache`)
- `MAX_WORKERS` - Worker pool size (default: `5`)
- `PORT` - HTTP server port (default: `8080`)
- `SESSION_SECRET` - Session encryption key (auto-generated if not set)
- `ADMIN_USERNAME` - Admin login username (default: `admin`)
- `ADMIN_PASSWORD` - Admin login password (default: `admin`)
- `LOGIN_TEXT` - Custom text to display under the login form. Useful for providing demo credentials (e.g., `LOGIN_TEXT="Demo: admin/admin"`). Supports basic HTML.
- `GIN_MODE` - Gin framework mode (`debug` for development)

### Itch.io Game Archiving

- `ITCH_API_KEY` - itch.io API key for downloading games (required for itch.io archiving)
- `ITCH_DL_PATH` - Path to itch-dl command (default: `itch-dl`)

**Dependencies for itch.io support:**
- Python 3.10+ with `itch-dl` package: `pip install itch-dl`
- itch.io API key: Generate at https://itch.io/user/settings → API Keys

### Storage Configuration

Arker supports both filesystem and S3-compatible storage backends.

#### Filesystem Storage (Default)
```bash
STORAGE_TYPE=filesystem  # or omit (default)
STORAGE_PATH=./storage
```

#### S3-Compatible Storage
```bash
STORAGE_TYPE=s3
S3_BUCKET=your-bucket-name        # Required
S3_REGION=us-east-1              # Default: us-east-1
S3_ACCESS_KEY_ID=your-key-id     # Optional: uses AWS credential chain if omitted
S3_SECRET_ACCESS_KEY=your-secret # Optional: uses AWS credential chain if omitted
S3_ENDPOINT=https://s3.example.com  # Optional: for non-AWS S3-compatible services
S3_PREFIX=arker/                 # Optional: prefix for all keys
S3_FORCE_PATH_STYLE=true         # Required for MinIO and some providers
```

**Supported S3-Compatible Services:**
- AWS S3
- MinIO
- Backblaze B2
- DigitalOcean Spaces
- Google Cloud Storage (S3 API)
- Any S3-compatible storage service

**Example Configurations:**

AWS S3:
```bash
STORAGE_TYPE=s3
S3_BUCKET=my-arker-archives
S3_REGION=us-west-2
```

MinIO:
```bash
STORAGE_TYPE=s3
S3_ENDPOINT=https://minio.example.com
S3_BUCKET=arker
S3_ACCESS_KEY_ID=minioadmin
S3_SECRET_ACCESS_KEY=minioadmin
S3_FORCE_PATH_STYLE=true
```

Backblaze B2:
```bash
STORAGE_TYPE=s3
S3_ENDPOINT=https://s3.us-west-002.backblazeb2.com
S3_BUCKET=my-arker-bucket
S3_REGION=us-west-002
S3_ACCESS_KEY_ID=your-b2-key-id
S3_SECRET_ACCESS_KEY=your-b2-secret
```

## Deployment Notes

### Docker Deployment

The Dockerfile includes all necessary dependencies including `itch-dl`. For production deployment:

1. **Set Environment Variables:**
   ```bash
   ITCH_API_KEY=your_itch_api_key_here  # Required for itch.io archiving
   ```

2. **Build and Deploy:**
   ```bash
   docker build -t arker .
   docker run -e ITCH_API_KEY="your_key" -p 8080:8080 arker
   ```

### Manual Installation

If not using Docker, install the Python dependencies manually:

```bash
# Install itch-dl for itch.io game archiving
pip3 install itch-dl

# Verify installation
python3 -m itch_dl --help
```

## License

MIT
