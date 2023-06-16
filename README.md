<img src="./images/IFX_LOGO_600.gif" align="right" width="150"/>

# P-DMA 2D Transfer

**This code example demonstrates the transmission of data by Peripheral Direct Memory Access (P-DMA) 2D Transfer, shows its operation, initial setting, and interrupt handling.**

## Device

The device used in this code example (CE) is:

- [TRAVEO™ T2G CYT4BF Series](https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/32-bit-traveo-t2g-arm-cortex-for-body/traveo-t2g-cyt4bf-series/)

## Board

The board used for testing is:

- TRAVEO™ T2G evaluation kit ([KIT_T2G-B-H_EVK](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g-b-h_evk/), [KIT_T2G-B-H_LITE](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g-b-h_lite/))

## Scope of work

This code example demonstrates the transmission of data by P-DMA with 2D-Transfer.

## Introduction

**P-DMA**

The TRAVEO™ T2G device supports two kinds of DMA controllers: Peripheral DMA (P-DMA) and Memory DMA (M-DMA). The main difference between the M-DMA and P-DMA controllers is that the M-DMA controller has dedicated channel logic (with channel state) for each channel, whereas the P-DMA reuses the channel logic for all channels.
<table border="1" style="border-collapse: collapse">
<thead><tr>
<th>Feature</th><th>P-DMA</th><th>M-DMA</th></tr></thead>
<tbody>
<tr><td><code>Focus</code></td><td>Low latency</td><td>High memory bandwidth</td></tr>
<tr><td><code>Used for</code></td><td>Transfer between peripheral and memory</td><td>Transfer between memories</td></tr>
<tr><td><code>Transfer engine</code></td><td>Shared between all channels</td><td>Dedicated for each channel</td></tr>
<tr><td><code>Transfer size</code></td><td colspan="2">8-bit/16-bit/32-bit</td></tr>
<tr><td><code>Channel priority</code></td><td>Four levels, Preemptable</td><td>Four levels</td></tr>
<tr><td><code>Transfer mode</code></td><td>Single<BR>1D/2D<BR>CRC transfer</td><td>Single<BR>1D/2D<BR>Memory copy<BR>Scatter</td></tr>
<tr><td><code>Descriptor</code></td><td colspan="2">Source and destination address<BR>Transfer size<BR>Channel action<BR>Data transfer mode<BR>Activation trigger type (4 types)<BR>Output trigger type (4 types)<BR>Interrupt type (4 types)<BR>Descriptor chaining</td></tr>
<tr><td><code>Access-control attributes</code><BR><i>The access attributes of the bus transfer that programmed the channel will be inherited</i></td><td colspan="2">Privileged/Unprivileged<BR>Secure/Non-secure<BR>Protection contexts</td></tr>
</tbody>
</table>

The P-DMA controller is part of the CPUSS and controls data transfer between peripherals and memory. This controller can be configured/programmed to perform multiple independent data transfers. Each data transfer is managed by a channel. The Descriptor type are Single transfer, 1D/2D transfer and CRC transfer.

**UART included in Serial Communication Block (SCB)**

- Supports UART protocol
  - Standard UART
  - Multi-processor mode
- SmartCard (ISO7816) reader
- IrDA
- Supports LIN
  - Break detection
  - Baud rate detection
  - Collision detection (ability to detect that a driven bit value is not reflected on the bus, indicating that another component is driving the same bus)
- Data frame size programmable from 4 to 16 bits
- Programmable number of STOP bits, which can be set in terms of half bit periods between 1 and 4
- Parity support (odd and even parity)
- Median filter on RX input
- Programmable oversampling
- Start skipping
- FIFO mode operation only
- Local loop-back control

More details can be found in [Technical Reference Manual (TRM)](https://www.infineon.com/dgdl/?fileId=5546d4627600a6bc017600bfae720007), [Registers TRM](https://www.infineon.com/dgdl/?fileId=5546d4627600a6bc017600be2aef0004) and [Data Sheet](https://www.infineon.com/dgdl/?fileId=5546d46275b79adb0175dc8387f93228).

## Hardware setup

This CE has been developed for:
- TRAVEO™ T2G evaluation kit ([KIT_T2G-B-H_EVK](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g-b-h_evk/))<BR>
<img src="./images/KIT_T2G-B-H_EVK.gif"/><BR>
No changes are required from the board's default settings.

- TRAVEO™ T2G Body High Lite evaluation kit ([KIT_T2G-B-H_LITE](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g-b-h_lite/))<BR>
<img src="./images/KIT_T2G-B-H_LITE.gif"/><BR>
No changes are required from the board's default settings.

## Implementation

This design consists of P-DMA, a user button. The P-DMA is set to initiate transfers by software triggers. Pressing the user button generates an interrupt. When interrupt occurs, P-DMA transfer is triggered by software. Then, P-DMA transfers data from a source address to a destination address.

The transmitted data are composed of 64 unique values on code flash memory, and transferred to UART TX FIFO using 2D transfer in single descriptor. The terminal software connected to KitProg3 COM port will print out the data transferred. The 2D transfer is configured as figure below:
<img src="./images/overview.png" width="800" /><BR>
This 2D transfer is configured as do X-loop four times by one trigger, and Y-loop is configured as perform four times as well. In X-loop, P-DMA fetches one-byte data from source table and expands it to 32bit data then transfers it to TX_FIFO_WR register of SCB.

**P-DMA configuration**

The P-DMA initialization is done in following steps.

- To initialize DMA Descriptor, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__descriptor__functions.html#gab1610fab6f2dd5fd25c031ac0aa9d8cb"><i>Cy_DMA_Descriptor_Init()</i></a> is called with using structure <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/structcy__stc__dma__descriptor__config__t.html"><i>cy_stc_dma_descriptor_config_t</i></a> and <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/structcy__stc__dma__descriptor__t.html"><i>cy_stc_dma_descriptor_t</i></a> which are auto-coded by Device Configurator as argument.

- To initialize P-DMA channel, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__channel__functions.html#gab0beed3c05362206ac66fc4327d8964b"><i>Cy_DMA_Channel_Init()</i></a> is called with using structure <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/structcy__stc__dma__channel__config__t.html"><i>cy_stc_dma_channel_config_t</i></a> which are auto-coded by Device Configurator as argument.
  - *P-DMA setting*<BR><img src="./images/DMA_DeviceConfigurator.png"/>

- Source and destination addresses are specified by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__descriptor__functions.html#ga2df838cf16522aa18ad713fd05107b4c"><i>Cy_DMA_Descriptor_SetSrcAddress()</i></a> and <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__descriptor__functions.html#ga2c7111653150314961db83f7941a03c2"><i>Cy_DMA_Descriptor_SetDstAddress()</i></a> respectively.

- To sets a descriptor as current for the specified P-DMA channel, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__channel__functions.html#ga4c5cca40de761c4b7827ce65538bf12d"><i>Cy_DMA_Channel_SetDescriptor()</i></a> is called with using structure <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/structcy__stc__dma__descriptor__t.html"><i>cy_stc_dma_descriptor_t</i></a> which are auto-coded by Device Configurator as argument.

- To make an interrupt when the data transfer is completed, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__channel__functions.html#gab3e0528d8e57ee04dbbee89ffca38545"><i>Cy_DMA_Channel_SetInterruptMask()</i></a> is called and the interrupt service routine is registered by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__sysint__functions.html#gab2ff6820a898e9af3f780000054eea5d"><i>Cy_SysInt_Init()</i></a> and enabled by *NVIC_EnableIRQ()*.

- After that enable P-DMA by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__channel__functions.html#ga13e6923cafe660ba825ba6d5b1841bf7"><i>Cy_DMA_Channel_Enable()</i></a> and <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__block__functions.html#gad14b53935dc8321f395e49a7dfec2c5a"><i>Cy_DMA_Enable()</i></a>.

**UART control**

The UART is initialized by the PDL function <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__scb__uart__general__functions.html#gacbc061f623d2fafbcfffcb58c5f2db51"><i>Cy_SCB_UART_Init()</i></a>.
  - The pin is specified to use KitProg3 COM port
  - The serial port parameters become to 8N1 and 115200 baud

Then call <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__scb__uart__general__functions.html#ga503d0737dadf35620306ef3571d84905"><i>Cy_SCB_UART_Enable()</i></a> to enable the UART channel.

To output some guidance of this CE to the terminal, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__scb__uart__low__level__functions.html#ga4092c34575a47087e08a58529451914b"><i>Cy_SCB_UART_PutString()</i></a> is called respectively.

**GPIO port pin initialization**

Initialization of the GPIO port pin is done in the <a href="https://infineon.github.io/mtb-hal-cat1/html/group__group__hal__gpio.html#gab93322030909d3af6a9fc1a3b2eccbaa"><i>cyhal_gpio_init()</i></a> function.

- Initialize the pin specified by *CYBSP_USER_BTN1* as input
- For the port of *CYBSP_USER_BTN1*, configuration of the GPIO interrupt is done once in the <a href="https://infineon.github.io/mtb-hal-cat1/html/group__group__hal__gpio.html#gaaf872e66c1934c8166f386a55e74707c"><i>cyhal_gpio_register_callback()</i></a> and <a href="https://infineon.github.io/mtb-hal-cat1/html/group__group__hal__gpio.html#ga0e0346810451d9940d31bb6111153593"><i>cyhal_gpio_enable_event()</i></a> function.

**P-DMA Transfer**

- When pressing the user button is detected, the *handle_GPIO_Interrupt_SW1()* is called and set the *g_isInterrupt_SW1*.

- Next, input a software trigger to P-DMA by calling <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__trigmux__functions.html#gad3c1d26d25a47bc4beca499bf0407c80"><i>Cy_TrigMux_SwTrigger()</i></a>. P-DMA performs a transfer for each trigger input, as a result 4 bytes from the source table are output to the terminal.

- *handle_PDMA_Interrupt()* is called when the transfer of each X-loop completion.
  - This handler firstly clears the interrupt by calling <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__channel__functions.html#ga9c5df2a8d891163cc1cc996e7d76c967"><i>Cy_DMA_Channel_ClearInterrupt()</i></a>.
  - To check the interrupt reason, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__dma__channel__functions.html#ga6ef61cb63dafd8b822dea5f2e6995614"><i>Cy_DMA_Channel_GetStatus()</i></a> is called.
    - If the reason is *CY_DMA_INTR_CAUSE_COMPLETION*, display a message to the terminal.

## Run and Test

For this CE, a terminal emulator is required for displaying outputs. Install a terminal emulator if you do not have one. Instructions in this document use [Tera Term](https://ttssh2.osdn.jp/index.html.en).

After code compilation, perform the following steps to flashing the device:

1. Connect the board to your PC using the provided USB cable through the KitProg3 USB connector.
2. Open a terminal program and select the KitProg3 COM port. Set the serial port parameters to 8N1 and 115200 baud.
3. Program the board using one of the following:
    - Select the CE project in the Project Explorer.
    - In the **Quick Panel**, scroll down, and click **[Project Name] Program (KitProg3_MiniProg4)**.
4. After programming, the CE starts automatically.
    - Confirm that the messages are displayed on the UART terminal.
      - *Terminal output on program startup*<BR><img src="./images/uart_start.png" width="640"/><BR>
      - *Print out the result after press USER Button*<BR><img src="./images/uart.png" width="640"/><BR>

5. You can debug the example to step through the code. In the IDE, use the **[Project Name] Debug (KitProg3_MiniProg4)** configuration in the **Quick Panel**. For details, see the "Program and debug" section in the [Eclipse IDE for ModusToolbox™ software user guide](https://www.infineon.com/dgdl/?fileId=8ac78c8c8386267f0183a8d7043b58ee).

**Note:** **(Only while debugging)** On the CM7 CPU, some code in *main()* may execute before the debugger halts at the beginning of *main()*. This means that some code executes twice: once before the debugger stops execution, and again after the debugger resets the program counter to the beginning of *main()*. See [KBA231071](https://community.infineon.com/t5/Knowledge-Base-Articles/PSoC-6-MCU-Code-in-main-executes-before-the-debugger-halts-at-the-first-line-of/ta-p/253856) to learn about this and for the workaround.

## References

Relevant Application notes are:

- AN235305 - GETTING STARTED WITH TRAVEO™ T2G FAMILY MCUS IN MODUSTOOLBOX™
- [AN220191](https://www.infineon.com/dgdl/?fileId=8ac78c8c7cdc391c017d0d3aee0b678a) - How To Use Direct Memory Access (DMA) Controller In TRAVEO™ T2G FAMILY
- [AN225401](https://www.infineon.com/dgdl/?fileId=5546d462749a7c2d01749b3592060cea) - How to Use Serial Communications Block (SCB) in Traveo II Family
- [AN219842](https://www.infineon.com/dgdl/?fileId=8ac78c8c7cdc391c017d0d3a490a6732) - How to use interrupt in TRAVEO™ II

ModusToolbox™ is available online:

- <https://www.infineon.com/modustoolbox>

Associated TRAVEO™ T2G MCUs can be found on:

- <https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/>

More code examples can be found on the GIT repository:

- [TRAVEO™ T2G Code examples](https://github.com/orgs/Infineon/repositories?q=mtb-t2g-&type=all&language=&sort=)

For additional trainings, visit our webpage:

- [TRAVEO™ T2G trainings](https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/32-bit-traveo-t2g-arm-cortex-for-body/traveo-t2g-cyt4bf-series/#!trainings)

For questions and support, use the TRAVEO™ T2G Forum:  

- <https://community.infineon.com/t5/TRAVEO-T2G/bd-p/TraveoII>  
