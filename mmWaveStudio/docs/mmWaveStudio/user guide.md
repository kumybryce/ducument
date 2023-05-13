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

> DAC1000 EVA的文档参考 [SPRUIJ4](https://www.ti.com/lit/ug/spruij4a/spruij4a.pdf?ts=1683702139364)

## 2. mmWaveStudio安装与运行

运行前安装：

1. 安装`mmWaveStudio`，针对`AWR2243`和`AWR1843`需要下载不同的版本
   
      - [ ] 1843：`02.01.01.00`
      - [ ] 2243: `03.00.00.14`

2. 安装32位的Matlab Runtime Engine（[Version 8.5.1](https://in.mathworks.com/supportfiles/downloads/R2015a/deployment_files/R2015aSP1/installers/win32/MCR_R2015aSP1_win32_installer.exe)），用于数据处理
3. 安装FTDI驱动，对于 `AWR2243` 来说，这一步需要连接板子（micro usb接口）