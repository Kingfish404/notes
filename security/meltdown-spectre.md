# Meltdown-Spectre

By: Jin Yu
Last Modified: 2021-10-16

## Overview

Meltdown and Spectre exploit critical vulnerabilities in modern processors.

These hardware vulnerabilities allow programs to steal data which is currently processed on the computer. While programs are typically not permitted to read data from other programs, a malicious program can exploit Meltdown and Spectre to get hold of secrets stored in the memory of other running programs. This might include your passwords stored in a password manager or browser, your personal photos, emails, instant messages and even business-critical documents.

Meltdown and Spectre work on personal computers, mobile devices, and in the cloud. Depending on the cloud provider's infrastructure, it might be possible to steal data from other customers.

There are also a lot of attack variants has been publish.

## Meltdown

Meltdown(rogue data cache load) melts security boundaries which are normally enforced by the hardware.
This attack breaks the most fundamental isolation between user applications and the operating system. This attack allows a program to access the memory, and thus also the secrets, of other programs and the operating system.

## Spectre

Spectre(bounds check bypass) base on speculative execution.
This attack breaks the isolation between different applications. It allows an attacker to trick error-free programs, which follow best practices, into leaking their secrets. In fact, the safety checks of said best practices actually increase the attack surface and may make applications more susceptible to Spectre.

<div style="break-after: page;"></div>

## Side-channel

Any information gained from the implementation of a computer system that unexpected.

## Attack Example

System memory is just like below.

```text
|    ---- UserMem ----     |      ---- SysMem ----     |
| from Mem[0] to Mem[100]  | from Mem[101] to Mem[200] |
|====|--arr--|========A====|---------target------------|
                      | length equal x |
```

使用如下代码进行`SysMem`中信息的获取，其中arr位于`UserMem`中，x的大小为A起始内存值到Sys Mem 中待攻击区域的内存长度，即 `&A[x] <==> &target`

```javascript
// Spectre attack
if(x<arr.length>){
    // 由于分支预测的存在，下方代码会被提前执行，然后回退
    y = arr[A[x]]
}

// Meltdown attack
for(i from arr.start to arr.end){
    getStartTime()
    y = arr[i]
    getEndTime()
    // Side-channel attack
    // 已经被访问过的会在缓存中被hit
    // 所以访问速度会很快
    MarkMinCost()
}
```

通过上述代码，arr中访问最快的地址的下标，即是`SysMem`中target数据的值

上述即为一个Meltdown attack的典型流程

<div style="break-after: page;"></div>

## Defence

对于`Meltdown-Spectre Attack`的防范目前主要关注与其攻击的核心点

- 防止被恶意训练及隔离敏感数据 - 敏感区域不缓存 from Meltdown defence
- 防止微体系结构上的变化 - 分支预测防范 from Spectre defence
- 防止侧信道观测微体系结构变化 - 防止信息获取 side-channel defence

可以通过软件方式来防范`Meltdown-Spectre Attack`，但是会导致系统运行速度降低，要从根本上解决这个问题，可能需要硬件配合[From Google's Blog]

## REF
- [Meltdown and spectre](https://meltdownattack.com/)
- [ZombieLoad attack | Github](https://github.com/IAIK/ZombieLoad)
- [maltdown | Github](https://github.com/IAIK/meltdown)
- [spectre-attack | Github](https://github.com/Eugnis/spectre-attack)