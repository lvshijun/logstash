# 输入插件\(Input\) {#%E8%BE%93%E5%85%A5%E6%8F%92%E4%BB%B6-input}

在 "Hello World" 示例中，我们已经见到并介绍了 logstash 的运行流程和配置的基础语法。从这章开始，我们就要逐一介绍 logstash 流程中比较常用的一些插件，并在介绍中针对其主要适用的场景，推荐的配置，作一些说明。

限于篇幅，接下来内容中，配置示例不一定能贴完整。请记住一个原则：Logstash 配置一定要有一个 input 和一个 output。在演示过程中，如果没有写明 input，默认就会使用 "hello world" 里我们已经演示过的_input/stdin_，同理，没有写明的 output 就是_output/stdout_。

以上请读者自明。

