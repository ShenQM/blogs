---
typora-root-url: ./
---

# CMOS 图像传感器简介（1）：像素结构

　　随着工艺的发展，CMOS图像传感器的性能已经赶上或超越CCD，再加上CMOS图像传感器在工艺上能很大程度与传统CMOS芯片兼容，它已经成为相机的主流传感器类型。由于只能硬件的迅猛发展，很多应用场景都将碰到CMOS传感器，因此本文从基础出发，介绍CMOS图像传感器的像素结构。 

## 1.图像传感器整体架构

　　CMOS图像传感器本质是一块芯片，主要包括：感光区阵列（Bayer阵列，或叫像素阵列）、时序控制、模拟信号处理以及模数转换等模块（如图1）。其中，各模块的作用分别为：

- 像素阵列：完成光电转换，将光子转换为电子。
- 时序控制：控制电信号的读出、传递。
- 模拟信号处理（ADC）：对信号去噪。（如用CDS去除reset noise、fpn等）

![CMOS图像传感器结构](http://micro.magnet.fsu.edu/primer/digitalimaging/images/cmos/cmoschipsfigure1.jpg)

<center>图1.1 CMOS传感器示例</center>

其中，像素阵列占整个芯片的面积最大，像素阵列是由一个个像素组成，它对应到我们看到每张图片中的每个像素。每个像素包括感光区和读出电路（后面小节会详细讨论），每个像素的信号经由模拟信号处理后，交由ADC进行模数转换后即可输出到数字处理模块。像素阵列的信号读出如下（参考图1.2）：

1. 每个像素在进行reset，进行曝光。
2. 行扫描寄存器，一行一行的激活像素阵列中的行选址晶体管。
3. 列扫描寄存器，对于每一行像素，一个个的激活像素的列选址晶体管。
4. 读出信号，并进行放大。

<center>

![CMOS传感器示意图](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c0/CMOS_Image_Sensor_Mechanism_Illustration.svg/2000px-CMOS_Image_Sensor_Mechanism_Illustration.svg.png)</center>

<center>图1.2 CMOS传感器信号读出示意图</center>

## 2.图像传感器像素结构

　　CMOS传感器上的主要部件是像素阵列，这是其与传统芯片的主要区别。每个像素的功能是将感受到的光转换为电信号，通过读出电路转为数字化信号，从而完成现实场景数字化的过程。像素阵列中的每个像素结构是一样的，如图2.1是典型的前照式像素结构，其主要结构包括：

1. On-chip-lens:该结构可以理解为在感光元件上覆盖的一层微透镜阵列，它用来将光线聚集在像素感光区的开口上。可以增加光电转化效率，减少相邻像素之间的光信号串扰。
2. Color filter：该结构是一个滤光片，包括红/绿/蓝三种，分别只能透过红色、绿色、蓝色对应波长的光线。该滤光片结构的存在，使得每个像素只能感应一种颜色，另外的两种颜色分量需要通过相邻像素插值得到，即demosaic算法。
3. Metal wiring：可以为金属排线，用于读出感光区的信号（其实就是像素内部的读出电路）。
4. Photodiode：即光电信号转换器，其转换出的电信号会经过金属排线读出。

![像素结构](http://www.extremetech.com/wp-content/uploads/2013/03/Wiring-diagram-for-a-typical-front-illuminated-sensor.gif)

<center>图2.1 像素结构</center>

其中，Photodiode和Metal wiring对CMOS传感器的性能影响最大（比如光电转换效率，读出噪声等），也是目前主流传感器厂商注重提高的工艺。为了方便叙述，下面将Photodiode和Metal wiring简称为Pixel（即包括每个像素内的感光区域和读出电路）。

### 2.1 Passive Pixel

　　最简单的Pixel结构只有一个PN结作为photodiode感光，以及一个与它相连的reset晶体管作为一个开关（如图2.2）。它的工作方式如下：

1. 在开始曝光之前，该像素的行选择地址会上电（图中未画出），从而RS会激活，连通PN结与column bus。同时列选择器会上电，此时PN结会被加载高反向电压（例如3.3 V）。在Reset（即PN结内电子空穴对达到平衡）完成后，RS将会被停止激活，停止PN结与column bus的连通。
2. 在曝光时间内，PN结内的硅在吸收光线后，会产生电子-空穴对。由于PN结内电场的影响，电子-空穴对会分成两个电荷载体，电子会流向PN结的$n^+$端，空穴会流向PN结的p-substrate。因此，经过曝光后的的PN结，其反向电压会降低。
3. 在曝光结束后，RS会被再次激活，读出电路会测量PN结内的电压，该电压与原反向电压之间的差，就是PN结接受到的光信号。（在主流sensor设计中，电压差与光强成正比关系）
4. 在读出感光信号后，会对PN结进行再次reset，准备下次曝光。

![passive pixel](/CMOS 图像传感器简介.assets/passive pixel.PNG)

<center>图2.2 像素结构</center>

这种像素结构，其读出电路完全位于像素外面，称为Passive Pixel。Passive Pixel的读出电路简单，整个Pixel的面积可以大部分用于构造PN结，所以其满阱电容一般会高于其他结构。但是，由于其信号的读出电路位于Pixel外面，它受到电路噪声的影响比Active Pixel（下一节会介绍）大。Passivel Pixel噪声较大有2个主要原因：

1. 相对读出电路上的寄生电容，PN结的电容相对较小。代表其信号的电压差相对较小，这导致其对电路噪声很敏感。
2. 如图2.3(b)，PN结的信号，先经过读出电路，才进行放大。这种情况，注入到读出信号的噪声会随着信号一起放大。

![active_passive_noise_compare](/CMOS 图像传感器简介.assets/active_passive_noise_compare.PNG)

<center>图2.3 Active Pixel和Passive Pixel噪声注入对比</center>

### 2.2 Active Pixel

　　Active Pixel指的是在像素内部有信号读出电路和放大电路的像素结构。如图2.3(a)，信号传出Pixel之前，就已经读出并放大，这减少了读出信号对噪声的敏感性。随着工艺的发展，基于Active Pixel的CMOS传感器在暗电流和噪声表现上有很大提升，Active Pixel结构随之成为了CMOS传感器的主流设计。

　　图2.4展示了基于PN结的Active Pixel结构，也成为3T像素结构（每个像素包含3个三极管）。在这种结构中，每个像素包含一个PN结作为感光元件，一个复位三极管RST，一个行选择器RS，以及一个信号放大器SF。其工作方式和Passive Pixel类似：

1. 复位。给PN结加载反向电压，或者说激活RST给PN结进行复位。复位完成后，不再导通RST。
2. 曝光。和在Passive Pixel中一样，光子打到PN结及硅基，被吸收后产生电子-空穴对。这些电子空穴对通过电场移动后，减小PN结上的反向电压。
3. 读出。在曝光完成后，RS会被激活，PN结中的信号经过运放SF放大后，读出到column bus。
4. 循环。读出信号后，重新复位，曝光，读出，不断的输出图像信号。

![active_pn_pixel](/CMOS 图像传感器简介.assets/active_pn_pixel.PNG)

<center>图2.4 PN结像素结构</center>

　　基于PN结的Active Pixel流行与90年代中期，它解决了很多噪声问题。但是由PN结复位引入的kTC噪声，并没有得到解决。为了解决复位kTC噪声，减小暗电流，引入了基于PPD结构（Pinned Photodiode Pixel）的像素结构。PPD pixel包括一个PPD的感光区，以及4个晶体管，所以也称为4T像素结构（如图2.5）。PPD的出现，是CMOS性能的巨大突破，它允许相关双采样（CDS）电路的引入，消除了复位引入的kTC噪声，运放器引入的1/f噪声和offset噪声。

![active_ppd_pixel](/CMOS 图像传感器简介.assets/active_ppd_pixel.PNG)

<center>图2.5 PPD像素结构</center>

　　仔细对比图2.4和2.5，发现示意图右边的结构基本一致。但他们的功能有明显差异，对于PPD，右边部分电路只是信号读出电路。读出电路与光电转换结构通过TX完全隔开，这样可以将光感区的设计和读出电路完全隔离开，有利于各种信号处理电路的引入（如CDS，DDS等）。另外，PPD感光区的设计采用的是p-n-p结构，减小了暗电流。PPD像素的工作方式如下：

1. 曝光。光照射产生的电子-空穴对会因PPD电场的存在而分开，电子移向n区，空穴移向p区。
2. 复位。在曝光结束时，激活RST，将读出区（$n^+$区）复位到高电平。
3. 复位电平读出。复位完成后，读出复位电平，其中包含运放的offset噪声，1/f噪声以及复位引入的kTC噪声，将读出的信号存储在第一个电容中。
4. 电荷转移。激活TX，将电荷从感光区完全转移到$n^+$区用于读出，这里的机制类似于CCD中的电荷转移。
5. 信号电平读出。接下来，将$n^+$区的电压信号读出到第二个电容。这里的信号包括：光电转换产生的信号，运放产生的offset，1/f噪声以及复位引入的kTC噪声。
6. 信号输出。将存储在两个电容中的信号相减（如采用CDS，即可消除Pixel中的主要噪声），得到的信号在经过模拟放大，然后经过ADC采样，即可进行数字化信号输出。

　　PPD像素结构有如下优点：

- 读出结构（$n^+$区）的kTC噪声完全被CDS消除。
- 运放器的offset和1/f噪声，都会因CDS得到明显改善。
- 感光结构因复位引起的kTC噪声，由于PPD电荷的全转移，变的不再存在。
- 光敏感度，它直接取决于耗尽区的宽度，由于PPD的耗尽区一直延伸到近$Si-SiO_{2}$界面，PPD的光感度更高。
- 由于p-n-p的双结结构，PPD的电容更高，能产生更高的动态范围。
- 由于$Si-SiO_{2}$界面由一层$p^+$覆盖，减小了暗电流。

由于PPD像素结构在暗电流和噪声方面的优异表现，近年来市面上的CMOS传感器都是以PPD结构为主。但是，PPD结构有4个晶体管，有的设计甚至有5个，这大大降低了像素的填充因子（即感光区占整个像素面积的比值），这会影响传感器的光电转换效率，进而影响传感器的噪声表现。在PPD结构中，像素的感光区和读出电路由TX晶体管隔开，相邻像素减可以共用读出电路（如图2.6）。图2.6所示的2x2像素共享读出电路，一共有7个晶体管，平均一个像素1.75个晶体管。这样可以大大减少每个像素中读出电路占用的面积，可以提高填充因子，这样可以使得像素面积更小（比如1微米）。然而，由于这2x2个像素的结构不再一致，会导致固定模式噪声的出现（FPN），这需要在后续图像处理中消除。

![shared_ppd](/CMOS 图像传感器简介.assets/shared_ppd.PNG)

<center>图2.6 共享读出电路的PPD像素结构</center>

## 3.总结

　　本文主要介绍了以下三个方面：

1. CMOS图像传感器的整体架构。阐明了其基本模块：像素整列、读出电路、模拟信号处理以及模数转换模块。
2. CMOS图像传感器的PN结像素结构。这里主要介绍了CMOS在发展初期的以中像素结构，但是它存在高暗电流和噪声的问题，在90年代中期始终无法与CCD在高端成像领域竞争。
3. CMOS图像传感器的PPD像素结构。介绍了PPD像素中感光区域与读出电路的分离，从而使得CDS能用于图像信号的读出，引发了CMOS图像传感器的变革。这大大减少了CMOS传感器的噪声和暗电流，使得CMOS传感器赶上并超越CCD，成为相机传感器的主流。
4. 介绍了PPD像素结构共享读出电路的方式。由于PPD读出电路复杂，减小了像素的感光区填充因子，引入了读出电路共享方式。该技术能够进一步减少像素的面积，目前已知的有1微米大小的像素。

文中主要介绍了像素结构的基础，后续会继续更新一些CMOS传感器的知识。

## 参考文献

[1] Albert THEUWISSEN. CMOS Image Sensors : State-Of-The-Art and Future Perspectives. [ESSDERC 2007 - 37th European Solid State Device Research Conference](https://ieeexplore.ieee.org/xpl/mostRecentIssue.jsp?punumber=4430869) 

[2] Junichi Nakamura etc. Image Sensors and Signal Processing for Digital Still Cameras. 2006. Taylor & Francis





