### phase机制基础

------

Phase在UVM中可以理解为是仿真片段或者仿真阶段，非常符合phase单词本意。

UVM方法学将仿真过程划分成了多个仿真阶段，然后还给这些仿真阶段都起了名字、预定义了功能、确定了先后关系和协作方式等等。这些概念的定义和协调仿真阶段的方式，被统称为Phase机制（Phase mechanism）。

Phase机制在基于UVM的仿真中尤其重要，它是整个仿真周期中的同步机制。每个环境组件（uvm_component）在仿真中都会执行一组预定义的仿真阶段，而且默认情况下，只有在所有组件都执行完了当前仿真阶段的任务，才可以进入到下一个仿真阶段

##### base

------

所有的仿真阶段被分成了三组：Build time phases, Run time phases, Clean-up phases。其中build time和clean-up time两组phases不消耗仿真时间，实现方法（method）用的函数（function）；Run time phases需要消耗仿真时间，是用任务（task）来实现的。

每组Phases又分别包含了多个具体的Phase，每一个Phase都有自己预定义的功能，在下图中被详细列出。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YJOEW8ib9oGPicgcTE8MaELicxCibcoFpAn6BazZcaMmTtLrpMlhD8vpAZ0SunGaHuz5Z4gRobwJ0LBLMibrD73UC1g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

标红的phase是在实际应用中被关注的比较多的phase，而其他phase经常可能因为各种具体的原因被忽略掉。尽管如此，UVM的仿真中这些phase都是存在的（指common_domain，默认会例化的域），只是没有具体的事情干的时候，phase就执行直接过去了。



##### why phase？

------

Phase机制是UVM定义的一套规范，约定了各个仿真阶段的功能，统一了环境组件的仿真步调。但严格来讲，用户是可以不完全遵守这些功能定义的，可以按照自己的需求去调整，更何况不同单位不同项目针对不同验证对象构造出来的验证环境也可能千差万别。

那为什么还要建议遵守Phase机制呢？

一方面Phase机制约定的功能具有普适性，且这套协作方式足够好用，从上面两个小节也可以看得出来。这套机制在时间尺度上将功能干净的隔离开，整齐有序，适用于绝大多数的功能验证。

另一方面，Phase机制的意义更在于对UVM通用性的保障。举个栗子，基于UVM的验证本身是面向对象的编程实践，这就意味着在仿真过程中可以动态地去实例化对象，如果不按照同一套时间和顺序规范去例化对象、连接组件、执行仿真和报告仿真结果，基于UVM的验证环境、组件或者验证IP的通用性都会大打折扣。



##### under phase(源代码)

------

UVM Phase机制中提出了domain、schedule、node、implementation等一些概念。如果不去深究phase机制的实现或者自定义phase以及phase的执行顺序，这些概念对于用户来说是可以忽略的。而当你需要去自定义新的phase，并且重新规划phase的执行顺序，那么这些概念还是要了解的。本文也将带着这个目的去梳理。

[SystemVerilog | UVM | 深入Phase机制，看懂Phase机制实现原理 (qq.com)](https://mp.weixin.qq.com/s/NfUbH7_7673LmA2CQC6mNw)