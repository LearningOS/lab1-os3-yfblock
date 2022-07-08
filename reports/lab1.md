# LAB1报告

## 实现功能

1. 为`TaskControlBlock`添加`task_syscall_times`和`task_start_time`记录任务执行`syscall`调用次数和任务第一次被调度的`time`。

2. 为`TaskManager`添加`get_current_task_info`和`update_current_task_syscall`，用于获取`TaskInfo`和更新`syscall`次数

3. 完善`sys_task_info`系统调用，在系统调用中将`*mut TaskInfo`转为`&mut TaskInfo`并传递给`get_current_task_info`进行数据填充

4. 在`TaskManager`的`run_first_task`中为`task0`更新第一次被调用的时间，并且在`run_next_task`中判断任务`task_start_time`是否被更新过（任务是否被调用过），如果没被更新过则更新为`get_time()`获取到的时间

5. 在`timer`中添加函数`time_to_ms`将系统的`time`转换为`ms`

## 简答作业

#### 1.出错行为

```rust
[ERROR] [kernel] PageFault in application, core dumped.
[ERROR] [kernel] IllegalInstruction in application, core dumped.
[ERROR] [kernel] IllegalInstruction in application, core dumped.
```

出错行为分别为`PageFault`由访问没有权限的内存页触发，`IllegalInstruction`在U态中使用`sret`指令和操作`sstatus`指令触发

#### 2.trap.S

答1：在进入`__restore`时，`a0`中的值为`trap_handler`函数的返回值，即`&mut TrapContext`（TrapContext的地址）。

答2：

```riscv
ld t0, 32*8(sp)
ld t1, 33*8(sp)
ld t2, 2*8(sp)
csrw sstatus, t0
csrw sepc, t1
csrw sscratch, t2
```

上述代码特殊处理了`sstatus`、`sepc`、`sscratch`三个寄存器。分别保存了调用之前CPU所处的状态（U/S），发生调用的地址也是恢复后返回地址、作为暂存用户态程序栈的地址。

答3：`x2`也是`sp`寄存器，如果恢复`x2`后面对于`sp`的读取将出现异常。`x4`寄存器在保存时没有保存，即不使用也不读取。

答4：L63`csrrw sp, sscratch, sp`之后`sp`保存用户栈地址，`sscratch`保存内核栈地址。

答5：状态切换发生在`sret`，`ecall`指令从`U-Mode`进入`S-Mode`，`sret`返回`U-Mode`，

答6：L13`csrrw sp, sscratch, sp`之后`sp`保存内核栈地址，`sscratch`保存用户栈地址。

答7：`ecall`指令

# 