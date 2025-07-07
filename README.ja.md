# rewrite-mp4-color

MP4内のHEVC映像の色情報(colour primaries, transfer characteristics, matrix coefficient)にパッチをあてて書き換えるツールです。

MP4コンテナのcolr boxとhvcC boxのいずれか、または両方の、数バイトを直接書き換えるだけなので、ffmpegのhevc_metadataビットストリームフィルタを用いるより圧倒的に高速で、処理は一瞬で終了します。

ただし、このツールにはffmpegのhevc_metadataにはない以下の制限事項があります。

## 制限事項

* fragmented MP4ファイルには対応していません
* colrがすでに存在する場合しか書き換えられません。つまり、元のファイルにcolrが存在しない場合にcolrを追加することはできません
* 同様に、元のhvcCのSPSに上記の色情報が含まれている場合は更新できますが、そうでない場合に追加することはできません
* SPSがhvcCの中だけでなく、HEVCビットストリームにインバンドで入った形でMP4のサンプルとして格納されている場合、それらは更新できません

このスクリプトの実行にはffmpegが必要です。


## 使用法

```
usage: rewrite-mp4-color OPTIONS FILE
OPTIONS:
    -p NUMBER  color primaries
    -t NUMBER  transfer characteristics
    -m NUMBER  matrix coefficients
    --colr     patch colr box
    --hvcC     patch hvcC
    --dry-run  don't actually rewrite
```

最低限-p/-t/-mのいずれか、また--colr/--hvcCのいずれかの指定が必要です。指定されたフィールド以外の値は書き換えられず、元の値が維持されます。


## 使用例

```
$ rewrite-mp4-color --dry-run -p 9 -t 18 -m 9 --colr --hvcC /path/to/video.mp4
patch hvcC SPS:
420101022000000300900000030000030096a001e020021c4d96566924cae6a121c120800001f480003a9804 ->
420101022000000300900000030000030096a001e020021c4d96566924cae6a1224120800001f480003a9804
patch colr:
6e636c780009000e000900 ->
6e636c7800090012000900
```
