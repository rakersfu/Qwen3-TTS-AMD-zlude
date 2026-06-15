🟩 环境准备
Python 版本：推荐 3.9–3.11（官方测试用 3.9.9）。

PyTorch：在 Windows + AMD 显卡下没有原生 ROCm 版，需要安装 CUDA 版 PyTorch，再用 ZLUDA 替换 DLL。
建议安装：

bat
pip install torch==2.7.0+cu118 torchvision==0.22.0+cu118 torchaudio==2.7.0+cu118 -f https://download.pytorch.org/whl/cu118
虚拟环境：推荐用 venv 或 conda，避免污染系统环境。

🟦 Whisper 安装
安装 Whisper 官方包

bat
pip install -U openai-whisper
或者安装最新源码版本：

bat
pip install git+https://github.com/openai/whisper.git
依赖库 requirements.txt（完整 WebUI 版）

text
numpy
numba
tqdm
more-itertools
setuptools

torch==2.7.0+cu118
torchvision==0.22.0+cu118
torchaudio==2.7.0+cu118

openai-whisper
tiktoken
ffmpeg

streamlit
fastapi
uvicorn
gradio

pydantic
requests
python-multipart

langchain
faiss-cpu
pdfplumber

triton>=2.0.0; platform_system=="Linux"
安装方式：

bat
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cu118
🟩 系统依赖
FFmpeg：在 Windows 下可用 Chocolatey 或 Scoop 安装：

bat
choco install ffmpeg
或

bat
scoop install ffmpeg
Rust：如果 tiktoken 没有预编译 wheel，需要安装 Rust：
下载 Rust 官方安装器，安装后确保 ~/.cargo/bin 在 PATH 中。
如果报错 No module named 'setuptools_rust'：

bat
pip install setuptools-rust
🟦 AMD 显卡支持（ZLUDA）
下载最新 ZLUDA Windows 包（推荐 Gitee 镜像）。

删除 venv\Lib\site-packages\torch\lib 下的旧 CUDA DLL。

复制 ZLUDA DLL 替换：

cublas.dll → cublas64_11.dll

cublasLt.dll → cublasLt64_11.dll

cusparse.dll → cusparse64_11.dll

cufft.dll → cufft64_10.dll

cufftw.dll → cufftw64_10.dll

nvrtc.dll → nvrtc64_112_0.dll

依赖：nvcuda.dll、nvml.dll、nccl.dll、zluda_redirect.dll、zluda_dump.dll

设置环境变量：

bat
set ZLUDA_VERBOSE=1
set HIP_VISIBLE_DEVICES=0
set CUDA_VISIBLE_DEVICES=0
测试 GPU 是否可用：

bat
zluda.exe -- python -c "import torch; print(torch.__version__); print(torch.version.cuda); print(torch.cuda.is_available())"
🟩 使用 Whisper
命令行转录：

bat
whisper audio.wav --model turbo
指定语言：

bat
whisper japanese.wav --language Japanese
翻译成英文：

bat
whisper japanese.wav --model medium --language Japanese --task translate
Python 调用：

python
import whisper
model = whisper.load_model("turbo")
result = model.transcribe("audio.mp3")
print(result["text"])
📌 总结：
在 Windows + AMD 显卡环境下，正确流程是：安装 CUDA 版 PyTorch → 安装 Whisper → 安装 FFmpeg/Rust → 用 ZLUDA 替换 DLL → 设置环境变量 → 测试 GPU → 使用 Whisper。
