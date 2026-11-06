![image](images/logo.png)
# Citrus

[English](README.md) | [简体中文](README.zh-CN.md)

Citrus 是一款面向长读长测序数据的 barcode（条形码）/ 引物拆分工具：读入测序数据，定位每条 read 两端的 barcode，再把 read 按 barcode 分流到各自的输出文件。基因组数据的预处理一步到位，下游分析拿到的数据也更干净。

> 本文档是 [English README](README.md) 的中文版本。两者内容如有出入，以英文版为准。

## 目录
- [安装](#安装)
- [快速上手](#快速上手)
- [命令行参数](#命令行参数)
- [Barcode 文件](#barcode-文件)
- [预定义 Barcode Kit](#预定义-barcode-kit)
- [拆分原理](#拆分原理)
- [输出](#输出)
- [性能](#性能)
- [常见问题](#常见问题)
- [作者](#作者)
- [许可与使用限制](#许可与使用限制)

## 安装
**解压即用**

Citrus 不需要编译安装，也不依赖额外的运行时：把下载到的压缩包解压到系统里任意位置，就能直接运行。

**使用步骤：**
1. 下载与你的 CPU 架构（`x86_64` 或 `aarch64`）对应的 `.tar.gz` 文件。
2. 解压到目标位置。
3. 运行 `citrus` 即可启动。

发布版二进制以动态链接方式依赖 glibc，要求宿主机 glibc 版本不低于 2.28（RHEL 8/9、Rocky 8/9、Debian 10+、Ubuntu 20.04+、ARM 服务器）。除此之外没有任何外部依赖。

## 快速上手
**Mode 1 barcode 拆分**
```bash
citrus -m 1 -f sample.fq.gz -b barcodes.txt -o output_dir -n sample -e 0.2 -s 150,100 -t threads
```
**Mode 2 barcode 拆分**
```bash
citrus -m 2 -f sample.fq.gz -b barcodes.txt -o output_dir -n sample -e 0.2 -s 150,100 -t threads
```
**多个输入文件**
```bash
citrus -m 1 -f sample1.fq.gz sample2.fq.gz -b barcodes.txt -o output_dir -n sample
```
**BAM 输入**
接受携带 MM/ML 标签的未比对 BAM 文件；其他所有标签原样透传。
```bash
citrus -m 1 --bam sample.bam -b barcodes.txt -o output_dir -n sample
```
**预定义 barcode kit**
```bash
citrus -m 2 -f sample.fq.gz -k CycloneSEQ_24_Barcode_Library_Prep_Set -o output_dir -n sample
```

## 命令行参数
| 参数 | 说明 | 必需 | 默认值 |
| :---: | --- | :---: | :---: |
| `-m, --mode` | 拆分模式。1：拆分 5' 端 barcode，若同时识别到 3' 端 barcode 则一并拆分；2：要求 5' 端和 3' 端都拆分 | 是 | - |
| `-f, --fastq` | 输入一个或多个 fastq 文件，多个文件用空格分隔 | 是¹ | - |
| `--bam` | 输入一个或多个携带 MM/ML 标签的未比对 BAM 文件，多个文件用空格分隔 | 是¹ | - |
| `-k, --kits` | 使用预定义的 barcode kit，与 `-b` 互斥 | 是² | - |
| `-b, --barcode` | 输入一个或多个 barcode 文件。若一条 read 两端都有 barcode，请在输入文件中用制表符分隔；当两端的 barcode 互为反向互补时，只需提供 5' 端 barcode；若一条 read 的一端有两个 barcode，barcode 文件中的序列应以逗号分隔 | 是² | - |
| `-n, --sample` | 样本名 | 是 | - |
| `-t, --thread` | 线程数 | 否 | 4 |
| `-o, --outdir` | 输出目录 | 是 | - |
| `-q, --testseq` | 输入测试序列文件，用于评估拆分准确度 | 否 | - |
| `-L, --min_len` | 短于最小长度的序列在读取输入时即被丢弃：不写入任何文件，也不计入统计。v3.3.0 之前，这类 read 会写入 `{sample}.length_filtered.fq.gz` | 否 | 0 |
| `-Q, --min_qual` | 由碱基质量计算（`-10·log10(mean(10^(-Q_i/10)))`，而非 read ID 中的 qscore）得到的 read 质量低于该阈值时，该 read 会被丢弃 | 否 | 7.0 |
| `-e, --err` | Levenshtein 编辑距离阈值：填整数表示绝对距离，填 0 到 1 之间的小数表示测序错误率。当 5' 端和 3' 端使用不同阈值时，用逗号分隔两个数值 | 否 | 0.2,0.2 |
| `-s, --shift` | 用于识别 barcode 序列的截取长度阈值。当 5' 端和 3' 端使用不同阈值时，用逗号分隔两个数值 | 否 | 150,100 |
| `-u, --trim_len` | 3' 端测序不完整时，为识别 barcode 而额外裁剪的长度 | 否 | 0 |
| `-c, --seed_size` | 种子（seed）长度 | 否 | 6 |
| `-d, --step_size` | 切取种子的步长 | 否 | 1 |
| `-r, --retain` | 在 read 上保留 barcode 序列 | 否 | - |
| `-g, --degenerate` | 允许 barcode 和引物中包含简并碱基 | 否 | - |
| `-i, --intermediate` | 输出中间结果，记录每条 read 的匹配细节 | 否 | - |
| `-F, --filter_chimeric` | 过滤嵌合体 read | 否 | - |
| `-a, --append` | 追加到已有输出文件，而不是覆盖 | 否 | - |
| `-h, --help` | 打印帮助信息 | 否 | - |
| `-V, --version` | 打印版本号 | 否 | - |

¹ `--fastq` 与 `--bam` 必须指定其一，两者互斥。
² `--barcode` 与 `--kits` 必须指定其一，两者互斥。

**关于默认值：** 使用 `-k` 指定预定义 kit 时，kit 自带的调优参数会取代上表中的默认值；命令行中显式设置的参数始终优先。

## Barcode 文件
barcode 文件以制表符分隔，包含两列或三列：`ID`、`5' 端 barcode 序列`，以及可选的 `3' 端 barcode 序列`。`ID` 可以是 barcode 编号，也可以是样本名、物种名、组织名等便于识别的名称。同一个 `ID` 也可以对应多组 barcode 序列。
如下例所示，若 `ID` 为 `BC1`，barcode 文件内容如下：
<table><tr><td>BC1</td><td>ATCG</td><td>TCAG</td></tr></table>

![image](images/barcode_demo.png)

**单端 barcode** —— 每行一个 barcode，两端使用同一序列：

```
barcode_id	barcode_sequence
BC01	ATCGATCG
BC02	GCTAGCTA
```

**双端 barcode** —— 每行两个 barcode：

```
barcode_id	5p_barcode	3p_barcode
BC01	ATGC	GGTA
BC02	TTGCATCA	CCTAGGTA
```

当 5' 和 3' 接头不同时，把同一 barcode ID 的两个接头写在同一行。Citrus 会自动识别该接头对的两个测序方向。例如：

```
BC01	ATGC	GGTA
```

该配置可匹配如下正向 read：

```
5' ATGC ... TACC 3'
```

其中 `TACC` 是 `GGTA` 的反向互补序列。它也能匹配相反方向：

```
5' GGTA ... GCAT 3'
```

其中 `GCAT` 是 `ATGC` 的反向互补序列。两种方向的 read 都会写入 `BC01` 的输出。如果要求 5' 和 3' 接头都必须检出，请使用 `--mode 2`；如果允许仅凭 5' 接头对 read 分类，请使用 `--mode 1`。

**不要列出镜像配对。** 如果 5' 和 3' 接头取自同一个共享池，并且列出了所有组合，那么 `(i, j)` 与 `(j, i)` 两种配对是重复的：

```
barcode_id	5p_barcode	3p_barcode
LB1_LB1	ATCGATCG	ATCGATCG
LB1_LB2	ATCGATCG	GCTAGCTA
LB2_LB2	GCTAGCTA	GCTAGCTA
LB2_LB1	GCTAGCTA	ATCGATCG
```

Citrus 会识别每个接头对的两种测序方向，因此 `LB1_LB2` 和 `LB2_LB1` 的匹配规则完全相同。一条 read 两端各带一个接头，而接头对 `ATCGATCG` / `GCTAGCTA` 与镜像配对 `GCTAGCTA` / `ATCGATCG` 在 read 上无法区分，于是两个 ID 的 read 都会写入按名称排序在前的那个 ID，镜像 ID 保持为空。程序不会给出任何警告。

> **提示。** 只列出两种顺序中的一种，例如所有 `i <= j` 的组合（N 个 barcode 对应 N(N+1)/2 个 ID：24 个 barcode 的 pool 为 300 个，96 个 barcode 的 pool 为 4656 个）。这样任一方向的 read 都会归到该 ID 下。
>
> 如果把所有组合都列出来，同一个 5' 接头会在多个 ID 中重复出现，导致一条 read 得到多个同样好的 5' 候选。在 `--mode 1` 下，这会禁用「仅凭 5' 接头分类」的兜底逻辑，3' 接头匹配不上的 read 会作为 LD-filtered 被丢弃。这种排布请使用 `--mode 2`。

## 预定义 Barcode Kit
用 `-k` 代替 `-b`，即可直接使用内置的 barcode kit 拆分：

| Kit | 别名 | barcode 数量 |
| :---: | --- | :---: |
| `CycloneSEQ_24_Barcode_Library_Prep_Set` | `24_Ligation_Barcodes` | 24 |
| `CycloneSEQ_96_Barcode_Library_Prep_Set` | `96_Ligation_Barcodes` | 96 |
| `CycloneSEQ_Rapid_24_Barcode_Library_Prep_Set` | - | 24 |

每个 kit 都自带一套调优参数。使用 `-k` 时，这套参数就是**实际生效的默认值**；不使用 `-k`（自己提供 `-b` 文件）时，生效的则是上文参数表里的通用默认值。命令行中显式设置的参数，在两种情况下都优先。三个 kit 的参数集合完全相同：

| 参数 | kit 值（`-k` 下生效） | 通用默认值（`-b` 下生效） |
| :--- | :---: | :---: |
| `--err` | `0.25,0.25` | `0.2,0.2` |
| `--shift` | `150,100` | `150,100` |
| `--seed_size` | 6 | 6 |
| `--step_size` | 1 | 1 |
| `--min_len` | 0 | 0 |
| `--min_qual` | 7.0 | 7.0 |
| `--trim_len` | 6 | 0 |

两套默认值只有两处不同：错误阈值更宽松（`--err`），以及 3' 端测序不完整时为识别 barcode 而裁剪 6 bp（`--trim_len`）。注意 `citrus --help` 显示的始终是通用默认值，并不是 `-k` 模式下的实际生效值。

## 拆分原理
Citrus 采用 **seed-and-extend（种子定位 + 延伸）** 策略：先用速度快的 k-mer 匹配把 barcode 可能出现的位置缩小到少数几个候选，再用 Levenshtein 编辑距离（LD）比对确认精确位置和精确误差。因此 barcode 不必与 read 完全一致——测序错误和小片段插入缺失只要在 `--err` 阈值内都可以容忍。

两端 barcode 碱基被切除后，read 才会写入对应 barcode 的输出：

```
5' ATGC ... insert ... TACC 3'   →   insert
   └ barcode ┘            └ 3' barcode, reverse-complemented ┘
```

1. **建立种子索引** —— 每个 5' 端 barcode 按 `--seed_size` 的长度（步长 `--step_size`）切成互相重叠的 k-mer，存入哈希表，每个 k-mer 指向包含它的 barcode。索引只构建一次，所有 worker 线程共享同一份。
2. **种子匹配与聚类** —— 用相同的 k-mer 窗口扫描 5' 端前 `--shift` 个碱基，按 barcode 收集命中位置并聚类；同一个密集簇中命中数最多的 barcode 成为候选，来自重复区或偶然相似性的零散命中被剔除。
3. **编辑距离验证** —— 对每个候选 barcode，在其种子簇附近的小窗口内做半全局 Levenshtein 比对，得到精确的编辑距离和 barcode 的结束坐标。距离在 `--err` 阈值以内（可以是绝对 LD，也可以是 barcode 长度的比例）时候选通过。
4. **3' 端处理** —— 末尾 `--shift` 个碱基以同样的方式匹配，3' 端 barcode 在该处以反向互补的形式出现。`--mode 2` 要求两端都匹配上；`--mode 1` 下若没找到 3' 端 barcode，就仅按 5' 端 barcode 分类，找到时则两端都切除。
5. **切除与输出** —— 匹配到的 barcode 碱基被切除（除非指定了 `--retain`），插入片段写入该 barcode 的输出。任何一步被否决的 read 都会进入 `Discarded/` 下对应的文件。

由于编辑距离这一步只在种子簇附近的小窗口内进行，每条 read 的比对开销被 `--shift` 限定住，不会随 read 长度增长。启用 `-F/--filter_chimeric` 后，Citrus 还会扫描 5' 端 barcode 切除后剩下的插入片段；如果再出现第二片密集的 barcode 种子区域，这条 read 就会被判定为嵌合体。

## 输出
```
output/
├── {sample}_{barcode_id}/
│   ├── {sample}_{barcode_id}_01.fastq.gz            # 拆分后的 read
│   └── {sample}_{barcode_id}_01.fastq.stats.tsv     # 该 barcode 的 read / 碱基计数
├── Discarded/
│   ├── {sample}.length_filtered.fq.gz               # 长度过滤的 read
│   ├── {sample}.qval_filtered.fq.gz                 # 质量过滤的 read
│   ├── {sample}.ld_filtered.fq.gz                   # 编辑距离过滤的 read
│   └── {sample}.chimeric.fq.gz                      # 嵌合体 read
├── Intermediate_results/                            # 使用 -i 时生成
│   ├── {sample}.splitted_record.tsv
│   ├── {sample}.ld_filtered_record.tsv
│   └── {sample}.chimeric_record.tsv
├── {sample}_demux.stats.txt                         # 详细统计报告
├── {sample}_barcodes.stats.tsv                      # 每个 barcode 的统计表
└── {sample}.barcode_cumulative.json                 # 累计 JSON 摘要
```

BAM 输入时，拆分输出和丢弃输出使用 `.bam` 扩展名而非 `.fastq.gz`/`.fq.gz`，每个 barcode 的统计文件名为 `{sample}_{barcode_id}_01.bam.stats.tsv`。

### 拆分结果与被丢弃的 read
`{sample}_{barcode_id}_01.fastq.gz` 保存分配给该 barcode 的 read，为标准 FASTQ 格式（每条 read 四行），gzip 压缩。默认会从 read 两端切除 barcode 序列；`--retain` 则把它们保留在输出中。

`Discarded/` 下的四个文件收集未能归入任何 barcode 的 read，每个类别一个文件：

| 文件 | 收录的 read |
|---|---|
| `{sample}.length_filtered.fq.gz` | 通过了 `--min_len`，但因其他长度原因被丢弃（短于最长 barcode，或切除后没有剩余插入片段）；自 v3.3.0 起，该文件不再包含短于 `--min_len` 的 read |
| `{sample}.qval_filtered.fq.gz` | 由碱基质量计算得到的 read 质量（把平均错误概率换算回 Q，而不是 read ID 中的 qscore）低于 `--min_qual` |
| `{sample}.ld_filtered.fq.gz` | 在 `--err` 阈值内没有匹配到任何 barcode |
| `{sample}.chimeric.fq.gz` | 被判定为嵌合体的 read；只有使用 `-F` 时才会检出该类别，否则文件为空 |

不使用 `-F` 时，嵌合体 read 不会被检出，会留在它所匹配到的那个 barcode 的拆分输出中。

### `{sample}_{barcode_id}_01.fastq.stats.tsv`
位于每个 barcode 子目录下，是一个带表头行和一行数据的 TSV 文件：

```
TotalReadNum	TotalBaseNum	PassBaseNum
{read_count}	{base_count}	{pass_base_count}
```

| 列 | 含义 |
|--------|---------|
| `TotalReadNum` | 分配给该 barcode 的 read 数（与下文的 `Read_count` 相同） |
| `TotalBaseNum` | 这些 read 的碱基总数，按原始 read 长度计算 |
| `PassBaseNum` | 切除 barcode 后剩余的碱基数（与下文的 `Passed_base_count` 相同） |

`TotalReadNum` 和 `TotalBaseNum` 不受 `--retain` 影响；`PassBaseNum` 会受影响，使用 `--retain` 时它等于 `TotalBaseNum`。这些数值是累计值，即包含之前用 `-a` 追加的运行结果。丢弃类别没有子目录，也不会生成该文件。

### `{sample}_barcodes.stats.tsv`
每个 barcode 一行的 TSV 表：一行表头，之后每个分类成功的 barcode 一行，丢弃类别不列出。行按 barcode ID 的自然顺序排序——连续数字按数值比较，因此 `barcode2` 排在 `barcode10` 之前。

**普通模式（默认）的列：**

| 列 | 含义 |
|--------|---------|
| `Barcode` | barcode ID |
| `Read_count` | 分配给该 barcode 的 read 数 |
| `Base_count` | 该 barcode 的碱基总数，按原始 read 长度求和 |
| `Passed_base_count` | 切除后保留序列的碱基总数 |
| `Read_N50` | 按切除后 read 长度分布计算的 N50 |
| `Read_mean_Q` | 该 barcode 的平均 read Q，取自每条 read ID 中的 Q 值，在错误率空间按 read 碱基数加权平均（保留两位小数）。ID 中无法解析出 Q 的 read 会被排除；若该 barcode 下的所有 read 都不带 Q，值为 `0.00` |

**准确度模式（`-q`）在上述 6 列之后追加 8 列：**

| 列 | 含义 |
|--------|---------|
| `True_read_count` | 验证为正确分类的 read 数 |
| `False_read_count` | 验证为错误分类的 read 数 |
| `Short_read_count` | 过短而无法验证的 read 数 |
| `Chimeric-I_read_count` | 由种子密度检出的嵌合体 read 数 |
| `Chimeric-II_read_count` | 由两条测试序列匹配检出的嵌合体 read 数 |
| `Unknown_read_count` | 未知 / 未确定的 read 数 |
| `Accuracy(%)` | `True_read_count` 占已验证 read 的百分比 |
| `Index_hopping_rate(%)` | `False_read_count` 占已验证 read 的百分比 |

**`--retain` 对该文件的影响：** `Read_count` 和 `Base_count` 不变；`Passed_base_count` 和 `Read_N50` 增大，因为它们统计的是切除 barcode 之后的序列，而 `--retain` 把这段序列留了下来。

### `{sample}_demux.stats.txt`
纯文本格式的完整统计报告，由三部分组成：

1. **每个 barcode 的表格**：列与 `{sample}_barcodes.stats.tsv` 相同（准确度模式下包含准确度列），按相同的自然顺序排序。
2. **类型 / Read / Base 汇总表**：跨 barcode 的合计，每个类别一行，包含 read 数、碱基数及其占总量的百分比：

   | 行 | 含义 |
   |-----|---------|
   | `Total` | 全部 read / 碱基 |
   | `Demultiplexed_count` / `Demultiplexed_percentage` | 成功分配到一个 barcode |
   | `Length-filtered_count` / `Length-filtered_percentage` | 长度过滤 |
   | `Qval-filtered_count` / `Qval-filtered_percentage` | 质量过滤 |
   | `LD-filtered_count` / `LD-filtered_percentage` | 编辑距离过滤 |
   | `Chimeric-filtered_count` / `Chimeric-filtered_percentage` | 嵌合体过滤 |
   | `CV` | 各 barcode 之间 read / 碱基计数的变异系数 |

   准确度模式会追加 `Accuracy(%)` 和 `Index_hopping_rate(%)` 两行。
3. **Barcode 分布矩阵**（仅准确度模式）：以真实 barcode 为行、预测 barcode 为列的混淆矩阵，直观展示 index hopping 的流向。行和列遵循与 barcode 表格相同的自然 ID 顺序。

### `{sample}.barcode_cumulative.json`
累计 JSON 摘要，**仅在普通模式下生成**（准确度模式不生成）。顶层字段：

| 字段 | 含义 |
|-------|---------|
| `run_total_passed_reads` / `run_total_passed_bases` | 通过 QC 的 read / 碱基总数 |
| `run_classified_reads` / `run_classified_bases` | 已分类的 read / 碱基计数 |
| `run_unclassified_reads` / `run_unclassified_bases` | 未分类（丢弃）的 read / 碱基计数 |
| `run_classified_reads_rate` / `run_classified_bases_rate` | 对应比例 |
| `run_unclassified_reads_rate` / `run_unclassified_bases_rate` | 对应比例 |
| `run_barcode_cumulatvie_info` | 数组，每个 barcode 一项 |

`run_barcode_cumulatvie_info` 中的每一项描述一个 barcode：

| 字段 | 含义 |
|-------|---------|
| `barcode_id` | barcode ID |
| `sequence` | 该 ID 注册的 barcode 序列 |
| `reads` / `bases` | 该 barcode 的 read 数和碱基数 |
| `reads_rate` / `bases_rate` | 该 barcode 占已分类 read / 碱基的比例 |
| `split_rate` | 保留字段，目前恒为 `0.0` |
| `reads_q_val` | 该 barcode 的众数 read Q |
| `total_reads_base` | 切除 barcode 后剩余的碱基数 |
| `N50` | 切除后 read 长度分布的 N50 |
| `q_bins` / `q_counts` | read 质量直方图：分箱边界（80.0 到 100.0，步长 0.1）及各分箱内的 read 数 |

该文件支持通过 `-a` 跨多次运行累计。

### `Intermediate_results/`（使用 `-i`）
三个 TSV 文件，每个都是一行表头加每条 read 一行：

| 文件 | 列 |
|------|---------|
| `{sample}.splitted_record.tsv` | `Record_ID`、`Barcode_ID`、`LD_of_5_end`、`End_pos_of_5_end`、`Seed_match_pos`、`LD_of_3_end`、`End_pos_of_3_end` |
| `{sample}.ld_filtered_record.tsv` | `Record_ID`、`Barcode_ID`、`LD`、`Seed_match_pos` |
| `{sample}.chimeric_record.tsv` | `Record_ID`、`Seed_count_per_win` |

这些文件用于单条 read 的可视化与问题排查。

## 性能
以下数据来自对同一批 **1000 万条 read** 的基准测试，使用 Citrus v3.4 拆分。除特别说明外，各次运行的版本、barcode 文件和所有参数完全一致，只改变输入格式或线程数。

### 输入格式：fastq 与 modBAM
同一批 read 拆分两次：一次从 17 GB 的 fastq 文件（`--fastq`）读入，一次从等价的 12 GB modBAM 文件（`--bam`）读入，均使用 32 线程。

| 输入 | 耗时 | 平均 CPU | 峰值内存 |
|---|---|---|---|
| fastq（`-f`） | 425 s | 4.4 核 | 2.3 GB |
| modBAM（`--bam`） | 333 s | 6.5 核 | 3.7 GB |

![fastq vs modBAM input](images/perf_input_formats.png)

modBAM 输入比 fastq 输入大约**快 20% 到 30%**，拆分结果等价。提速来自更高的并行度，而并行度的差异则源于输入解码方式的不同。modBAM 侧的 MM/ML 标签重算实测开销可以忽略，不是性能瓶颈。

### 线程数扩展性
同样这 1000 万条 read，分别用 4、8、16、32 线程拆分：

| 线程数 | fastq | modBAM |
|---|---|---|
| 4 | 990 s | 702 s |
| 8 | 507 s | 378 s |
| 16 | 411 s | 334 s |
| 32 | 455 s | 330 s |

![thread-count sweep](images/perf_thread_sweep.png)

CPU 利用率随请求的线程数非线性增长。超过 16 线程后基本不再提升：fastq 反而变慢（32 线程 455 s，16 线程是 411 s），modBAM 也只再快 4 s。

## 常见问题

### 1. `--mode 1` 和 `--mode 2` 有什么区别？

两种模式定位 5' 端 barcode 的方式完全相同，对能找到 3' 端 barcode 的 read，结果也一致。区别只体现在**没找到 3' 端 barcode** 时如何处理。

| | `--mode 1` | `--mode 2` |
|---|---|---|
| 5' 端 barcode | 必需 | 必需 |
| 3' 端 barcode | 可选 | 必需 |
| 没匹配到 3' 端 barcode 的 read | 仅按 5' 端 barcode 分类 | 作为 LD-filtered 丢弃 |
| 切除 | 找到 3' 端 barcode 时切除两端，否则只切 5' 端 | 总是切除两端 |

`--mode 1` 的兜底逻辑有两个细节：

- **只在 5' 端归属唯一时才生效**，也就是恰好只有一个 barcode 的编辑距离最小。若多个 barcode 并列，read 会像 `--mode 2` 一样被当作 LD-filtered 丢弃。这正是 [Barcode 文件](#barcode-文件) 中「列出全部组合」那种排布不能用于 `--mode 1` 的原因：同一个 5' 端接头被多个 ID 共用，5' 端归属永远不唯一。
- **仅按 5' 端 barcode 分类时，3' 端不会被切除**，因此输出 read 仍然带着那一端测到的序列。

**如何选择：** 两端都保证存在时（双端 index 文库、双端 barcode kit）使用 `--mode 2`；3' 端经常缺失、截断或不可靠时使用 `--mode 1`。`--mode 1` 能拆分 `--mode 2` 能拆分的全部 read，外加只带可用 5' 端 barcode 的 read，所以拿不准时它是更稳妥的选择——代价是这些多出来的 read 仅仅凭 5' 端 barcode 就被接受。

### 2. 如何把目录下全部（或部分）FASTQ/BAM 文件传给 `--fastq`/`--bam`？

`--fastq` 和 `--bam` 接受一个或多个明确的文件路径，用空格分隔。Citrus 本身既不遍历目录也不展开通配符——展开由 shell 完成，因此直接使用 shell 通配符即可：

```bash
# 目录下所有 FASTQ
citrus -m 1 -f /data/run1/*.fq.gz -b barcodes.txt -o out -n sample

# 带编号的文件，按序号顺序展开（bash 花括号展开）
citrus -m 1 -f /data/run1/sample_{1..10}.fq.gz -b barcodes.txt -o out -n sample

# 选定的子集
citrus -m 1 -f /data/run1/A.fq.gz /data/run1/B.fq.gz -b barcodes.txt -o out -n sample

# 目录下所有 BAM
citrus -m 1 --bam /data/run1/*.bam -b barcodes.txt -o out -n sample

# 递归包含子目录（sort -z 保证顺序确定）
find /data -name '*.fq.gz' -print0 | sort -z | xargs -0 citrus -m 1 -b barcodes.txt -o out -n sample -a -f
```

几个容易踩的坑：

- **不要给通配符加引号。** 加了引号的 `"*.fq.gz"` 会原样传给 Citrus，并以 `No such file or directory` 报错。通配符完全没有匹配到文件时也一样——先用 `ls` 确认。
- **第一个文件必须可读。** 无法打开时整次运行中止。列表中靠后的文件无法打开时只会给出警告并跳过，运行继续处理其余文件——请留意日志中的 `Warning: failed to open ...`。
- **不要传目录。** Citrus 期望的是文件，传目录会导致运行失败。
- **`xargs` 可能把过长的文件列表拆成多次 Citrus 运行。** 它默认串行执行，因此多次运行不会重叠——但每次运行都写入同一个输出目录并覆盖上一次写入的内容，最终只有最后一批保留下来。上面的示例因此传入了 `--append`，让各批结果累加；注意把同一条命令再次运行到同一目录会再追加一遍。追加要求每次运行使用相同的 barcode 集合和相同的 `--shift` 阈值，否则 Citrus 会报错退出。**不要**加 `-P`（并行）：多个 Citrus 进程写同一个输出目录会损坏输出的 FASTQ/BAM 文件和 `.summary.bincode` 文件。
- **文件按给定顺序读取**，并拼接成单一的 read 流。`find` 返回的是目录顺序而非排序后的结果，因此需要确定顺序时请通过 `sort -z`（自然顺序用 `sort -zV`，`sample_2` 排在 `sample_10` 之前）处理。注意线程数大于 1 时 read 由并行 worker 写出，输出文件内的顺序本来就不是输入顺序。
- `--fastq` 与 `--bam` 互斥；两者都有时请分两次运行。

**带编号的文件集。** 在 bash 中，`{1..10}` 会在通配符解析之前展开为十个独立参数，因此 `sample_{1..10}.fq.gz` 变成 `sample_1.fq.gz` … `sample_10.fq.gz`。有两点值得注意：

- 文件按**序号顺序**（1、2、……、10）读取。单纯的 `*` 通配符则按字典序排列，会把 `sample_10` 紧接在 `sample_1` 之后。顺序不影响结果，只决定 read 进入流程的先后。
- 不存在的文件会把自己的名字留在参数列表里，Citrus 随后会尝试打开这个名字：如果它是第一个文件，运行中止；否则给出警告并跳过。如果需要避免这种情况，请事先剔除空缺：

  ```bash
  files=(); for i in {1..10}; do f=/data/run1/sample_${i}.fq.gz; [ -f "$f" ] && files+=("$f"); done
  citrus -m 1 -f "${files[@]}" -b barcodes.txt -o out -n sample
  ```

花括号展开是 bash/zsh 的特性——POSIX `sh`（dash）不支持，会原样把模式交给 Citrus。

### 3. 为什么输入已经通过长度和质量过滤，仍有一些 read 因长度和质量被过滤？

因为这两个过滤都不是简单地与阈值比一次大小，而且这里用到的 Q 值也不是 read ID 中打印的那个 qscore。

**长度——`--min_len` 不是唯一的长度过滤。** 另外两种长度原因也会让长度超过 `--min_len` 的 read 进入 `Discarded/{sample}.length_filtered.fq.gz`：

- **短于最长 barcode**（`barcode_len_max`）：这类 read 不可能包含 barcode，因此在匹配前就被丢弃。
- **切除后没有剩余序列**：barcode 位于 read 的最末端，或者比对消耗了整条 read（barcode/接头二聚体的典型情况），切除 barcode 后没有插入片段剩下。

该文件中的记录以未切除的形式写出（原始 SEQ/QUAL），因此长度可能超过 `--min_len`——在 "length-filtered" 文件中看到 100 bp 甚至更长的 read 是正常的。

本身短于 `--min_len` 的 read 走的是另一条路径：自 v3.3.0 起，普通模式在读取输入时就会跳过它们，因此它们不会写入任何文件，也不计入统计（`{sample}_demux.stats.txt` 的 `Total` 行因此可能小于输入记录数）。准确度模式（`--testseq`）没有读取阶段的预过滤；在那里它们会进入 `length_filtered`。如果输入来自之前的一次 Citrus 运行，还有一点需要注意：那次运行的 `--min_len` 检查看到的是未切除的 read，而它的输出是切除后的，因此一条带两个 16 bp barcode 的 132 bp read 在当时以 132 bp 通过了检查，进入本次运行却只有 100 bp（而 100 bp 的 read 进入时只剩 68 bp）。

**质量——read Q 由碱基质量计算，而不是从 read ID 读取。** Citrus 使用整数 QUAL 字符串：

```
read_q = -10 · log10( mean( 10^(-Q_i / 10) ) )
```

对到达时的**整条 read** 计算。read ID 中的 qscore 由 basecaller 写入，遵循另一套口径，且这套口径在 basecaller 版本间发生过变化：

- **旧版本**基于 read 的首尾 chunk 计算，忽略中间部分。
- **当前版本**对长度不低于 100 bp 的 read 忽略首尾 40 bp，否则按全长计算，并采用同样的错误概率公式。

因此这两个量是对不同碱基集合求的平均值；此外 Citrus 基于整数 QUAL（上限 Q40）计算，而 basecaller 使用其内部值。在短 read 上，或中间段质量较低的 read 上，两者可能相差约 ±0.5–1 Q，因此 ID 中显示 `qscore 8` 的 read 仍可能被 `--min_qual 7` 过滤。如果之前的过滤是由其他工具基于 read ID 的 qscore 而非碱基质量执行的，也存在同样的情况。

如果输入来自之前的一次 Citrus 运行，还有一点不同：那次运行的 `--min_qual` 检查针对的是包含 barcode 的原始 read，而本次运行评估的是切除后的 read；去掉 barcode 碱基可能让原本卡在阈值边缘的 read 跨过阈值（我们的测试中为 7.44 → 6.69）。

**快速定位一条消失的 read：**

| 这条 read 在哪里 | 被什么过滤了 |
|---|---|
| 任何文件中都没有，且 `Total` 计数偏少 | `--min_len`，在读取阶段应用（普通模式） |
| `Discarded/{sample}.length_filtered.fq.gz` | 通过了 `--min_len`，但短于 `barcode_len_max`，或切除后没有剩余插入片段 |
| `Discarded/{sample}.qval_filtered.fq.gz` | Citrus 计算出的 read Q 低于 `--min_qual` |
| `Discarded/{sample}.ld_filtered.fq.gz` | 在 `--err` 阈值内没有匹配到 barcode |

> **提示。** 默认情况下 barcode 会从输出 read 上切除，因此 Citrus 的输出文件无法再用同一套 barcode 拆分：这些 read 会作为 LD-filtered 被丢弃，因为 barcode 已不在其中。如果打算对输出再次运行 Citrus，请在生成输入时使用 `--retain`。

### 4. 拆分后的总数据量（拆分成功和拆分失败都算上），为什么比原始输入少？

有三个互相独立的原因会让输出比输入小，其中通常只有第一个会明显。

**一、成功拆分的 read 被切掉了 barcode。** 默认情况下，每个成功分类的 read 两端的 barcode 都会被切除，因此写进 barcode 文件的序列和质量值都比输入短。`Discarded/` 下的 read 是**未切除**的原样输出，不参与这项损失。所以净损失恰好等于成功拆分 read 的被切除碱基数：`--mode 2` 下每条 read 切掉两个 barcode，`--mode 1` 兜底时切掉一个。加上 `--retain` 这一项就消失了，输出里保留的是完整的原始 read。

**二、短于 `--min_len` 的 read 根本没有写盘。** 自 v3.3.0 起，普通模式在读取输入时就会跳过它们，这些 read 不写入任何文件，统计里也没有。默认 `--min_len 0` 时不会有损失；一旦设了阈值，短于阈值的 read 就从输出里整体消失。准确度模式（`--testseq`）没有读入阶段的预过滤，在那里它们会进入 `length_filtered`。

**三、输出被重新压缩了一遍。** 每个 barcode 文件是各自独立的 gzip 流，用的压缩器、压缩级别和分块方式都和输入文件不同，同样的字节压出来可能略有出入。这一项只影响文件体积：把输出解压后和输入逐条比对，除了上面说的切除之外没有别的差别。BAM 输入时这种比较意义更小——输出是重新编码的，两个文件的容器开销本来就不必一致。

**统计口径不受切除影响。** `{sample}_demux.stats.txt` 的 `Total` 行按 read 的**原始长度**统计，被切掉的 barcode 碱基仍算在里面。真正留存下来的碱基数看 `{sample}_barcodes.stats.tsv` 的 `Passed_base_count` 列；单个 barcode 则是 `{sample}_{barcode_id}_01.fastq.stats.tsv` 里的 `PassBaseNum`（切除后），与旁边未切除的 `TotalBaseNum` 相对照。

一个实测例子——1000 条 read、127,200 个碱基，两端各一个 16 bp 的 barcode：

| 运行方式 | 写出的 read 数 | 写出的碱基数 | 输出体积（gzip） |
|---|---|---|---|
| 输入 | 1000 | 127,200 | 121,780 B |
| `--mode 2`（默认） | 1000 | 101,588 | 105,734 B |
| `--mode 2 --retain` | 1000 | 127,200 | 123,492 B |
| `--mode 2 --min_len 120` | 800 | 79,988 | 83,356 B |

默认运行一条 read 都没丢，少掉的 25,612 个碱基全是被切掉的 barcode。`--retain` 把它们补了回来——解压后逐条比对，输出与输入完全一致。`--min_len 120` 则把 200 条短于 120 bp 的 read 整个丢掉，文件和统计里都不再出现。

## 作者

Ming Lian 连明 (lianming@genomics.cn)

Jiayuan Zhang 张嘉远 (zhangjiayuan@genomics.cn)

## 许可与使用限制
**仅供研究使用**

本软件严格限于个人研究目的使用，严禁商业使用。具体而言：

**允许：** 个人学术研究、个人学习及非商业性实验。
**不允许：** 任何形式的商业应用、二次分发，或直接、间接产生收入的使用。包括但不限于集成到商业产品、将本软件作为服务提供，或用于商业获利。

如需商业授权或许可，请联系我们。
