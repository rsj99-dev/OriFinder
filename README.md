# OriFinder

Methanogenic Archaeal Plasmid ori Sequence Finder。

## 功能特点

- 🔍 **AT盒子识别**: 自动识别DNA序列中连续13个以上的AT碱基对
- 📍 **复制起点注释**: 基于AT盒子密度识别复制起点区域
- 📄 **格式转换**: 将FASTA格式转换为带注释的GenBank格式
- ⚙️ **参数可调**: 支持自定义AT盒子长度、窗口大小等参数
- 🖥️ **命令行工具**: 提供简单易用的命令行接口
- 📦 **批量处理**: 支持批量处理目录中的多个FASTA文件
- ⚡ **并行处理**: 支持多线程并行处理，提高处理效率

## 安装

### 从源码安装

```bash
git clone <repository-url>
cd OriFinder
pip install -e .
```

### 依赖要求

- Python >= 3.7
- BioPython >= 1.79

## 使用方法

### 命令行使用

#### 单文件处理

```bash
# 基本用法
orifinder input.fasta output.gb

# 自定义参数
orifinder input.fasta output.gb --min-at-length 12 --window-size 1200 --min-atbox-count 5
```

#### 批量处理

```bash
# 批量处理目录中的所有FASTA文件
orifinder --batch input_dir output_dir

# 启用并行处理（推荐）
orifinder --batch input_dir output_dir --parallel

# 自定义并行线程数
orifinder --batch input_dir output_dir --parallel --max-workers 8

# 不递归搜索子目录
orifinder --batch input_dir output_dir --no-recursive

# 批量处理with自定义参数
orifinder --batch input_dir output_dir --parallel --min-at-length 12 --window-size 1200
```

#### 命令行参数说明

**基本参数：**
- `input_file`: 输入的FASTA文件路径（单文件模式）或输入目录路径（批量模式）
- `output_file`: 输出的GenBank文件路径（单文件模式）或输出目录路径（批量模式）

**分析参数：**
- `--min-at-length`: AT盒子的最小长度（默认: 13）
- `--window-size`: 分析ori区域的滑动窗口大小（默认: 1000）
- `--min-atbox-count`: 窗口内AT盒子的最小数量来标记为ori（默认: 4，即大于3个）

**批量处理参数：**
- `--batch`: 启用批量处理模式
- `--parallel`: 启用并行处理（批量模式）
- `--max-workers`: 最大并行线程数（默认: 4）
- `--recursive`: 递归搜索子目录（默认启用）
- `--no-recursive`: 不递归搜索子目录

### Python API使用

#### 单文件处理

```python
from orifinder import ATBoxFinder

# 创建分析器实例
finder = ATBoxFinder(
    min_at_length=13,    # AT盒子最小长度
    window_size=1000,    # 滑动窗口大小
    min_atbox_count=4    # ori区域最小AT盒子数量
)

# 处理FASTA文件
finder.process_fasta("input.fasta", "output.gb")
```

#### 批量处理

```python
from orifinder import BatchProcessor

# 创建批量处理器实例
processor = BatchProcessor(
    min_at_length=13,    # AT盒子最小长度
    window_size=1000,    # 滑动窗口大小
    min_atbox_count=4,   # ori区域最小AT盒子数量
    max_workers=4        # 最大并行线程数
)

# 批量处理目录中的所有FASTA文件
summary = processor.process_batch(
    input_dir="input_directory",
    output_dir="output_directory",
    recursive=True,      # 递归搜索子目录
    parallel=True        # 启用并行处理
)

# 查看处理结果
print(f"总文件数: {summary['total_files']}")
print(f"成功处理: {summary['successful']}")
print(f"处理失败: {summary['failed']}")
```

#### 高级用法

```python
from orifinder import ATBoxFinder
from Bio import SeqIO

# 创建分析器
finder = ATBoxFinder()

# 读取序列
for record in SeqIO.parse("input.fasta", "fasta"):
    sequence = str(record.seq)
    
    # 寻找AT盒子
    at_boxes = finder.find_at_boxes(sequence)
    print(f"发现 {len(at_boxes)} 个AT盒子")
    
    # 寻找ori区域
    ori_regions = finder.find_ori_regions(sequence, at_boxes)
    print(f"发现 {len(ori_regions)} 个ori区域")
```

## 算法原理

### AT盒子识别


1. 使用正则表达式 `[AT]{13,}` 识别连续的AT碱基对
2. 默认最小长度为13个碱基对，可通过参数调整
3. 对每个识别到的AT盒子进行位置标记

### 复制起点(ori)识别

1. 使用滑动窗口方法分析整个DNA序列
2. 默认窗口大小为1000个碱基对
3. 统计每个窗口内的AT盒子数量
4. 当窗口内AT盒子数量大于3个时，标记为ori区域
5. 合并重叠的ori区域

### 输出格式

生成的GenBank文件包含以下注释：

- **ATbox**: AT盒子区域，标记为 `misc_feature`
- **ori**: 复制起点区域，标记为 `rep_origin`

## 示例

### 单文件处理示例

```bash
# 处理单个FASTA文件
orifinder plasmid.fasta plasmid_annotated.gb
```

### 批量处理示例

```bash
# 批量处理质粒目录中的所有FASTA文件
orifinder --batch "质粒fasta" "批量处理结果" --parallel --max-workers 8
```

这将：
1. 搜索"质粒fasta"目录中的所有FASTA文件
2. 使用8个线程并行处理
3. 将结果保存到"批量处理结果"目录
4. 生成详细的处理报告

## 输出文件查看

生成的GenBank文件可以使用以下软件查看：

- **SnapGene**: 专业的分子生物学软件
- **Geneious**: 生物信息学分析平台
- **ApE (A plasmid Editor)**: 免费的质粒编辑器
- **Benchling**: 在线分子生物学平台
- 任何支持GenBank格式的生物信息学软件

## 技术细节

### 文件结构

```
OriFinder/
├── orifinder/
│   ├── __init__.py
│   ├── core.py          # 核心算法实现
│   └── cli.py           # 命令行接口
├── example.py           # 使用示例
├── setup.py            # 安装配置
└── README.md           # 说明文档
```

### 核心类和方法

- `ATBoxFinder`: 主要分析类
  - `find_at_boxes()`: 识别AT盒子
  - `find_ori_regions()`: 识别ori区域
  - `process_fasta()`: 处理FASTA文件

- `BatchProcessor`: 批量处理类
  - `find_fasta_files()`: 查找FASTA文件
  - `process_single_file()`: 处理单个文件
  - `process_batch()`: 批量处理文件

## 许可证

MIT License

## 贡献

欢迎提交Issue和Pull Request来改进这个项目。

## 更新日志

### v0.1
- 初始版本发布
- 实现AT盒子识别功能
- 实现ori区域注释功能
- 提供命令行接口
- 支持FASTA到GenBank格式转换
