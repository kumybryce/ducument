# mmWaveStudio文档翻译
## 1. 简介
mmWaveStudio GUI用于TI雷达设备的评估与使用。mmWaveStudio通过SPI向设备发送指令来控制设备，雷达的ADC数据使用DCA1000 EVM(基带板)来采集，数据处理采用Matlab的runtime，处理结果可以在该GUI显示。
mmWaveStudio GUI的特征如下：

1. 电路板控制（切换SOP、重置等）
2. 通过RS232连接设备
3. 通过RS232烧录固件
4. 通过Radar API命令来配置TI的雷达设备（即LUA命令，需要学习LUA）
5. 使用DCA1000采集原始ADC数据
6. 数据处理以及可视化

> DAC1000 EVA的文档参考 SPRUIJ4 