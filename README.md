# LLVM-PGO
调研：使用llvm编译linux内核，并使用pgo机制进行优化
## Profile-guided optimization for the kernel(with gcc)  
### 参考资料：  
[Linux Weekly News](https://lwn.net/Articles/830300/)  
[Project PPT](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)  
[Using gcov with the Linux kernel](https://www.kernel.org/doc/html/v5.8/dev-tools/gcov.html)
### 概要
* Build and install kernel with instrumentation：  
  在编译时要使能GCOV选项：
  ```bash
  CONFIG_DEBUG_FS=y
  CONFIG_GCOV_KERNEL=y
  CONFIG_GCOV_PROFILE_ALL=y
  ```
* Run scenario：  
  运行实际工作负载，可在/sys/kernel/debug/gcov中发现*.gcda; *.gcno等文件，记录着运行时的性能数据  
* optimizing the Kernel:  
  重新编译，增加"-fprofile-use"选项，如：
  ```bash
  KCFLAGS="-fprofile-use=/home/user81/gcov-test/generic-instr/gcov -Wno-coverage-mismatch -Wno-error=coverage-mismatch"
  ```
  上述方案主要利用了gcc的gcov工具收集任务运行数据做pgo优化，而clang/llvm的gcov工具（llvm-cov）与其并不兼容。因此采用llvm编译内核并进行pgo优化与上述方案有所差别，但是整体的思路基本相同。  
  另外，在上述项目中还提到了LTO优化，暂未展开调研。

## Building Linux Kernel with Clang/LLVM  
* [Profiling with Instrumentation](https://clang.llvm.org/docs/UsersManual.html#profiling-with-instrumentation)  
  1）在编译时增加"-fcs-profile-generate"或"-fprofile-generate"选项  
  2）运行代码得到*.profraw文件
  3）使用llvm-profdata工具合并得到*.profdata文件

* optimizing the Kernel:  
  似乎clang/llvm已经原生的支持了对于内核的pgo优化：[pgo: add clang's Profile Guided Optimization infrastructure](https://lwn.net/ml/linux-kernel/20210407211704.367039-1-morbo@google.com/)    
  即将实际工况下运行得到的*.profdata文件作为编译的输入即可：  
  ```bash
  $ make LLVM=1 KCFLAGS=-fprofile-use=vmlinux.profdata ...
  ```
  同时上述Linux weekly news的文章中讲到：此种对于内核的profiling是clang原生的   
  ```bash
  # Note that this method of profiling the kernel is clang-native, unlike the clang support in kernel/gcov.
  ```  
  但是，目前此种方案似乎只在x86架构上进行了验证：  
  ```bash
  #This initial submission is restricted to x86, as that's the platform we know works. 
  #This restriction can be lifted once other platforms have been verified to work with PGO.
  ```
## Problems  
* [Profiling with Instrumentation](https://clang.llvm.org/docs/UsersManual.html#profiling-with-instrumentation) 中给出的收集.profdata的示例是基于一个简单的源代码文件说明的，至于编译linux内核并运行得到proraw文件并整合成编译器可用的profdata文件的过程需要再实践中探索  
* 目前调研到的方案似乎暂时只在x86平台上进行了验证  
* 可能还存在一些工具链构建方面的问题有待于在实践中进一步学习
## To do 
* 目前主要是补充了一些相关的基础知识，有了整体的认识和理解，下一步要在实践中探索,计划在x86架构下尝试pgo三部曲:  
"instrumentation"->"Running scenario"->"optimizing the Kernel"  

  
  
  



