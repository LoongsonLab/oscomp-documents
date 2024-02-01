
# 平台支持

目前支持两个平台，一个是Qemu模拟器，另一个是2k1000的开发板。下面详细介绍各自平台：


## Qemu virt平台
这是个以Loongson-3A5000 LS7A1000的平台，其主要的配置如下所示：

``` shell
qemu-system-loongarch64 -M virt,dumpdtb=la_virt.dtb -smp 1
fdtdump la_virt.dtb > la_virt.dtb.dts
```


```text
/dts-v1/;
// magic:		0xd00dfeed
// totalsize:		0x100000 (1048576)
// off_dt_struct:	0x40
// off_dt_strings:	0x5dc
// off_mem_rsvmap:	0x30
// version:		17
// last_comp_version:	16
// boot_cpuid_phys:	0x0
// size_dt_strings:	0xcf
// size_dt_struct:	0x59c

/ {
    interrupt-parent = <0x00008001>;
    #size-cells = <0x00000002>;
    #address-cells = <0x00000002>;
    compatible = "linux,dummy-loongson3";
    pcie@20000000 {
        ranges = <0x01000000 0x00000000 0x00004000 0x00000000 0x18004000 0x00000000 0x0000c000 0x02000000 0x00000000 0x40000000 0x00000000 0x40000000 0x00000000 0x40000000>;
        reg = <0x00000000 0x20000000 0x00000000 0x08000000>;
        dma-coherent;
        bus-range = <0x00000000 0x0000007f>;
        linux,pci-domain = <0x00000000>;
        #size-cells = <0x00000002>;
        #address-cells = <0x00000003>;
        device_type = "pci";
        compatible = "pci-host-ecam-generic";
    };
    platform-bus@16000000 {
        interrupt-parent = <0x00008001>;
        ranges = <0x00000000 0x00000000 0x16000000 0x02000000>;
        #address-cells = <0x00000001>;
        #size-cells = <0x00000001>;
        compatible = "qemu,platform", "simple-bus";
    };
    intc@10000000 {
        phandle = <0x00008001>;
        reg = <0x00000000 0x10000000 0x00000000 0x00000100>;
        compatible = "loongarch,ls7a";
        ranges;
        #size-cells = <0x00000002>;
        #address-cells = <0x00000002>;
        interrupt-controller;
        #interrupt-cells = <0x00000003>;
    };
    rtc@100d0100 {
        reg = <0x00000000 0x100d0100 0x00000000 0x00000100>;
        compatible = "loongson,ls7a-rtc";
    };
    serial@1fe001e0 {
        clock-frequency = <0x05f5e100>;
        reg = <0x00000000 0x1fe001e0 0x00000000 0x00000100>;
        compatible = "ns16550a";
    };
    flash@1d000000 {
        bank-width = <0x00000004>;
        reg = <0x00000000 0x1d000000 0x00000000 0x01000000>;
        compatible = "cfi-flash";
    };
    fw_cfg@1e020000 {
        dma-coherent;
        reg = <0x00000000 0x1e020000 0x00000000 0x00000018>;
        compatible = "qemu,fw-cfg-mmio";
    };
    memory@80000000 {
        device_type = "memory";
        reg = <0x00000002 0x80000000 0x00000002 0x30000000>;
    };
    memory@0 {
        device_type = "memory";
        reg = <0x00000002 0x00000000 0x00000002 0x10000000>;
    };
    cpus {
        #size-cells = <0x00000000>;
        #address-cells = <0x00000001>;
        cpu-map {
            socket0 {
                core0 {
                    cpu = <0x00008000>;
                };
            };
        };
        cpu@0 {
            phandle = <0x00008000>;
            reg = <0x00000000>;
            compatible = "loongarch,Loongson-3A5000";
            device_type = "cpu";
        };
    };
    chosen {
        stdout-path = "/serial@1fe001e0";
    };
};
```

具体的代码可查询[QEMU Virt(LoongArch64), virt.c](https://github.com/qemu/qemu/blob/master/hw/loongarch/virt.c)

## 广东龙芯2K1000星云板(2k1000平台)

### 1. 简介

广东龙芯2K1000星云板基于龙芯2K1000处理器，采用单板的方式设计。该开发板主要包含CPU、内
存、双网口、USB、RS232、RS485、CAN等主要芯片及常用外设接口，同时通过插针形式的引出其他
接口。

本板卡采用标准化贴片焊接工艺，接口丰富，具有稳定、安全、可靠等特点，可广泛应用于国防、电
力、交通、医疗、金融、通信、能源等行业领域。

### 2.板卡规格
主要参数如下：
| 项目        |   参数 |
|-|-|
| CPU        |   龙芯2K1000LA（LoongArch64 架构兼容） |
| CPU        |   核心数 双核 |
| CPU        |   主频 800MHz-1000MHz |
| 内存容量    |   1GB DDR3（2片512MB） |
| 存储容量    |   32GB SSD(M.2 2242) |
| 以太网      |   2路千兆RJ45 |
| USB接口     |   2路USB2.0 HOST，其中一路与4G模块接口复用 |
| LCD接口     |   1路LCD接口（支持24位输出） |
| 按钮        |   2个（电源、复位按钮） |
| LED灯       |   4个用户自定义LED灯 |
| 按键        |   3个按键 |
| 调试串口     |   1个MicroUSB |
| RS232接口    |   2路 |
| RS485接口    |   2路 |
| CAN接口      |   2路 |
| M.2         |  KEY-B 接口 1路 (SATA) |
| M.2         |  KEY-E 接口 1路 (PCIE x1, USB2.0)（可接WIFI模块/AI模块(默认算力2Tops)） |
| 4G模块接口     |   1路 (与USB复用) |
| EJTAG接口     |   1路 (1.27连接器) |
| RTC电池座      |   1路2PIN |
| I2C接口        |   2路 |
| SPI接口        |   1路 |
| 串口           |   2路TTL |
| PWM接口        |   4路 |
| GPIO接口       |   8路 |
| 固件           |  /内核 U-Boot 2022/Linux 5.10 |
| 文件系统        |   Buildroot 2021/OpenHarmony 3/Loongnix Embeded |
| 电源输入        |   5V/3A |
| 工作温度        |   0 ~ +70℃ |
| 环境湿度        |   20% ~ 90%（无凝结） |
| 存储温度        |   -40 ~ +85℃ |
| 产品尺寸        |   150mm×109mm |


### 3.资源如下：

```text
/dts-v1/;
#include <dt-bindings/interrupt-controller/irq.h>
#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/clock/loongson2k1000-clock.h>

/ {
	model = "loongson-2k1000";
	compatible = "loongson,ls2k";
	#address-cells = <2>;
	#size-cells = <2>;

	aliases {
		ethernet0 = &gmac0;
		ethernet1 = &gmac1;
		serial0 = &cpu_uart0;
		serial1 = &uart1;
		serial2 = &uart2;
		serial3 = &uart3;
		serial4 = &uart4;
		serial5 = &uart5;
		serial6 = &uart6;
		serial7 = &uart7;
		serial8 = &uart8;
		serial9 = &uart9;
		serial10 = &uart10;
		serial11 = &uart11;
		i2c0 = &i2c0;
		i2c1 = &i2c1;
	};

	chosen {
		stdout-path = "serial0:115200n8";
		bootargs = "earlycon";
	};

	cpus {
		#address-cells = <1>;
		#size-cells = <0>;

		cpu0: cpu@0 {
			device_type = "cpu";
			compatible = "loongarch";
			reg=<0>;
			numa-node-id = <0>;
		};

		cpu1: cpu@1 {
			device_type = "cpu";
			compatible = "loongarch";
			reg=<1>;
			numa-node-id = <0>;
		};
	};

	cpuic: interrupt-controller {
		compatible = "loongson,cpu-interrupt-controller";
		interrupt-controller;
		#interrupt-cells = <1>;
	};

	icu: interrupt-controller@1fe01400 {
		compatible = "loongson,2k1000-icu";
		interrupt-controller;
		#interrupt-cells = <1>;
		reg = <0 0x1fe01400 0 0x40
			0 0x1fe01040 0 16>;
		interrupt-parent = <&cpuic>;
		interrupt-names = "cascade";
		interrupts = <3>; /* HW IP1 */
	};

	pinirq_ctrl: pinirq_ctrl@1fe00530 {
		compatible = "pinctrl-single";
		reg = <0 0x1fe00530 0 0x8>;
		#address-cells = <1>;
		#size-cells = <0>;
		#pinctrl-cells = <2>;
		pinctrl-single,bit-per-mux;
		pinctrl-single,register-width = <32>;
		pinctrl-single,function-mask = <0x1>;
		status = "disabled";
	};

	soc {
		compatible = "ls,nbus", "simple-bus";
		#address-cells = <2>;
		#size-cells = <2>;
		ranges = <0 0x10000000 0 0x10000000 0 0x10000000
			0 0x2000000  0 0x2000000  0 0x2000000
			0 0x20000000 0 0x20000000 0 0x10000000
			0 0x40000000 0 0x40000000 0 0x40000000
			0xfe 0x00000000 0xfe 0x00000000 0 0x40000000>;

		dma-coherent;

		osc_clk: osc-clock {
			#clock-cells = <0>;
			compatible = "fixed-clock";
			clock-frequency = <100000000>;
			clock-output-names = "ref_clk";
		};

		clks: clock-controller@1fe00480 {
			compatible = "loongson,ls2x-clk";
			reg = <0 0x1fe00480 0 1>;
			clocks = <&osc_clk>;
			clock-names = "ref_clk";
			#clock-cells = <1>;
		};

		cpu_uart0: serial@0x1fe20000 {
			compatible = "ns16550a";
			pinctrl-names = "default";
			pinctrl-0 = <&uart0_4_default>;
			reg = <0 0x1fe20000 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <0>;
			no-loopback-test;
			status = "okay";
		};

		uart1: serial@0x1fe20100 {
			compatible = "ns16550a";
			pinctrl-names = "default";
			pinctrl-0 = <&uart1_4_default>;
			reg = <0 0x1fe20100 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <0>;
			no-loopback-test;
			status = "disable";
		};

		uart2: serial@0x1fe20200 {
			compatible = "ns16550a";
			pinctrl-names = "default";
			pinctrl-0 = <&uart2_4_default>;
			reg = <0 0x1fe20200 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <0>;
			no-loopback-test;
			status = "disable";
		};

		uart3: serial@0x1fe20300 {
			compatible = "ns16550a";
			reg = <0 0x1fe20300 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <0>;
			no-loopback-test;
		};

		uart4: serial@0x1fe20400 {
			compatible = "ns16550a";
			reg = <0 0x1fe20400 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <1>;
			no-loopback-test;
		};

		uart5: serial@0x1fe20500 {
			compatible = "ns16550a";
			reg = <0 0x1fe20500 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <1>;
			no-loopback-test;
		};

		uart6: serial@0x1fe20600 {
			compatible = "ns16550a";
			reg = <0 0x1fe20600 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <1>;
			no-loopback-test;
			status = "disable";
		};

		uart7: serial@0x1fe20700 {
			compatible = "ns16550a";
			reg = <0 0x1fe20700 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <1>;
			no-loopback-test;
			status = "disable";
		};

		uart8: serial@0x1fe20800 {
			compatible = "ns16550a";
			reg = <0 0x1fe20800 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <2>;
			no-loopback-test;
			status = "disable";
		};

		uart9: serial@0x1fe20900 {
			compatible = "ns16550a";
			reg = <0 0x1fe20900 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <2>;
			no-loopback-test;
			status = "disable";
		};

		uart10: serial@0x1fe20a00 {
			compatible = "ns16550a";
			reg = <0 0x1fe20a00 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <2>;
			no-loopback-test;
			status = "disable";
		};

		uart11: serial@0x1fe20b00 {
			compatible = "ns16550a";
			reg = <0 0x1fe20b00 0 0x10>;
			clocks = <&clks CLK_UART>;
			interrupt-parent = <&icu>;
			interrupts = <2>;
			no-loopback-test;
			status = "disable";
		};

		pioA:gpio@0x1fe00500 {
			compatible = "loongson,loongson3-gpio";
			reg = <0 0x1fe00500 0 0x38>;
			ngpios = <64>;
			gpio_base = <0>;
			conf_offset = <0>;
			out_offset = <0x10>;
			in_offset = <0x20>;
			int_offset = <0x30>;
			in_start_bit = <0>;
			gpio-controller;
			#gpio-cells = <2>;

			support_irq;
			interrupt-parent = <&icu>;
			interrupts =
				<60>, <61>, <62>, <63>, <58>, <58>,
				<58>, <58>, <58>, <58>, <58>, <58>,
				<58>, <58>, <58>, <>,   <58>, <58>,
				<58>, <58>, <58>, <58>, <58>, <58>,
				<58>, <58>, <58>, <58>, <58>, <58>,
				<58>, <58>, <59>, <59>, <59>, <59>,
				<59>, <>,   <59>, <59>, <59>, <59>,
				<>,   <>,   <59>, <59>, <59>, <59>,
				<59>, <59>, <59>, <59>, <59>, <59>,
				<59>, <59>, <59>, <59>, <59>, <59>,
				<59>, <59>, <59>, <59>;
		};

		pmc: syscon@0x1fe27000 {
			compatible = "syscon";
			reg = <0x0 0x1fe27000 0x0 0x58>;
		};

		reboot {
			compatible ="syscon-reboot";
			regmap = <&pmc>;
			offset = <0x30>;
			mask = <0x1>;
		};

		poweroff {
			compatible ="syscon-poweroff";
			regmap = <&pmc>;
			offset = <0x14>;
			mask = <0x3c00>;
			value = <0x3c00>;
		};

		otg: otg@40000000 {
			compatible = "loongson,dwc2-otg";
			reg = <0 0x40000000 0 0x40000>;
			interrupt-parent = <&icu>;
			interrupts = <49>;
			dma-mask = <0x0 0xffffffff>;
			status = "disabled";
		};

		ohci@40070000 {
			compatible = "loongson,ls2k-ohci", "generic-ohci";
			reg = <0 0x40070000 0 0x8000>;
			interrupt-parent = <&icu>;
			interrupts = <51>;
			dma-mask = <0x0 0xffffffff>;
		};

		ehci@40060000 {
			compatible = "loongson,ls2k-ehci", "generic-ehci";
			reg = <0 0x40060000 0 0x8000>;
			interrupt-parent = <&icu>;
			interrupts = <50>;
			/* enable 64 bits dma-mask nedd setting 0x1fe00420 |= 1 << 36*/
			dma-mask = <0 0xffffffff>;
		};

		i2c0: i2c@1fe21000 {
			compatible = "loongson,ls-i2c";
			reg = <0 0x1fe21000 0 0x8>;
			interrupt-parent = <&icu>;
			interrupts = <22>;
			#address-cells = <1>;
			#size-cells = <0>;
			status = "disabled";
		};

		i2c1: i2c@1fe21800 {
			#address-cells = <1>;
			#size-cells = <0>;

			compatible = "loongson,ls-i2c";
			reg = <0 0x1fe21800 0 0x8>;
			interrupt-parent = <&icu>;
			interrupts = <23>;

			status = "disabled";
		};

		dc: dc@400c0000 {
			compatible = "loongson,ls2k1000-dc";
			reg = <0 0x400c0000 0 0x00010000>;
			interrupt-parent = <&icu>;
			interrupts = <28>;
			dma-mask = <0x00000000 0xffffffff>;
			lsdc,relax_alignment;
			status = "disabled";
		};

		gpu@40080000 {
			compatible = "vivante,gc";
			reg = <0 0x40080000 0 0x00040000>;
			interrupt-parent = <&icu>;
			interrupts = <29>;
			dma-mask = <0x00000000 0xffffffff>;
		};

		sata: ahci@400e0000 {
			compatible = "snps,spear-ahci";
			reg = <0 0x400e0000 0 0x10000>;
			interrupt-parent = <&icu>;
			interrupts = <19>;
			dma-mask = <0x0 0xffffffff>;
			pinctrl-0 = <&ahci_default>;
			pinctrl-names = "default";
			status = "disabled";
		};

		rtc0: rtc@1fe27800{
			#address-cells = <1>;
			#size-cells = <1>;
			compatible = "loongson,ls2k-rtc";
			reg = <0 0x1fe27800 0 0x100>;
			interrupt-parent = <&icu>;
			interrupts = <52>;
		};

		pwm0: pwm@1fe22000{
			compatible = "loongson,ls2k-pwm";
			reg = <0 0x1fe22000 0 0x10>;
			clocks = <&clks CLK_PWM>;
			clock-names = "pwm-clk";
			interrupt-parent = <&icu>;
			interrupts = <24>;
			#pwm-cells = <2>;
			pinctrl-0 = <&pwm0_default>;
			pinctrl-names = "default";
			status = "disabled";
		};

		pwm1: pwm@1fe22010{
			compatible = "loongson,ls2k-pwm";
			reg = <0 0x1fe22010 0 0x10>;
			clocks = <&clks CLK_PWM>;
			clock-names = "pwm-clk";
			interrupt-parent = <&icu>;
			interrupts = <25>;
			#pwm-cells = <2>;
			pinctrl-0 = <&pwm1_default>;
			pinctrl-names = "default";
			status = "disabled";
		};

		pwm2: pwm@1fe22020 {
			compatible = "loongson,ls2k-pwm";

			reg = <0 0x1fe22020 0 0xf>;
			clocks = <&clks CLK_PWM>;
			clock-names = "pwm-clk";
			interrupt-parent = <&icu>;
			interrupts = <34>;

			#pwm-cells = <2>;
			pinctrl-0 = <&pwm2_default>;
			pinctrl-names = "default";

			status = "disabled";
		};

		pwm3: pwm@1fe22030 {
			compatible = "loongson,ls2k-pwm";

			reg = <0 0x1fe22030 0 0xf>;
			clocks = <&clks CLK_PWM>;
			clock-names = "pwm-clk";
			interrupt-parent = <&icu>;
			interrupts = <35>;

			#pwm-cells = <2>;
			pinctrl-0 = <&pwm3_default>;
			pinctrl-names = "default";

			status = "disabled";
		};

		gmac0: ethernet@40040000 {
			compatible = "snps,dwmac-3.70a", "ls,ls-gmac";
			reg = <0 0x40040000 0 0x8000>;
			interrupt-parent = <&icu>;
			interrupts = <12 13>;
			interrupt-names = "macirq", "eth_wake_irq";
			phy-mode = "rgmii";
			bus_id = <0x0>;
			phy_addr = <0xffffffff>;
			dma-mask = <0xffffffff 0xffffffff>;
			status = "disabled";
		};

		gmac1: ethernet@40050000 {
			compatible = "snps,dwmac-3.70a", "ls,ls-gmac";
			reg = <0 0x40050000 0 0x8000>;
			pinctrl-0 = <&gmac1_default>;
			pinctrl-names = "default";
			interrupt-parent = <&icu>;
			interrupts = <14 15>;
			interrupt-names = "macirq", "eth_wake_irq";
			phy-mode = "rgmii";
			bus_id = <0x1>;
			phy_addr = <0xffffffff>;
			dma-mask = <0xffffffff 0xffffffff>;
			status = "disabled";
		};

		/* APB DMA controller nodes:
		 * apbdma node specify the commom property for dma node.
		 * the #config-nr must be 2,Used to provide APB sel region
		 * and APB DMA controler information.
		 */
		apbdma: apbdma@1fe00438{
			compatible = "loongson,ls-apbdma";
			reg = <0 0x1fe00438 0 0x8>;
			#config-nr = <2>;
		};
		/* DMA node should specify the apbdma-sel property using a
		 * phandle to the controller followed by number of APB sel
		 * region(max 9) and number of APB DMA controller(max 4).
		*/

		dma0: dma@1fe00c00 {
			compatible = "loongson,ls-apbdma-0";
			reg = <0 0x1fe00c00 0 0x8>;
			apbdma-sel = <&apbdma 0x0 0x0>;
			#dma-cells = <1>;
			dma-channels = <1>;
			dma-requests = <1>;
		};

		dma1: dma@1fe00c10 {
			compatible = "loongson,ls-apbdma-1";
			reg = <0 0x1fe00c10 0 0x8>;
			apbdma-sel = <&apbdma 0x5 0x1>;
			#dma-cells = <1>;
			dma-channels = <1>;
			dma-requests = <1>;
			status = "disabled";
		};

		dma2: dma@1fe00c20 {
			compatible = "loongson,ls-apbdma-2";
			reg = <0 0x1fe00c20 0 0x8>;
			apbdma-sel = <&apbdma 0x6 0x2>;
			#dma-cells = <1>;
			dma-channels = <1>;
			dma-requests = <1>;
			status = "disabled";
		};

		dma3: dma@1fe00c30 {
			compatible = "loongson,ls-apbdma-3";
			reg = <0 0x1fe00c30 0 0x8>;
			apbdma-sel = <&apbdma 0x7 0x3>;
			#dma-cells = <1>;
			dma-channels = <1>;
			dma-requests = <1>;
			status = "disabled";
		};

		dma4: dma@1fe00c40 {
			compatible = "loongson,ls-apbdma-4";
			apbdma-sel = <&apbdma 0x0 0x0>;
			reg = <0 0x1fe00c40 0 0x8>;
			#dma-cells = <1>;
			dma-channels = <1>;
			dma-requests = <1>;
			status = "disabled";
		};

		mmc: sdio@1fe2c000 {
			#address-cells = <2>;
			compatible = "loongson,ls2k_sdio";
			reg = <0 0x1fe2c000 0 0x1000>;
			interrupt-parent = <&icu>;
			interrupts = <31>;
			interrupt-names = "ls2k_mci_irq";
			clocks = <&clks CLK_APB>;

			cd-gpio = <&pioA 22 0>;
			dmas = <&dma1 1>;
			dma-names = "sdio_rw";
			dma-mask = <0xffffffff 0xffffffff>;

			status = "disabled";
		};

		spi0: spi@1fff0220 {
			compatible = "loongson,ls-spi";
			#address-cells = <1>;
			#size-cells = <0>;
			reg = <0 0x1fff0220 0 0x10>;
			clocks = <&clks CLK_SPI>;
			clock-names = "sclk";
			status = "disabled";
		};

		nand: nand@1fe26040 {
			#address-cells = <2>;
			compatible = "loongson,ls-nand";
			reg = <0 0x1fe26040 0 0x0
			       0 0x1fe26000 0 0x20>;
			interrupt-parent = <&icu>;
			interrupts = <44>;
			interrupt-names = "nand_irq";

			dmas = <&dma0 1>;
			dma-names = "nand_rw";
			dma-mask = <0xffffffff 0xffffffff>;

			status = "disabled";
		};

		/* CAN controller nodes:
		 * If you want to use the "can" function,enable the "can"
		 * controller by configure general configuration register 0.
		 */
		can0: can@1fe20c00{
			compatible = "nxp,sja1000";
			reg = <0 0x1fe20c00 0 0xff>;
			interrupt-parent = <&icu>;
			interrupts = <16>;
			pinctrl-0 = <&can0_default>;
			pinctrl-names = "default";

			status = "disabled";
		};
		can1: can@1fe20d00{
			compatible = "nxp,sja1000";
			reg = <0 0x1fe20d00 0 0xff>;
			interrupt-parent = <&icu>;
			interrupts = <17>;
			pinctrl-0 = <&can1_default>;
			pinctrl-names = "default";

			status = "disabled";
		};

		hda: sound@0x400d0000 {
			compatible = "loongson,ls2k-audio";
			reg = <0 0x400d0000 0 0xffff>;
			interrupt-parent = <&icu>;
			interrupts = <4>;
			pinctrl-0 = <&hda_default>;
			pinctrl-names = "default";

			status = "disabled";
		};

		i2s: ls1x-i2s@1fe2d000 {
			compatible = "loongson,ls1x-i2s";
			reg = <0 0x1fe2d000 0 0xff>;
			interrupt-parent = <&icu>;
			interrupts = <5>;
			clocks = <&clks CLK_APB>;
			status = "disabled";
			pinctrl-names = "default";
			pinctrl-0 = <&i2s_default>;
		};

		pcm: loongson1-pcm-audio {
			compatible = "loongson,pcm-audio";
			interrupt-parent = <&icu>;
			interrupts = <46>,
							<47>;
			loongson,dma_chan_w = <2>;
			loongson,dma_chan_r = <3>;
			status = "disabled";
		};

		tsensor: tsensor@1fe01500 {
			compatible = "loongson,ls2k-tsensor";
			reg = <0 0x1fe01500 0 0x30>;
			id = <0>;
			interrupt-parent = <&icu>;
			interrupts = <7>;
			#thermal-sensor-cells = <1>;
		};

		tz: thermal-zones {
			cpu_thermal: cpu-thermal {
				polling-delay-passive = <1000>;
				polling-delay = <5000>;
				thermal-sensors = <&tsensor 0>;

				trips {
					cpu_alert: cpu-alert {
						temperature = <60000>;
						hysteresis = <5000>;
						type = "active";
					};

					cpu_crit: cpu-crit {
						temperature = <125000>;
						hysteresis = <5000>;
						type = "critical";
					};
				};
			};
		};

		wdt:watchdog@1ff6c030{
			compatible = "loongson,ls2x-wdt";
			reg = <0 0x1fe27030 0 0xC>;
			clocks = <&clks CLK_WDT>;
			clock-names = "wdt-clk";
			status = "okay";
		};

		pcie@0 {
			compatible = "loongson,ls2k1000-pci";
			#interrupt-cells = <1>;
			bus-range = <0x1 0x16>;
			#size-cells = <2>;
			#address-cells = <3>;

			reg = < 0xfe 0x00000000 0 0x20000000>;
			ranges = <0x2000000 0x0 0x60000000 0 0x60000000 0x0 0x20000000 /* mem */
				0x01000000 0 0x00008000 0 0x18008000 0x0 0x8000>;

			pcie0_port0: pci_bridge@9,0 {
				compatible = "pciclass060400",
						   "pciclass0604";
				reg = <0x4800 0x0 0x0 0x0 0x0>;
				interrupts = <32>;
				interrupt-parent = <&icu>;

				#interrupt-cells = <1>;
				interrupt-map-mask = <0 0 0 0>;
				interrupt-map = <0 0 0 0 &icu 32>;
			};

			pcie0_port1: pci_bridge@10,0 {
				compatible = "pciclass060400",
						   "pciclass0604";
				reg = <0x5000 0x0 0x0 0x0 0x0>;
				interrupts = <33>;
				interrupt-parent = <&icu>;

				#interrupt-cells = <1>;
				interrupt-map-mask = <0 0 0 0>;
				interrupt-map = <0 0 0 0 &icu 33>;
			};

			pcie0_port2: pci_bridge@11,0 {
				compatible = "pciclass060400",
						   "pciclass0604";
				reg = <0x5800 0x0 0x0 0x0 0x0>;
				interrupts = <34>;
				interrupt-parent = <&icu>;

				#interrupt-cells = <1>;
				interrupt-map-mask = <0 0 0 0>;
				interrupt-map = <0 0 0 0 &icu 34>;
			};

			pcie_port3: pci_bridge@12,0 {
				compatible = "pciclass060400",
						   "pciclass0604";
				reg = <0x6000 0x0 0x0 0x0 0x0>;
				interrupts = <35>;
				interrupt-parent = <&icu>;

				#interrupt-cells = <1>;
				interrupt-map-mask = <0 0 0 0>;
				interrupt-map = <0 0 0 0 &icu 35>;
			};

			pcie1_port0: pci_bridge@13,0 {
				compatible = "pciclass060400",
						   "pciclass0604";
				reg = <0x6800 0x0 0x0 0x0 0x0>;
				interrupts = <36>;
				interrupt-parent = <&icu>;

				#interrupt-cells = <1>;
				interrupt-map-mask = <0 0 0 0>;
				interrupt-map = <0 0 0 0 &icu 36>;
			};

			pcie1_port1: pci_bridge@14,0 {
				compatible = "pciclass060400",
						   "pciclass0604";
				reg = <0x7000 0x0 0x0 0x0 0x0>;
				interrupts = <37>;
				interrupt-parent = <&icu>;

				#interrupt-cells = <1>;
				interrupt-map-mask = <0 0 0 0>;
				interrupt-map = <0 0 0 0 &icu 37>;
			};

		};
	};
};

```
