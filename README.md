# 格式转换工具集

一个可直接打开的图片转换网页，以及两份供支持 `SKILL.md` 的编码助手使用的转换技能：图片格式转换、Office／PDF 文档转换。

## 选择使用方式

| 入口 | 用途 | 所需环境 |
| --- | --- | --- |
| [图片转换网页](image-converter.html) | PNG、JPEG、WebP 输出，质量调整、缩放、批量下载 | 支持 Canvas 的现代浏览器 |
| [image-converter](skill/SKILL.md) | 用 Pillow 转换、缩放和批量处理图片 | 支持技能的助手、Python、Pillow |
| [document-converter](skill-docs/SKILL.md) | Office 转 PDF，从 PDF 提取可编辑内容 | 支持技能的助手、LibreOffice／Python 库 |

网页只处理图片。技能提供操作说明和代码示例，由助手结合实际文件执行，并不是独立命令行程序。

## 快速开始：图片网页

```bash
git clone https://github.com/1m01m0/format-converter.git
cd format-converter
```

在浏览器中打开 `image-converter.html`，然后：

1. 拖入图片或点击选择文件。
2. 选择 PNG、JPEG 或 WebP；JPEG／WebP 可调节质量。
3. 保留原尺寸、按比例缩放，或指定宽高。
4. 点击转换，检查预览和大小，再单独下载或批量下载。

批量下载会依次触发多个文件下载，浏览器可能要求允许多文件下载。自定义宽高直接使用输入值，不自动保持宽高比。

图片在本机通过 `FileReader` 和 Canvas 处理，当前代码没有图片上传接口。页面样式加载 Tailwind CDN，因此完整样式依赖网络，不能视为完全离线的自包含页面。

## 使用技能

```text
skill/SKILL.md            # image-converter 源定义
skill-docs/SKILL.md       # document-converter 源定义
image-converter.skill    # 图片技能归档包（ZIP）
document-converter.skill # 文档技能归档包（ZIP）
image-converter.html     # 独立图片网页
```

安装时以所用助手当前支持的技能导入方式为准。支持 `.skill` 导入的客户端可使用归档包；通过目录发现技能的客户端，应把内容放入独立技能目录，并确保 `SKILL.md` 位于该目录顶层。

也可以直接把 [图片技能说明](skill/SKILL.md) 或 [文档技能说明](skill-docs/SKILL.md) 提供给助手，说明输入路径、输出目录和转换要求。图片技能提到的网页位于仓库根目录，按需一并保留。

示例请求：

```text
把 input/ 中的 PNG 转成 JPEG，透明区域使用白色背景，质量 90，
按原比例缩小到最长边 1600 像素，输出到 output/，保留原文件。
```

```text
把 report.docx 转成 PDF，输出到 output/。
转换后检查字体、分页、表格和图片，说明发现的排版差异。
```

执行取决于助手的文件访问权限、工具能力和本机依赖。仅导入技能不会自动安装转换引擎。

## 转换能力与质量预期

| 方向 | 引擎／方法 | 需要检查 |
| --- | --- | --- |
| 常见栅格图片互转 | 技能用 Pillow；网页用 Canvas | 透明通道、色彩、尺寸、大小 |
| DOCX／PPTX／XLSX → PDF | LibreOffice headless | 字体、分页、工作表打印范围 |
| PDF → DOCX | pdf2docx | 复杂布局、表格、图片位置 |
| PDF → XLSX | pdfplumber + openpyxl 提取表格 | 单元格边界、合并单元格、数据类型 |
| PDF → PPTX | PyMuPDF 提取文字、python-pptx 创建页面 | 输出是文字骨架，需要重新排版 |

PDF 反向转换不能保证还原原始 Office 文件。扫描件需要额外 OCR；仓库未提供完整 OCR 转换流水线。图片动画、多页图像和 EXIF 等元数据也不保证保留。网页没有为透明图片转 JPEG 显式填充背景，转换后应检查底色。

## 本地依赖

图片技能的基础依赖可安装在虚拟环境中：

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install Pillow
```

文档反向转换按需要安装：

```bash
python -m pip install pdf2docx pdfplumber pymupdf python-docx python-pptx openpyxl
```

Office → PDF 需要系统中存在 LibreOffice 的 `soffice` 命令。安装后可验证一次转换：

```bash
mkdir -p output
soffice --headless --convert-to pdf --outdir output report.docx
```

将 `report.docx` 替换为真实输入文件。HEIC／HEIF、SVG 等扩展格式需要额外依赖和解码设置，不属于网页保证支持的范围。

## 排查与贡献

- **网页不能读取文件：** 确认浏览器能解码输入格式；大图和批量任务可能超过可用内存。
- **PDF 字体变化：** 在转换环境安装原文档使用的字体，再检查输出分页。
- **PDF 表格错位：** 检查是否为扫描图或复杂排版；人工核对后再使用结果。
- **技能未出现：** 检查助手的技能发现路径及解压层级，避免多嵌套一层目录。

欢迎在 [Issues](https://github.com/1m01m0/format-converter/issues) 提供格式、运行环境和可公开的最小示例。修改技能时同步检查对应 `.skill` 包；修改网页时验证透明 PNG、JPEG／WebP 输出、缩放及批量下载。

## 许可

当前仓库未附带明确的许可证文件。复用或分发前请向维护者确认许可；所用转换库与应用遵循各自许可证。
