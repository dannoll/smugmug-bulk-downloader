# SmugMug Bulk Downloader & Metadata Export Tool

![Python 3.12+](https://img.shields.io/badge/python-3.12+-blue.svg)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Platform: macOS (Apple Silicon tested)](https://img.shields.io/badge/platform-macOS-lightgrey.svg)

A Python CLI that recursively **counts** all folders/photos in your SmugMug account (with validation), then bulk-downloads originals + full metadata (titles/tags/descriptions) for both images and galleries. **Pre/post download counts + logging verify 100% completeness**. A complete SmugMug backup tool for leaving SmugMug, migrating your account, or creating full local backups.

## SmugMug Backup & Export Use Cases

- **Full account export before canceling SmugMug** - Download everything before closing your account
- **Migrating albums to another platform** - Export to Lightroom, local NAS, Google Photos, or other gallery services
- **Regular local backups of all photos + metadata** - Keep offline copies with complete titles, descriptions, and tags
- **Preserving photo organization** - Maintains folder hierarchy and all metadata for future reference

## Features

- **Recursively walks all folders and galleries** in a SmugMug account with hierarchical tree view
- **Downloads original images** in parallel with retry and comprehensive logging
- **Exports gallery and photo metadata** (title, caption/description, keywords/tags) to text files
- **Resume-safe** - Skips already-downloaded files, allowing interrupted downloads to continue
- **Duplicate detection** - Identifies true duplicates (same ImageKey) and handles filename collisions
- **Intelligent quality fallback** - Original → X5Large → X4Large → etc., ensures highest quality available
- **Comprehensive logging** - Creates detailed logs for failures, duplicates, renamed files, and skipped images
- **OAuth 1.0a authentication** - Secure API access
- **Managed with `uv`** - Fast, reliable modern Python dependency management
- **Actively maintained** - Tested on Python 3.12+ and macOS (Apple Silicon)

## Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) package manager
- A SmugMug account with API access

## Setup

### 1. Install uv (if not already installed)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2. Clone or download this project

```bash
cd /path/to/smugmug-bulk-downloader
```

### 3. Install dependencies

```bash
uv sync
```

### 4. Register a SmugMug API Application

1. Go to [SmugMug API Developer Portal](https://api.smugmug.com/api/developer/apply)
2. Fill out the application form
3. Set the **Callback URL** to: `http://localhost:8080/`
4. Note your **API Key** and **API Secret**

### 5. Authenticate with SmugMug

Run the authentication script:

```bash
uv run python auth.py
```

Follow the prompts:
1. Enter your API Key
2. Enter your API Secret
3. Your browser will open for authorization
4. Authorize the application
5. Copy the output credentials

### 6. Create config.py

Copy the example config file:

```bash
cp config-example.py config.py
```

Edit `config.py` and fill in:
- The credentials from the auth script
- Your SmugMug nickname (username)

Example `config.py`:

```python
API_KEY = 'abc123...'
API_SECRET = 'def456...'
ACCESS_TOKEN = 'ghi789...'
ACCESS_SECRET = 'jkl012...'
NICKNAME = 'johndoe'  # Your SmugMug username
```

## Usage

### Counting Albums and Images

Run the counter to see all albums and image counts:

```bash
uv run python main.py
```

Or activate the virtual environment first:

```bash
source .venv/bin/activate
python main.py
```

Add `--debug` flag for detailed output:

```bash
uv run python main.py --debug
```

### Downloading All Content

To download your entire SmugMug library:

```bash
uv run python download.py
```

The script will:
1. Prompt you for a download location (default: ~/SmugMug-Downloads)
2. Ask for confirmation before starting
3. Download all folders, albums, and images preserving the hierarchy
4. Save album metadata as `Album_Name.txt` in each gallery folder
5. Save image metadata (title, caption, keywords) as `.txt` files alongside each image
6. Skip files that already exist (resume support)
7. Show progress for each album
8. Display a summary when complete

**Features:**
- Downloads **original quality images** using SmugMug's ImageSizeDetails API
- Intelligent fallback chain: Original → X5Large → X4Large → X3Large → etc.
- **Duplicate detection** - Identifies true duplicates (same ImageKey) and skips them
- **Filename collision handling** - Automatically renames different images with same filename
- **No data loss** - "Skipped" images are saved to `Skipped_Images` directory instead of being lost
- **Saves image metadata** (title, caption, keywords) in `.txt` files alongside images
- **Saves album/gallery metadata** (title, description, keywords) in album-named `.txt` files
- **Detailed logging** - Creates logs for failures, duplicates, renamed files, and skipped images
- Preserves folder hierarchy
- Concurrent downloads (5 images at a time per album)
- Resume support - rerun to continue interrupted downloads
- Sanitizes filenames for filesystem compatibility
- Progress indicators for large albums

### Sample Output

```
================================================================================
SmugMug Album Counter for: johndoe
================================================================================

Fetching folder structure...
Found 3 top-level folders

============================================================
Travel/
============================================================
  ├─ Europe 2024: 127 images
  ├─ Asia Trip: 89 images
  ├─ Beach Vacation: 45 images
────────────────────────────────────────────────────────────
Subtotal for Travel: 261 images

============================================================
Family/
============================================================
  ├─ Holidays: 156 images
  ├─ Birthday Parties: 78 images
────────────────────────────────────────────────────────────
Subtotal for Family: 234 images

============================================================
Events/
============================================================
  ├─ Wedding 2023: 312 images
  ├─ Corporate Event: 67 images
────────────────────────────────────────────────────────────
Subtotal for Events: 379 images

================================================================================
GRAND TOTAL
================================================================================
Total Albums: 8
Total Images: 874
================================================================================
```

## Project Structure

```
smugmug-bulk-downloader/
├── main.py              # Album counter script
├── download.py          # Complete download script
├── auth.py              # OAuth authentication helper
├── config-example.py    # Configuration template
├── config.py            # Your credentials (git-ignored)
├── pyproject.toml       # Project dependencies
├── uv.lock              # Dependency lockfile
├── .gitignore           # Git ignore rules
├── CHANGELOG.md         # Version history
├── LICENSE              # MIT License
└── README.md            # This file
```

## Troubleshooting

### Authentication Fails

- Ensure your callback URL is set to `http://localhost:8080/` in your SmugMug app settings
- Make sure port 8080 is not in use
- Check that your API Key and Secret are correct

### "config.py not found"

- Make sure you've created `config.py` from the example
- Verify all fields are filled in
- Check that NICKNAME matches your SmugMug username

### Rate Limiting

SmugMug API has rate limits. If you have many albums, the script may take time to complete. The counter script fetches data sequentially to avoid hitting rate limits. The download script uses concurrent downloads (5 at a time) to balance speed with API limits.

### Download Interruptions

If the download script is interrupted:
- Simply rerun it with the same download location
- It will skip files that already exist
- Only missing files will be downloaded

### Duplicate Detection & Handling

The script automatically detects and handles two types of duplicates:

#### 1. True Duplicates (Same ImageKey)
When SmugMug's API returns the same image multiple times (same ImageKey), the script:
- Downloads it only once
- Logs it to `duplicates.txt`
- Shows count in summary as "True duplicates skipped"

#### 2. Filename Collisions (Different Images, Same Name)
When two **different** images have the same filename:
- Both images are downloaded
- The second is renamed (e.g., `photo.jpg` → `photo_2.jpg`)
- Logged to `renamed_files.txt`
- Shows count in summary as "Filename collisions renamed"

**Important**: Filename collisions mean you have different images with identical names. The renaming prevents data loss.

### Failed Downloads

If any images fail to download, the script creates a `failed_downloads.txt` file in your download location with:
- Filename of each failed image
- Album name where it belongs
- Specific reason for the failure (HTTP error, timeout, no URL, etc.)

**Example** (`failed_downloads.txt`):
```
SmugMug Download Failures
================================================================================
Total failed: 3
================================================================================

File: sunset.jpg
Album: California Coast
Reason: HTTP 403 error
--------------------------------------------------------------------------------

File: portrait.jpg
Album: Family Photos
Reason: No download URL found (no ImageSizeOriginal, ArchivedUri, LargestImage, or ImageDownload)
--------------------------------------------------------------------------------

File: beach.jpg
Album: Vacation 2024
Reason: Download timeout (>30s)
--------------------------------------------------------------------------------
```

Review this file to understand what went wrong and potentially retry the download.

### Skipped Images

If an image already exists at its intended location during download, instead of skipping it:
- **The image is downloaded to the `Skipped_Images` directory**
- The album structure is preserved within `Skipped_Images`
- Details are logged to `skipped_images.txt`

This prevents any data loss and allows you to investigate later.

**On Resume Downloads**: These are files you already downloaded. They're saved to `Skipped_Images` for comparison.

**On Fresh Downloads**: This indicates:
- API returned the same image multiple times (without same ImageKey)
- Threading race condition
- Images can be compared to verify if they're true duplicates

**File Structure Example**:
```
SmugMug-Downloads/
├── California_Coast/
│   └── sunset.jpg           ← Original download
├── Skipped_Images/
│   └── California_Coast/
│       └── sunset.jpg       ← "Duplicate" saved here instead of being skipped
└── skipped_images.txt       ← Log file
```

**Example** (`skipped_images.txt`):
```
SmugMug Skipped Images (Already Existed)
================================================================================
Total skipped: 36
================================================================================

File: sunset.jpg
Album: California Coast
ImageKey: abc123xyz
Original Path: /path/to/California_Coast/sunset.jpg
Saved to: /path/to/Skipped_Images/California_Coast/sunset.jpg
--------------------------------------------------------------------------------
```

You can then compare files to see if they're identical or different versions.

### Log Files Summary

The script creates these log files in your download location:

- `download_log.txt` - **Complete log of all processing messages** (everything shown on terminal)
- `failed_downloads.txt` - Images that failed to download (errors)
- `duplicates.txt` - True duplicates (same ImageKey returned multiple times)
- `renamed_files.txt` - Filename collisions (different images with same filename)
- `skipped_images.txt` - Images that already existed (saved to `Skipped_Images` directory instead)

The `download_log.txt` file contains a complete record of the entire download session with timestamps, allowing you to review what happened even if you weren't watching the terminal output.

### Disk Space

Before running the download script, ensure you have enough disk space for your entire SmugMug library. Check the counter output first to estimate space needed (assuming ~2-5MB per image on average).

### Image Quality

The download script prioritizes getting the **original uploaded image** quality:

1. **First choice**: Fetches `ImageSizeOriginal` via the ImageSizeDetails API endpoint
2. **Fallback chain**: If original isn't available, tries progressively smaller sizes (X5Large, X4Large, X3Large, X2Large, XLarge, Large)
3. **Additional fallbacks**: ArchivedUri → LargestImage endpoint → ImageDownload

This ensures you get the highest quality version available for each image.

### Image Metadata

For each image downloaded, the script automatically creates a companion text file containing:
- **Title**: The image title from SmugMug
- **Caption**: The image caption/description
- **Keywords**: Comma-separated keywords/tags

**Filename pattern**: If your image is `photo.jpg`, the metadata file will be `photo.txt`

**Example image metadata file** (`sunset.txt`):
```
Title: Golden Hour Sunset
Caption: Beautiful sunset over the Pacific Ocean, taken from Big Sur
Keywords: sunset, ocean, california, landscape, golden hour
```

### Album/Gallery Metadata

For each album/gallery folder, the script creates a metadata text file named after the album (with spaces replaced by underscores) containing:
- **Gallery Title**: The album/gallery name
- **Gallery Description**: The album/gallery description
- **Keywords**: Album-level keywords

**Filename pattern**: Album "Paris 2024 Trip" creates `Paris_2024_Trip.txt`

**Example album metadata file** (`Paris_2024_Trip.txt`):
```
Gallery Title: Paris 2024 Trip
Gallery Description: Photos from our spring vacation in Paris, including visits to the Eiffel Tower, Louvre, and Montmartre
Keywords: paris, france, travel, vacation, europe
```

The metadata files are only created if the image or album has title, caption, description, or keywords defined. Empty metadata is not saved.

**File structure example**:
```
SmugMug-Downloads/
├── Europe Travel Photos/
│   ├── Paris/
│   │   ├── Paris.txt                  ← Album metadata
│   │   ├── eiffel_tower.jpg
│   │   ├── eiffel_tower.txt           ← Image metadata
│   │   ├── louvre_museum.jpg
│   │   └── louvre_museum.txt          ← Image metadata
```

## License

MIT License - feel free to use and modify as needed.

## Acknowledgements

This project was inspired by and/or built with reference to:

- [am133/SmugMug-Image-Downloader](https://github.com/am133/SmugMug-Image-Downloader) – original idea for a Python-based SmugMug downloader.
- [andyjsmith/SmugMug-Downloader](https://github.com/andyjsmith/SmugMug-Downloader) – earlier approach to bulk exporting images.
- [phpSmug](https://github.com/lildude/phpSmug) – helped clarify SmugMug API patterns and folder traversal.
- [CustomSmug Downloader](https://customsmug.com/tutorial/how-to-use-the-smugmug-downloader/) – inspired capture of file metadata, and later gallery metadata.

## Credits

Built with:
- [requests-oauthlib](https://github.com/requests/requests-oauthlib) for OAuth authentication
- [uv](https://docs.astral.sh/uv/) for dependency management
- [SmugMug API](https://api.smugmug.com/) for accessing album data
