---
title: graphbuilder
toc: true
date: 2023-05-04 22:26:59
tags:
- 
categories:
- 
typora-root-url: ..\img
---

# 整体



![grc-类组织关系.drawio](/../../../../../工作图库/grc-类组织关系.drawio.svg)



<!-- more -->

* 在build 阶段初始化了那些信息

1. 初始化一个全局的单例执行器，并且可以设置图的name
2. 添加add_vertex的到对应的GraphVertexBuilder，并且为其添加依赖和输出（逻辑概念？）
3. builder 开始finish所有GraphVertexBuilder的vertexs（也就是算子）
   1. 清空**_producers_for_data_index**和**_data_index_for_name**(数据 _target对应索引的)
   2. 遍历GraphVertexBuilder的_vertexes
   3. 每个GraphVertexBuilder执行finish
      1. 每个GraphDependencyBuilder开始build【每个依赖对应一个builder】
         1. 保存 _target(依赖数据）和index的关系到  _data_index_for_name
         2. 并保存 每个依赖的_target_index 和  _condition_index

      2. 每个GraphEmitBuilder开始build【每个输出对应一个builder]
         1. 保存 _target（输出数据）和index的关系到  _data_index_for_name
         2. 保存 _target_index和GraphVertexBuilder的关系到 _producers_for_data_index

      3. 创建 对应的 _processor_creator(实际的算子对象创建函数)【::std::function<::std::unique_ptr<GraphProcessor>()>】
      4. processor->config【默认直接转发】

   4. 遍历_producers_for_data_index对算子set_allow_trivial
   5. 如果一个输出数据被两个GraphVertexBuilder产出，则认为是**非trivial**的

4. Builder 开始调用build()构图
   1. set_executor(*_executor);来设置图执行器*
   2. set_page_allocator(*_page_allocator)设置内存分配方式_
   3. initialize_data(_data_index_for_name(hash)); _
      1. 初始化数据到_data_for_name【flat_hash_map<<::std::string, GraphData*> 】
      2. _初始化数据到 _data(std::vector<GraphData>)，**每一个输出或者输入都是GraphData**

   4. initialize_vertexes(_vertexes.size());用GraphVertexBuilder的数组初始化std::vector<GraphVertex>的数组
   5. 遍历图执行GraphVertexBuilder的数组，然后每个GraphVertexBuilder进行build
      1. GraphVertex保存graph和GraphVertexBuilder的信息
      2. 遍历该graph 的 GraphDependencyBuilder并build
         1.  从_data中根据GraphDependencyBuilder保存的target_index获取GraphData
         2. 给该GraphData设置GraphDependency(add_successor())
         3. 设置依赖(dependency)的算子；dependency.source(vertex);
         4.  设置依赖(dependency)对应的数据；dependency.target(target);
         5. 如果condition不为空，增加condition关联的依赖dependency；并增加该依赖dependency的条件

      3. 遍历该graph的GraphEmitBuilder并build(**GraphVertex保存了依赖和输出**)
         1. 通过GraphEmitBuilder的_target_index获取输出的GraphData
         2. 将GraphData的算子节点保存到该raphData的_producers(std::vector<vertex>）里面；data.producer(vertex);
         3. 返回出该输出节点emit = &GraphData

      4. vertex.setup()
         1. 用宏ANYFLOW_INTERFACE声明变量
         2. 调用GraphProcessor::setup()

      5. 检查GraphData是否build成功
         1. one_data.error_code()
         2. one_data.check_safe_mutable();依赖不超过1个，是安全的.???

5. 在run之前要设置最开始的输入数据，如：*(a1->emit<int>()) = 100;
   1. 析构commiter的时候会调用release
   2. 设置当前GraphData的_closure为SEALED_CLOSURE
   3. 对该commiter的依赖数据判断是否ready
      1. 如果数据是其他算子的依赖，将该算子添加到runnable_vertexes里面

   4. 遍历runnable_vertexes
   5.  vertex->invoke(runnable_vertexes);

6. 在run阶段创建了那些信息

* 用单例执行器创建对应的闭包，等待的依赖的vertex也增加depend_vertex_add
* 遍历每个输出数据
  * 将闭包绑定到输出数据中
    * 闭包的_waiting_data_num加1
    * 闭包的_waiting_data保存当前要run的数据
    * 
  * 递归激活每个输出数据
    * 创建一个vector来模拟栈
    * 输出数据自己触发递归的激活，（当前的输出数据先激活，然后递归遍历当前数据依赖的数据）
      * 判断当前数据是否激活(默认没有激活)并将状态设置为_active=true
      * 判断数据是否为ready,用特殊地址(0xFFFL)和有意义的地址进行比较作为条件
        * 并将数据添加到activating（正在？）的模拟栈中
    * 从模拟栈中获取一个数据，并且该数据将递归激活
      * 遍历当前数据的算子producers,让producer【相当于算子激活】激活(只能激活一次)
        * 保存闭包到每一个算子（vertex)
        * 节点算子无依赖数据（size==0);将算子直接记入可算子执行列表(runnable_vertexes)
        * 保存该算子的依赖数（输入）量到_waiting_num（表示是等待的个数）
        * 激活具体的算子（GraphProcessor::on_activate）
        * 循环遍历激活该算子的GraphDependency(输入数据），并记录激活时已经就绪的依赖的数目（目的是用来判断当前数据就绪？？）
          * 根据当前依赖【逻辑上的概念】（也就是输入数据）的waiting_num来返回转台（？）
            * case 0
              * 判断条件是否成立(_condition)
              * 判断依赖状态（_depend_state.exchange(1, ::std::memory_order_relaxed) != 2）
              * 判断GraphDependency自己对应的数据（GraphData _target)是否ready[_ready = _target->ready();]
            * case 1
              * 判断没有条件
                * 依赖自己对应的数据（GraphData _target)激活（trigger） _target->trigger(activating_data);
                  * 判断并设置激活
                  * 判断是否ready
              * 判断有条件
                * _condition->trigger(activating_data);
            * case 2：
              * condition未就绪，激活condition
        * 该算子的等待数为没有就绪的  _waiting_num(用总的依赖减去已经就绪的)
* 串行执行可运行模拟队列里面的算子（主意是否区分trivial算子）
  * 遍历每个依赖是否是必要的，是否是ready
  * _builder->graph().executor().run(this, GraphVertexClosure(*_closure, *this));
    * 内置图执行器执行run
      * 图节点执行run
        * 检查闭包是否处于完成状态；closure.finished()(大概率false)
        * 运行具体的processor
        * 数据在release 的时候也会触发算子节点的运行
        * 定义具体的成员，也就是运行__anyflow_prepare_interface
* 启动闭包（context->fire()）
  * 依赖GraphData减1【depend_data_sub】
  * 依赖的vertex减1【depend_vertex_sub】

并行：





# 问题

* GraphDependency是很重要的概念，和GraphData怎么理解呢？

GraphData绑定在一个GraphDependency上？？

auto& target = graph.data()[_target_index];

  target.add_successor(dependency);

  dependency.source(vertex);

  dependency.target(target);

* GraphData::release()非常重要的思路，析构的时候释放数据
* x
* 
