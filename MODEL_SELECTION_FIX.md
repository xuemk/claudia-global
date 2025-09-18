# 模型选择问题修复总结

## 问题描述

用户报告虽然模型选择器正确显示了动态配置的模型（如"ZhipuAI/GLM-4.5"），但在实际发送消息时系统仍然使用默认的Claude模型ID（"claude-3-5-sonnet-20241022"）而不是用户选择的模型ID。

## 问题分析

通过代码审查发现问题出现在 `FloatingPromptInput` 组件的模型选择逻辑中：

### 原始问题
1. **初始化问题**: `selectedModel` 被初始化为 `defaultModel`（"sonnet-3-5"）
2. **条件逻辑错误**: `useEffect` 中的条件 `if (models.length > 0 && !selectedModel)` 永远不会为真，因为 `selectedModel` 已经有值
3. **无法更新**: 即使动态加载了正确的模型列表，`selectedModel` 仍然保持为默认值
4. **传递错误ID**: `handleSend` 传递的是未更新的 `selectedModel`，导致API收到错误的模型ID

### 数据流分析
```
环境变量 → get_available_models API → FloatingPromptInput.models
                                                 ↓
用户选择 → setSelectedModel → selectedModel (被卡在默认值)
                                     ↓
handleSend → onSend(prompt, selectedModel) → API调用 (错误的模型ID)
```

## 修复方案

### 1. 修复 selectedModel 初始化
**文件**: `src/components/FloatingPromptInput.tsx`

**修改前**:
```typescript
const [selectedModel, setSelectedModel] = useState<ClaudeModel>(defaultModel);
```

**修改后**:
```typescript
const [selectedModel, setSelectedModel] = useState<ClaudeModel>(""); // 空字符串强制从动态模型设置
```

### 2. 添加日志记录
在关键位置添加日志以便调试：

**模型设置日志**:
```typescript
setSelectedModel(defaultModelToUse);
logger.info(`[FloatingPromptInput] Set selected model to: ${defaultModelToUse}`);
```

**模型选择日志**:
```typescript
logger.info(`[FloatingPromptInput] Model selected: ${model.id}`);
```

**发送消息日志**:
```typescript
logger.info(`[FloatingPromptInput] Sending prompt with model: ${selectedModel}`);
```

## 验证流程

1. **启动应用程序**
2. **配置环境变量组** (包含 MID_1、MNAME_1、MDESC_1)
3. **启用环境变量组**
4. **检查模型选择器** - 应显示配置的模型
5. **选择动态模型** - 查看控制台日志确认选择
6. **发送消息** - 查看控制台日志确认使用正确的模型ID
7. **验证API调用** - 确认后端收到正确的模型ID

## 预期结果

修复后的行为：
```
动态模型加载 → 第一个模型被自动选择 → 用户可选择其他模型 → 发送时使用正确的模型ID
```

用户现在应该能够：
- 看到正确的动态模型列表
- 选择任意动态配置的模型
- 发送消息时使用选择的模型ID（如"ZhipuAI/GLM-4.5"）
- 在控制台看到相关的调试日志

## 技术要点

- **类型兼容性**: 动态模型ID被转换为 `ClaudeModel` 类型以保持兼容性
- **getApiModel 函数**: 正确处理动态模型ID，直接返回原始ID而不进行映射
- **向后兼容**: 保持对原有Claude模型的支持
- **错误处理**: 在无可用模型时显示适当的提示信息

## 相关文件

- `src/components/FloatingPromptInput.tsx` - 主要修复文件
- `src/lib/api.ts` - API调用层，使用 `getApiModel()` 转换
- `src/types/models.ts` - 模型类型定义和转换函数
- `src-tauri/src/commands/agents.rs` - 后端获取动态模型API
- `src-tauri/src/commands/claude.rs` - 后端Claude执行命令

## 测试建议

1. 测试无环境变量配置的情况
2. 测试单个模型配置的情况
3. 测试多个模型配置的情况
4. 测试模型选择和切换功能
5. 验证实际API调用使用正确的模型ID