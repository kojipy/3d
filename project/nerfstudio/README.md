# nerfstudio

## setup
- GPU : RTX5060Ti VRAM 16GB

### download dataset
`nf-download-data`コマンドでサンプルファイルをダウンロードすることができるが、リンク切れを起こしているものが多い点に注意。

2026/6/11 時点
- blender : NG
- sitcoms3d : OK
- nerfstudio : NG
- record3d : NG
- dnerf : NG
- phototourism : OK
- sdfstudio : OK
- nerfosr : OK
- mill19 : OK
- eyefultower : NG


以下のコマンドでsdfstudioデータセットをダウンロード
```
ns-download-data sdfstudio
```

## train

**`sdfstudio`のデータセットでプログラムが動作し始めることを確認。**
以下のコマンドで学習を開始することができる。
※ 但しCUDAのバージョンが異なることで(5060Ti非対応)学習処理が回らない。
```
ns-train neus-facto --data data/sdfstudio/sdfstudio-demo-data/dtu-scan65/ --vis viewer
```