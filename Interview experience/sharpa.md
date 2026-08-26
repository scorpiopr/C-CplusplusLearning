# 岗位
无线软件开发工程师
## 岗位职责
负责研发机器人无线通信方案，优化通信协议，解决多机器人协同中的干扰与延迟问题；
实现低延迟、高可靠性无线通信方案，涉及 SDR、Wi-Fi、LoRa、蓝牙、4G/5G 等一种或多种技术；
跟踪通信网络领域的先进技术，支持机器人的通信技术研究。
## 职位要求
2. 掌握c/c++编程，有RTOS或Linux上开发经验；
3. 精通至少一种无线技术（SDR/Wi-Fi /LoRa/蓝牙/5G），了解物理层与MAC层原理；
# 要点
## dB、dBm和dBW
| 单位 | 类型与定义 | 核心计算公式 | 0 值的含义 (参考基准) | 核心用途与应用场景 |
|---|---|---|---|---|
| dB (分贝) | 相对值 （表示两个功率的比值或放大倍数） | $\text{dB} = 10 \lg\left(\frac{P_{\text{out}}}{P_{\text{in}}}\right)$ (电压/电流比值时为 $20\lg$) | $0\text{ dB} = 1$ 倍 （即 $P_{\text{out}} = P_{\text{in}}$） | 1. 描述放大器增益（如信噪比提升） 2. 描述线路损耗 / 衰减（如信号衰减 -3dB） |
| dBm (分贝毫瓦) | 绝对值 （表示功率的绝对大小） | $\text{dBm} = 10 \lg\left(\frac{P}{1\text{ mW}}\right)$ | $0\text{ dBm} = 1\text{ mW}$ | 1. 弱电 / 无线通信设备发射与接收功率 2. Wi-Fi/4G/5G 信号强度（RSSI，如 -70dBm） |
| dBW (分贝瓦) | 绝对值 （表示功率的绝对大小） | $\text{dBW} = 10 \lg\left(\frac{P}{1\text{ W}}\right)$ | $0\text{ dBW} = 1\text{ W}$ | 1. 强电 / 高功率系统（如雷达、卫星通信） 2. 宏基站发射功率（如 43dBm = 20W = 13dBW） |

## 三大黄金法则（3-10-30 法则）
### 1. 功率互换口算口诀

* $\pm 3\text{ dB}$ 规则：功率翻倍或减半（$+3\text{ dB} = \times 2$，$-3\text{ dB} = \div 2$）。
* $\pm 10\text{ dB}$ 规则：功率变为 10 倍或 $1/10$（$+10\text{ dB} = \times 10$，$-10\text{ dB} = \div 10$）。

### 2. dBm 与 dBW 的空间转换

* $\pm 30$ 规则：$\text{dBW} = \text{dBm} - 30$（因为 $1\text{ W} = 1000\text{ mW}$，而 $10\lg(1000) = 30$）。

### 3. 混合运算对错判定机制（面试极易挖坑）

* 绝对值 $\pm$ 相对值 $=$ 绝对值：$\text{dBm} + \text{dB} = \text{dBm}$（正确，表示发射功率经过放大器放大了）。
* 绝对值 $-$ 绝对值 $=$ 相对值：$\text{dBm} - \text{dBm} = \text{dB}$（正确，表示两个功率点之间的差值/损耗）。
* 绝对值 $+$ 绝对值 $=$ 毫无数学意义：$\text{dBm} + \text{dBm}$（错误，编译或物理逻辑直接报错）。
## 信道映射：
  逻辑信道（CCCH/DCCH/DTCH）、传输信道（BCH/DL-SCH/UL-SCH）到物理信道的映射关系。

# 问题
## 编程题：找到无序数组第k大的数
## 机械手2路信号输入，包括控制手自由度的矩阵数据，实现手指的运动，需要设计哪些接口实现哪些具体功能

### 一、 外部输入接口（2路信号接收）
这部分接口负责从外界（如5G无线模组、上层视觉AI或主控系统）接收2路输入信号，并将其高效地存入 EulerOS 的内存中。
1. 信号 1：控制手自由度的矩阵数据（高带宽、流式数据）

* 功能：接收表示手部关节角度、位置或速度的齐次变换矩阵或关节配置矩阵（例如：$4 \times 4$ 空间位姿矩阵，或 $M \times N$ 的关节目标角度矩阵）。
* 接口设计（API 示例）：
  * int init_matrix_receiver_socket(const char* ip, int port); —— 初始化 UDP/TCP 监听套接字（推荐 UDP，因为机械手控制对时延极度敏感，允许少量丢包，但不能容忍 TCP 重传带来的阻塞）。
   * int parse_hand_matrix(const uint8_t* raw_buf, size_t len, float* out_matrix); —— 高效解析原始二进制流为浮点数矩阵，需在用户态进行严格的 数据校验（如 CRC32 或校验和），防止无线传输中的错码导致机械手误动伤人。

2. 信号 2：状态/触发控制信号（低频、高可靠）

* 功能：接收离散的控制指令（如：开始抓取、紧急停止、切换模式、握力调节、复位等）。
* 接口设计（API 示例）：
* int register_command_callback(uint16_t cmd_id, void (*callback)(void* arg)); —— 回调函数注册接口。当收到特定控制指令（如基站/远端下发的 EMERGENCY_STOP）时，立即触发中断级或高优先级线程的回调。

### 二、 内部核心处理模块接口（Linux 用户态核心层）
数据进入 EulerOS 后，需要通过高效的进程间通信 (IPC) 和核心算法，将矩阵数据转化为具体的“手指运动指令”。
1. 共享内存与无锁队列接口（数据传输）
为了实现无延迟的数据传递，避免传统的 Linux read/write 带来内核态与用户态的内存拷贝，建议设计共享内存 (Shared Memory) + 环形缓冲区 (Ring Buffer) 接口。

* 接口设计：
  * shm_ringbuf_t* create_shm_queue(key_t key, size_t buffer_size); —— 创建基于共享内存的无锁环形队列。
   * int enqueue_matrix_data(shm_ringbuf_t* queue, const float* matrix); —— 写端（网络接收线程）将矩阵压入队列。
   * int dequeue_matrix_data(shm_ringbuf_t* queue, float* matrix); —— 读端（运动学计算线程）从队列读取矩阵。

2. 运动学逆解与轨迹规划接口（功能核心）

* 功能：将输入的空间自由度矩阵（笛卡尔空间），转化为机械手各个手指关节的旋转角度（关节空间）。
* 接口设计：
  * int calculate_inverse_kinematics(const float* target_matrix, float* out_joint_angles); —— 逆运动学算法接口。输入 $4 \times 4$ 矩阵，输出如 5 根手指共 15 个自由度的目标角度。
   * int trajectory_interpolation(const float* current_angles, const float* target_angles, float step, float* next_step_angles); —— 轨迹插补接口。防止手指运动幅度过大导致电机过载，将其分解为平滑的微步。

### 三、 底层硬件/驱动接口（控制手指运动）
将计算出的关节角度，转化为硬件能识别的电信号或总线协议数据包。在机器人中，通常走 EtherCAT、CAN-Open 或 UART (RS485) 总线。
1. 总线通信与同步接口

* 功能：将所有手指的运动指令打包，通过总线在同一个控制周期（如 1ms 或 2ms）内同步发送给各个手指的伺服电机。
* 接口设计：
  * int init_bus_driver(const char* device_node); —— 初始化驱动节点（如 /dev/ttyUSB0 或 /dev/ethercat0）。
   * int pack_motor_commands(const float* joint_angles, uint8_t* tx_packet); —— 按照电机协议（如 Modbus 或自定义 CAN 帧）封包。
   * int sync_write_motors(const uint8_t* tx_packet, size_t len); —— 同步写入接口。利用 Linux 的 ioctl 或通过驱动直接操作硬件，确保所有手指的电机同时收到指令。

2. 安全保护与状态反馈接口

* 功能：实时读取手指电机的传感器数据（角度反馈、当前电流/触觉限制），防止夹毁物体或限位碰撞。
* 接口设计：
  * int read_motor_feedback(int motor_id, float* current_angle, float* current_torque); —— 读取指定手指电机的角度和扭矩。
   * int set_safety_torque_limit(int motor_id, float max_torque); —— 设置手指的最大安全握力。一旦超过该阈值，触发保护，手指停止加力。

------------------------------
  
