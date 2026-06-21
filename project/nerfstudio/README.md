# nerfstudio

## setup
- GPU : RTX5060Ti VRAM 16GB

### download dataset
`nf-download-data`コマンドでサンプルファイルをダウンロードすることができるが、リンク切れを起こしているものが多い点に注意。

2026/6/11 時点

| データセット | ダウンロード | 学習に使用できるか |
| :-----: | :-----: | :-----: |
| blender | NG | - |
| sitcoms3d | OK |  |
| nerfstudio | NG | - |
| record3d | NG | - |
| dnerf | NG | - |
| phototourism | OK |  |
| sdfstudio | OK |  |
| nerfosr | NG | - |
| mill19 | OK |  |
| eyefultower | NG | - |



例 : 以下のコマンドでsdfstudioデータセットをダウンロード
```
ns-download-data sdfstudio
```

## train

[使用できるメソッド一覧](https://docs.nerf.studio/nerfology/methods/index.html#implemented-methods)から学習させたいモデルを選択する。
- Instant-NGP
- Splatfacto
- Splatfacto-W
- Instruct-NeRF2NeRF
- Instruct-GS2GS
- SIGNeRF
- K-Planes
- LERF
- LiveScene
- Feature-Splatting
- Mip-NeRF
- NeRF
- Nerfacto
- Nerfbusters
- NeRFPlayer
- Tetra-NeRF
- TensoRF
- Generfacto
- PyNeRF
- SeaThru-NeRF
- Zip-NeRF
- BioNeRF
- NeRFtoGSandBack
- OpenNeRF

使用するデータセットに応じてパーサーを指定する必要がある。用意されているパーサーは以下の通り。
- nerfstudio-data
- minimal-parser
- arkit-data
- blender-data
- instant-ngp-data
- nuscenes-data
- dnerf-data
- phototourism-data
- dycheck-data
- scannet-data
- sdfstudio-data
- nerfosr-data
- sitcoms3d-data
- scannetpp-data
- colmap


**`sdfstudio`のデータセットでプログラムが動作し始めることを確認。**
以下のコマンドで学習を開始することができる。
※ 但しCUDAのバージョンが異なることで(5060Ti非対応)学習処理が回らない。
```
ns-train neus-facto sdfstudio-data --data /data/sdfstudio/sdfstudio-demo-data/dtu-scan65
```