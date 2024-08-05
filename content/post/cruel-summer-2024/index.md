---
title: "Cruel Summer in 2024"
subtitle: "To record what I did in the summer of 2024"

summary: "To record what I did in the summer of 2024"

date: 2024-06-30T16:40:00+08:00

lastmod: 2024-08-04T11:11:00+08:00

draft: false


# show_date: true
# links:
#   - icon_pack: fab
#     icon: twitter
#     name: Follow
#     url: 'https://twitter.com/Twitter'
#   - icon_pack: fab
#     icon: medium
#     name: Originally published on Medium
#     url: 'https://medium.com'
featured: false
authors:
  - admin

categories:
- daily-record
---


### 8.5

+ Learn how to use snort

```bash
sudo ./snort -vd -i <interface_name> "port 9999"
curl http://localhost --connect-timeout 2000
mkdir dir
```

+ Learn more about [bash](https://liujiacai.net/blog/2024/04/05/robust-shell-scripting). Useful material: [Shell Style Guide from Google](https://google.github.io/styleguide/shellguide.html)

```bash
if [[ "foo" == "f*" ]] ; then echo hello; else  echo test; fi
# blank is important
if [[ "foo" == f* ]] ; then echo hello ; else echo test; fi
for (( i=0;i<5;i++)) ; do echo hello ; done
# Here document
 cat <<EOF                               
`ls`
EOF
# (())
if [ $((1+1)) -eq 2 ] ; then echo yes ; fi

# Why export?
x=1
export y=2
# case statement
case $p in
world)
echo hello 
;;
esac

declare -a arr=(a b c)
declare -a map=([hello]=world)
echo "${arr[@]}"
echo "${arr}"

echo $$
echo $@
echo $?

. example.sh
```

+ Learn about the implication of batch size

### 8.4

+ Read something about diffusion model

> A note about promiscuous vs. non-promiscuous sniffing: The two techniques are very different in style. In standard, non-promiscuous sniffing, a host is sniffing only traffic that is directly related to it. Only traffic to, from, or routed through the host will be picked up by the sniffer. Promiscuous mode, on the other hand, sniffs all traffic on the wire. In a non-switched environment, this could be all network traffic. The obvious advantage to this is that it provides more packets for sniffing, which may or may not be helpful depending on the reason you are sniffing the network. However, there are regressions. Promiscuous mode sniffing is detectable; a host can test with strong reliability to determine if another host is doing promiscuous sniffing. Second, it only works in a non-switched environment (such as a hub, or a switch that is being ARP flooded). Third, on high traffic networks, the host can become quite taxed for system resources.

+ `perf` usage notes

```bash
taskset --cpu-list 2 ls # 2 is not a mask
taskset -p pid
```

+ Learn about isolate certain cpu cores. [link](https://unix.stackexchange.com/questions/326579/how-to-ensure-exclusive-cpu-availability-for-a-running-process/326585#326585)
+ Read [A Brief History of JavaScript Frameworks](https://primalskill.blog/a-brief-history-of-javascript-frameworks)

### 8.3

+ Read something about reinforcement learning and GAN
+ Read [Pytorch: An imperative style, high-performance deep learning library](https://arxiv.org/abs/1912.01703)

### 8.1-8.2

+ Back to Xi'an
+ Read Matrix Calculus

### 7.31

+ Read [PROGRAMMING WITH PCAP](https://www.tcpdump.org/pcap.html)

### 7.30

+ Read [计算机系统会议论文是如何评审的](https://zhaoxiahust.github.io/blog/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%B3%BB%E7%BB%9F%E4%BC%9A%E8%AE%AE%E8%AE%BA%E6%96%87%E6%98%AF%E5%A6%82%E4%BD%95%E8%AF%84%E5%AE%A1%E7%9A%84.pdf) written by Haibo Chen

### 7.29

+ Read [Mapping High Level Constructs to LLVM IR](https://mapping-high-level-constructs-to-llvm-ir.readthedocs.io/en/latest/basic-constructs/local-variables.html)

### 7.26-7.27

+ Read [Kubernetes Tutorials](https://kubernetes.io/docs/tutorials/)

### 7.25

+ Watch video about [Flash Attention](https://www.bilibili.com/video/BV1UT421k7rA)

### 7.24

+ Watch [Kaiming He's Talk @ CUHK](https://www.bilibili.com/video/BV1Zz421B7Vf)

### 7.23

+ Read [The Ultimate Guide to Kubernetes Networking and Multi-Tenant Gateways](https://blog.moelove.info/the-ultimate-guide-to-kubernetes-networking-and-multi-tenant-gateways)
+ Watch video about [Automatic Mixed Precision](https://www.bilibili.com/video/BV1qJ4m1w7ur)
+ Read [Vector Database](https://guangzhengli.com/blog/zh/vector-database/)

### 7.22

+ Read [Hardware Accelerator (CUDA, DSA, etc.)](https://openmlsys.github.io/chapter_accelerator/index.html)
+ Watch video about [KV Cache, Page Attention](https://www.bilibili.com/video/BV1kx4y1x7bu)
+ Watch video about [Distributed Training(DeepSpeed)](https://www.bilibili.com/video/BV1mm42137X8/)
+ Watch video about [Reinforcement Learning: PPO](https://www.bilibili.com/video/BV1iz421h7gb)
+ Watch video about [GPU (Tensor Core)](https://www.bilibili.com/video/BV1rH4y1c7Zs) SIMT, Warp(32 threads), Streaming Multiprocessor, CUDA core(more general) VS. Tensor Core(add and multiply in a period), Special Function Unit, Compute Intensity
+ Watch [Kaiming He's Talk on Learning Deep Representation](https://www.bilibili.com/video/BV1sW421c7SK) (really insightful talk！)

### 7.21

+ Read [Distributed Training](https://openmlsys.github.io/chapter_distributed_training/index.html) (Pipeline Parallelisim，Model Parallelism，Parameter Server，Collective Communication)
+ Read [Programming Interface](https://openmlsys.github.io/chapter_programming_interface/index.html) (Python & C++)
+ Read [Computation Graph](https://openmlsys.github.io/chapter_computational_graph/index.html) in ML framework

> 一类是学习率不受梯度影响的随机梯度下降（Stochastic Gradient Descent）及SGD的一些改进方法，如带有Momentum的SGD；另一类是自适应学习率如AdaGrad、RMSProp、Adam等。

### 7.20

+ Read [Python Numpy Tutorial](https://cs231n.github.io/python-numpy-tutorial/)
+ Read [Gradient Check](https://cs231n.github.io/neural-networks-3/)
+ Read [Image Similarity Search and Cosine Similarity](https://www.trybackprop.com/blog/linalg101/part_3_build_image_search)
+ Find [GNN Course](http://web.stanford.edu/class/cs224w/) and [Deep Reinforcement Learning](https://rail.eecs.berkeley.edu/deeprlcourse/)

### 7.19

+ Read [Visualize CNN](https://cs231n.github.io/understanding-cnn/)
+ Read [SGD](https://cs231n.github.io/optimization-1/)
+ Read [Simple Neural Network](https://cs231n.github.io/neural-networks-case-study/)

### 7.18

+ Read [Learning Research](https://github.com/pengsida/learning_research)

### 7.17

+ Read [Generative Model(Autoregressive Model & VAE & GAN)](https://cs231n.stanford.edu/slides/2024/lecture_13.pdf)
+ Read [RNN](https://cs231n.stanford.edu/slides/2024/lecture_7.pdf)

### 7.16

+ Read [Transfer Learning, Batch Normalization,Case Study on AlexNet, VGG, GoogleNet and ResNet](https://cs231n.stanford.edu/slides/2024/lecture_6_part_1.pdf)
+ Read [Why Momentum Really Works](https://distill.pub/2017/momentum/)
+ Read [How to apply for CVE](https://blog.csdn.net/m0_43397796/article/details/130985872)
+ Read [Training in Practice](https://cs231n.stanford.edu/slides/2024/lecture_6_part_2.pdf)
+ Read [Object Detection and Semantic Segmantation](https://cs231n.stanford.edu/slides/2024/lecture_9.pdf)
+ Why 'zero centered ' is better?

### 7.11-7.15

+ Interview...
+ Review Linear Algebra & Probability & Advanced Mathematic

### 7.8

+ Read [Classification](https://cs231n.github.io/classification/)
+ Read [Linear Classification](https://cs231n.github.io/linear-classify/)

### 7.6-7.7

+ Finish [dynamic linker lab](https://github.com/ruitianzhong/dynamic-linker)

### 7.5

+ Finish [static linker lab](https://github.com/ruitianzhong/linker)

### 7.4

+ Interview...

### 7.2

+ Read [Convolutional Neural Networks (CNNs / ConvNets)](https://cs231n.github.io/convolutional-networks/)
  
### 7.1

+ Read [Attention? Attention!](https://lilianweng.github.io/posts/2018-06-24-attention/)
+ Read [Derivatives, Backpropagation, and Vectorization](https://cs231n.stanford.edu/handouts/derivatives.pdf)

### 6.30

+ Read [Understanding LSTM Networks](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)
+ Read [Back Propagation](http://cs231n.github.io/optimization-2)

### 6.29

+ Read [The Illustrated Transformer](http://jalammar.github.io/illustrated-transformer/)
+ Read [The Illustrated BERT](https://jalammar.github.io/illustrated-bert/)
+ Read [Visual and Interactive Guide to the Basics of Neural Networks](https://jalammar.github.io/visual-interactive-guide-basics-neural-networks/)
