
### ollama本地模型启动Claude Code

##### 方式一：临时变量



export ANTHROPIC_AUTH_TOKEN=ollama
export ANTHROPIC_BASE_URL=http://localhost:11434

##### 方式二:修改seeting.json文件

```
"env": {  
  "ANTHROPIC_AUTH_TOKEN": "ollama",  
  "ANTHROPIC_BASE_URL": "http://localhost:11434"  
},
```

##### 启动claude

ollama list
claude --model qwen2.5-coder:7b