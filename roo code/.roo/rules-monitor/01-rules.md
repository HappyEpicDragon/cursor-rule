# Experiment Monitor 规则

你负责从启动到报告的完整闭环，**绝不能只启动实验就返回**。

## 全闭环工作流（必须完整执行）

### 步骤 1：写监控脚本

根据主 Agent 给你的启动命令，创建自包含的 monitor 脚本：

```bash
#!/bin/bash
# scripts/monitor_<实验名>.sh

# === 启动 ===
<启动命令> > <LOG_PATH> 2>&1 &
PID=$!
echo "[monitor] PID=$PID started at $(date)"

# === 等待完成 ===
STATUS_FILE=<STATUS_PATH>
while kill -0 $PID 2>/dev/null; do
  sleep <间隔>
done
wait $PID
EXITCODE=$?

# === 写状态报告 ===
{
  echo "## 实验状态报告"
  echo "- 时间: $(date)"
  echo "- Exit code: $EXITCODE"
  if [ $EXITCODE -eq 0 ]; then
    echo "- 状态: ✅ 成功"
  else
    echo "- 状态: ❌ 异常"
    echo '```'
    tail -20 <LOG_PATH>
    echo '```'
  fi
  echo "- 关键指标:"
  grep -E '<指标关键词>' <LOG_PATH> | tail -10
} > $STATUS_FILE
echo "[monitor] 报告已写入 $STATUS_FILE"
```

### 步骤 2：tmux 启动

```bash
chmod +x scripts/monitor_<实验名>.sh
tmux new-session -d -s <session_name> "bash scripts/monitor_<实验名>.sh"
```

启动后等 10-20 秒验证进程正常：
```bash
tmux ls && nvidia-smi --query-gpu=index,utilization.gpu,memory.used --format=csv,noheader
```

### 步骤 3：轮询等待

- 短实验（<10 分钟）：`sleep <预计秒数> && cat <STATUS_PATH>`
- 长实验（>10 分钟）：循环检查
```bash
while [ ! -f <STATUS_PATH> ]; do sleep 60; done
cat <STATUS_PATH>
```

### 步骤 4：返回结果

读取 STATUS_FILE 内容，通过 `attempt_completion` 返回状态摘要。只返回摘要，不贴完整日志。

## 多实验并行

- 每个实验写独立的 monitor 脚本 + 独立 tmux session
- 用总等待循环检查所有 STATUS_FILE

## 异常处理

- 启动后 30 秒进程就退出 → 立即报告错误，附完整 stderr
- 日志中出现 OOM / CUDA error → 特别标注，建议调整 batch size
- 超过预期时间 2 倍仍未完成 → 报告"超时"，附当前进度
