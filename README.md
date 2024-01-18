# pytorch-gfx803-for-Whisper (WEBUI)
Description and whl files for using Whisper with gfx803 RX580

# PyTorch ROCm gfx803

build pytorch 1.13.1 with ROCm support for Whisper

```
Linux Mint 21.2
Radeon RX 580 8GB
RoCm 5.5

Python 3.10.6
- pytorch 1.13.1
- torchvision 0.14.1
```
## Install dependencies

```bash
sudo apt autoremove rocm-core amdgpu-dkms
sudo apt install libopenmpi3 libstdc++-12-dev libdnnl-dev ninja-build libopenblas-dev libpng-dev libjpeg-dev
```

## Install ROCm

```bash
sudo -i
sudo echo ROC_ENABLE_PRE_VEGA=1 >> /etc/environment
sudo echo HSA_OVERRIDE_GFX_VERSION=8.0.3 >> /etc/environment
# Reboot after this

wget https://repo.radeon.com/amdgpu-install/5.5/ubuntu/jammy/amdgpu-install_5.5.50500-1_all.deb
sudo apt install ./amdgpu-install_5.5.50500-1_all.deb
sudo amdgpu-install -y --usecase=rocm,hiplibsdk,mlsdk

sudo usermod -aG video $LOGNAME
sudo usermod -aG render $LOGNAME

# verify
rocminfo
clinfo
```

## Build

You may need to install addional dependencies, and the build will take a long time.

#in home directory create directory pytorch1.13.1
#Build Torch
```bash
cd pytorch1.13.1
git clone --recursive https://github.com/pytorch/pytorch.git -b v1.13.1
cd pytorch
pip install cmake mkl mkl-include
pip install -r requirements.txt
sudo ln -s /usr/lib/x86_64-linux-gnu/librt.so.1 /usr/lib/x86_64-linux-gnu/librt.so
export PATH=/opt/rocm/bin:$PATH ROCM_PATH=/opt/rocm HIP_PATH=/opt/rocm/hip
export PYTORCH_ROCM_ARCH=gfx803
export PYTORCH_BUILD_VERSION=1.13.1 PYTORCH_BUILD_NUMBER=1
export USE_CUDA=0 USE_ROCM=1 USE_NINJA=1
python3 tools/amd_build/build_amd.py
python3 setup.py bdist_wheel
#pip install dist/torch-1.13.1-cp310-cp310-linux_x86_64.whl --force-reinstall

```

### Build torchvision

```bash
cd ..
git clone https://github.com/pytorch/vision.git -b v0.14.1
cd vision
export BUILD_VERSION=0.14.1
FORCE_CUDA=1 ROCM_HOME=/opt/rocm/ python3 setup.py bdist_wheel
#pip install dist/torchvision-0.14.1-cp310-cp310-linux_x86_64.whl --force-reinstall

```
If you are using WHISPER-WEBUI you have to turn of FASTER-WHISPER because it does not work with ROCm, therefore you have to call it with argument --disable_faster_whisper

### install in WHISPER-WEBUI
```bash
cd Whisper-WebUI
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip wheel
pip uninstall torch torchvision
pip3 install /home/*******/pytorch1.13.1/pytorch/dist/torch-1.13.1-cp310-cp310-linux_x86_64.whl
pip3 install /home/*******/pytorch1.13.1/vision/dist/torchvision-0.14.1-cp310-cp310-linux_x86_64.whl
pip list | grep 'torch'
python3 app.py --disable_faster_whisper
```
******* is your home directory username

I have tested a 45 minutes mp4 viedeo in language german to transcibe on my local computer (language setting was also preset to german)
1) with faster-whisper and CPU mode
2) with whisper and RX580

Results (which was not really a clean test - because other programs (firefox and so on where running during the test)
1: fasterwhisper with cpu 27.0 minutes 44 seconds!
2: whisper with gpu rx580 8GB (gfx803) 22.0 minutes 54 seconds!

## Reference
- https://github.com/openai/whisper
- https://github.com/jhj0517/Whisper-WebUI
- https://github.com/RadeonOpenCompute/ROCm/issues/1659
- https://github.com/xuhuisheng/rocm-gfx803
- https://github.com/xuhuisheng/rocm-gfx803/issues/27#issuecomment-1534048619
- https://github.com/xuhuisheng/rocm-gfx803/issues/27#issuecomment-1892611849
