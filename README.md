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

Thoughout this tutorial the name for the Vivado project is `pl_to_ps_interrupt_example`.

## Creating a Custom AXI4 IP

After you successfully created a new Vivado project do the following steps to create a custom AXI IP which will issue the interrupts from the PL to the PS with an AXI slave interface.

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

After the successful creation of the new IP a new Vivado project was opened. In this project you can find the Vivado generated Verilog code for the AXI4-Lite slave and a top module (wrapper) which contains the AXI4-Lite slave.

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

`interrupt_0`and `interrupt_1` will be connected to `interrupt_0` and `interrupt_1` of the top module. To add `interrupt_0` and `interrupt_1` to the top module navigate to `// Users to add ports here` and add the following:

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


2. Choose a name, directory, and specify a source set for the block diagram. In this tutorial everything stays at the default.

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


13. Check _S00\_AXI_ in the tree on the left-hand side. Select _/processing\_system7\_0/FCLK\ CLK0_ in the list of _Clock Connections_ and click on OK.

    ![block diagram clock connection](./images/block_diagram12.png "block diagram clock connection")


14. After the connection automation is done click on ![block diagram regenerate layout](./images/block_diagram13.png "block diagram regenerate layout") to regenerate the layout of the block diagram. Your block diagram should now look like this:

    ![block diagram axi4 connection done](./images/block_diagram14.png "block diagram axi4 connection done")


15. To connect the _interrupt\_0_ and _interrupt\_1_ outputs of your custom AXI4 IP to the Zynq PS. Add another module by right-clicking on the white background and choose _Add IP_ and select _Concat_.

    ![block diagram concat](./images/block_diagram15.png "block diagram concat")


16. Connect the _In0_ and _In1_ inputs of the _xlconcat\_0_ module to the _interrupt\_0_ and _interrupt\_1_ outputs of your custom AXI4 IP by hovering with the curser over on of _interrupt\_*_ outputs and drawing a line a with the pencil to one of the _In*_ inputs. Do this for both _interupt\_*_ outputs.

    ![block diagram draw connection](./images/block_diagram16.png "block diagram connection")


17. Afterwards connection the _dout_ output of the _xlconcat\_0_ to the _IRQ\_F2P_ input of the Zynq PS.

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
