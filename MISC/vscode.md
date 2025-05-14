## 更改插件位置
在 VS Code 中，插件（扩展）默认存储在用户目录下的 `C:/user/用户名/.vscode/extensions` 文件夹中，当插件数量较多时会占用大量空间。以下是将插件目录迁移到其他位置的方法：
### 方法一：使用命令行参数（临时方案）
在启动 VS Code 时，通过 `--extensions-dir` 参数指定插件目录：
```bash
code --extensions-dir "D:/vscode-extensions"  # Windows
code --extensions-dir "/data/vscode-extensions"  # Linux/macOS
```
- **缺点**：每次启动都需手动添加参数，不方便。
### 方法二：创建快捷方式（Windows）
1. **右键点击 VS Code 图标** → **属性**
2. **在 “目标” 字段末尾添加参数**：
    ```plaintext
    "C:\Users\用户名\AppData\Local\Programs\Microsoft VS Code\Code.exe" --extensions-dir "D:/vscode-extensions"
    ```
3. **保存并使用新快捷方式启动**。
### 方法三：修改环境变量（推荐）
通过设置 `VSCODE_EXTENSIONS` 环境变量永久更改路径：
#### Windows
1. **搜索 “环境变量”** → 打开 “编辑系统环境变量”
2. **新建系统变量**：
    - **变量名**：`VSCODE_EXTENSIONS`
    - **变量值**：`D:\vscode-extensions`（自定义路径）
3. **重启 VS Code**，插件将自动安装到新路径。
#### Linux/macOS
1. **编辑 `~/.bashrc` 或 `~/.zshrc`**：
    ```bash
    export VSCODE_EXTENSIONS="/data/vscode-extensions"
    ```
2. **应用更改**：
    ```bash
    source ~/.bashrc  # 或 source ~/.zshrc
    ```
### 方法四：迁移现有插件（可选）
若已安装大量插件，可迁移现有文件而非重新安装：
1. **关闭 VS Code**
2. **复制原插件目录**：
    ```bash
    # Windows
    robocopy "C:\Users\用户名\.vscode\extensions" "D:\vscode-extensions" /E
    
    # Linux/macOS
    cp -r ~/.vscode/extensions /data/vscode-extensions
    ```
3. **按上述方法设置新路径**
4. **验证**：打开 VS Code，查看扩展管理器中是否显示已安装插件。
### 验证方法
```bash
code --list-extensions --show-versions  # 查看插件路径
```
输出应显示新路径下的插件列表。
### 注意事项
1. **路径要求**：避免包含空格或特殊字符的路径（尤其 Windows）。
2. **权限问题**：确保新路径有读写权限（Linux/macOS 可能需要 `sudo`）。
3. **便携版 VS Code**：若使用便携版，插件默认存储在安装目录下的 `data/extensions`，无需额外配置。
通过以上方法，可有效将 VS Code 插件占用的空间转移到其他磁盘，释放系统盘空间。

## 为不同环境设置不同插件
1. 选择配置文件
![[vscode配置文件.png]]
2. 新建配置文件，其余可以保持默认，但拓展必须选择无
![[新建vscode配置文件.png]]