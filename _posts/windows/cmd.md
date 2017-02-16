title: dos命令使用记录
date: 2017-02-16 16:34:00
toc: true
tags: [windows, dos]
categories: windows
keywords: [windows, dos, command, 命令]
desciption: 
---

输出目录树（tree）
---------

命令：`tree /f > dir_pic_tree.txt`，输出的结果在 cmd 的当前目录下，如果不带 `> dir_pic_tree.txt`，则直接打印到 cmd 界面。

例如输出 Pictures 目录的目录树：

```
C:\Users\R04220\Pictures>tree /f
文件夹 PATH 列表
卷序列号为 0000009A B477:AC59
C:.
│  dir_pic_tree.txt
│  paper.小组完成情况.png
│  paper.延期数据.png
│  paper.总数据.png
│  paper.总数据2.png
│  save-spotlight.ps1
│  
├─Camera Roll
├─Saved Pictures
└─Spotlight
    ├─CopyAssets
    ├─Horizontal
    │      07d13665d2a42b6a9b1308d76ea9d43ede14964ac843f5c646e11d8537323c75.jpg
    │      2eabdd4e869a9ba575a2c7ab85026357cfc21f27aba7e5aefb06ef0611a66a8c.jpg
    │      3260bc4d05d2d4b9426e114713aece847a7807b26fe85ead14ccf1d4b5653f5f.jpg
    │      389abcdbb34cd0352e7e9c216859a1373a9a0beeb245b6f866c4459604e5fc83.jpg
    │      4422e3ac278425ebdf387990995c10159dd787dfc3914bca5062208b6cd4af8b.jpg
    │      4699a04f5e6c7ecf0c86891a1bb98933dac79228b21633d8a34f153e3c1d3180.jpg
    │      498f917a9bbc3ce7bf061ef6bb2f832aaa37fc813e40c94d4f5f9f23c3567a5b.jpg
    │      51ed125c041ab491a2494aad469656480d30e2e07c019634329de28613b68aac.jpg
    │      52e51ea8bfb1886d16a84c83f2bee187b0b129cd3260339cf54f596882fabedc.jpg
    │      56127afeae00317c8ee4efebbed57156e9ec6c0a2be7c6f5e164428a37805abf.jpg
    │      611ae451bfcb5a31fed1deb8766815bd8b522e67f737ce4ee38f40d51d1a659d.jpg
    │      7586db4f8911367a593e56e98f92fccd1d78bd34c8b30052b71b94a859eac1c0.jpg
    │      8659dbdf640884f349f2a010048ed000ccc600c90897427f3e6d27972e15bfda.jpg
    │      888733fb259bbdc503f23baf71603d81d01b068dca3f8e5c62248eee1092c17e.jpg
    │      8e0c39f07085cd7862d7472c12204e8ecaa82d62e580c68664faec960babcddd.jpg
    │      90bfcc1d948170fd252992ceaf68c59596b86ef26cf4d448f9e8000348a152ae.jpg
    │      b2c87ee5f28245a081a6d6181a31840a84cc6c983d2c4f1d524ab5e879e41edd.jpg
    │      cacc63578dda6099e9d6465b8247f118e60de9eb4aa81d2cecd5107628028a0e.jpg
    │      d6f886a44d933673742740f2df0fd045a75d2457b80ce52deac53d6d50da0ae2.jpg
    │      daab366938f9d2fc1f6783121248a75089dec6bf5adfbb6d8447710793efd5a6.jpg
    │      db6c55aaea5ca497ad8fed0219e9742551594d612f46a291daf351bc106ef759.jpg
    │      f6eb0171205b66d1f01c947af1144093544c7eab4716c59f1bf872703af26409.jpg
    │      
    └─Vertical
            0a965b06725e3eb45058e1e1c699d529f026aa170fa43696659801d56807ee8c.jpg
            149be707844d99bf1d13b6f6ee0071952861e6ec9d475e5d588139f6477e7b3e.jpg
            1fbbd6779aa1407e13c2837941b61650f06f35a282b29f478d07f4bd77ca926e.jpg
            212909107619586a7a3b6abe628a9ef9a1bead207b3d0b88f6ac6b6ffcb2070f.jpg
            25befdccd1b9f6b7ed91bcbfca73656998f6e563fea160dca01be9cc1a4cd747.jpg
            30a0cdf04b7d029e34f5f75ac25493c66ceac9734c2d6accf9008638a5475953.jpg
            3a93ca26063b7552eaff43c70a142aaabd53e99940fa0a25a248bdfa9d96796f.jpg
            44fe2b0dca73fadd35d818cb83356e1431f3b0f9729ee9742f5f227e0948de72.jpg
            4c6d76fbf970c420376041ec4ad5a97601add0d56838859acaec60bae3227fc1.jpg
            4e8ea94abf449377fcc3e45c4033e4e2ef0f91a43c66873d03e351cb640f0264.jpg
            5b8006c1069151d12ae83c175af746b84aa11ae46e4834c2f0535b4dc2aa55c5.jpg
            60ecfce0af8371c7f3fe030f3c1d040605aaa18e2b44de65c7e09c82aee765d3.jpg
            68cbd1d229910ccd15329f9989c13a09fcf869fd09b86d5195bb1cd398b7f80c.jpg
            69ed52e624a7bf2e2b95b91339982186d3cd8b85b8a4308b0004c95286cb98d6.jpg
            70a0927503e8adc8841749d379177c4f30a03477c305b4f0e595015e308cde0b.jpg
            774f1a1a02fd2a4c6612b8f4b4654613a6bbeca5f4bfc2bbf438a0b822321754.jpg
            79ae9d23c38685fac276058333d066a728d75a9bc7324e66d395707fa4913140.jpg
            815e87ed8cd0d24bdf30132500a23a80f936f808ff5b653807f8de8d7df05a74.jpg
            89d40db523257c5397d54e6719d2ac201e3f13b060aecce38663d42ccaba3996.jpg
            938eae85fb0af3deda7bd9386c7f0e28d87b64f87736c47f9ddde52306de15e1.jpg
            a931c3312424a1d9878db81e041f1a6d5704e990d7dd34386ca18e6dc107015e.jpg
            d4335387d80d96e64933e71a284d0ea466b2e4a95afe1e54a9244efd20caff8e.jpg
            e37f97eb49a3459589815c0543648a737904e4d91397da43c3618bcaa1a4f75a.jpg
```

