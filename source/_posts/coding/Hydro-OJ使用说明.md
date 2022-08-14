---
title: Hydro-OJ 使用说明
date: 2022-5-21 10:00:23
tags:
  - oj
  - docker
categories:
  - coding
abbrlink: hydro-hepler
---

本`OJ`的具体搭建方式：[Docker搭建Hydro-OJ系统 · xiabee-瞎哔哔](https://blog.xiabee.cn/posts/hydro-docker/)

以下内容为用户文档

## 超级管理员

* 超级管理员是`OJ`系统的最高权限，能够直接控制`OJ`的全部内容，不建议设置多个

### 使用要求
·
* 拥有服务器`ssh`权限，且能够直接控制服务器容器

### 创建方法

- 在`OJ`右上角注册一个账号，此时账号的`UID`为2·
- 回到服务器的终端，使用 `docker oj-backend exec -it hydrooj cli user setSuperAdmin 2` 将 UID 为 2 的用户设置为超级管理员。
- 使用 `docker oj-backend exec -it pm2 restart hydrooj` 重启以使管理员更改立刻生效。
- 前往 “题库” 面板，查看创建的示例题目是否正常工作。

### 设置角色

* `管理域->管理用户->添加用户`

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2fzplb344j319x0o4n6r.jpg)

* 输入用户`UID`或者用户名，设置其角色

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2fzqs7hakj30ht09w3z7.jpg)

#### 

* 目前已设置好的角色为老师和学生：
  
  ![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2fzrz7smtj30h009d3za.jpg)

### 自动加域

* 设置加域链接：注册后点击链接直接加入

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2g0rvv90kj318w0nwn5m.jpg)

### 维护方法

* 具体内容详见官方文档：[维护 | Hydro](https://hydro.js.org/docs/system/maintain/)

## Teacher

* 老师常用模块为创建/删除题目、创建/删除训练、创建/删除作业
* 未自动加域的老师，可以联系超级管理员，手动加入域中

### 创建题目

* `题库->创建题目`
  
  ![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2fzza7irtj318p0dkq7u.jpg)

* 填写相关信息和描述，并翻到最下方，点击`创建`
  
  ![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2g02cjb3aj30ue0f3q8k.jpg)

    若未填写分组，则默认分配至当前组

* 添加测试数据和附件
  
  ![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2g045qiwjj30vh0kgdjq.jpg)

### 测试数据文件格式（手动）

* 对于一般的题目，只需提供 `.in` 和 `.out/.ans` 文件

* 文件名中必须含有数字，如`1.in`和`1.out`。形如 `sample.in` 的文件是不会被自动识别

* 例如手动上传如下文件：
  
  ```bash
  .
  ├── a1.in
  ├── a1.out
  ├── a2.in
  ├── a2.out
  ├── a3.in
  └── a3.out
  ```
  
  测试数据将被自动识别，并使用 1S 256MB 的限制。

### 测试数据文件格式（自动）

* 上传`config.yaml`（推荐通过`评测设置`在线编辑题目配置）

* `config.yaml`文件格式如下：
  
  ```yaml
  # 题目类型，可以为 default(比对输出，可以含spj), objective(客观题), interactive(交互题)
  type: default
  
  # 全局时空限制（此处的限制优先级低于测试点的限制）
  time: 1s
  memory: 128m
  
  # 输入输出文件名（例：使用 foo.in 和 foo.out），若使用标准 IO 删除此配置项即可
  filename: foo
  
  # 此部分设置当题目类型为 default 时生效
  # 比较器类型，支持的值有 default（直接比对，忽略行末空格和文件末换行）, ccr, cena, hustoj, lemon, qduoj, syzoj, testlib(比较常用)
  checker_type: default
  # 比较器文件（当比较器类型不为 default 时填写）
  # 文件路径（位于压缩包中的路径）
  # 将通过扩展名识别语言，与编译命令处一致。在默认配置下，C++ 扩展名应为 .cc 而非 .cpp
  checker: chk.cc
  
  # 此部分设置当题目类型为interactive时生效
  # 交互器路径（位于压缩包中的路径）
  interactor: interactor.cc
  
  # Extra files 额外文件
  # These files will be copied to the working directory 这些文件将被复制到工作目录。
  # 提示：您无需手动上传 testlib.h。
  user_extra_files:
    - extra_input.txt
  judge_extra_files:
    - extra_file.txt
  
  # Test Cases 测试数据列表
  # If neither CASES or SUBTASKS are set(or config.yaml doesn't exist), judge will try to locate them automaticly.
  # 如果 CASES 和 SUBTASKS 都没有设置或 config.yaml 不存在， 系统会自动尝试识别数据点。
  # We support these names for auto mode: 自动识别支持以下命名方式：
  # 1. [name(optional)][number].(in/out/ans)         RegExp: /^([a-zA-Z]*)([0-9]+).in$/
  #   examples: 
  #     - c1.in / c1.out
  #     - 1.in / 1.out
  #     - c1.in / c1.ans
  # 2. input[number].txt / output[number].txt        RegExp: /^(input)([0-9]+).txt$/
  #   - example: input1.txt / input2.txt
  #
  # The CASES option has higher priority than the SUBTASKS option!
  # 在有 CASES 设置项时，不会读取 SUBTASKS 设置项！
  #
  # The CASES option has been deprecated in the new version, please use the more personalized SUBTASKS!
  # CASES 已于新版本中被废弃，请使用个性化程度更高的SUBTASKS！
  # score: 50     # 单个测试点分数
  # time: 1s      # 时间限制
  # memory: 256m  # 内存限制
  # cases:
  #   - input: abc.in
  #     output: def.out
  #   - input: ghi.in
  #     output: jkl.out
  # 或使用Subtask项：
  subtasks:
    - score: 30
      type: min # 可选 min/max/sum，分别表示取所有测试点最小值、所有测试点最大值、所有测试点之和
      time: 1s
      memory: 64m
      cases:
        - time: 0.5s
          memory: 32m # 可对单个测试点单独设置时间限制和内存限制
          input: a.in
          output: a.out
        - input: b.in
          output: b.out
    - score: 70
      time: 0.5s
      memory: 32m
      if: [0] # 可选，传入数组，表示仅在subtask0通过时此subtask才计分
      cases:
        - input: c.in
          output: c.out
        - input: d.in
          output: d.out
  
  # 提交语言限制
  # 列举出所有本题允许使用的语言对应的代码（需要和评测机 lang.yaml 内的语言代码相同）
  # 使用语言ID而非名称！对于有子类的选项，请详细至子分类！
  langs:
    - c
    - cc
    - cc.cc11o2
  ```

* 可以在[官方题库](https://hydro.ac/d/system_test/p)中下载数据进行参考

### 创建作业

* `作业->创建作业`

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2g0fi36mtj316q0oqagl.jpg)

* 设置相关字段：
  
  ![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h2g0gkxy87j30pd0mxq6u.jpg)

### 创建比赛

* 同上

## Students

* 创建账号

* 点击自动加域链接，加入域中

* 可以认领作业、参加比赛、发布题解等

## Refference

> [用户文档 | Hydro](https://hydro.js.org/docs/user/)
> 
> [Docker搭建Hydro-OJ系统 · xiabee-瞎哔哔](https://blog.xiabee.cn/posts/hydro-docker/)
