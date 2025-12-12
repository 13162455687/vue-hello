<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// API配置
const apiUrl = ref('http://190.2.253.38:10005/BGEJZ/api/CallService') // 用户提供的API地址
const method = ref('POST')
const endpoint = ref('/smlyej02_app_inq')
const headers = ref('{"Content-Type": "application/json"}')
const requestBody = ref('')

// 分页配置
const currentPage = ref(1)
const pageSize = ref(10) // 默认limit为10
const totalItems = ref(0)

// 原始请求体模板
const requestBodyTemplate = `{
 	"__blocks__": {
 	 	"0": {
 	 	 	"meta": {
 	 	 	 	"columns": [
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 0,
 	 	 	 	 	 	"name": "DRIVER_EMPNO",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "司机账号"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 1,
 	 	 	 	 	 	"name": "STOCK_CODE",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "库区代码"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 2,
 	 	 	 	 	 	"name": "VEHICLE_NO",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "车牌号"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 3,
 	 	 	 	 	 	"name": "RESERVATION_DATE",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "预约日期"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 4,
 	 	 	 	 	 	"name": "STATUS",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "预约单状态"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 5,
 	 	 	 	 	 	"name": "offset",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "分页起始"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 6,
 	 	 	 	 	 	"name": "limit",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "分页条数"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 7,
 	 	 	 	 	 	"name": "operator_id",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "操作者工号"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 8,
 	 	 	 	 	 	"name": "operator_cname",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "操作者姓名"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 9,
 	 	 	 	 	 	"name": "RESERVATION_DATE_START",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "预约日期起"
 	 	 	 	 	},
 	 	 	 	 	{
 	 	 	 	 	 	"pos": 10,
 	 	 	 	 	 	"name": "RESERVATION_DATE_END",
 	 	 	 	 	 	"type": "C",
 	 	 	 	 	 	"descName": "预约日期止"
 	 	 	 	 	}
 	 	 	 	]
 	 	 	},
 	 	 	"rows": [
 	 	 	 	[
 	 	 	 	 	"",
 	 	 	 	 	"",
 	 	 	 	 	"",
 	 	 	 	 	"",
 	 	 	 	 	"",
 	 	 	 	 	"1",
                     "20",
                     "195966",
                     "张博凯",
                     "",
                     ""
 	 	 	 	]
 	 	 	],
 	 	 	"attr": {
 	 	 	}
 	 	}
 	}
}`

// 组件挂载时初始化请求体
onMounted(() => {
  requestBody.value = requestBodyTemplate
})

// 请求结果
const response = ref(null)
const loading = ref(false)
const error = ref(null)

// 动态生成请求体，包含分页参数
const getRequestBody = () => {
  // 使用用户输入的请求体，如果为空则使用模板
  let bodyStr = requestBody.value.trim() || requestBodyTemplate
  // 移除请求体中的注释
  bodyStr = removeJsonComments(bodyStr)
  
  const offset = (currentPage.value - 1) * pageSize.value + 1
  const bodyObj = JSON.parse(bodyStr)
  
  try {
    // 尝试更新分页参数（如果请求体包含预期结构）
    if (bodyObj.__blocks__ && bodyObj.__blocks__["0"] && 
        bodyObj.__blocks__["0"].rows && bodyObj.__blocks__["0"].rows[0]) {
      bodyObj.__blocks__["0"].rows[0][5] = offset.toString() // offset
      bodyObj.__blocks__["0"].rows[0][6] = pageSize.value.toString() // limit
    }
  } catch (error) {
    // 如果结构不匹配，不影响请求发送，直接返回用户输入的请求体
    console.warn('请求体结构不匹配，无法自动更新分页参数')
  }
  
  return JSON.stringify(bodyObj)
}

// 移除JSON中的注释
const removeJsonComments = (jsonString) => {
  // 移除单行注释和多行注释
  return jsonString
    // 移除单行注释（// ...）
    .replace(/\/\/.*$/gm, '')
    // 移除多行注释（/* ... */）
    .replace(/\/\*[\s\S]*?\*\//g, '')
    // 移除多余的空白字符
    .trim()
}

// 发送API请求
const sendRequest = async () => {
  try {
    // 输入校验
    if (!apiUrl.value.trim()) {
      throw new Error('API地址不能为空')
    }
    
    // 校验API地址格式
    try {
      new URL(apiUrl.value)
    } catch {
      throw new Error('API地址格式不正确，请输入完整的URL，如：http://example.com')
    }
    
    if (!endpoint.value.trim()) {
      throw new Error('接口路径不能为空')
    }
    
    // 校验请求头格式
    let parsedHeaders = {}
    if (headers.value) {
      try {
        parsedHeaders = JSON.parse(removeJsonComments(headers.value))
      } catch {
        throw new Error('请求头格式不正确，请输入有效的JSON格式')
      }
    }
    
    // 非GET请求时校验请求体
    let requestData = undefined
    if (method.value !== 'GET') {
      // 使用用户输入的请求体，如果为空则使用模板
      const bodyStr = requestBody.value.trim() || requestBodyTemplate
      try {
        // 先检查用户输入的请求体是否有效（移除注释后）
        JSON.parse(removeJsonComments(bodyStr))
        // 再检查动态生成的请求体是否有效
        requestData = JSON.parse(getRequestBody())
      } catch {
        throw new Error('请求体格式不正确，请输入有效的JSON格式')
      }
    }
    
    loading.value = true
    error.value = null
    
    const config = {
      url: `${apiUrl.value}${endpoint.value}`,
      method: method.value,
      headers: parsedHeaders,
      data: requestData
    }
    
    const result = await axios(config)
    response.value = result.data
    // 更新总条数（来自__blocks__里Table0里attr里count字段）
    if (result.data?.__blocks__?.Table0?.attr?.count) {
      totalItems.value = parseInt(result.data.__blocks__.Table0.attr.count)
    } else if (result.data?.__blocks__?.Table0?.rows) {
      // 如果没有返回count，暂时用返回的行数作为总条数
      totalItems.value = result.data.__blocks__.Table0.rows.length
    }
  } catch (err) {
    error.value = err.message
    response.value = null
  } finally {
    loading.value = false
  }
}

// 预设一些常见的Webproxy API调用
const presetApis = [
  {
    name: '获取代理列表',
    method: 'GET',
    endpoint: '/proxies',
    body: '',
    headers: '{"Content-Type": "application/json"}'
  },
  {
    name: '测试代理连接',
    method: 'POST',
    endpoint: '/test',
    body: '{"proxy": "http://localhost:8080"}',
    headers: '{"Content-Type": "application/json"}'
  },
  {
    name: '设置代理配置',
    method: 'PUT',
    endpoint: '/config',
    body: '{"enabled": true, "proxyUrl": "http://localhost:8080"}',
    headers: '{"Content-Type": "application/json"}'
  }
]

// 应用预设API
const applyPreset = (preset) => {
  method.value = preset.method
  endpoint.value = preset.endpoint
  requestBody.value = preset.body
  headers.value = preset.headers
}

// 导出Excel功能
const exportToExcel = () => {
  if (!response.value || !response.value.__blocks__?.Table0) {
    return
  }
  
  const tableData = response.value.__blocks__.Table0
  const columns = tableData.meta?.columns || []
  const rows = tableData.rows || []
  
  // 构建CSV内容，添加UTF-8 BOM以解决Excel乱码问题
  let csvContent = 'data:text/csv;charset=utf-8,' + '\uFEFF'
  
  // 添加表头
  const headerRow = columns.map(col => col.name || `列${columns.indexOf(col) + 1}`)
  csvContent += headerRow.join(',') + '\n'
  
  // 添加数据行
  rows.forEach(row => {
    const csvRow = row.map(cell => {
      // 处理包含逗号、引号等特殊字符的数据
      if (typeof cell === 'string') {
        // 如果包含逗号、双引号或换行符，需要用双引号包裹
        if (cell.includes(',') || cell.includes('"') || cell.includes('\n')) {
          // 转义双引号为两个双引号
          return `"${cell.replace(/"/g, '""')}"`
        }
      }
      return cell
    })
    csvContent += csvRow.join(',') + '\n'
  })
  
  // 创建下载链接并触发下载
  const encodedUri = encodeURI(csvContent)
  const link = document.createElement('a')
  link.setAttribute('href', encodedUri)
  link.setAttribute('download', `API_结果_${new Date().getTime()}.csv`)
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}

// 组件挂载时初始化
onMounted(() => {
  // 可以在这里初始化一些配置
})
</script>

<template>
  <div class="webproxy-api">
    <h2>Webproxy API调用  VUE版本</h2>
    <h2>外部系统对接物流提升二步项目WebProxy接口自测</h2> 
    
    <!-- 左右布局容器 -->
    <div class="main-content">
      <!-- API配置区 -->
      <div class="config-section">
        <!-- 配置内容滚动容器 -->
        <div class="config-content">
          <div class="input-group">
            <label for="apiUrl">API地址：</label>
            <input 
              id="apiUrl" 
              v-model="apiUrl" 
              type="text" 
              placeholder="http://localhost:3000/api/webproxy"
            />
          </div>
          
          <div class="input-row">
            <div class="input-group">
              <label for="method">请求方法：</label>
              <select id="method" v-model="method">
                <option value="GET">GET</option>
                <option value="POST">POST</option>
                <option value="PUT">PUT</option>
                <option value="DELETE">DELETE</option>
              </select>
            </div>
            
            <div class="input-group">
              <label for="endpoint">接口路径：</label>
              <input 
                id="endpoint" 
                v-model="endpoint" 
                type="text" 
                placeholder="/proxy"
              />
            </div>
          </div>
          
          <div class="input-group">
            <label for="headers">请求头（JSON）：</label>
            <textarea 
              id="headers" 
              v-model="headers" 
              rows="3"
              placeholder='{"Content-Type": "application/json"}'
            ></textarea>
          </div>
          
          <div class="input-group" v-if="method !== 'GET'">
            <label for="requestBody">请求体（JSON）：</label>
            <textarea 
              id="requestBody" 
              v-model="requestBody" 
              rows="5"
              placeholder='{"url": "https://example.com"}'
            ></textarea>
          </div>
          
          <!-- 预设API -->
          <div class="presets">
            <h3>预设API：</h3>
            <div class="preset-buttons">
              <button 
                v-for="(preset, index) in presetApis" 
                :key="index"
                @click="applyPreset(preset)"
              >
                {{ preset.name }}
              </button>
            </div>
          </div>
        </div>
        
        <!-- 发送按钮固定在底部 -->
        <div class="send-btn-container">
          <button 
            class="send-btn" 
            @click="sendRequest" 
            :disabled="loading"
          >
            {{ loading ? '发送中...' : '发送请求' }}
          </button>
        </div>
      </div>
    
    <!-- 中间：完整JSON结果 -->
    <div class="json-section">
      <h3>完整请求结果</h3>
      
      <!-- 错误信息 -->
      <div v-if="error" class="error-message">
        <strong>错误：</strong>{{ error }}
      </div>
      
      <!-- 响应数据 -->
      <div v-else class="response-data">
        <div v-if="response" class="json-content-wrapper">
          <!-- 添加一个内部容器，确保宽度超出父容器 -->
          <div style="min-width: calc(100% + 1px); width: fit-content;">
            <pre>{{ JSON.stringify(response, null, 2) }}</pre>
          </div>
        </div>
        
        <!-- 空状态 -->
        <div v-else-if="!loading" class="json-content-wrapper">
          <!-- 添加一个内部容器，确保宽度超出父容器 -->
          <div style="min-width: calc(100% + 1px); width: fit-content;">
            <div class="empty-state-content">
              点击"发送请求"按钮开始调用API
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 右侧：Table0表格数据 -->
    <div class="table-section">
      <div class="table-header-container">
        <h3>转换成表格</h3>
        <button 
          class="export-btn" 
          @click="exportToExcel" 
          :disabled="!response || !response.__blocks__?.Table0"
        >
          导出excel
        </button>
      </div>
      
      <!-- 错误信息 -->
      <div v-if="error" class="error-message">
        <strong>错误：</strong>{{ error }}
      </div>
      
      <!-- 响应数据 -->
      <div v-else class="response-data">
        <div class="table-container">
          <!-- 添加一个内部容器，确保宽度超出父容器 -->
          <!-- 分页控件 -->
          <div v-if="response && response.__blocks__?.Table0" class="pagination-container">
            <div class="pagination-info">
              共 {{ totalItems }} 条记录，第 {{ currentPage }} / {{ Math.ceil(totalItems / pageSize) || 1 }} 页
            </div>
            <div class="pagination-controls">
              <button 
                @click="currentPage > 1 && (currentPage--, sendRequest())"
                :disabled="currentPage === 1 || loading"
                class="pagination-btn"
              >
                上一页
              </button>
              <span class="current-page">{{ currentPage }}</span>
              <button 
                @click="currentPage < Math.ceil(totalItems / pageSize) && (currentPage++, sendRequest())"
                :disabled="currentPage >= Math.ceil(totalItems / pageSize) || loading"
                class="pagination-btn"
              >
                下一页
              </button>
            </div>
            <div class="page-size-selector">
              <label>每页显示：</label>
              <select 
                v-model.number="pageSize"
                @change="currentPage = 1, sendRequest()"
                class="page-size-input"
              >
                <option :value="10">10</option>
                <option :value="20">20</option>
                <option :value="50">50</option>
                <option :value="100">100</option>
                <option :value="500">500</option>
                <option :value="1000">1000</option>
              </select>
            </div>
          </div>
          
          <div style="min-width: calc(100% + 1px); width: fit-content;">
            <table v-if="response" class="result-table">
              <!-- 表头 -->
              <thead>
                <tr>
                  <th 
                    v-for="(col, index) in response.__blocks__?.Table0?.meta?.columns || []" 
                    :key="index" 
                    class="table-header"
                  >
                    {{ col.name || `列${index + 1}` }}
                  </th>
                </tr>
              </thead>
              <!-- 数据行 -->
              <tbody>
                <!-- 直接遍历response.__blocks__.Table0.rows，使用兜底数组 -->
                <tr v-for="(row, rowIndex) in (response.__blocks__?.Table0?.rows || [])" :key="rowIndex" class="table-row">
                  <td 
                    v-for="(cell, cellIndex) in row" 
                    :key="cellIndex" 
                    class="table-cell"
                    :data-label="response.__blocks__?.Table0?.meta?.columns[cellIndex]?.name || `列${cellIndex + 1}`"
                  >
                    {{ cell }}
                  </td>
                </tr>
                <!-- 无数据时显示 -->
                <tr v-if="(response.__blocks__?.Table0?.rows || []).length === 0">
                  <td colspan="100%" class="empty-cell">
                    无数据
                  </td>
                </tr>
              </tbody>
            </table>
            
            <!-- 空状态 -->
            <div v-else-if="!loading" class="empty-cell">
              点击"发送请求"按钮开始调用API
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  </div>
</template>

<style scoped>
.webproxy-api {
  width: 100%;
  margin: 0 auto;
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  min-height: calc(100vh - 40px);
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow: hidden;
  box-sizing: border-box;
}

/* 左中右布局容器 */
.main-content {
  display: flex;
  gap: 24px;
  flex-wrap: nowrap;
  width: 100%;
  max-width: 2400px;
  justify-content: center;
  align-items: stretch;
  min-height: 700px;
  height: auto;
  max-height: calc(100vh - 120px);
  overflow: hidden;
  box-sizing: border-box;
}

h2 {
  text-align: center;
  color: white;
  margin-bottom: 25px;
  font-size: 28px;
  font-weight: 600;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h3 {
  font-size: 20px;
  font-weight: 600;
  color: #667eea;
  margin-bottom: 20px;
  padding: 12px 20px;
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.1) 0%, rgba(118, 75, 162, 0.1) 100%);
  border-radius: 12px;
  border: 1px solid rgba(102, 126, 234, 0.2);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.15);
  display: inline-block;
  transition: all 0.3s ease;
}

h3:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.2);
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.15) 0%, rgba(118, 75, 162, 0.15) 100%);
  border-color: rgba(102, 126, 234, 0.4);
}

/* 修复表格头部容器中的h3样式 */
.table-header-container h3 {
  margin: 0;
  padding: 10px 18px;
  font-size: 18px;
}

.config-section {
  background-color: white;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  width: 35%;
  flex-shrink: 1;
  transition: all 0.3s ease;
  border: 1px solid #e2e8f0;
  min-height: 700px;
  min-width: 350px;
  height: auto;
  max-height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-sizing: border-box;
}

/* 配置内容滚动容器 */
.config-content {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding-right: 8px;
  margin-bottom: 20px;
  /* 自定义滚动条 */
  scrollbar-width: thin;
  scrollbar-color: rgba(102, 126, 234, 0.5) rgba(102, 126, 234, 0.1);
}

.config-content::-webkit-scrollbar {
  width: 8px;
}

.config-content::-webkit-scrollbar-track {
  background: rgba(102, 126, 234, 0.1);
  border-radius: 4px;
}

.config-content::-webkit-scrollbar-thumb {
  background: rgba(102, 126, 234, 0.5);
  border-radius: 4px;
  transition: all 0.3s ease;
}

.config-content::-webkit-scrollbar-thumb:hover {
  background: rgba(118, 75, 162, 0.6);
}

/* 发送按钮容器 */
.send-btn-container {
  border-top: 1px solid rgba(102, 126, 234, 0.2);
  padding-top: 20px;
  margin-top: auto;
  background: linear-gradient(to bottom, rgba(255, 255, 255, 0) 0%, rgba(255, 255, 255, 0.95) 50%);
}

/* 发送按钮美化 */
.send-btn {
  width: 100%;
  padding: 14px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
  position: relative;
  overflow: hidden;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.send-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.5);
  background: linear-gradient(135deg, #764ba2 0%, #667eea 100%);
}

.send-btn:active:not(:disabled) {
  transform: translateY(0);
  box-shadow: 0 3px 10px rgba(102, 126, 234, 0.4);
}

.config-section:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}

.input-row {
  display: flex;
  gap: 12px;
  margin-bottom: 20px;
}

.input-group {
  margin-bottom: 20px;
}

.input-group label {
  display: inline-block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #667eea;
  font-size: 14px;
  transition: all 0.3s ease;
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.1) 0%, rgba(118, 75, 162, 0.1) 100%);
  padding: 6px 12px;
  border-radius: 20px;
  border: 1px solid rgba(102, 126, 234, 0.2);
  box-shadow: 0 2px 4px rgba(102, 126, 234, 0.1);
}

.input-group label:hover {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.15) 0%, rgba(118, 75, 162, 0.15) 100%);
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(102, 126, 234, 0.15);
  border-color: rgba(102, 126, 234, 0.4);
}

.input-group input,
.input-group select,
.input-group textarea {
  width: 100%;
  padding: 10px 14px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 14px;
  font-family: inherit;
  transition: all 0.3s ease;
  background: #ffffff;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  text-align: left;
  text-indent: 0;
}

.input-group input:hover,
.input-group select:hover,
.input-group textarea:hover {
  border-color: #cbd5e0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.input-group input:focus,
.input-group select:focus,
.input-group textarea:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  background: #ffffff;
}

.input-group textarea {
  resize: vertical;
  min-height: 80px;
  text-align: left;
  text-indent: 0;
}

.presets {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f7fafc;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
}

.presets h3 {
  margin-bottom: 12px;
  font-size: 15px;
  color: #2d3748;
  border-bottom: none;
  padding-bottom: 0;
  font-weight: 600;
}

.preset-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.preset-buttons button {
  padding: 8px 14px;
  background-color: #ffffff;
  border: 2px solid #e2e8f0;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  color: #4a5568;
  transition: all 0.3s ease;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.preset-buttons button:hover {
  background-color: #667eea;
  color: white;
  border-color: #667eea;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(102, 126, 234, 0.3);
}

.preset-buttons button:active {
  transform: translateY(0);
}

.send-btn {
  width: 100%;
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
  position: relative;
  overflow: hidden;
}

.send-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.5);
}

.send-btn:active:not(:disabled) {
  transform: translateY(0);
}

.send-btn:disabled {
  background: #a0aec0;
  cursor: not-allowed;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  opacity: 0.7;
}

/* 加载动画 */
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.send-btn:disabled::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 20px;
  margin: -10px 0 0 -10px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

.result-section {
  background-color: white;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  width: 75%;
  overflow-x: auto;
  transition: all 0.3s ease;
  border: 1px solid #e2e8f0;
}

.result-section:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}

/* 中间：完整JSON结果区 */
.json-section {
  background-color: white;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  width: 30%;
  flex-shrink: 1;
  transition: all 0.3s ease;
  border: 1px solid #e2e8f0;
  min-height: 700px;
  min-width: 300px;
  height: auto;
  max-height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-sizing: border-box;
}

.json-section:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}

/* 右侧：表格数据区 */
.table-section {
  background-color: white;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  width: 35%;
  flex-shrink: 1;
  transition: all 0.3s ease;
  border: 1px solid #e2e8f0;
  min-height: 700px;
  min-width: 350px;
  height: auto;
  max-height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-sizing: border-box;
}

.table-section:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}

/* 表格头部容器 */
.table-header-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

/* 导出Excel按钮 */
.export-btn {
  padding: 10px 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
}

.export-btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.export-btn:disabled {
  background: #a0aec0;
  cursor: not-allowed;
  box-shadow: none;
  opacity: 0.7;
}

.response-data {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.95) 0%, rgba(247, 250, 252, 0.95) 100%);
  padding: 20px;
  border-radius: 12px;
  overflow-x: auto;
  overflow-y: auto;
  max-height: calc(100% - 20px);
  height: auto;
  flex-grow: 1;
  border: 1px solid rgba(102, 126, 234, 0.2);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.1);
  transition: all 0.3s ease;
  margin-top: 15px;
  display: block;
  position: relative;
  backdrop-filter: blur(10px);
}

.response-data:hover {
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.15);
  border-color: rgba(102, 126, 234, 0.3);
}

/* 美化滚动条 */
.response-data::-webkit-scrollbar,
.json-content-wrapper::-webkit-scrollbar,
.table-container::-webkit-scrollbar,
textarea::-webkit-scrollbar {
  width: 10px;
  height: 12px;
}

.response-data::-webkit-scrollbar-track,
.json-content-wrapper::-webkit-scrollbar-track,
.table-container::-webkit-scrollbar-track,
textarea::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.response-data::-webkit-scrollbar-thumb,
.json-content-wrapper::-webkit-scrollbar-thumb,
.table-container::-webkit-scrollbar-thumb,
textarea::-webkit-scrollbar-thumb {
  background: #667eea;
  border-radius: 6px;
  transition: all 0.3s ease;
  border: 2px solid #f1f1f1;
}

.response-data::-webkit-scrollbar-thumb:hover,
.json-content-wrapper::-webkit-scrollbar-thumb:hover,
.table-container::-webkit-scrollbar-thumb:hover,
textarea::-webkit-scrollbar-thumb:hover {
  background: #764ba2;
  transform: scaleY(1.2);
  box-shadow: 0 2px 6px rgba(102, 126, 234, 0.4);
}

.response-data pre {
  margin: 0;
  font-family: 'Courier New', Courier, monospace;
  font-size: 13px;
  line-height: 1.6;
  color: #333;
  background: #ffffff;
  padding: 15px;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  white-space: pre;
  overflow: visible;
  min-width: fit-content;
  text-align: left;
}

/* 空状态内容样式 */
.empty-state-content {
  text-align: center;
  padding: 20px;
  color: #a0aec0;
  font-size: 14px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 15px;
  min-height: calc(100% - 40px);
  height: calc(100% - 40px);
  margin: -15px;
  background-color: #ffffff;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.empty-state-content::before {
  content: "📡";
  font-size: 48px;
  opacity: 0.5;
  transition: all 0.3s ease;
}

.error-message {
  background-color: #fff5f5;
  border: 1px solid #fed7d7;
  padding: 15px;
  border-radius: 8px;
  color: #c53030;
  font-size: 14px;
  margin-bottom: 15px;
  display: flex;
  align-items: flex-start;
  gap: 10px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.error-message::before {
  content: "⚠️";
  font-size: 18px;
  flex-shrink: 0;
  margin-top: 1px;
}

.empty-state-content {
  text-align: center;
  padding: 100px 20px;
  color: #a0aec0;
  font-size: 14px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 15px;
  min-height: 200px;
}

.empty-state-content::before {
  content: "📡";
  font-size: 48px;
  opacity: 0.5;
  transition: all 0.3s ease;
}

.empty-cell {
  text-align: center;
  padding: 60px 20px;
  color: #a0aec0;
  font-style: italic;
  font-size: 14px;
  background-color: #fafafa;
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 表格样式 */
.result-table {
  width: 100%;
  border-collapse: collapse;
  margin: 10px 0;
  background-color: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.08);
  font-size: 14px;
  transition: all 0.3s ease;
}

.result-table:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
}

.result-table th,
.result-table td {
  padding: 14px 16px;
  text-align: left;
  border-bottom: 1px solid #e9ecef;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: all 0.3s ease;
}

.result-table th {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  font-weight: 600;
  border-bottom: none;
  min-width: 120px;
  position: sticky;
  top: 0;
  z-index: 10;
}

.result-table tr:last-child td {
  border-bottom: none;
}

.result-table tr:hover td {
  background-color: #f7fafc;
  color: #2d3748;
}

/* 表格容器样式 */
.table-container {
  background-color: #f8f9fa;
  padding: 15px;
  overflow-x: scroll;
  overflow-y: auto;
  max-height: calc(100% - 40px);
  height: auto;
  flex-grow: 1;
  border-radius: 8px;
  border: 1px solid #e9ecef;
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.05);
  margin: 0;
  transition: all 0.3s ease;
  width: 100%;
  display: block;
  position: relative;
  /* 强制显示滚动条，即使内容没有超出 */
  scrollbar-width: thick;
  scrollbar-color: #667eea #f1f1f1;
}

.table-container table {
  /* 确保表格宽度超出容器，触发滚动条 */
  min-width: calc(100% + 1px);
  width: fit-content;
}

.table-container:hover {
  box-shadow: inset 0 2px 6px rgba(0, 0, 0, 0.08);
}

/* 表格单元格样式优化 */
.table-header {
  font-weight: bold;
  text-transform: uppercase;
  font-size: 12px;
  letter-spacing: 0.8px;
  opacity: 0.9;
}

.table-row:nth-child(even) {
  background-color: #fafafa;
}

.table-row:nth-child(even):hover td {
  background-color: #f7fafc;
}

.table-cell {
  vertical-align: middle;
  color: #4a5568;
}

/* 空单元格样式 */
.empty-cell {
  text-align: center;
  padding: 60px 20px;
  color: #a0aec0;
  font-style: italic;
  font-size: 14px;
  background-color: #fafafa;
  min-height: calc(100% - 120px);
  display: flex;
  align-items: center;
  justify-content: center;
}

/* JSON内容包装器 */
.json-content-wrapper {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.95) 0%, rgba(247, 250, 252, 0.95) 100%);
  padding: 20px;
  border-radius: 12px;
  border: 1px solid rgba(102, 126, 234, 0.2);
  overflow-x: scroll;
  overflow-y: auto;
  max-height: calc(100% - 40px);
  height: auto;
  flex-grow: 1;
  transition: all 0.3s ease;
  display: block;
  width: 100%;
  position: relative;
  /* 强制显示滚动条，即使内容没有超出 */
  scrollbar-width: thick;
  scrollbar-color: #667eea #f1f1f1;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.1);
  backdrop-filter: blur(10px);
}

.json-content-wrapper:hover {
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.15);
  border-color: rgba(102, 126, 234, 0.3);
}

.json-content-wrapper pre {
  margin: 0;
  font-family: 'Courier New', Courier, monospace;
  font-size: 13px;
  line-height: 1.6;
  color: #333;
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  padding: 20px;
  border-radius: 10px;
  border: 1px solid rgba(102, 126, 234, 0.2);
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.1);
  white-space: pre;
  overflow: visible;
  /* 确保内容宽度超出容器，触发滚动条 */
  min-width: calc(100% + 1px);
  width: fit-content;
  display: block;
  transition: all 0.3s ease;
}

.json-content-wrapper pre:hover {
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.15);
  border-color: rgba(102, 126, 234, 0.3);
}

.json-content-wrapper .empty-state-content {
  text-align: center;
  padding: 100px 20px;
  color: #a0aec0;
  font-size: 14px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 15px;
  min-height: 200px;
}

.json-content-wrapper .empty-state-content::before {
  content: "📡";
  font-size: 48px;
  opacity: 0.5;
  transition: all 0.3s ease;
}

/* 区块样式 */
.blocks-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.block-item {
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #e9ecef;
}

.block-item h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #495057;
  font-size: 16px;
}

/* 空数据样式 */
.empty-data {
  text-align: center;
  padding: 20px;
  color: #6c757d;
  background-color: white;
  border-radius: 4px;
  border: 1px dashed #dee2e6;
}

/* 原始区块样式 */
.block-raw {
  background-color: white;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
  overflow-x: auto;
}

/* 无blocks字段样式 */
.no-blocks {
  background-color: white;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
  overflow-x: auto;
}

/* 分页样式 */
.pagination-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20px;
  padding: 15px;
  background-color: white;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  flex-wrap: wrap;
  gap: 15px;
}

.pagination-info {
  color: #4a5568;
  font-size: 14px;
  font-weight: 500;
}

.pagination-controls {
  display: flex;
  align-items: center;
  gap: 12px;
}

.pagination-btn {
  padding: 8px 16px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 2px 6px rgba(102, 126, 234, 0.3);
  min-width: 80px;
}

.pagination-btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(102, 126, 234, 0.4);
}

.pagination-btn:disabled {
  background: #a0aec0;
  cursor: not-allowed;
  opacity: 0.7;
  box-shadow: none;
}

.current-page {
  font-size: 16px;
  font-weight: 600;
  color: #667eea;
  min-width: 30px;
  text-align: center;
}

.page-size-selector {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #4a5568;
  font-size: 14px;
}

.page-size-input {
  padding: 6px 10px;
  border: 2px solid #e2e8f0;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  color: #4a5568;
  background-color: white;
  transition: all 0.3s ease;
  min-width: 80px;
}

.page-size-input:hover {
  border-color: #cbd5e0;
}

.page-size-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

@media (max-width: 768px) {
  /* 小屏幕下切换为上下布局 */
  .main-content {
    flex-direction: column;
  }
  
  .config-section,
  .json-section,
  .table-section {
    width: 100%;
    min-height: 500px;
  }
  
  .input-row {
    flex-direction: column;
  }
  
  .preset-buttons {
    flex-direction: column;
  }
  
  .preset-buttons button {
    width: 100%;
  }
  
  /* 响应式表格 */
  .result-table {
    display: block;
    overflow-x: auto;
  }
  
  .result-table thead,
  .result-table tbody,
  .result-table tr,
  .result-table th,
  .result-table td {
    display: block;
    text-align: right;
  }
  
  .result-table thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }
  
  .result-table tr {
    margin-bottom: 15px;
    border: 1px solid #dee2e6;
    border-radius: 4px;
    overflow: hidden;
  }
  
  .result-table td {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px;
    border-bottom: 1px solid #f8f9fa;
    text-align: right;
    position: relative;
  }
  
  .result-table td:before {
    content: attr(data-label);
    position: absolute;
    left: 12px;
    width: 45%;
    padding-right: 10px;
    font-weight: 600;
    text-align: left;
    color: #495057;
  }
  
  .result-table td:last-child {
    border-bottom: none;
  }
}
</style>