Internet Engineering Task Force (IETF) | M.Q.C. van Beurden

Request for Comments: [9639](https://www.rfc-editor.org/rfc/rfc9639)

Cagegory: Standards Track | A. Weaver

Published: December 2024

ISSN: 2070-1721

---

# Free Lossless Audio Codec (FLAC)

## 摘要

本文定义了 Free Lossless Audio Codec (FLAC) 格式及其流式子集。
FLAC 的设计目的是减少存储数字音频信号所需要的计算机存储空间。
这种操作是无损的，即不会丢失信息。
FLAC 是自由的，因其规范是开放的、其参考实现是开源的。
相较于其他无损音频编码格式，FLAC 格式的复杂度较低，且仅需要很少的计算资源即可完成编码和解码。
针对多种不同平台的 FLAC 的解码功能已单独实现，同时，实现编码和解码不需要浮点数运算。

## Status of This Memo

This is an Internet Standards Track document.

This document is a product of the Internet Engineering Task Force (IETF). It represents the consensus of the IETF community. It has received public review and has been approved for publication by the Internet Engineering Steering Group (IESG). Further information on Internet Standards is available in Section 2 of RFC 7841.

Information about the current status of this document, any errata, and how to provide feedback on it may be obtained at [https://www.rfc-editor.org/info/rfc9639.](https://www.rfc-editor.org/info/rfc9639.)

## Copyright Notice

Copyright (c) 2024 IETF Trust and the persons identified as the document authors. All rights reserved.

This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents [https://trustee.ietf.org/license-info](https://trustee.ietf.org/license-info) in effect on the date of publication of this document. Please review these documents carefully, as they describe your rights and restrictions with respect to this document. Code Components extracted from this document must include Revised BSD License text as described in Section 4.e of the Trust Legal Provisions and are provided without warranty as described in the Revised BSD License.

## 1. 引言

本文定义 Free Lossless Audio Codec (FLAC) 格式及其流式子集。
FLAC 文件和 FLAC 流能以编码脉冲编码调制（Pulse-Code Modulation）音频，其声道数（channels）为 1 到 8 个，采样率（sample rates）可从 1 到 1048575 赫兹，位深度（bit depths）为 4 到 32 比特。
大多数编码、解码 FLAC 格式的工具是针对 CD 音频优化的，即 2 声道、44.1 kHz 采样率和 16 位位深度的 PCM 音频。

FLAC 能够实现无损压缩的原因在于：音频信号往往与其相邻的信号密切相关。
通用压缩工具经常使用字典、进行游程编码或是利用长期重复性（long-term repetition），FLAC 与之相对，其仅在非常短的周期内（short term）移除重复，最多会回顾 32 个样本。

FLAC 格式提供的编码方式在应用于使用以有符号表示、且以 0 为中心的采样样本的 PCM 音频信号上最佳。
采样样本以无符号表示的音频信号必须按照此文档中的指示转换为有符号表示才能获得较为合理的压缩效果。
FLAC 格式不适合压缩非 PCM 音频。

## 2. Notation and Conventions（符号表示和约定）

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119) [RFC8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

书写为 u(n) 的值表示一个 n 位无符号大端序整数。
书写为 s(n) 的值表示一个 n 位有符号大端序整数（二进制补码表示）。
必要时，n 以方程形式表示，并含有 \*（乘法符号）、/（除法符号）、+（加法符号）或 -（减法符号）。
用省略号表示所含比特位范围，例如 u(m...n)。

本文中所提到的移位均为算术移位（译注：算术移位中，左移时低位补 0，右移时高位保持不变）。

FLAC 格式能用于存储数字音频与其他数字信号，但本文所使用的术语均为数字音频的特定术语。
因使用更加通用的术语会被视为表述不够清晰，若读者的兴趣在于将 FLAC 格式应用于非音频场景，请自行将音频领域的特定术语转换为更通用的术语。

## 3. 定义（Definitions）

### 无损压缩（lossless compression）
减少存储数据所需要的计算机存储空间，同时不需要移除或不可逆地修改原有数据。
换句话说，解压无损压缩的数据所获得的数据将与原始数据完全一致。

### 有损压缩（lossy compression）
类似无损压缩，但需要移除、不可逆地修改原有数据或是仅保留近似数据以进一步减少所需的存储空间。
换句话说，解压有损压缩的数据所获得的数据是原始数据的近似值（approximation）。

### 块（block）
单声道或多声道的线性 PCM 音频中的一（小）节。

### 子块（subblock）
某个声道内所对应的块（block）中的所有采样样本（sample）。
其中，某个特定的块中的所有子块均包含相同数量的采样样本。

### 帧（frame）
包含帧头（frame header）、一个或多个子帧（subframe）和帧尾（frame footer）。
它编码了一个对应的块中的内容。

### 子帧（subframe）
一个编码了的子块（subblock）。
一个帧内的所有子帧都编码了相同数量的采样样本（sample）。
在应用了声道间去相关性（interchannel decorrelation）的情况下，一个子帧对应两个子块的（预采样）平均值或是（预采样）差值，而非直接对应一个子块。参见 [#4.2 小节](#interchannel-decorrelation)。

### 声道间采样样本（interchannel samples）
所有声道中采样样本的数量。
例如，一秒长的 44.1 kHz 音频中有 44100 个声道间采样样本，意味着每个声道的采样样本的数量都是 44100 个。

### 块大小（block size）
一个块（block）或编码在一帧（frame）内的声道间采样样本数量。

### 位深度（bit depth）
或称每采样样本比特数（bits per sample），是用于存储每个采样样本所使用的位（bit）数。
一个块（block）中的所有子块（subblock）所使用的位深度**必须**相同，但由于声道间去相关性，一个帧内的子帧们**可以**使用不同的位深度（关于声道间去相关性请参见 [#4.2 小节](#interchannel-decorrelation)）。

### 预测器（predictor）
一种基于之前的采样样本，预测后续音频信号中采样样本的模型。
FLAC 使用此类预测器来移除信号中的冗余部分，以此实现压缩信号的目的。

### 线性预测器（linear predictor）
使用线性预测方式的预测器（参见[线性预测](https://en.wikipedia.org/w/index.php?title=Linear_prediction&oldid=1169015573)）。
也被称之为**线性预测编码（linear predictive coding）**。
使用线性预测器时，每一项预测值都是之前样本的线性组合（也是“线性”这一名称的由来）。
线性预测器具有因果关系时间离散性有限冲激效应（Finite Impulse Response）（参见[有限冲激响应](https://en.wikipedia.org/w/index.php?title=Finite_impulse_response&oldid=1240945295)）。

### 固定预测器（fixed predictor）
在所有 FLAC 文件下模型参数全部相同的线性预测器，因此不需要存储。

### 预测器序号（predictor order）
预测器所使用的之前的采样样本数量。
例如，一个序号为 4 的预测器使用某个特定样本之前的 4 个样本进行预测。
在 FLAC 中，预测器所使用的样本都是连续的，并且都是在需要被预测的样本之前的样本。

### 残余（residual）
残余样本（residual samples），从一个子块（subblock）中减去（subtract）预测器后剩余的音频信号。
若预测器能够移除信号中的冗余部分，那么留下的信号中的样本（即残余样本）将会获得比原始信号更接近 0 的数值（平均而言）。

### Rice 编码
一段可变长度编码（参见[可变长度编码](https://en.wikipedia.org/w/index.php?title=Variable-length_code&oldid=1220260423)）。
对接近 0 的样本使用短编码，而后对远离 0 的样本逐渐采用较长的编码。
这利用了残余样本通常接近 0 的观察结果。

### Muxing
Short for multiplexing。多路复用，将多个流合并为一个流或将多个文件合并为一个文件。在本文的语境中，复用特指 [#10 小节](#container-mappings)中描述的，将 FLAC 流嵌入到一个容器中。

## 4. 概念一览（Conceptual Overview）

FLAC 文件的编码过程类似于其他众多的音频编解码器，如下列步骤所示。
解码过程与编码过程正好相反。

1. 分割成块（blocking）（参见 [#4.1 小节](#分割成块)。输入数据会被分割成许多连续的块。

2. 声道间去相关性（interchannel decorrelation）（参见 [#4.2 小节](#声道间去相关性)）。对于立体声音频流，FLAC 格式允许将其左右声道转换为一个中部信号和一个左侧信号或是一个右侧信号以移除声道间的冗余数据。对于每个块而言，选择哪一种转换方式都是独立的。

3. 预测（prediction）（参见 [#4.3 小节](#预测)）。这一步会将信号中的冗余部分移除，要完成这一点，需要为每个子块（subblock）或是其在上一步中的转换结果保存为预测器。预测器由简单的数学描述组成，顾名思义，它将以某个样本的前几个样本对这个样本进行预测。由于这种预测几乎不可能与实际完全一致，误差会被传递到下一步中。每个子块的预测器与其他子块之间完全独立。由于编码器和解码器都了解预测方式，压缩后的流中仅需要包含预测器的参数即可。若对于某个子块而言没有可用的预测器，那么这段信号就会被按原样保存起来，不进行压缩，同时跳过下一步。

4. 残余编码（residual coding）（参见 [#4.4 小节](#残余编码)）。由于预测器不会按原样描述原始信号，原始信号与预测信号的差值（称为误差或残余信号）将是无损编码的。如果预测器有效，每个样本的残余信号相比原始信号而言将拥有更少的位（bit）。FLAC 利用 Rice 编码（Golomb 编码的一个子集），使用 4 位或 5 位（bit）参数编码残余信号。

除此以外，FLAC 还指定了一个元数据系统（参见 [#8 小节]），允许在流的开头处包含有关流的任意信息。

### 4.1. 分割成块（blocking）
音频数据的分块大小会直接影响压缩率。
如果块太小，分块后产生的大量帧意味着帧头部中将有不成比例的字节。
而如果块太大，信号的特征会相差巨大，将导致编码器无法找到合适的预测器。
为了简化编码器和解码器的设计，FLAC 强制规定，除了最后一个块，最小的块大小为 16 个样本，最大的块大小为 65535 个样本。
允许最后一个块不足 16 个样本是因为这么做可以在不使用数据填充的情况下匹配编码音频的长度。

虽然 FLAC 文件中的块大小不必是固定不变的，但要找到能够实现最大压缩所需要的块大小优化方案常常是十分困难的。因此，FLAC 流中的块有着明确的固定大小或可变大小，同时，如果流中的块大小是固定不变的，为了小幅度提升压缩率，流中会存储块的数量而非样本数量。

### 4.2. 声道间去相关性（interchannel decorrelation）

许多音频文件中的声道之间有相关性。
FLAC 格式通过编码立体声声道的两个子块中所有样本的平均值（中声道（mid channel））或编码立体声声道的两个子块之间的差值（边声道（side channel）），而非直接将子块编码为子帧来利用立体声音频中声道间的相关性。
以下是一些可能的组合：

#### 各声道独立
独立编码所有声道。
所有的非立体声音频文件都**必须**以这种方式进行编码。

#### 中间（mid-side）
将左右子块转换为中间（mid）、侧边（side）子帧。
中间子帧的样本计算方法是：将对应的左右侧样本相加，再对求和结果右移 1 位（bit）得到计算结果。
侧边子帧的样本计算方法是：将对应左侧样本减去右侧样本，得到的差值就是计算结果。
解码时，所有的中声道（mid channel）样本都需要左移 1 位（bit）。
如果侧边声道（mid channel）是奇数，在中声道样本左移 1 位后，还需要加 1 得到最终结果。
重建左声道时，需要将对应的中间、侧边子帧中的样本相加，再对球和结果右移 1 位得到计算结果。
对于右声道而言，需要将中间声道的样本减去侧边声道的样本，再将差值右移 1 位得到计算结果。

#### 左侧（left-side）
左子块编码后，需要使用左右子块来编码侧边子帧。
侧边子帧的构建方法与中间子帧的相同。
解码时，右子块由对应的左子帧减去侧边子帧得到。

#### 右侧（side-right）
先用左右子块编码侧边子帧，之后编码右子块。
侧边子帧的构建方法与中间子帧的相同。
解码时，左子块由对应的右子帧减去侧边子帧得到。

表示侧边声道时需要额外的 1 位位深度，因为相减得到的样本值，再经过右移 1 位后得到的最终结果可能是任何给定的位深度的两倍大。
中间声道的左移不会导致数据丢失，因为中间子帧的一个奇数样本总伴随有对应侧边子帧中的一个奇数样本，这意味着丢失的最低位能够通过侧边子帧中获取和恢复。

### 4.3. 预测（prediction）

FLAC 格式有 4 种对输入信号建模的方式：
