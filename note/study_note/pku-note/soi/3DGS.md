## TODO
run these with GPU on server
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202411111357007.png?token=ALRC6ISFVCVGCZ44CBENM4THGGOPI)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202411111358795.png?token=ALRC6IV66OT2FPEMAKB25M3HGGOUS)

## office
article
code
[GitHub - graphdeco-inria/gaussian-splatting: Original reference implementation of "3D Gaussian Splatting for Real-Time Radiance Field Rendering"](https://github.com/graphdeco-inria/gaussian-splatting?tab=readme-ov-file)

## log
### 11.18
ubuntu22.04 self 
#### install
[Site Unreachable](https://zhuanlan.zhihu.com/p/685698909)
安装成功
```
conda create -n gaussian_splatting python=3.10
conda activate gaussian_splatting
# 安装pytorch 2.2.1版本,cuda 12.1
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
SET DISTUTILS_USE_SDK=1
pip install submodules\diff-gaussian-rasterization
pip install submodules\simple-knn
pip install plyfile
pip install tqdm

```

