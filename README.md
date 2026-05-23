# EEG_with_mne
> To keep track of my learning process.

### 1. preprocessing
首先，需要知道，预处理不应该被视为一个机械的模板化流程，而是应该要思考，怎么处理这组数据，更有利于之后的分析。Take **ASR** for example，它作为Artifact Subspace Reconstruction的方法，本身是基于统计视角的，i.e. 异常信号可能被识别为噪声。如果我们要做患者被试与正常被试的EEG差异分析，这个时候去使用ASR，个人认为可能存疑。  
Now, back to the pipeline, 经过主包白天的学习，大概有了更深的理解，here is the process as follows:
> - **reading**：读取数据，文件格式不同，读取方式有差异【P.S. 具体函数都可以在[MNE官网](https://mne.tools)查看学习】，主包是`.set`文件，故使用`mne.io.read_raw_eeglab(path)`。
  >> tips：读取之后先看一下数据长啥样`mata.plot()` & 有哪些信息`mata.info`。
> - **电极定位**：This action can make computer know where are the different electrical signals from. 最直观的一点是，可以绘制彩色的PSD图。对于绝大多数情况，`raw.set_montage("standard_1020")`足矣，当然也可以自己设定。
> - **去除坏导**（人工）：首先需要定义什么是“bad channels” ———— 完全无信号、噪声过大（杂乱/振幅大）、漂移（基线漂移）、工频干扰（由交流电源，如50Hz或60Hz电网引起的电气干扰，叠加在有用信号上的周期性噪声）。`通过交互的图片，可以选择坏导`，之后可以用相邻电极信号的平均赋给坏导，减少损失。
> - **filter**：滤波滤的是波的频率，大致有四种方式：高通（去除低频）、低通（去除高频）、band-pass（截取区间内的）、band-stop（去除一段频率区间的信号）。由于主包用的是公开数据，所以就按照作者的处理的方式选择了band-pass，保留了\[0.5Hz, 45Hz]的部分（这一部分$\delta、\thata、\alpha、\beta、\gamma$的波段都有涵盖哦）
> ```python
> raw.load_data() #By default, MNE does not load data into main memory to conserve resources.
> raw.filter(l_freq=0.5, h_freq=45)
> raw.notch_filter(50) #工频去噪
> raw.plot()
> ```

> - **re-referencing**：重新处理参考电极。因为EEG得到的电信号（微伏级）其实是一个相对值，零点就是参考电极，但是参考电极也是电极呀，也可能有噪音什么的，所以这里重新处理一下参考，没毛病。这里选择“**average**” ———— `raw.set_eeg_reference("average")`，从统计的角度，具有规律性的信号不会因为平均而减弱，但随机产生的噪音会。重参考之后，会更加展现数据的真实面貌，但这些体现了预处理EEG，不是为了100%拿到“标准答案”，而是借助数学，极大可能帮我们还原目标信号的实况。
> - **ICA**：independent component analysis，独立成分分析，类似于PCA主成分分析，也是会通过变换坐标，找出相互独立的成分（不再是原来的成分）。也就是说想肌电、眼动、心跳等等，是可以被分出来的，但也是人工，所以或许有实际经验能区辨得清做这个工作会更好，不然就把好信号排掉了。
> ```python
> # ICA
> from mne.preprocessing import ICA
> 
> #注意：要现在环境装 picard 这个包
> ica = ICA(n_components=10, method='picard', random_state=97, max_iter = 'auto')
> ica.fit(raw)
> 
> raw.load_data()
> ica.plot_sources(raw, show_scrollbars=True, block=True, title='请选择要去除的成分')
> plt.show()
> ica.plot_components()
> 
> raw_clean = ica.apply(raw)
> 
> raw.plot(title="原始数据")
> raw_clean.plot(title="ICA清洗后") #对比ICA效果
> ```

> - 分段：这一步也可以在滤波之前，因为数据更少，后续处理时间快，缺点是做ICA不是更方便。
> - ……

---

欧克，其实这也是主包今日更实际认识到的EEG预处理流程，并且也上手试了，唯一的遗憾是好像不太会认这个数据，就是做ICA的时候不能很确定。好消息是主包已经可以非常顺畅地用prompt搭建环境哩！
