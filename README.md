# rewrite-mp4-color

A tool for patching and rewriting color information—specifically color primaries, transfer characteristics, and matrix coefficients—for HEVC video streams inside MP4 containers.

It directly rewrites a few bytes in either or both the colr and hvcC boxes of the MP4 container, making it significantly faster than using the hevc_metadata bitstream filter in ffmpeg. The process completes almost instantly.

However, this tool has several limitations not present in ffmpeg's hevc_metadata:

## Limitations

- Fragmented MP4 files are not supported.
- The colr box must already exist in the input file; this tool cannot insert a new colr box.
- The tool can only update color information if it is already present in the SPS within the hvcC box; it cannot add new fields if missing.
- It does not update SPS NAL units that are stored as part of the actual video bitstream (i.e., in-band samples), only those embedded in hvcC.

This script requires ffmpeg to be installed and accessible from your system path.

## Usage

```
usage: rewrite-mp4-color OPTIONS FILE
OPTIONS:
    -p NUMBER  color primaries
    -t NUMBER  transfer characteristics
    -m NUMBER  matrix coefficients
    --colr     patch the colr box
    --hvcC     patch the hvcC box
    --dry-run  show what would be rewritten, without modifying the file
```

You must specify at least one of -p, -t, or -m, as well as at least one of --colr or --hvcC. Fields that are not explicitly specified will not be modified—their original values will be preserved.

## Example

```
$ rewrite-mp4-color --dry-run -p 9 -t 18 -m 9 --colr --hvcC /path/to/video.mp4
patch hvcC SPS:
420101022000000300900000030000030096a001e020021c4d96566924cae6a121c120800001f480003a9804 ->
420101022000000300900000030000030096a001e020021c4d96566924cae6a1224120800001f480003a9804
patch colr:
6e636c780009000e000900 ->
6e636c7800090012000900
```
