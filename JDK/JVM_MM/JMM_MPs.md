# Java内存模型-多处理器

原文：http://ifeve.com/jmm-cookbook-mps/



​        本文总结了在多处理器(MPs)中常用的的处理器列表，处理器相关的信息都可以从链接指向的文档中得到（一些网站需要通过注册才能得到相应的手册）。当然，这不是一个完全详细的列表，但已经包括了我所知道的在当前或者将来Java实现中所使用的多核处理器。下面所述的关于处理器的列表和内容也不一定权威。我只是总结一下我所阅读过的文档，但是这些文档也有可能是被我误解了，一些参考手册也没有把Java内存模型(JMM)相关的内容阐述清楚，所以请协助我把本文变得更准确。

​        一些很好地讲述了跟内存屏障(barriers)相关的硬件信息和机器(machines)相关的特性的资料并没有在本文中列出来，如《Hans Boehm的原子操作库([Hans Boehm’s atomic_ops library](http://www.hpl.hp.com/research/linux/atomic_ops/))》,  《Linux内核源码([Linux Kernel Source](http://kernel.org/))》, 和 《Linux可扩展性研究计划([Linux Scalability Effort](http://lse.sourceforge.net/))》。Linux内核中所需的内存屏障与这里讨论的是非常一致的，它已被移植到大多数处理器中。不同处理器所支持的潜在内存模型的相关描述，可以查阅[Sarita Adve et al, Recent Advances in Memory Consistency Models for Hardware Shared-Memory Systems](http://rsim.cs.uiuc.edu/~sadve/)和 [Sarita Adve and Kourosh Gharachorloo, Shared Memory Consistency Models: A Tutorial](http://rsim.cs.uiuc.edu/~sadve/).



**sparc-TSO**

Ultrasparc 1, 2, 3 (sparcv9)都支持全存储顺序模式（TSO:Total Store Orde)，Ultra3s只支持全存储顺序模式（TSO:Total Store Orde)。(Ultra1/2的RMO(Relax Memory Order)模式由于不再使用可以被忽略了)相关内容可进一步查看 [UltraSPARC III Cu User’s Manual](http://www.sun.com/processors/manuals/index.html) 和 [The SPARC Architecture Manual, Version 9 ](http://www.sparc.com/resource.htm)。



**x86 (和 x64)**

英特尔486+，AMD以及其他的处理器。在2005到2009年有很多规范出现，但当前的规范都几乎跟TSO一致，主要的区别在于支持不同的缓存模式，和极端情况下的处理(如不对齐的访问和特殊形式的指令)。可进一步查看[The IA-32 Intel Architecture Software Developers Manuals: System Programming Guide](http://www.intel.com/products/processor/manuals/) 和 [AMD Architecture Programmer’s Manual Programming](http://www.amd.com/us-en/Processors/DevelopWithAMD/0,,30_2252_875_7044,00.html)。



**ia64**

安腾处理器。可进一步查看 [Intel Itanium Architecture Software Developer’s Manual, Volume 2: System Architecture](http://developer.intel.com/design/itanium/manuals/iiasdmanual.htm)。



**ppc (POWER)**

尽管所有的版本都有相同的基本内存模型，但是一些内存屏障指令的名字和定义会随着时间变化而变化。下表中所列的是从Power4开始的版本；可以查阅架构手册获得更多细节。查看 [MPC603e RISC Microprocessor Users Manual](http://www.motorola.com/PowerPC/), [MPC7410/MPC7400 RISC Microprocessor Users Manual ](http://www.motorola.com/PowerPC/), [Book II of PowerPC Architecture Book](http://www-106.ibm.com/developerworks/eserver/articles/archguide.html), [PowerPC Microprocessor Family: Software reference manual](http://www-3.ibm.com/chips/techlib/techlib.nsf/techdocs/F6153E213FDD912E87256D49006C6541), [Book E- Enhanced PowerPC Architecture](http://www-3.ibm.com/chips/techlib/techlib.nsf/techdocs/852569B20050FF778525699600682CC7), [EREF: A Reference for Motorola Book E and the e500 Core](http://e-www.motorola.com/webapp/sps/site/overview.jsp?nodeId=03M943030450467M0ys3k3KQ)。关于内存屏障的讨论请查看[IBM article on power4 barriers](http://www-1.ibm.com/servers/esdd/articles/power4_mem.html), 和 [IBM article on powerpc barriers](http://www-106.ibm.com/developerworks/eserver/articles/powerpc.html).



**arm**

arm版本7以上。请查看 [ARM processor specifications](http://infocenter.arm.com/help/index.jsp) **alpha** 21264x和其他所以版本。请查看[Alpha Architecture Handbook](http://www.alphalinux.org/docs/alphaahb.html)



**pa-risc**
HP pa-risc实现。请查看[pa-risc 2.0 Architecture](http://h21007.www2.hp.com/dspp/tech/tech_TechDocumentDetailPage_IDX/1,1701,2533,00.html)手册。

**下面是这些处理器所支持的屏障和原子操作：**

<table border="1" cellspacing="1" cellpadding="2">
<tbody>
<tr>
<td><b>Processor</b></td>
<td><b>LoadStore</b></td>
<td><b>LoadLoad</b></td>
<td><b>StoreStore</b></td>
<td><b>StoreLoad</b></td>
<td><b>Data<br>
dependency<br>
orders loads?</b></td>
<td><b>Atomic<br>
Conditional</b></td>
<td><b>Other<br>
Atomics</b></td>
<td><b>Atomics<br>
provide<br>
barrier?</b></td>
</tr>
<tr>
<td>sparc-TSO</td>
<td>不执行操作</td>
<td>不执行操作</td>
<td>不执行操作</td>
<td>membar<br>
(StoreLoad)</td>
<td>是</td>
<td>CAS:<br>
casa</td>
<td>swap,<br>
ldstub</td>
<td>全部</td>
</tr>
<tr>
<td>x86</td>
<td>不执行操作</td>
<td>不执行操作</td>
<td>不执行操作</td>
<td>mfence or<br>
cpuid or<br>
locked insn</td>
<td>是</td>
<td>CAS:<br>
cmpxchg</td>
<td>xchg,<br>
locked insn</td>
<td>全部</td>
</tr>
<tr>
<td>ia64</td>
<td><em>combine<br>
</em>和<br>
st.rel 或者<br>
ld.acq</td>
<td>ld.acq</td>
<td>st.rel</td>
<td>mf</td>
<td>是</td>
<td>CAS:<br>
cmpxchg</td>
<td>xchg,<br>
fetchadd</td>
<td>部分 +<br>
acq/rel</td>
</tr>
<tr>
<td>arm</td>
<td>dmb<br>
(见下文)</td>
<td>dmb<br>
(见下文)</td>
<td>dmb-st</td>
<td>dmb</td>
<td>只能间接</td>
<td>LL/SC:<br>
ldrex/strex</td>
<td></td>
<td>仅针对部分</td>
</tr>
<tr>
<td>ppc</td>
<td>lwsync<br>
(见下文)</td>
<td>lwsync<br>
(见下文)</td>
<td>lwsync</td>
<td>hwsync</td>
<td>只能间接</td>
<td>LL/SC:<br>
ldarx/stwcx</td>
<td></td>
<td>仅针对部分</td>
</tr>
<tr>
<td>alpha</td>
<td>mb</td>
<td>mb</td>
<td>wmb</td>
<td>mb</td>
<td>否</td>
<td>LL/SC:<br>
ldx_l/stx_c</td>
<td></td>
<td>仅针对部分</td>
</tr>
<tr>
<td>pa-risc</td>
<td>不执行操作</td>
<td>不执行操作</td>
<td>不执行操作</td>
<td>不执行操作</td>
<td>是</td>
<td><em>build<br>
from<br>
</em>ldcw</td>
<td>ldcw</td>
<td><em>无</em></td>
</tr>
</tbody>
</table>



**说明：**

- 尽管上面一些单元格中所列的屏障指令比实际需要的特性更强，但可能是最廉价的方式获得所需要的效果。
- 上面所列的屏障指令主要是为正常的程序内存的使用而设计的，IO和系统任务就没有必要用特别形式/模式的缓存和内存。举例来说，在x86 SPO中，StoreStore屏障指令(“sfence”)需要写合并(WC)缓存模式，其目的是用在系统级的块传输等地方。操作系统为程序和数据使用写回(Writeback)模式，这就不需要StoreStore屏障了。
- 在x86中，任何lock前缀的指令都可以用作一个StoreLoad屏障。（在Linux内核中使用的形式是无操作的lock指令，如addl $0,0(%%esp)。)。除非必须需要使用像CAS这样lock前缀的指令，否则使用支持SSE2扩展版本（如奔腾4及其后续版本）的mfence指令似乎是一个更好的方案。cpuid指令也是可以用的，但是比较慢。
- 在ia64平台上，LoadStore，LoadLoad和StoreStore屏障被合并成特殊形式的load和store指令–它们不再是一些单独的指令。ld.acq就是(load;LoadLoad+LoadStore)和st.rel就是(LoadStore+StoreStore;store)。这两个都不提供StoreLoad屏障–因此你需要一个单独的mf屏障指令。
- 在ARM和ppc平台中，就有可能通过non-fence-based指令序列取代load fences。这些序列和以及他们应用的案例在[Cambridge Relaxed Memory Concurrency Group](http://www.cl.cam.ac.uk/~pes20/ppc-supplemental/)著作中都有描述。
- sparc membar指令不但支持所有的4种屏障模式，而且还支持组合模式。但是StoreLoad模式需要在TSO中。在一些UltraSparcs中，不管任何模式下membar指令总是能让StoreLoad生效。
- 在与这些流指令有关的情况中，X86处理器支持”streaming SIMD” SSE2扩展只需要LoadLoad ‘lfence’
- 虽然PA-RISC规范并不强制规定，但所有HP PA-RISC的实现都是顺序一致，因此没有内存屏障指令。
- 唯一的在pa-risc上的原始原子操作(atomic primitive)是ldcw, 一种test-and-set的形式，通过它你可以使用一些技术建立原子条件更新(atomic conditional updates)，这些技术在 [HP white paper on spinlocks](http://h21007.www2.hp.com/hpux-devtools/CXX/hpux-devtools.0106/0014.html)中可以找到.
- 在不同的字段宽度(field width，包括4个字节和8个字节版本）里，CAS和LL/SC在不同的处理器上会使用多种形式。
- 在sparc和x86处理器中，CAS有隐式的前后全StoreLoad屏障。sparcv9架构手册描述了CAS不需要post-StoreLoad屏障特性，但是芯片手册表明它确实在ultrasparcs中存在这个特性。
- 只有在内存区域进行加载和存储(loaded/stored)时，ppc和alpha, LL/SC才会有隐式的屏障，但它不再有更通用的屏障特性。
- 在内存区域中进行加载或存储时, ia64 cmpxchg指令也会有隐式的屏障，但还会额外加上可选的.acq（post-LoadLoad+LoadStore）或者.rel（pre-StoreStore+LoadStore)修改指令。cmpxchg.acq形式可用于MonitorEnter，cmpxchg.rel可用于MonitorExit。在上述的情况中，exits和enters在没有被确定匹配的情况下，就需要ExitEnter(StoreLoad)屏障。
- Sparc,x86和ia64平台支持unconditional-exchange (swap, xchg). Sparc ldstub是一个one-byte test-and-set。 ia64 fetchadd返回前一个值并把它加上去。在x86平台，一些指令（如add-to-memory）能够使用lock前缀的指令执行原子操作。