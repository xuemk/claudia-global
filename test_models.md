# 测试动态模型获取功能

## 测试计划

### 1. 后端API测试
- ✅ get_available_models API已实现
- ✅ 从启用的环境变量组中读取模型
- ✅ 支持MODEL_*_ID、MODEL_*_NAME、MODEL_*_DESCRIPTION模式

### 2. 前端组件测试
- ✅ FloatingPromptInput.tsx 动态加载模型
- ✅ AgentExecution.tsx 动态加载模型
- ✅ API接口 getAvailableModels() 已添加

### 3. 类型系统测试
- ✅ 模型类型定义已更新
- ✅ 支持动态模型和向后兼容
- ✅ 新增DynamicModelInfo接口

## 测试环境变量组配置示例

为了测试功能，需要在环境变量组中配置以下变量：

```
MODEL_1_ID=gpt-4o
MODEL_1_NAME=GPT-4o
MODEL_1_DESCRIPTION=OpenAI的最新模型

MODEL_2_ID=claude-3-5-sonnet-20241022
MODEL_2_NAME=Claude 3.5 Sonnet
MODEL_2_DESCRIPTION=Anthropic的平衡性能模型

MODEL_3_ID=gemini-pro
MODEL_3_NAME=Gemini Pro
MODEL_3_DESCRIPTION=Google的高性能模型
```

## 预期结果

1. 模型选择器应该显示3个模型而不是固定的Claude模型
2. 模型名称应该使用MODEL_*_NAME中的值
3. 模型描述应该显示MODEL_*_DESCRIPTION中的内容
4. 如果没有配置模型，应该显示"无可用模型"的提示

## 测试状态

- ✅ 后端实现完成
- ✅ 前端组件更新完成  
- ✅ 类型定义更新完成
- 🔄 等待运行时测试验证

## 重要文件

1. **后端API**: `src-tauri/src/commands/agents.rs` - get_available_models函数
2. **前端API**: `src/lib/api.ts` - getAvailableModels方法
3. **类型定义**: `src/types/models.ts` - 动态模型类型
4. **组件**: 
   - `src/components/FloatingPromptInput.tsx`
   - `src/components/AgentExecution.tsx`
5. **文档**: `docs/MODEL_CONFIGURATION.md`

## 功能验证检查清单

- [ ] 启动应用程序
- [ ] 在设置页面创建环境变量组
- [ ] 配置MODEL_*_ID等变量
- [ ] 启用环境变量组
- [ ] 检查模型选择器是否显示配置的模型
- [ ] 测试模型选择功能
- [ ] 验证Agent执行时使用正确的模型ID