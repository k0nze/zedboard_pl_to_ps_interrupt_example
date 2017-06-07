<!--
Author: Konstantin Lübeck (University of Tübingen, Chair for Embedded Systems)
-->

# Setting up a PL to PS interrupt on the Zedboard

This tutorial shows you how to setup a PL to PS interrupt on the Zedboard using Vivado and the Xilinx SDK

## Requirements

- Vivado 2016.4
- Zedboard

## Creating a New Vivado Project

This part is pretty straight forward. You can look up how to setup a new Vivado project here:

[https://github.com/k0nze/zedboard_axi4_master_burst_example#creating-a-new-vivado-project](https://github.com/k0nze/zedboard_axi4_master_burst_example#creating-a-new-vivado-project)

Throughout this tutorial the name for the Vivado project is `pl_to_ps_interrupt_example`.

## Creating a Custom AXI4 IP

After you successfully created a new Vivado project carry out the following steps to create a custom AXI IP which will issue the interrupts from the PL to the PS with an AXI4-Lite slave interface.

1. Open: _Menu -> Tools -> Create and Package IP_.

    ![menu - tools - create and package ip](./images/create_and_package_ip01.png "menu - tools - create and package ip")


2. Click _Next >_

    ![click next](./images/create_and_package_ip02.png "click next")


3. Choose _Create a new AXI4 peripheral_.

    ![choose create a new axi4 peripheral](./images/create_and_package_ip03.png "choose create a new axi4 peripheral")


4. Choose a name, description and location for the new AXI4 peripheral. The name in this tutorial is `axi4_pl_interrupt_generator` and the location is `[...]/ip_repo`.

    ![choose name and description](./images/create_and_package_ip04.png "choose name and description")


5. Keep the AXI4-Lite slave interface and click _Next >_

    ![click next](./images/create_and_package_ip05.png "click next")


6. Choose _Edit IP_ and click _Finish_

    ![edit ip](./images/create_and_package_ip06.png "edit ip")


## Edit AXI4 IP

After the successful creation of the new AXI4 IP a new Vivado project was opened. In this project you can find the Vivado generated Verilog code for the AXI4-Lite slave and a top module (wrapper) which contains the AXI4-Lite slave.

![sources](./images/edit_ip01.png "sources")

* `axi4_pl_interrupt_generator_v1_0` contains the top module
* `axi4_pl_interrupt_generator_v1_0_S00_AXI_inst`contains the Verilog code for the AXI4-Lite slave.

The AXI4-Lite slave will be used to set and clear the interrupt from the PS.

Double-click on `axi4_pl_interrupt_generator_v1_0_S00_AXI_isnt` and navigate to the ports definition and add your own ports under `// Users to add ports here`.

```verilog
// Users to add ports here
output wire interrupt_0,
output wire interrupt_1,
// User ports ends
```

Those two wires will later be connected to outputs of the top module which will be then connected to the interrupt ports of the PS.

Navigate to `// Add user logic here` and add the following:

```verilog
// Add user logic here
assign interrupt_0 = slv_reg0[0:0];
assign interrupt_1 = slv_reg1[1:1];
// User logic ends
```

This will connect the wires to the LSB of the slave registers 0 and 1 to which the PS can write directly (`slv_reg0[0:0]` and `slv_reg1[0:0]`). 

The newly added ports of the AXI4-Lite slave also have to be added to the module instantiation in the top module `axi4_pl_interrupt_generator`. Double-click on `axi4_pl_interrupt_generator` in the sources tree and navigate to: `// Instantiation of Axi Bus Interface S00_AXI` and add the new ports to the port map:

```verilog
// Instantiation of Axi Bus Interface S00_AXI
axi4_pl_interrupt_generator_v1_0_S00_AXI # ( 
    .C_S_AXI_DATA_WIDTH(C_S00_AXI_DATA_WIDTH),
    .C_S_AXI_ADDR_WIDTH(C_S00_AXI_ADDR_WIDTH)
) axi4_pl_interrupt_generator_v1_0_S00_AXI_inst (
    
    .interrupt_0(interrupt_0),
    .interrupt_1(interrupt_1),
``` 

`interrupt_0` and `interrupt_1` will be connected to `interrupt_0` and `interrupt_1` of the top module. To add `interrupt_0` and `interrupt_1` to the top module navigate to `// Users to add ports here` and add the following:

```verilog
// Users to add ports here
output wire interrupt_0,
output wire interrupt_1,
// User ports ends
``` 

When the PS sets the LSB of `slv_reg0` or `slv_reg1` in the AXI4-Lite slave from 0 to 1 a rising edge will be seen at the output ports `interrrupt_0` and `inpterrupt_1` of the `axi4_pl_interrupt_generator`. This concludes the edits in the Verilog code of the AXI4-Lite slave. 

1. Click on the tab _Package IP - axi\_pl\_interrupt\_generator_.
    
    ![package ip](./images/edit_ip02.png "package ip")


2. Click on _File Groups_ in _Packaging Steps_.

    ![file groups](./images/edit_ip03.png "file groups")


3. Click on _Merge changes from File Groups Wizard_.

    ![merge changes](./images/edit_ip04.png "merge changes")


4. Click on _Customization Parameters_ in _Packaging Steps_.

    ![costomziation parameters](./images/edit_ip05.png "customization parameters")


5. Click on _Merge changes from customization Parameters Wizard_.

    ![merge changes](./images/edit_ip06.png "merge changes")


6. Click on _Review and Package_ in _Packaging Steps_.

    ![review and package](./images/edit_ip07.png "review and package")
    

7. Click on _Re-Package IP_.

    ![re-package ip](./images/edit_ip08.png "re-package ip")
    

8. Click _Yes_ to close the Project for the _axi4\_pl\_interrupt\_generator_.

    ![close project](./images/edit_ip09.png "close project")
   

You can go back to the Verilog code by clicking on _Flow Navigator -> Project Manager -> IP Catalog_.

![edit ip ip catalog](./images/edit_ip10.png "edit ip ip catalog")

And navigate to _User Repository -> AXI Peripheral -> axi4\_pl\_interrupt\_generator\_v1.0_ and right-click to open the context menu an choose _Edit in IP Packager_.

![edit ip edit in ip packager](./images/edit_ip11.png "edit ip edit in ip packager")


## Zynq Block Diagram

So that your custom AXI4 IP can be implemented on the Zynq PL and connected to the Zynq PS you have to create a block diagram in Vivado. The following steps will show you how to do that:

1. Click on _Flow Navigator -> IP Integrator -> Create Block Diagram_.

    ![create block diagram](./images/block_diagram01.png "create block diagram")


2. Choose a name, directory, and specify a source set for the block diagram. In this tutorial everything stays at its default.

    ![block diagram choose name](./images/block_diagram02.png "block diagram choose name")


3. Right-click on the white background of the _Diagram_ tab and choose _Add IP_.

    ![block diagram add ip](./images/block_diagram03.png "block diagram add ip")

4. From the list of IPs choose _ZYNQ7 Processing System_ (this is the Zynq PS) and double-click on it. 

    ![block diagram zynq ps](./images/block_diagram04.png "block diagram zynq ps")


5. You can now see the Zynq PS in the block diagram. Click on _Run Block Automation_ to connect the Zynq PS with the memory.

    ![block diagram run block automation](./images/block_diagram05.png "block diagram run block automation")


6. Leave everything at the default values and click on _OK_.

    ![block diagram run block automation ok](./images/block_diagram06.png "block diagram run block automation ok")


7. To connect the interrupt ports of your AXI4 IP to the Zynq PS the Zynq PS needs interrupt ports. To enable those interrupt ports double-click on the Zynq PS in the block diagram.

    ![block diagram zynq ps](./images/block_diagram07.png "block diagram zynq ps")

8. In the _Re-customize IP_ window go to _Page -> Navigator -> Interrupts_.

    ![block diagram interrupts](./images/block_diagram08.png "block diagram interrupts")


9. Unfold _Fabric Interrupts -> PL-PS Interrupt Ports_ and check _IRQ\_F2P[15:0]_ and click _OK_.

    ![block diagram irq f2p](./images/block_diagram09.png "block diagram irq f2p")


10. Now its time to add your custom AXI4 IP. Right-click on the white background of the _Diagram_ tab and choose _Add IP_.

    ![block diagram add ip](./images/block_diagram03.png "block diagram add ip")


11. From the list of IPs choose _axi4\_pl\_interrupt\_generator\_v1.0_ (this is your custom AXI4 IP) and double click on it to add it to the block diagram. 

    ![block diagram axi4 ip](./images/block_diagram10.png "block diagram axi4 ip")


12. Click on _Run Connection Automation_ to connect your custom AXI4 IP to the Zynq PS via the AXI4 bus. 

    ![block diagram run connection automation](./images/block_diagram11.png "block diagram connection automation")


13. Check _S00\_AXI_ in the tree on the left-hand side. Select _/processing\_system7\_0/FCLK\_CLK0_ in the list of _Clock Connections_ and click on OK.

    ![block diagram clock connection](./images/block_diagram12.png "block diagram clock connection")


14. After the connection automation is done click on ![block diagram regenerate layout](./images/block_diagram13.png "block diagram regenerate layout") to regenerate the layout of the block diagram. Your block diagram should now look like this:

    ![block diagram axi4 connection done](./images/block_diagram14.png "block diagram axi4 connection done")


15. To connect the _interrupt\_0_ and _interrupt\_1_ outputs of your custom AXI4 IP to the Zynq PS. Add another module by right-clicking on the white background and choose _Add IP_ and select _Concat_.

    ![block diagram concat](./images/block_diagram15.png "block diagram concat")


16. Connect the _In0_ and _In1_ inputs of the _xlconcat\_0_ module to the _interrupt\_0_ and _interrupt\_1_ outputs of your custom AXI4 IP by hovering with the curser over on of _interrupt\_*_ outputs and drawing a line a with the pencil to one of the _In*_ inputs. Do this for both _interupt\_*_ outputs.

    ![block diagram draw connection](./images/block_diagram16.png "block diagram connection")


17. Afterwards connect the _dout_ output of the _xlconcat\_0_ to the _IRQ\_F2P_ input of the Zynq PS.

    ![block diagram zynq ps interrupt connection](./images/block_diagram17.png "block diagram zynq ps interrupt connection")


18. The block diagram is now finished. In the _Sources Panel_ navigate to _Design Sources -> design\_1_.

    ![block diagram sources panel](./images/block_diagram18.png "block diagram sources panel")


19. Right-click on _design\_1_ and choose _Create HDL Wrapper_. This generates HDL code for the block diagram which is necessary for the synthesis.

    ![block diagram create hdl wrapper](./images/block_diagram19.png "block diagram create hdl wrapper")


20. Choose _Let Vivado manage wrapper and auto-update_ and click _OK_. This will always update your HDL wrapper when the block diagram was changed.

    ![block diagram update hdl wrapper](./images/block_diagram20.png "block diagram update hdl wrapper")


21. Afterwards output products for the your block diagramm have to generated. Navigate in the Sources Panel to _design\_1\_i_, right-click on it and choose _Generate Output Products_

    ![block diagram generate output products](./images/block_diagram21.png "block diagram generate output products")


22. Leave everything at its default and click _Generate_. 

    ![block diagram generate](./images/block_diagram22.png "block diagram generate")


## Synthesis and implementation
To bring the custom AXI4 IP with the block diagram to the Zynq PL you have to synthesize and implement it.

1. Start the synthesis by click on Run Synthesis in _Flow Navigator -> Synthesis_.

    ![run synthesis](./images/synthesis_and_implementation01.png "run synthesis")


2. Leave everything at its default and click _OK_ to launch the systhesis. 

    ![launch synthesis](./images/synthesis_and_implementation02.png "launch synthesis")


3. After the synthesis is finished choose _Run implementation_ and click on _OK_ to run the implementation.

    ![run implementation](./images/synthesis_and_implementation03.png "run implementation")


4. When the implementation is finished choose _Generate Bitstream_ and click on _OK_ to generate the bitstream which contains the configuration data for the Zynq PL. 

    ![generate bitstream](./images/synthesis_and_implementation04.png "generate bitstream")


5. Lastly, when the bitstream generation is finished you can look at the reports to see if all contraints are fulfilled. Choose _View Reports_ and click OK. (However, this is not necessary here since the design is very simple).

    ![view reports](./images/synthesis_and_implementation05.png "view reports")


## Software for the Zynq PS

To program the Zedboard and talk to it via UART you have to connect it to the power supply and connect two USB cables from your computer to the following USB ports on the Zedboard.

![zedboard connection](./images/zedboard01.jpg "zedboard connection")

Make sure your Zedboard is turned on. If the green _POWER_ led is on the Zedboard is turned on.

![zedboard led](./images/zedboard02.jpg "zedboard led")

The C program which will be transferred to the Zynq PS is going to setup the interrupt system of the Zynq PS and enables the interrupts for the `IRQ_F2P[1:0]` ports for a rising edge. When the interrupt system is enabled the interrupts will be generated by writing a `1` into `slv_reg0[0:0]` and `slv_reg1[0:0]`. This will trigger the interrupt service routines `isr0` and `isr1` which will clear the interrupts by writing a `0` to `slv_reg0[0:0]` or `slv_reg1[0:0]`. Then the interrupt for `IRQ_F2P[1:1]` will be disabled and by writing a `1` into `slv_reg0[0:0]` and `slv_reg1[0:0]` new interrupts will be generated. This will only trigger the interrupt service routine `isr0` since the interrupt `IRQ_F2P[1:1]` is disabled.

1. You have to export the hardware configuration to the Xilinx SDK. Go to _Menu -> File -> Export -> Export Hardware ..._.

    ![menu export hardware](./images/software01.png "menu export hardware")

2. Check _Include bitstream_ and click _OK_.

    ![export hardware](./images/software02.png "export hardware")


3. To launch the Xilinx SDK go to _Menu -> File -> Launch SDK_

    ![lauch sdk](./images/software03.png "lauch sdk")

4. When the Xilinx SDK is ready create a new project by going to _Menu -> File -> New -> Application Project_.

    ![new application project](./images/software04.png "new application project")


5. Choose a _Project name_ and leave all other parameters at their default value and click on _Next >_. The _Project name_ in this tutorial is _axi4\_pl\_interrupt\_generator\_test_. 

    ![project name](./images/software05.png "project name")


6. Choose _Hello World_ under _Available Templates_ and click on finish. This creates a simple Hello World program for the Zynq PS.

    ![hello world](./images/software06.png "hello world")


7. After the project was successfully created open `helloworld.c` under _Project Explorer -> axi4\_pl\_interrupt\_generator\_test -> src -> helloworld.c_ 

    ![helloworld.c](./images/software07.png "helloworld.c")


8. Replace the Hello World C code with:

    ```c
    #include <stdio.h>
    #include "platform.h"
    #include "xil_printf.h"
    #include "xbasic_types.h"
    #include "xscugic.h"
    #include "xil_exception.h"

    #define INTC_INTERRUPT_ID_0 61 // IRQ_F2P[0:0]
    #define INTC_INTERRUPT_ID_0 62 // IRQ_F2P[1:1]

    // instance of interrupt controller
    static XScuGic intc;

    // address of AXI PL interrupt generator
    Xuint32* baseaddr_p = (Xuint32*) XPAR_AXI4_PL_INTERRUPT_GENERATOR_0_S00_AXI_BASEADDR;

    int setup_interrupt_system();

    void isr0 (void *intc_inst_ptr);
    void isr1 (void *intc_inst_ptr);
    void nops(unsigned int num); 

    int main() {
        init_platform();

        xil_printf("== START ==\n\r");
        // set interrupt_0/1 of AXI PL interrupt generator to 0
        *(baseaddr_p+0) = 0x00000000;
        *(baseaddr_p+1) = 0x00000000;

        xil_printf("Checkpoint 1\n\r");

        // set interrupt_0/1 of AXI PL interrupt generator to 1
        *(baseaddr_p+0) = 0x00000001;
        *(baseaddr_p+1) = 0x00000001;

        xil_printf("Checkpoint 2\n\r");
        // read interrupt_0/1 of AXI PL interrupt generator
        xil_printf("slv_reg0: 0x%08x\n\r", *(baseaddr_p+0));
        xil_printf("slv_reg1: 0x%08x\n\r", *(baseaddr_p+1));

        // set interrupt_0/1 of AXI PL interrupt generator to 0
        *(baseaddr_p+0) = 0x00000000;
        *(baseaddr_p+1) = 0x00000000;

        xil_printf("Checkpoint 3\n\r");
        // read interrupt_0/1 of AXI PL interrupt generator
        xil_printf("slv_reg0: 0x%08x\n\r", *(baseaddr_p+0));
        xil_printf("slv_reg1: 0x%08x\n\r", *(baseaddr_p+1));

        xil_printf("Checkpoint 4\n\r");
        // setup and enable interrupts for IRQ_F2P[1:0]
        int status = setup_interrupt_system();
        if (status != XST_SUCCESS) {
             return XST_FAILURE;
        }

        xil_printf("Checkpoint 5\n\r");
        nops(1000);
        // set interrupt_0 of AXI PL interrupt generator to 1 (isr0 will be called)
        *(baseaddr_p+0) = 0x00000001;

        xil_printf("Checkpoint 6\n\r");
        nops(1000);
        // set interrupt_1 of AXI PL interrupt generator to 1 (isr1 will be called)
        *(baseaddr_p+1) = 0x00000001;

        // disable interrupts for IRQ_F2P[1:1]
        XScuGic_Disable(&intc, INTC_INTERRUPT_ID_1);

        xil_printf("Checkpoint 7\n\r");
        nops(1000);
        // set interrupt_0 of AXI PL interrupt generator to 1 (isr0 will be called)
        *(baseaddr_p+0) = 0x00000001;

        xil_printf("Checkpoint 8\n\r");
        nops(1000);
        // set interrupt_1 of AXI PL interrupt generator to 1
        // (isr1 wont be called since interrupts for IRQ_F2P[1:1] are disabled)
        *(baseaddr_p+1) = 0x00000001;

        xil_printf("== STOP ==\n\r");

        cleanup_platform();
        return 0;
    }

    // interrupt service routine for IRQ_F2P[0:0]
    void isr0 (void *intc_inst_ptr) {
        xil_printf("isr0 called\n\r");
        *(baseaddr_p+0) = 0x00000000;
    }

    // interrupt service routine for IRQ_F2P[1:1]
    void isr1 (void *intc_inst_ptr) {
        xil_printf("isr1 called\n\r");
        *(baseaddr_p+1) = 0x00000000;
    }

    // sets up the interrupt system and enables interrupts for IRQ_F2P[1:0]
    int setup_interrupt_system() {

        int result;
        XScuGic *intc_instance_ptr = &intc;
        XScuGic_Config *intc_config;

        // get config for interrupt controller
        intc_config = XScuGic_LookupConfig(XPAR_PS7_SCUGIC_0_DEVICE_ID);
        if (NULL == intc_config) {
            return XST_FAILURE;
        }

        // initialize the interrupt controller driver
        result = XScuGic_CfgInitialize(intc_instance_ptr, intc_config, intc_config->CpuBaseAddress);

        if (result != XST_SUCCESS) {
            return result;
        }

        // set the priority of IRQ_F2P[0:0] to 0xA0 (highest 0xF8, lowest 0x00) and a trigger for a rising edge 0x3.
        XScuGic_SetPriorityTriggerType(intc_instance_ptr, INTC_INTERRUPT_ID_0, 0xA0, 0x3);

        // connect the interrupt service routine isr0 to the interrupt controller
        result = XScuGic_Connect(intc_instance_ptr, INTC_INTERRUPT_ID_0, (Xil_ExceptionHandler)isr0, (void *)&intc);

        if (result != XST_SUCCESS) {
            return result;
        }

        // enable interrupts for IRQ_F2P[0:0]
        XScuGic_Enable(intc_instance_ptr, INTC_INTERRUPT_ID_0);

        // set the priority of IRQ_F2P[1:1] to 0xA8 (highest 0xF8, lowest 0x00) and a trigger for a rising edge 0x3.
        XScuGic_SetPriorityTriggerType(intc_instance_ptr, INTC_INTERRUPT_ID_1, 0xA8, 0x3);

        // connect the interrupt service routine isr1 to the interrupt controller
        result = XScuGic_Connect(intc_instance_ptr, INTC_INTERRUPT_ID_1, (Xil_ExceptionHandler)isr1, (void *)&intc);

        if (result != XST_SUCCESS) {
            return result;
        }

        // enable interrupts for IRQ_F2P[1:1]
        XScuGic_Enable(intc_instance_ptr, INTC_INTERRUPT_ID_1);

        // initialize the exception table and register the interrupt controller handler with the exception table
        Xil_ExceptionInit();

        Xil_ExceptionRegisterHandler(XIL_EXCEPTION_ID_INT, (Xil_ExceptionHandler)XScuGic_InterruptHandler, intc_instance_ptr);

        // enable non-critical exceptions
        Xil_ExceptionEnable();

        return XST_SUCCESS;
    }

    void nops(unsigned int num) {
        int i;
        for(i = 0; i < num; i++) {
            asm("nop");
        }
    }
    ```

9. Program the Zynq PL with the previously generated bitstream by going to _Menu -> Xilinx Tools -> Program FPGA_.

    ![menu program fpga](./images/software08.png "menu program fpga")


10. Click on _Program_.

    ![program fpga](./images/software09.png "program fpga")


11. The Zynq PS will write the content of all `xil_printf("...");` statments to the UART.
    
    On Linux you can connect to the UART of the Zedboard with the following `picocom` command:
```bash
picocom /dev/ttyACM0 -b 115200 -d 8 -y n -p 1
```
12. After you programed the Zynq PL you can and connected to the UART you can build and run the Program on the Zynq PS by clicking on ![build](./images/software10.png "build") and then on ![run](./images/software11.png "run").


13. Choose _Lauch on Hardware (System Debugger)_ and click on _OK_.

    ![run on hardware](./images/software12.png "run on hardware")


14. You should see the following output in `picocom`:

```
== START ==
Checkpoint 1
Checkpoint 2
slv_reg0: 0x00000001
slv_reg1: 0x00000001
Checkpoint 3
slv_reg0: 0x00000000
slv_reg1: 0x00000000
Checkpoint 4
Checkpoint 5
isr0 called
Checkpoint 6
isr1 called
Checkpoint 7
isr0 called
Checkpoint 8
== STOP ==
```

You can leave `picocom` with [CTRL]+[A] [CTRL]+[Q].
