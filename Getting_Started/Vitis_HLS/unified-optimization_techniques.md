﻿<table class="sphinxhide" width="100%">
 <tr>
   <td align="center"><img src="https://raw.githubusercontent.com/Xilinx/Image-Collateral/main/xilinx-logo.png" width="30%"/><h1>Vitis™ Application Acceleration Tutorials</h1>

   </td>
 </tr>
 <tr>
 <td>
 </td>
 </tr>
</table>


# 3. Using Optimization Techniques

This lab discusses various optimization techniques that you can use to reconcile performance concerns related to your design. You are going to be setting and changing configuration options in the `hls_config.cfg` file to drive the synthesis process using the Config File Editor. 

1.  Open the `hls_config.cfg` file. 

There are a number of ways to find the file. The last time you opened it from the `vitis-comp.json` of the HLS component. This time go to the Vitis Components Explorer, expand the HLS component folders, and click the `hls_config.cfg` file in the *Settings* folder. 

## Loop Pipeline

 ![Config Compile](./images/unified-hls-config-compile.png)

2.  Select the **Compile** category on the left side of the Config File Editor. 
3.  Locate the `compile.pipeline_loops` option and set it to 5. 

This indicates that the tool should automatically unroll loops with six iterations or more. The default setting is 64.

4.  Change the view of the Config Editor to the **Source Editor** by selecting the icon at the top of the editor window. 

 ![Config Toolbar](./images/unified-hls-config-toolbar.png)

This lets you examine and edit the `hls_config.cfg` as a text file file rather than through the GUI. Notice the addition of the `syn.compile.pipeline_loops=5` command in the config file. 

5.  In the Flow Navigator click the **Run** command under C Synthesis to rerun with the new directive.
6.  Examine the updated reports to see if there is any performance improvement. 

If any of the reports are opened when the tool regenerates them you will see an out-of-date notice appear in the open report. You will need to close the report and then reopen it.

 ![Report Out-of-Date](./images/unified-hls-synthesis-out-of-date.png)

## Pipeline Directive

**ACTION:** Back out the prior change before proceeding. Go back to the Config Editor window and hover your mouse to the left of the `compile.pipeline_loops` command to display and click the **More Actions** command. Click the **Reset to Default** command. Alternatively, you can just remove the command from the `hls_config.cfg` file. 

Another possible optimization is to tell the tool that a function or loop should occur before processing another sample. The `Pipeline` pragma or directive defines an acceptable level of performance, and can eliminate II violations from the reports because the latency would then match your specification. The overall latency of an application could indicate that perhaps `II=4` is acceptable for some loops.

This configuration might be an acceptable response to II violations when the loops are not in the critical path of the design, or they represent a small problem relative to some larger problems that must be resolved. In other words, not all violations need to be resolved, and in some cases, not all violations can be resolved. They are simply artifacts of performance.

7.  In the Settings Form of the Config File Editor go to the bottom of the categories on the left side and go to **Design Directives > Pipeline**. This lets you manage the [hls.syn.directive.pipeline](https://docs.amd.com/access/sources/dita/topic?Doc_Version=2024.1%20English&url=ug1399-vitis-hls&resourceid=tjy1677219494422.html) in your design. 

8.  Click **Add Item** under Pipeline. This opens the `dct.cpp` source code for the HLS component, and also opens the Directive editor as decribed in [Adding Pragmas and Directives](https://docs.amd.com/access/sources/dita/topic?Doc_Version=2024.1%20English&url=ug1399-vitis-hls&resourceid=gip1583519972576.html). 

9.  In the **HLS Directive** view, select the `dct_2d` function, and click '+' to add a directive or pragma. This opens the **Add Directive** dialog box as shown in the following figure. 

![Add Directive](./images/add_directive.png)

10. Select the **PIPELINE** directive and specify an II of 4. You can select either **Source File** to add it as a pragma directly to your code, or **Config File** to add it as a directive to the HLS configuration file. Click **OK** to add the pragma or directive to your design. 

11.  In the Flow Navigator click the **Run** command under C Synthesis to rerun with the new directive.
12.  Examine the updated reports to see if there is any performance improvement.

## Assign Dual-Port RAMs with BIND_STORAGE

**ACTION:** Back out the prior change before proceeding. Depending on whether you added **PIPELINE** as a pragma to the source code, or a directive to the config file, you can remove it from the appropriate location. You can also select the pragma in the **HLS Directive** view and delete it from there. 

In some designs, a Guidance message `Unable to schedule load operation...` indicates a load/load (or read/read conflict) issue with memory transactions. In these cases rather than accepting the latency, you could try to optimize the implementation to achieve the best performance (II=1).

The specific problem of reading or writing to memory can possibly be addressed by increasing the available memory ports to read from, or to write to. One approach is to use the BIND_STORAGE pragma or directive to specify the type of device resource to use in implementing the storage. BIND_STORAGE defines a specific device resource for use in implementing a storage structure associated with a variable in the RTL. For more information, refer to [BIND_STORAGE](https://docs.amd.com/access/sources/dita/topic?Doc_Version=2024.1%20English&url=ug1399-vitis-hls&resourceid=imo1677218583234.html). 

Looking at the **Storage Report** section of the Synthesis report you can see that the tool has implemented the `buf_2d_in` variable with a `ram_s2p`. This allows reading on one port while writing on the other. But the RAM_2P allows simultaneous reading on both ports, or reading on one and writing on the other. This might offer some performance improvement. 

13.  In the Config Editor select **Add Item** for **Bind Storage** to open the Directive Editor. In the HLS Directive view navigate to the `dct` function, select the buf_2d_in variable, and select **Add Directive**. 
14.  In the Add Directive dialog box select the **BIND_STORAGE** pragma, specify **type** of `ram_2p` and **impl** of `bram`, and click **OK** to add the directive or pragma to your design.

**TIP:** You can also edit the `hls_config.cfg` file and add the following line directly: `syn.directive.bind_storage=dct buf_2d_in impl=bram type=ram_2p`

15. Run C Synthesis again and examine the results. 
 
## Using Array_Partition

**ACTION:** Back out the prior change before proceeding. 

Another approach to solve memory port conflicts is to use the `Array_Partition` directive to reconfigure the structure of an array. `Array_Partition` lets you partition an array into smaller arrays or into individual registers instead of one large array. This effectively increases the amount of read and write ports for the storage and potentially improves the throughput of the design. However, `Array_Partition` also requires more memory instances or registers, and so increases area and resource consumption. For more information, refer to [ARRAY_PARTITION](https://docs.amd.com/access/sources/dita/topic?Doc_Version=2024.1%20English&url=ug1399-vitis-hls&resourceid=muz1677218527052.html).

16.  In the Config Editor select **Add Item** for **Array Partition** to open the Directive Editor. In the HLS Directive view navigate to the `dct` function, select the `buf_2d_out` variable, and select **Add Directive**. 
17.  In the Add Directive dialog box select the **ARRAY_PARTITION** pragma, specify **type** of `cyclic` and **factor** of `8`, and click **OK** to add the directive or pragma to your design.
18.  Repeat this process for the `col_inbuf` variable of the `dct_2d` function,  with the same settings for the **ARRAY_PARTITION** pragma. 

The reason for choosing a cyclic partition with a factor of 8 has to do with the code structures involved. The loop is processing an 8x8 matrix, which requires taking eight passes through the outer loop, and eight passes through the inner loop. By selecting a cyclic array partition, with a factor of 8, you are creating eight separate arrays that each get read once per iteration. This eliminates any contention for accessing the memory during the pipeline of the loop. 

Run synthesis and re-examine the Synthesis Summary report to see the results of this latest change. 

## Next Step

**ACTION:** Back out the prior change before proceeding. 

Now that you have examined different optimizations for different issues in the design, there is one more optimization to explore: [the DATAFLOW optimization](./unified-dataflow_design.md).
</br>
<hr/>
<p align="center" class="sphinxhide"><b><a href="/README.md">Return to Main Page</a> — <a href="./README.md">Return to Start of Tutorial</a></b></p>


<p class="sphinxhide" align="center"><sub>Copyright © 2020–2023 Advanced Micro Devices, Inc</sub></p>

<p class="sphinxhide" align="center"><sup><a href="https://www.amd.com/en/corporate/copyright">Terms and Conditions</a></sup></p>

