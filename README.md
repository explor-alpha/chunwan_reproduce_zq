# 春晚舞蹈机器人复现（环境配置）    ——郑群 上海大学

> 应 **Datawhale** 邀分享 **50 系显卡 & WSL 本地部署方案**：[my 方案](./完整方案_春晚舞蹈机器人复刻.md)            

> **原教程（云端部署 & 40系显卡）**参考：[Datawhale-every-embodied](https://github.com/datawhalechina/every-embodied/tree/main/07-%E6%9C%BA%E5%99%A8%E4%BA%BA%E6%93%8D%E4%BD%9C%E3%80%81%E8%BF%90%E5%8A%A8%E6%8E%A7%E5%88%B6/Locomotion) 的[07-机器人操作、运动控制/Locomotion/01春晚舞蹈机器人复刻](https://github.com/datawhalechina/every-embodied/blob/main/07-%E6%9C%BA%E5%99%A8%E4%BA%BA%E6%93%8D%E4%BD%9C%E3%80%81%E8%BF%90%E5%8A%A8%E6%8E%A7%E5%88%B6/Locomotion/01%E6%98%A5%E6%99%9A%E8%88%9E%E8%B9%88%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%A4%8D%E5%88%BB.md)   

## 功能实现

- 输入：文本动作描述或已有视频  
- 输出：机器人动作（`robot_motion*.pkl`）与可视化（Web/viser/MuJoCo）  
- 支持多人轨迹（all tracks）与多机器人 MuJoCo 录制  

```text
Prompt / Video -> PromptHMR -> SMPL-X -> GMR -> Robot Motion
```

> **1. PromptHMR（Prompt-based Human Mesh Recovery）**  
> - **功能**：视觉感知/机器人的眼睛。它的任务是看视频。当你给它一段人跳舞的视频时，它负责从 2D 的画面里，精准地识别出人的三维姿态、骨骼角度和体型（即 SMPL-X 模型）。     
> - **为什么需要它**：机器人看不懂像素视频，它需要知道人的手肘弯了多少度、膝盖抬了多高。PromptHMR 就是负责把“视频”变成“3D 数据”的工具。   
> - **来源**：第三方的学术论文代码库，故克隆到 third_party 目录。  

> **2. GMR（Generative Motion Retargeting）**  
> - **功能**：运动控制/大脑/机器人的“运动翻译官”。它的任务是把人的动作“翻译”给机器人。  
> - 人有 206 块骨头，机器人（比如 Unitree G1）只有几十个电机。人的腿长，机器人的腿短。GMR 负责解决这些差异，计算出“为了模仿这个人的动作，机器人的每个电机应该转动多少度”，同时保证机器人不摔倒。         
> - **为什么需要它**：直接把人的动作数据塞给机器人，机器人会抽搐或者散架。GMR 保证了动作的平滑和物理可行性。  
> - **来源**：第三方的学术论文代码库，故克隆到 third_party 目录。  

## 🎬 Show results

<div align="center">
<img src="show_result/show_result.gif" width="90%">
<p><b>Web UI</b></p>
</div>


## 文件结构

```
chunwan_reproduce_zq
 ┣ 完整方案_春晚舞蹈机器人复刻.md         # Key：My 方案
 ┃
 ┣ notes                              # 原理
 ┃ ┣ detail_cpp扩展编译.md
 ┃ ┣ detail_环境变量配置(conda级).md
 ┃ ┣ 原理_NVIDIA-50系显卡.pdf
 ┃ ┗ 原理_cpp扩展编译 & CUDA.pdf
 ┃
 ┣ show_result
 ┃ ┣ show_result.gif
 ┃ ┗ show_result.mp4
 ┃
 ┗ README.md
```

### 实际落地难点  
1. **动作映射精度**：从视频提取的 SMPL-X 姿态到机器人关节的映射（Retargeting）存在物理限制。  
2. **环境感知缺失**：目前主要是开环动作复刻，缺乏对实际地面摩擦、碰撞和障碍物的实时反馈。  
3. **动力学约束**：视频中的人类动作可能超出机器人的电机扭矩或平衡极限。例如在马年春晚武术机器人表演中，高难度的腾空与受力对伺服响应要求极高。  

### 展望：IsaacSim 仿真 & RL  
- **优势**：极高的物理仿真精度，支持复杂的碰撞检测、摩擦力模型和传感器仿真。  
- **价值**：本项目主要完成“动作映射”，IsaacSim 则负责“动作稳健性训练”。  