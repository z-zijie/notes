什么是MLIR
---
MLIR 即 Multi-Level Intermediate Representation, 多级中间表示

MLIR是一个可重复使用和可扩展的编译器基础设施, aims to 解决软件栈碎片化的问题，降低DSL编译器的构建成本，并aid in connecting existing compilers together. 粗浅的可理解为是一个可以快速开发Domain Specific Compiler的平台，特别是用于FPGA\NPU等异构芯片上的编译器。

更多的细节可以在[MLIR: A Compiler Infrastructure for the End of Moore's Law](https://arxiv.org/pdf/2002.11054.pdf)这篇论文中，值得一提的是，这篇文章的一作Chris Lattner是LLVM项目的主要发起人与作者之一，同时也是Swift语言的创始人，Clang编译器的作者。可以说MLIR是Chris继LLVM, Clang, Swift之后又一个重要项目。其未来的发展值得期待。

后期有空将出一片该文章的阅读笔记。


## IR
先说说基本概念，什么是IR(Intermediate Representation)

通俗的来讲，C/C++/Python/...等我们熟知的语言是为人类编写和阅读而设计的，我们可以轻松的读懂代码想要表达的意思，但机器不行。所以需要Compiler将这些Frontend表达转换为硬件的Instruction. 显然这个过程不是一蹴而就的，这里就需要IR作为载体，将Frontend 逐步lower到Backend表达上.

在深度学习领域，IR作为一种中间的数据结构，可以把模型在不同的框架间转换。例如`TVM Relay`, `TorchScript`, `ONNX`.
`ONNX`最初由Microsoft和Facebook提出，旨在定义一套标准化的算子格式。各个深度学习框架都可以使用ONNX提供的算子定义表达自己的模型。这样就解决了同样的模型在各个框架中间互相转换的问题。但是由于各个框架的设计理念存在差异，同样的算子在TensorFlow和PyTorch中的功能定义有可能有所不同，即便是同一个框架，伴随着迭代与更新也会有算子定义的变化或废弃。ONNX无法支持所有版本的算子实现。
从另一个角度来讲，ONNX作为`基础设施`是不能频繁的迭代更新的。

`TorchScript`被提出是用来解决PyTorch动态图模式执行效率太低的问题。用户可以将编写好的动态图代码转化为于语言无关的TorchScript表达，这样使得C++测的API也可以方便的调用。

`TVM` 提出的`Relay IR`也是DSL的一种，它是函数式的可微静态语言，非常有利于表达AI领域的ML模型。这套IR不仅解决了PyTorch依赖Python control flow的问题，同时解决了Transformer等一众模型中需要的dynamic shape特性。总的来说`Relay IR`具备一个编程语言的基本特性，表达能力和完备程度也是极高的。但是它依赖于TVM，一切能力也需要建立在TVM的基础上。

总的来说，IR就是不同范畴之间的交流渠道，既可以是不同深度学习框架之间的，也可以是不同抽象层级之间的。


## MLIR
MLIR解决了哪些问题，或者说MLIR希望解决哪些问题？

当下的现状是有能力公司或组织，为了自身的需求和业务场景，设计实现了一套行之有效的IR及相关软件栈。众多不同又类似的IR表达不仅造成了大量的重复开发，又导致了用户被某一框架生态绑定。

其实各个IR都为了解决一样的问题，如图优化。大家为了优化一个DAG，需要基于IR实现多个图优化Pass，而对于不同的IR可能功能是完全一致的，造成了大量的重复开发。
其次，当一个新的更优秀的IR的出现后，需要基于它自身的设计去迁移一系列的优化Pass，甚至说，为了在某一IR上开发新功能需要开发者完全的了解熟悉这套IR的设计理念，学习代价是巨大的。
另一方面，之前说过从高层抽象到底层抽象并不是一蹴而就的。以XLA的HLO为例，在经过图层面的优化后，还需要针对融合后的算子做绑定硬件的针对性优化。如基于硬件的`Share Memory`/`Unify Buffer`大小，需要对循环做一些切片；对于NPU上常见的SIMD指令，需要了解熟悉硬件行为，ISA特性等细节，而这些和GPU又是截然不同的。再往后昨晚指令映射后，还需要了解基于LLVM IR的Codegen调用toolchain，等等。总而言之，从IR到ISA的跨度是非常大的。

MLIR的提出，就是为了解决这些问题。MLIR使用Dialect，让各个框架的IR可以使用同一套IR重写，并且拥有相互转换的能力。同时Dialect还可以抽象出不同阶段不同层级的抽象，用来解决高级IR到ISA跨度太大的问题。另一方面，MLIR属于LLVM的一部分，共享了LLVM的生态。

## Ongoing
当前业界的主要深度学习框架都在积极的接入MLIR，如
 - [torch-mlir](https://github.com/llvm/torch-mlir)
 - [mlir-hlo](https://github.com/tensorflow/mlir-hlo)
