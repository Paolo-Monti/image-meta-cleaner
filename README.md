# IMClean

IMClean is a native 64-bit Windows utility for detecting, reporting, and removing
EXIF and embedded C2PA/JUMBF metadata from image files. It includes both an
accessible graphical interface and a command-line interface. No external runtime
libraries are required.

The graphical executable is named `imclean-gui.exe`; the console executable is
named `imclean.exe`. The two executables can be used independently.

## Features

- Scans individual files, multiple paths, or complete folders.
- Processes subfolders progressively with `--recursive`, without building a full
  file list before showing results.
- Reports EXIF and embedded C2PA/JUMBF metadata in clear English text.
- Removes EXIF only, C2PA/JUMBF only, or both metadata categories.
- Provides a safe command-line `--dry-run` mode that previews changes without
  writing files.
- Optionally creates non-destructive backup copies before modification.
- Writes an optional UTF-8 log file.
- Detects image formats from their binary signatures instead of trusting file
  extensions alone.
- Avoids re-encoding image data and preserves pixel content whenever metadata is
  removed.
- Uses atomic file replacement and rejects a write if the original file changes
  while it is being processed.
- Skips directory junctions and symbolic links during recursive scans to avoid
  cycles and unintended traversal outside the requested folder.
- Provides a responsive Windows GUI based on owner-drawn controls, with a live
  results table, image preview, metadata details, status badges, and progress.
- Accepts files and folders through dialogs, drag and drop, or GUI command-line
  arguments.
- Exports GUI scan results as a UTF-8 text report.

## Supported image formats

| Format | Extensions | EXIF support | C2PA/JUMBF support |
| --- | --- | --- | --- |
| JPEG | `.jpg`, `.jpeg` | Detection, report, and removal | Detection, report, and removal of embedded APP11 JUMBF data |
| PNG | `.png` | Detection, report, and removal | Detection, report, and removal of embedded C2PA/JUMBF chunks |
| WebP | `.webp` | Detection, report, and removal | Detection, report, and removal of embedded RIFF chunks |
| TIFF | `.tif`, `.tiff` | Detection, report, and removal for classic little-endian and big-endian TIFF | Detection, report, and removal of the embedded C2PA Manifest Store tag |

Classic TIFF files with magic number 42 are supported. BigTIFF files with magic
number 43 and 64-bit offsets are not currently supported.

## Usage

### Graphical interface

Start `imclean-gui.exe`, then use **Add Files** or **Add Folder**. Choose the
required options and select one of the following actions:

- **Scan** detects and reports metadata without changing images.
- **Remove EXIF** removes EXIF metadata only.
- **Remove C2PA** removes embedded C2PA/JUMBF metadata only.
- **Remove All** removes both metadata categories.

The **Recursive** option includes matching files in subfolders. **Create backup**
stores a safe copy before each modification. The **Pattern** field accepts
multiple file masks separated by semicolons. The graphical interface always
asks for confirmation before a removal operation; dry-run mode remains
available in the command-line interface for scripts and automation.

Use the **View selected inputs** link in the upper-right corner to inspect the
complete list of queued files and folders, remove selected entries, or clear
the input list. Results can be sorted by clicking a column header; EXIF and
C2PA/JUMBF sorting shows detected metadata first on the initial click. Sorting
is temporarily suspended while an operation is running and is applied once
when the operation finishes, avoiding unnecessary redraws. Hover over a
truncated path in the metadata panel to see the complete path in a tooltip.
When EXIF or C2PA/JUMBF metadata is detected, hover over its blue label to see
the details tooltip, then click the label to open a scrollable dialog containing
the complete file path and every available parsed metadata detail.

You can drag files or folders directly onto the window. You can also start the
GUI with one or more paths; those inputs are added and scanned automatically:

```bat
imclean-gui.exe "C:\Photos"
imclean-gui.exe photo.jpg scan.png
```

Use **Export Report** to save the current results as a UTF-8 text file. Selecting
a result displays its preview, file information, parsed EXIF fields, and embedded
C2PA/JUMBF details. IMClean detects embedded provenance data but does not verify
C2PA digital signatures.

### Command-line interface

```text
imclean [command] [options] <file-or-folder> [...]
```

If no command is supplied, `scan` is used automatically. Running the program
without parameters displays the complete built-in English guide and does not
modify any file.

### Commands

| Command | Description |
| --- | --- |
| `scan` | Detect and display embedded metadata. This is the default command. |
| `remove-exif` | Remove EXIF metadata only. |
| `remove-c2pa` | Remove embedded C2PA/JUMBF metadata only. |
| `remove-all` | Remove both EXIF and embedded C2PA/JUMBF metadata. |

### Options

| Option | Description |
| --- | --- |
| `-r`, `--recursive` | Include all supported images in subfolders. |
| `-n`, `--dry-run` | Preview removal operations without changing files or creating backups. |
| `-b`, `--backup` | Create a backup before each modification. |
| `-v`, `--verbose` | Display additional processing details. |
| `--log <file>` | Duplicate console output to a UTF-8 log file. |
| `--` | Treat all following arguments as paths, including names beginning with a dash. |
| `-h`, `--help` | Display the built-in guide. |

## Examples

Scan one JPEG file. The explicit `scan` command is optional:

```bat
imclean photo.jpg
imclean scan photo.jpg
```

Scan every supported image in a folder and its subfolders:

```bat
imclean "C:\Photos" --recursive
```

Preview EXIF removal without changing the PNG file:

```bat
imclean remove-exif photo.png --dry-run
```

Remove EXIF metadata from a TIFF file and keep the original as a backup:

```bat
imclean remove-exif scan.tiff --backup
```

Preview C2PA/JUMBF removal from an entire folder:

```bat
imclean remove-c2pa "C:\Photos" -r --dry-run
```

Remove both metadata categories from several files and save a log:

```bat
imclean remove-all a.jpg b.png c.webp d.tif --backup --log cleanup.log
```

Process a file whose name begins with a dash:

```bat
imclean scan -- -photo.jpg
```

## Recommended workflow

For valuable or irreplaceable images, use the following sequence:

1. Run `scan` to inspect the detected metadata.
2. Run the intended removal command with `--dry-run`.
3. Repeat the command without `--dry-run` and add `--backup`.
4. Verify the resulting image with your usual image viewer or workflow.

Backups never overwrite existing files. IMClean first creates `filename.ext.bak`,
then `filename.ext.bak.1`, `filename.ext.bak.2`, and so on.

## Safety and file integrity

IMClean edits supported metadata structures directly and does not decode or
re-encode image pixels. New content is written to a temporary file in the same
folder and then installed using Windows atomic replacement facilities. Temporary
files are cleaned up after failures whenever possible.

Before replacing an image, IMClean checks the file identity, size, and last-write
time again. If another process has modified or replaced the file since it was
read, the operation stops instead of overwriting the newer content.

Malformed or truncated containers are handled conservatively. If safe removal
cannot be guaranteed, the file is left unchanged and an error is reported.

## Known limitations

- Proprietary MakerNote formats, private string encodings, and every possible
  TIFF field type are not fully decoded in reports.
- JPEG APP11 segments structurally identified as JUMBF are removed as a group.
  If a JPEG uses JUMBF for non-C2PA data, that data may also be removed by
  `remove-c2pa` or `remove-all`.
- BigTIFF is not supported. Classic TIFF metadata is removed without moving or
  re-encoding pixel data; detached payloads are cleared only when they do not
  overlap pixels, IFD structures, or still-referenced data.
- C2PA manifests may be remote or externally referenced. IMClean handles only
  embedded metadata in the supported containers. It does not validate digital
  signatures and does not download remote manifests.
- XMP, IPTC, ICC profiles, comments, and unrelated metadata are not removed,
  except when they are contained inside a selected JPEG JUMBF segment.
- Future container variants or metadata identifiers may require an update.

Removing C2PA data discards embedded provenance information. Keep an original
copy whenever authenticity or chain-of-custody information may be important.

### Requirements

- Windows 64-bit.
- The release binaries require no external runtime libraries.

## Release verification

Published releases include `SHA256SUMS.txt`. Verify a downloaded executable
with PowerShell before running it:

```powershell
Get-FileHash .\imclean-gui.exe -Algorithm SHA256
```

Compare the resulting value with the matching line in `SHA256SUMS.txt`.

## License

Copyright (c) 2026 Paolo Monti.

IMClean is distributed under the custom **IMClean Non-Commercial
No-Derivatives License 1.0**. You may run the software for personal,
educational, academic, research, charitable, or other non-commercial purposes.

Modification is not permitted. You may not alter, adapt, translate, patch, or
create derivative works from the binaries, nor distribute a
modified version. Commercial use is also prohibited without prior written
authorization from Paolo Monti.

See [`LICENSE.txt`](LICENSE.txt) for the complete terms.

## Disclaimer

The software is provided "as is", without warranties or conditions of any kind,
to the maximum extent permitted by law. The author is not liable for data loss,
metadata loss, loss of provenance information, or other damages arising from its
use. Always keep verified backups of important files.

## Author

Paolo Monti  
Copyright (C) 2026
