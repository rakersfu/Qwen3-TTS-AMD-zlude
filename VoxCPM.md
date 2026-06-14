🟩 1. 克隆项目代码
在命令行（CMD 或 PowerShell）执行：

bash
git clone https://github.com/OpenBMB/VoxCPM.git
cd VoxCPM
🟦 2. 创建并激活虚拟环境
建议使用 Python 3.9+。

Windows
bash
python -m venv venv
.\venv\Scripts\activate
Linux / macOS
bash
python3 -m venv venv
source venv/bin/activate
激活后命令行前缀会变成 (venv)，表示你已经进入虚拟环境。

🟧 3. 安装依赖（开发模式）
在虚拟环境中执行：

bash
pip install -e .
-e 参数表示 editable mode，源码修改后无需重新安装即可生效。

🟩 4. 启动 VoxCPM
运行以下命令启动应用，并指定端口为 8808：

bash
python app.py --port 8808
启动后，在浏览器访问：

Code
http://127.0.0.1:8808
🟦 5. 可选：一键启动脚本（Windows）
如果你想双击运行，可以在项目目录下创建 start_voxcpm.bat，内容如下：

bat
@echo off
chcp 65001 >nul

:: 获取当前目录
set "BASEDIR=%~dp0"
cd /d "%BASEDIR%"

echo -----------------------------------------------------------------------
echo     * VoxCPM Starter (with venv check) *
echo -----------------------------------------------------------------------
echo.
echo 当前脚本目录：
echo %BASEDIR%
echo.

:: 确认路径
set /p CONFIRM="请确认以上路径是否正确 (Y/N)："
if /I "%CONFIRM%" NEQ "Y" (
    echo 已取消操作，请检查路径后重新运行。
    pause
    exit /b
)

:: 检查 venv
if not exist "%BASEDIR%venv\Scripts\python.exe" (
    echo ❌ 未找到 venv，请先执行: python -m venv venv
    pause
    exit /b
)

:: 激活 venv
call "%BASEDIR%venv\Scripts\activate.bat"

:: 启动 VoxCPM
python app.py --port 8808

pause
📌 总结
完整流程就是：

克隆项目

创建虚拟环境

激活虚拟环境

安装依赖

启动应用
