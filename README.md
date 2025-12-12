# Webproxy API调用组件

## 组件概述

`WebproxyApi.vue` 是一个功能完整的API调用和数据展示组件，支持多种HTTP请求方法，提供直观的用户界面用于配置API请求、查看响应数据和导出结果。

## 主要功能

### 1. API请求配置
- **支持多种请求方法**：GET、POST、PUT、DELETE
- **动态API地址和接口路径**
- **可配置请求头**（JSON格式）
- **可编辑请求体**（JSON格式）
- **预设API模板**：提供常用API配置快速选择

### 2. 数据展示
- **完整JSON响应展示**：格式化显示API返回的原始JSON数据
- **表格化数据展示**：自动将响应中的表格数据转换为直观的表格形式
- **分页功能**：支持自定义每页显示条数和页码切换
- **数据统计**：显示总记录数和当前页码信息

### 3. 数据导出
- **Excel导出**：支持将表格数据导出为CSV格式，自动处理特殊字符和中文编码
- **自定义文件名**：导出文件名包含时间戳，避免重复

### 4. 用户体验优化
- **加载状态**：显示请求发送中的状态
- **错误处理**：清晰展示请求错误信息
- **响应式设计**：适配不同屏幕尺寸，小屏幕下自动切换为上下布局
- **现代化UI**：采用渐变色彩、圆角设计和流畅的交互动画

## 组件结构

组件采用三栏布局设计：

```
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│                 │ │                 │ │                 │
│   API配置区     │ │   JSON结果区    │ │   表格数据区    │
│                 │ │                 │ │                 │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### 左侧：API配置区
- API地址输入框
- 请求方法下拉选择
- 接口路径输入框
- 请求头编辑区
- 请求体编辑区
- 预设API按钮组
- 发送请求按钮

### 中间：JSON结果区
- 完整的API响应JSON数据展示
- 支持滚动查看长文本
- 错误信息提示

### 右侧：表格数据区
- 响应数据的表格化展示
- 分页控件（上一页/下一页/当前页码）
- 每页显示条数选择
- 导出Excel按钮

## 核心实现

### 1. 请求体动态生成

组件会根据当前页码和每页显示条数，动态计算并更新请求体中的分页参数：

```javascript
const getRequestBody = () => {
  const offset = (currentPage.value - 1) * pageSize.value + 1
  const bodyObj = JSON.parse(requestBodyTemplate)
  // 更新分页参数
  bodyObj.__blocks__["0"].rows[0][5] = offset.toString() // offset
  bodyObj.__blocks__["0"].rows[0][6] = pageSize.value.toString() // limit
  return JSON.stringify(bodyObj)
}
```

### 2. API请求处理

使用Axios发送HTTP请求，处理响应数据和错误：

```javascript
const sendRequest = async () => {
  try {
    loading.value = true
    error.value = null
    
    const config = {
      url: `${apiUrl.value}${endpoint.value}`,
      method: method.value,
      headers: headers.value ? JSON.parse(headers.value) : {},
      data: method.value !== 'GET' ? JSON.parse(getRequestBody()) : undefined
    }
    
    const result = await axios(config)
    response.value = result.data
    // 更新总条数
    if (result.data?.__blocks__?.Table0?.attr?.count) {
      totalItems.value = parseInt(result.data.__blocks__.Table0.attr.count)
    } else if (result.data?.__blocks__?.Table0?.rows) {
      totalItems.value = result.data.__blocks__.Table0.rows.length
    }
  } catch (err) {
    error.value = err.message
    response.value = null
  } finally {
    loading.value = false
  }
}
```

### 3. 数据导出功能

将表格数据转换为CSV格式，添加UTF-8 BOM解决中文乱码问题：

```javascript
const exportToExcel = () => {
  // 构建CSV内容，添加UTF-8 BOM
  let csvContent = 'data:text/csv;charset=utf-8,' + '\uFEFF'
  
  // 添加表头和数据行
  // ...
  
  // 创建下载链接并触发下载
  const encodedUri = encodeURI(csvContent)
  const link = document.createElement('a')
  link.setAttribute('href', encodedUri)
  link.setAttribute('download', `API_结果_${new Date().getTime()}.csv`)
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}
```

## 使用方法

### 1. 基本使用

直接在Vue应用中引入并使用该组件：

```vue
<template>
  <div>
    <WebproxyApi />
  </div>
</template>

<script setup>
import WebproxyApi from './components/WebproxyApi.vue'
</script>
```

### 2. 配置API请求

1. 在左侧配置区输入API地址和接口路径
2. 选择合适的请求方法
3. 根据需要配置请求头和请求体
4. 点击"发送请求"按钮

### 3. 查看响应数据

- 在中间区域查看完整的JSON响应
- 在右侧区域查看表格化的数据
- 使用分页控件浏览更多数据

### 4. 导出数据

1. 确保已获取到有效的表格数据
2. 点击右侧区域的"导出excel"按钮
3. 浏览器会自动下载CSV格式的文件

## 预设API模板

组件内置了几个常用的API模板：

1. **获取代理列表**：GET请求，获取所有代理服务器列表
2. **测试代理连接**：POST请求，测试指定代理服务器的连接状态
3. **设置代理配置**：PUT请求，更新代理服务器配置

## 自定义配置

### 修改默认API地址

在组件代码中修改`apiUrl`的初始值：

```javascript
const apiUrl = ref('http://your-default-api-url.com/api/CallService')
```

### 修改默认请求体模板

修改`requestBodyTemplate`变量，自定义默认的请求体结构：

```javascript
const requestBodyTemplate = `{
  "__blocks__": {
    "0": {
      // 自定义请求体结构
    }
  }
}`
```

### 添加自定义预设API

在`presetApis`数组中添加新的预设API配置：

```javascript
const presetApis = [
  // 现有预设...
  {
    name: '自定义API',
    method: 'POST',
    endpoint: '/custom-endpoint',
    body: '{"key": "value"}',
    headers: '{"Content-Type": "application/json"}'
  }
]
```

## 技术栈

- Vue 3
- Axios
- 原生JavaScript（CSV导出）
- CSS3（响应式设计、动画效果）

## 浏览器兼容性

- Chrome/Edge 80+
- Firefox 75+
- Safari 13+

## 响应式设计

组件采用响应式设计，在小屏幕设备上会自动调整布局：

- 屏幕宽度 < 768px：切换为上下布局
- 表格在小屏幕下自动转换为卡片式布局
- 输入控件自适应宽度

## 性能优化

- 延迟加载：仅在发送请求后加载数据
- 虚拟滚动：支持大量数据的高效展示
- 防抖处理：避免频繁请求导致的性能问题

## 错误处理

- 网络错误提示
- 无效JSON格式提示
- 空数据状态处理
- 加载超时处理

## 更新日志

### v1.0.0
- 初始版本发布
- 支持基本API请求功能
- 实现JSON和表格数据展示
- 添加数据导出功能

## 作者

组件由张博凯开发，用于内部系统的API调试和数据查询。

## 许可证

MIT License
