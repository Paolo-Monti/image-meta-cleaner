# IMClean

IMClean is a native 64-bit Windows command-line utility for detecting, reporting,
and removing EXIF and embedded C2PA/JUMBF metadata from image files. 
No external runtime libraries are required.

The executable is named `imclean.exe`.

## Features

- Scans individual files, multiple paths, or complete folders.
- Processes subfolders progressively with `--recursive`, without building a full
  file list before showing results.
- Reports EXIF and embedded C2PA/JUMBF metadata in clear English text.
- Removes EXIF only, C2PA/JUMBF only, or both metadata categories.
- Provides a safe `--dry-run` mode that previews changes without writing files.
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

Scan every supported image in a folder and its subfolders (wildcards ? and * are supported):

```bat
imclean "C:\Photos\pic*" --recursive
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

## License

Copyright (c) 2026 Paolo Monti.

IMClean is distributed under the custom **IMClean Non-Commercial
No-Derivatives License 1.0**. You may run the software for personal,
educational, academic, research, charitable, or other non-commercial purposes.

Modification is not permitted. You may not alter, adapt, translate, patch, or
create derivative works from the binaries, nor distribute a
modified version. Commercial use is also prohibited without prior written
authorization from Paolo Monti.

This is a non-commercial, no-derivatives license rather than
an OSI-approved open-source license. See [`LICENSE.txt`](LICENSE.txt) for the
complete terms.

## Disclaimer

The software is provided "as is", without warranties or conditions of any kind,
to the maximum extent permitted by law. The author is not liable for data loss,
metadata loss, loss of provenance information, or other damages arising from its
use. Always keep verified backups of important files.

## Author

Paolo Monti  
Copyright (C) 2026
