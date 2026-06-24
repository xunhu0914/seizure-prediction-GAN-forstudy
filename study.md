utils/load_signals.py 确实是项目中最早的数据处理部分，也是把“原始 EEG/SEEG 信号”变成后面模型能用的“STFT 图像 / 数组”的地方。  
它包含了若干针对不同公开数据集的读取器（loader）、若干预处理（preprocess）函数，以及一个面向外部的类 LoadSignals，main.py 就是通过这个类来获取/生成数据的。
负责从磁盘读取不同格式的 EEG/SEEG 原始数据（不同数据集格式不同），提取“预ictal / 非发作”段或其它感兴趣段。
把这些原始时序信号切窗并计算短时傅里叶变换（STFT），把时频图（spectrogram）转成模型的输入张量格式。
支持把处理结果缓存（hickle 文件），或者把 STFT 按文件保存为 .npy（供 DCGAN 训练使用）。
提供一个统一入口类 LoadSignals，外面只需调用 LoadSignals(...).apply(...) 即可得到 X,y 或把 STFT 文件保存到目录。
load_signals_CHBMIT(...)：读取 CHB-MIT（.edf）格式的数据并返回原始矩阵（或 yield 多段），会使用 mne 库。
