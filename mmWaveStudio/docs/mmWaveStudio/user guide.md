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
3. 安装FTDI驱动（mmwave_studio_\<ver>\ftdi），对于 `AWR2243` 来说，这一步需要连接板子（Mini-B接口）
4. XDS Emulation Software Package v6.0.579.0 及以上版本

运行时板子连线

1. 1843需要两根能传数据的`Micro-B`线、一根网线、一个电源适配器
   ![](https://img-blog.csdnimg.cn/img_convert/b5f5bcb2769a3556fb78c53237a83839.png)
   > 注意：基带板有两个`micro-B`口，接上面那一个！知道设备管理器中出现以下六个端口(4个`AR`开头的，2个`XDS110`开头的)才能进行下一步，否则该装驱动装驱动
   ![20230522210105](https://blog-kumybryce-1300357628.cos.ap-nanjing.myqcloud.com/personal/liujiang/document/20230522210105.png)

2. 2243需要一根网线和一个电源适配器

## 3. 自动化脚本

1. 在GUI中，点击一个按钮或者一个命令被触发，都可以在output窗口中看到其对应的LUA命令
2. 脚本中的命令都以`ar1`开头。将所有命令存成LUA脚本可以实现自动化
3. 执行自动化脚本可以在`RUN`中选择脚本并运行
   ![20230522203115](https://blog-kumybryce-1300357628.cos.ap-nanjing.myqcloud.com/personal/liujiang/document20230522203115.png)

## 4. GUI中各个Tab的作用

### 4.1 Connect Tab

> 该Tab中可以对板子进行连接和固件下载，如果是使用GUI，根据按钮后面括号的数字顺序来进行操作，当前应该做的操作GUI中按钮会变成蓝色。在进行该步骤之前，确保`No. of Devices Detected`为1，否则要刷新，这个时候FDTI也应该处于`Connected`状态！

1. Reset Control

   ```lua
   ar1.PowerOff()
   Opening Gpio Control Port()
   ar1.Disconnect()
   ar1.Calling_ATE_DisconnectTarget()
   ar1.Calling_IsConnected()
   ar1.FullReset()
   Closing Board Control Port()
   Closing Gpio Control Port()
   ar1.SOPControl(2)
   ```

2. RS232 Operations

   ```lua
   ar1.Connect(14,115200,1000) -- 14是指‘XDS110 Class Application/User UART’端口号，115200是波特率，1000不知道是啥
   ar1.SelectChipVersion("XWR1843") -- 设置板子类型，应该还要设置工作频率
   ar1.frequencyBandSelection("77G")
   ```

3. Firmware Download

   ```lua
   ar1.DownloadBSSFw("C:\\ti\\mmwave_studio_02_01_01_00\\rf_eval_firmware\\radarss\\xwr18xx_radarss.bin")
   ar1.GetBSSFwVersion()
   ar1.GetBSSPatchFwVersion()
   ar1.DownloadMSSFw("C:\\ti\\mmwave_studio_02_01_01_00\\rf_eval_firmware\\masterss\\xwr18xx_masterss.bin")
   ar1.GetMSSFwVersion()
   ```

4. SPI Connection & RF Power Up

   ```lua
   ar1.PowerOn(0, 1000, 0, 0)
   ar1.RfEnable()
   ```

### 4.2 Static Config Tab

1. 设置发送和接收天线数量

   ```lua
   ar1.ChanNAdcConfig(1, 1, 1, 1, 1, 1, 1, 2, 1, 0) -- 前三位代表发射天线配置，接着4位是接收天线配置，1代表开，0代表关，后三位咱不知道是啥
   ```

2. LP Mode （选择`Regular ADC`）

   ```lua
   ar1.LPModConfig(0, 0)
   ```

3. RfInit

   ```lua
   ar1.RfInit()
   ```

   > 返回值案例：
   Time stamp, Temperture: 595478,49; APLL Status, Update: 1, 0; SynthVCO1 Status, Update: 1, 1; SynthVCO2 Status, Update: 1, 1; LODist Status, Update: 1, 1; RxADCDC Status, Update: 1, 1; HPFcutoff Status, Update: 1, 1; LPFcutoff Status, Update: 1, 1; PeakDetector Status, Update: 1, 1; TxPower Status, Update: 1, 1; RxGain Status, Update: 1, 1; TxPhase Status, Update: 1, 1; RxIQMM Status, Update: 1, 1;

### 4.3 Data Config Tab

> 通过这个tab，用户可以配置板子采集的ADC数据传输到PC端的文件路径

### 4.4 Sensor Config Tab

> 通过这个tab，用户可以配置波形参数

1. Sensor Config

```lua
ar1.ProfileConfig(0, 77, 100, 6, 60, 0, 0, 0, 0, 0, 0, 29.982, 0, 256, 10000, 0, 0, 30)
-- 0 profile id 可以单独为一帧内每一个chirp配置不同的波形参数
-- 77 start freq(GHz)
-- 100 idle time(us)
-- 6 ADC start time(us)
-- 60 ramp end time(us)
-- 0 o/p pwr backoff TX0(dB)
-- 0 o/p pwr backoff TX1(dB)
-- 0 o/p pwr backoff TX2(dB)
-- 0 phase shifter TX0(deg)
-- 0 phase shifter TX1(deg)
-- 0 phase shifter TX2(deg)
-- 29.982 freq slop(MHz/us)
-- 0 Tx start time(us)
-- 256 ADC samples
-- 1000 sample rate(ksps)
-- 0 不知道
-- 0 不知道
-- 30 Rx Gain(dB)
```

2. Chirp Config

   >

3. Frame Config

   ```
   注意：
   1. 在帧配置中增加了一个新的选项"Dummy Chirps (End)"。这表示在帧末尾虚拟chirp的数量。这些chirp的ADC数据、CP和CQ数据不会从设备中发出
   2. 在配置非零虚拟chirp的情况下，Studio自带的Matlab后处理仅支持chirp循环数等于1。在帧配置过程中还发出警告："如果( chirp循环不等于1，虚拟chirp不等于0 )，Matlab后处理将不能正常工作"。
   3. 确保在发布Frame Configuration之前打开HSDCPro软件
   4. Matlab Run Time Engine ( Version 8.5.1 )和HSDCPro软件是使用mm Wave Studio GUI中的后处理工具的先决条件
   5. 当frame配置为0时，会无限采数。
   ```

4. IP address config
   
   将对应网口配置成如下形式
   ![20230523144355](https://blog-kumybryce-1300357628.cos.ap-nanjing.myqcloud.com/personal/liujiang/document/20230523144355.png)

### 5. Register Operation Tab

> 该tab可以用来读写寄存器

1. 寄存器的读写
   
2. GPIO_0
   
   **利用GPIO_0输出ADC有效信号，触发外部仪器进行合成器相关测量**
   该功能可以用作雷视融合同步

### 6. Continuous Streaming Tab

> Continuous Streaming选项卡可用于射频测量。该选项卡包含数据通路的配置，用于将捕获的ADC样本连续无损传输到外部主机

### 7. BPM Config Tab Operations

> 使用BPM选项卡，在每个TX中执行与BPM (二元相位调制)特征相关的静态配置。选取起止指标。配置TX并点击set按钮。

### 8. Capture Json

### 9. Matlab控制GUI

在mmWaveStudio中，调用`RSTD.NetStart()`命令可以默认监听2777端口，可以将该命令添加到`C:\ti\mmwave_studio_01_00_00_01\ mmWaveStudio\Scripts\Startup.lua`中，这样软件启动即可执行该命令

在matlab中操作开发板的逻辑是通过2777端口将LUA命令发送给GUI，GUI再执行这些LUA命令。

所以在matlab中有两个文件用于完成这个操作：

- `Init_RSTD_Connection.m`：用于与2777端口建立通讯连接
- `RSTD_Interface_Example.m`：用于书写业务代码，该脚本中首先调用`Init_RSTD_Connection`脚本，再通过`RtttNetClientAPI.RtttNetClient.SendCommand(Lua_String)`发送LUA命令

```matlab
%% Init_RSTD_Connection.m
function ErrStatus = Init_RSTD_Connection(RSTD_DLL_Path)
if (strcmp(which('RtttNetClientAPI.RtttNetClient.IsConnected'),''))
    disp('Adding RSTD Assembly');
    RSTD_Assembly = NET.addAssembly(RSTD_DLL_Path);
    if ~strcmp(RSTD_Assembly.Classes{1},'RtttNetClientAPI.RtttClient')
        disp('RSTD Assembly not loaded correctly. Check DLL path');
        ErrStatus = -10;
        return
    end
    Init_RSTD_Connection = 1;
elseif ~RtttNetClientAPI.RtttNetClient.IsConnected()
Init_RSTD_Connection = 1;
else
    Init_RSTD_Connection = 0;
end
if Init_RSTD_Connection
    disp('Initializing RSTD client');
    ErrStatus = RtttNetClientAPI.RtttNetClient.Init();
    if (ErrStatus ~= 0)
        disp('Unable to initialize NetClient DLL');
        return;
    end
    disp('Connecting to RSTD client');
    ErrStatus = RtttNetClientAPI.RtttNetClient.Connect('127.0.0.1',2777);
    if (ErrStatus ~= 0)
        disp('Unable to connect to mmWaveStudio');
        disp('Reopen port in mmWaveStudio. Type RSTD.NetClose() followed RSTD.NetStart()')
        return;
    end
    pause(1);%Wait for 1sec. NOT a MUST have.
end
disp('Sending test message to RSTD');
Lua_String = 'WriteToLog("Running script from MATLAB\n", "green")';
ErrStatus = RtttNetClientAPI.RtttNetClient.SendCommand(Lua_String);
if (ErrStatus ~= 30000)
    disp('mmWaveStudio Connection Failed');
end
disp('Test message success');
end
```

```matlab
%% RSTD_Interface_Example.m
addpath(genpath('.\'))
RSTD_DLL_Path = 'C:\ti\mmwave_studio_02_01_01_00\mmWaveStudio\Clients\RtttNetClientController\RtttNetClientAPI.dll'; % 需要根据实际情况更改
ErrStatus = Init_RSTD_Connection(RSTD_DLL_Path);
if (ErrStatus ~= 30000)
    disp('Error inside Init_RSTD_Connection');
    return;
end
% 举例：使用GUI执行一个LUA脚本
%strFilename ='C:\\ti\\mmwave_studio_01_00_00_01\\mmWaveStudio\\Scripts\\Example_script_AllDevices.lua';
%Lua_String = sprintf('dofile("%s")',strFilename);
%ErrStatus =RtttNetClientAPI.RtttNetClient.SendCommand(Lua_String);
```

### 10. LUA脚本

> 不用打开mmWaveStudio程序也可以执行脚本实现采数。原理是将所有的LUA命令存放在LUA脚本中，使用mmWaveStudio.exe去调用脚本，实现自动化。

