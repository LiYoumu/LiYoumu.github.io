---
title: Python Textual运行Python子程序
date: 2025-11-02
tag:
  - Python
---

### 程序

示例程序如下：
```python
from textual.app import App, ComposeResult
from textual.widgets import Button, Log
from textual.containers import Container
import subprocess
import asyncio
import os

class ScriptRunnerApp(App):
    """运行外部Python文件的版本"""
    
    CSS = """
    Screen {
        layout: vertical;
    }
    
    #button-container {
        height: auto;
        padding: 1;
        border: solid $accent;
        background: $panel;
    }
    
    #log-container {
        height: 1fr;
        border: solid $accent;
        background: $surface;
    }
    
    Button {
        width: 100%;
    }
    
    Log {
        height: 100%;
    }
    """
    
    def __init__(self):
        super().__init__()
        self.script_path = "your_script.py"  # 替换为你的脚本路径
    
    def compose(self) -> ComposeResult:
        """组合界面组件"""
        yield Container(
            Button("运行外部脚本", id="run-button"),
            id="button-container"
        )
        yield Container(
            Log(id="output-log"),
            id="log-container"
        )
    
    def on_button_pressed(self, event: Button.Pressed) -> None:
        """按钮点击事件处理"""
        if event.button.id == "run-button":
            asyncio.create_task(self.run_external_script())
    
    async def run_external_script(self) -> None:
        """运行外部Python脚本文件（流式输出）"""
        log_widget = self.query_one("#output-log", Log)
        log_widget.clear()
        
        if not os.path.exists(self.script_path):
            log_widget.write_line(f"错误: 找不到脚本文件 {self.script_path}")
            return
        
        log_widget.write_line(f"开始运行脚本: {self.script_path}")
        
        try:
            # 运行外部Python文件
            process = await asyncio.create_subprocess_exec(
                "python", 
                "-u",  # 无缓冲模式
                self.script_path,
                stdout=subprocess.PIPE,
                stderr=subprocess.STDOUT,
                bufsize=0  # 无缓冲
            )
            
            # 实时流式读取输出
            async for line in self.read_lines(process.stdout):
                log_widget.write_line(line)
            
            # 检查返回码
            return_code = await process.wait()
            if return_code == 0:
                log_widget.write_line(f"\n✓ 脚本执行成功")
            else:
                log_widget.write_line(f"\n✗ 脚本执行失败，返回码: {return_code}")
                
        except Exception as e:
            log_widget.write_line(f"执行过程中发生错误: {str(e)}")
    
    async def read_lines(self, stream):
        """异步逐行读取流数据"""
        while True:
            line = await stream.readline()
            if not line:
                break
            yield line.decode('utf-8').rstrip()

if __name__ == "__main__":
    app = ScriptRunnerApp()
    app.run()
```

### 特点

- 真正的实时输出：每行输出立即显示，无需等待整个脚本完成
- 内存效率：不会累积大量输出数据在内存中
- 更好的用户体验：用户可以立即看到脚本的执行进度
- 处理长时间运行脚本：即使脚本运行很长时间，输出也会实时显示
