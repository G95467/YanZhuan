# 焰转 · 完整源代码

本目录包含「焰转」v1.0.2 的全部源代码：

- `src/YanZhuan.App`：WPF 宿主、WebView2 桥接、格式转换引擎（媒体 / 文档 / Word-Excel / Word-PDF / PDF-PPT / OCR / CSV-JSON / 字幕 / ESD）、设置、临时 HTTP 下载服务
- `src/YanZhuan.App/ui`：HTML / CSS / JavaScript 界面
- `src/EsdHelper`：ESD → ISO 的管理员辅助程序（DISM + IMAPI2）
- `assets`：应用图标与界面资源
- `InstallerSource`：单文件安装程序与卸载程序的完整源码
- `使用说明.txt`：应用使用说明

## 新增文档能力的实现方式

- Word -> Excel：读取 DOCX 内部 XML，把正文和表格分别写入标准 XLSX，不依赖 Office。
- Word -> PDF：Pandoc 生成自包含 HTML，再调用 Microsoft Edge 无头打印为 PDF。
- PDF -> PPT：Windows.Data.Pdf 渲染每页为 PNG，再写入带母版/主题/空白版式的 PPTX 模板。
- 图片 -> 文字：Windows.Media.Ocr，使用系统 OCR 语言包。
- 视频 -> GIF：FFmpeg 调色板/缩放转换。

这些能力不需要安装 Microsoft Office；Word/PDF/OCR 的功能依赖 Windows 自带组件。

## 运行环境

- Windows 10 / 11 x64
- .NET 10 SDK（编译时）/ .NET 10 Desktop Runtime（运行时）
- Microsoft Edge WebView2 Runtime
- 安装目录内的 `tools` 已包含 FFmpeg 与 Pandoc，编译时请保持 tools 目录存在

## 编译应用

在源码根目录执行：

```powershell
dotnet build src/YanZhuan.App/YanZhuan.App.csproj -c Release
dotnet publish src/YanZhuan.App/YanZhuan.App.csproj -c Release -r win-x64 --self-contained false -o publish
```

## 编译 ESD 辅助程序

```powershell
dotnet publish src/EsdHelper/EsdHelper.csproj -c Release -r win-x64 --self-contained false -o esd-publish
copy esd-publish/EsdHelper.exe tools/EsdHelper.exe
```

## 安装包

`InstallerSource` 内有安装程序和卸载程序的完整 C# 源码，使用 .NET Framework 自带的
csc.exe 编译为单个安装 EXE；安装包会把应用、tools、完整源代码和流光下载器安装程序副本
一起安装到同一目录。详细构建方式见 `InstallerSource/build.ps1`。

安装后的根目录会包含：

- `焰转.exe`
- `tools\`（FFmpeg、Pandoc、EsdHelper）
- `流光下载器-安装程序-v1.0.2.exe`（根目录中的安装程序副本）
- `Source\`（完整源代码）
- `使用说明.txt`、`THIRD-PARTY-NOTICES.txt`

## 大文件与流光下载器

应用不会自动安装流光下载器。源文件或转换结果达到或超过 300MB 时，界面会询问用户是否使用
流光下载器高速下载；只有用户选择允许时才打开安装目录内的安装程序，安装完成后由用户
返回应用点击重新检测。检测到安装后，应用通过本机回环地址提供带随机令牌的临时 HTTP
链接，并支持剪贴板复制。链接约 20 分钟有效，应用关闭后失效。

## 许可与第三方组件

FFmpeg（GPL）、Pandoc（GPL）、WebView2 和 Windows DISM/IMAPI2 的说明见
`THIRD-PARTY-NOTICES.txt`。流光下载器安装程序副本的版权归其作者所有，仅作为用户
明确选择后的安装入口，不静默安装。
