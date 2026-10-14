# rewrite-mp4-color

A tool for patching and rewriting color information—specifically color primaries, transfer characteristics, and matrix coefficients—for HEVC and AV1 video streams inside MP4 containers.

It directly rewrites a few bytes in the colr and hvcC boxes of the MP4 container (for AV1, the av1C box and the sequence header OBUs in the samples), making it significantly faster than using the hevc_metadata/av1_metadata bitstream filters in ffmpeg. The process completes almost instantly.

However, this tool has several limitations not present in ffmpeg's hevc_metadata:

## Limitations

- Fragmented MP4 files are not supported.
- The colr box must already exist in the input file; this tool cannot insert a new colr box.
- The tool can only update color information if it is already present in the SPS within the hvcC box; it cannot add new fields if missing.
- It does not update SPS NAL units that are stored as part of the actual video bitstream (i.e., in-band samples), only those embedded in hvcC.

### AV1

- With --av1C, the sequence header OBU in av1C and every in-band sequence header OBU stored in the samples are rewritten (in AV1, each key frame carries a sequence header, and decoders usually use those).
- The sequence header must already contain color description (color_description_present_flag); it cannot be added.
- Rewriting to or from (1, 13, 0) (BT.709/sRGB/Identity) changes the syntax of the rest of the sequence header, so it cannot be done in-place and is rejected.

--hvcC requires ffmpeg to be installed and accessible from your system path. --av1C and --colr do not need ffmpeg.

## Usage

```
usage: rewrite-mp4-color OPTIONS FILE
OPTIONS:
    -p NUMBER  color primaries
    -t NUMBER  transfer characteristics
    -m NUMBER  matrix coefficients
    --colr     patch the colr box
    --hvcC     patch the hvcC box
    --av1C     patch the av1C box and in-band AV1 sequence headers
    --dry-run  show what would be rewritten, without modifying the file
```

You must specify at least one of -p, -t, or -m, as well as at least one of --colr, --hvcC or --av1C. Fields that are not explicitly specified will not be modified—their original values will be preserved.

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
