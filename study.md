utils/load_signals.py 确实是项目中最早的数据处理部分，也是把“原始 EEG/SEEG 信号”变成后面模型能用的“STFT 图像 / 数组”的地方。  
它包含了若干针对不同公开数据集的读取器（loader）、若干预处理（preprocess）函数，以及一个面向外部的类 LoadSignals，main.py 就是通过这个类来获取/生成数据的。
负责从磁盘读取不同格式的 EEG/SEEG 原始数据（不同数据集格式不同），提取“预ictal / 非发作”段或其它感兴趣段。
把这些原始时序信号切窗并计算短时傅里叶变换（STFT），把时频图（spectrogram）转成模型的输入张量格式。
支持把处理结果缓存（hickle 文件），或者把 STFT 按文件保存为 .npy（供 DCGAN 训练使用）。
提供一个统一入口类 LoadSignals，外面只需调用 LoadSignals(...).apply(...) 即可得到 X,y 或把 STFT 文件保存到目录。
  #  例如
load_signals_CHBMIT(...)：读取 CHB-MIT（.edf）格式的数据并返回原始矩阵（或 yield 多段），会使用 mne 库。
在指定的病人目录（例如 data_dir/chb01/）中读取所有 .edf 文件列表。
根据 seizure_summary.csv 决定哪些文件有发作（用于 ictal/preictal）。
对每个选中的 EDF：
使用 mne.io.read_raw_edf 读取（preload=True 会把数据读进内存）。
pick_channels(chs) 保证通道顺序统一（仓库中有 utils/CHBMIT_channels.py 指定每个病人的通道子集）。
把 EDF 转为 NumPy 矩阵（samples × channels）。
若提取 preictal（发作前）段，基于 szstart（秒）与 sph（分钟）与 SOP（30 分钟）计算样本索引，切出 [st - SOP : st] 的那段；若当前文件开头不够，会尝试从上一个 EDF 拼接。
对 interictal，若 special_interictal 指定了 start/stop，则取该段；否则返回整个录制作为 interictal 段（后续 preprocess 会切窗、做 STFT）。
