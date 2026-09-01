# Changelog

All notable changes to IMClean are documented in this file. A new section will
be added whenever the application version changes; previously published entries
will remain unchanged.

## [1.2] - 2026-08-31

### Added

- Added image support for GIF, SVG, DNG, HEIF/HEIC, AVIF, and JPEG XL, in
  addition to JPEG, PNG, WebP, and classic TIFF.
- Added direct detection, reporting, and removal of supported EXIF and embedded
  C2PA/JUMBF structures without external runtime libraries.
- Added GUI input management for files and folders, including drag and drop,
  command-line paths, recursive scanning, wildcard patterns, and a dialog for
  reviewing, removing, or clearing queued inputs.
- Added sortable result columns, full-path tooltips for truncated names and
  paths, metadata-detail dialogs, image previews, and double-click opening with
  the Windows default image application.
- Added clear error causes in the metadata details panel and UTF-8 report export
  with the IMClean version recorded in the report header.
- Added optional safe backups, atomic replacement, source-file identity checks,
  and protection against recursive traversal through directory links.
- Added representative test images and automated coverage for all supported
  formats, malformed files, container safety, wildcard searches, GUI layout,
  rendering, and result sorting.

### Changed

- All application messages, dialogs, reports, and built-in help text are in
  English.
- The no-argument command-line guide now lists every supported image format.
- GUI removal commands run a fresh pass over the queued inputs and use wording
  that reflects the complete processing scope instead of the current row
  selection.
- The GUI dry-run option was removed; dry-run remains available in the console
  application for scripts and automation.
- Both executables now display version 1.2 before the author information and
  include synchronized Windows version resources.

### Fixed

- Fixed truncated EXIF status values, including `Not applicable`, in the result
  table.
- Fixed result ordering so sorting does not unexpectedly move the current view
  from one end of the list to the other.
- Fixed column headers remaining interactive when the result list is empty.
- Fixed the queued-input dialog so **Remove selected** immediately follows the
  current selection state.
- Fixed modal-dialog layout, clipping, close behavior, and border rendering.
- Fixed excessive flicker caused by sorting during an active scan.

### Security and integrity

- Release executables are compressed with PEPack LZMA and protected with its
  strong integrity guard.
- Added a verifiable IAT integrity manifest stored in a read-only,
  non-executable section.
- The release build now verifies both the packed-file integrity footer and the
  IAT integrity manifest before packaging.
- Published binaries are accompanied by SHA-256 checksums.
