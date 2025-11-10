# 2api-to-CFwork

# 2025年11月4日 18:16:37（2.0提示词，推荐大家使用1.0提示词，那样更小白化）：
```
角色扮演：
你是一位世界顶级的首席开发者体验架构师 (Principal Developer Experience Architect)，兼具 Serverless 架构师与全栈设计大师的深厚功力。你的核心哲学是“为开发者打造如同F1赛车驾驶舱般的工具——信息密集、响应迅捷、绝对可靠”。你痴迷于将复杂的后端服务，通过精妙的工程设计，转化为一个单一、自包含、自带“终极说明书”的艺术品级 Cloudflare Worker。

核心任务：
我将提供一个完整的 Python API 项目源代码（或其他语言的项目，你可以根据我提供的项目自动转换自动识别哈，有可能是c++项目也有可能是docker等等那些项目）。你的任务是：

核心转换： 将该 Python 项目的后端代理逻辑，完整、无损地迁移到一个高性能的 Cloudflare Worker 中。
体验封装： 在此基础上，为 Worker 的根路径 (/) 构建一个以“开发者驾驶舱”为核心的、信息架构与交互体验无懈可击的、完全中文化的 HTML 交互界面。
最终交付物：
一个单一的、格式化良好的、可直接部署的 JavaScript 文件，包含所有 Worker 逻辑、HTML、CSS 和客户端 JS。

第一部分：对生成的 Worker 后端逻辑的技术要求 (Backend Logic)
这部分是转换的基础，必须优先、准确地实现。

架构：配置即代码 (Configuration-as-Code)

在生成的 JS 文件顶部，创建一个 const CONFIG = {...} 对象，将所有核心输入参数（如 API_MASTER_KEY, UPSTREAM_URL, PROJECT_NAME, PROJECT_VERSION, DEFAULT_MODEL 等）清晰地组织在内。后续所有逻辑都必须从此 CONFIG 对象读取。
路由 (Routing)

GET /: 返回第二部分定义的“开发者驾驶舱” HTML 页面。
/v1/**: 执行 API 代理逻辑。
其他所有路径: 返回包含错误信息的 404 JSON 响应。
API 代理逻辑 (/v1/**)

认证: 严格校验 Authorization: Bearer <KEY>，必须与 CONFIG.API_MASTER_KEY 全等。失败则返回 401 JSON 错误。
可观测性 (Observability):
请求水印: 为每个进入的 API 请求生成 crypto.randomUUID() 作为请求 ID，同时添加到 X-Request-ID (转发给上游) 和 X-Worker-Trace-ID (响应给客户端) 响应头中，以便端到端追踪。
请求转发与协议现代化:
将客户端请求（方法、头、主体）准确转发至 CONFIG.UPSTREAM_URL 对应的路径。
HTTP/3 优先: 在 fetch 请求中，应暗示 Cloudflare 优先使用 HTTP/3 与上游通信。
Brotli 压缩: 自动对返回给客户端的文本类响应（HTML, JSON）应用 Brotli 压缩。
智能流式处理与错误处理:
完美代理SSE: 必须完美支持并原样代理上游的 Server-Sent Events 流。
背压 (Backpressure) 处理: 利用 ReadableStream 的内在机制，优雅地处理客户端消费速度慢于上游生产速度的情况，防止 Worker 内存溢出。
捕获上游错误（网络问题、非2xx状态码），并转化为对开发者友好的、结构化的 JSON 错误响应。
性能与缓存:
利用 Cloudflare Cache API 对上游的幂等、可缓存请求（如 GET /v1/models）进行缓存，缓存键应包含路径和认证信息摘要。
第二部分：对生成的“开发者驾驶舱”页面的硬性要求 (UI/UX & Functionality)
在满足第一部分功能的基础上，构建以下前端体验。

核心技术与美学

前端技术: 整个页面必须由自定义元素 (Custom Elements) 构成 (如 <main-layout>, <status-indicator>, <live-terminal>)，并使用 Shadow DOM 实现样式隔离。同时，必须实现渐进增强，在 JS 失效时核心信息依然可见。
状态管理: 客户端 JS 需实现一个精巧的状态机来管理 UI 状态 (INITIALIZING, HEALTH_CHECKING, READY, REQUESTING, STREAMING, ERROR)，并严格同步所有交互元素的外观和行为。在加载状态下，必须显示骨架屏 (Skeleton Screen) 效果。
主题与美学:
主题: 深色背景 (#121212)，白色/灰色文本 (#E0E0E0, #888888)。
高亮: 所有可交互、可复制的关键信息使用醒目的琥珀色 (#FFBF00)。
布局: 桌面端双栏，移动端单栏。
图标: 所有图标必须是高质量的内联 SVG。
核心功能板块 (全中文界面)

顶部标题栏: 左侧显示项目名和版本号，右侧放置 <status-indicator> 组件实时展示上游健康状况。
左栏：📋 即用情报 (Actionable Intelligence)
使用 <info-panel> 组件，清晰展示 API 地址、API 密钥（带复制和显/隐功能）、默认模型。
右栏：🚀 实时交互终端 (Live Terminal)
使用 <live-terminal> 组件，包含：
AI 输出窗口: 支持 Markdown 的流式响应显示。
请求日志 & 性能洞察: 以虚拟滚动表格展示历史请求（ID, 状态, TTFB, 总耗时, 速率），并实时计算最近10次的成功率和平均耗时。
指令输入区: 支持自动增高的 <textarea> 和状态同步的 发送/取消 按钮。
附加情报区 (Collapsible Sections)
使用 <details> 元素实现，默认折叠。
[ ⚙️ 主流客户端集成指南 ]: 使用 <client-guides> 组件，通过 Tab 展示 ChatGPT-Next-Web, LobeChat, cURL, Python 的预填值配置块。
[ 🔌 兼容接口参考 ]: 表格列出所有代理的 API 接口及方法。
[ 🛠️ 调试与复现工具箱 ]: 展示 上游接口、项目模式，并提供“查看上次请求详情”和“一键复现 cURL”的功能。
最终指令：
现在，请严格遵循以上这份包含后端逻辑和前端体验的完整蓝图，结合我接下来提供的 Python 项目源代码，开始转换，生成那个单一、完整、自包含且可立即部署的“艺术品级” JavaScript Worker 文件。

注释也要中文化哈，还有就是web界面使用说明一定要中文化

我最终只想要稳定部署无bug无错误的

类似于这种错误你要比避免：
ncaught SyntaxError: Invalid or unexpected token at worker.js:345:20

我下方可以给你几个我的成功品你可以参考参考一下：
// =================================================================================
//  项目: umint-2api (Cloudflare Worker 单文件版)
//  版本: 8.0.5 (代号: Chimera Synthesis - Final)
//  作者: 首席AI执行官 (Principal AI Executive Officer)
//  协议: 奇美拉协议 · 综合版 (Project Chimera: Synthesis Edition)
//  日期: 2025-11-10
//
//  描述:
//  本文件是一个完全自包含、可一键部署的 Cloudflare Worker。它将 umint-ai.hf.space
//  的后端服务，无损地转换为一个高性能、兼容 OpenAI 标准的 API，并内置了一个
//  功能强大的"开发者驾驶舱"Web UI，用于实时监控、测试和集成。
//
//  v8.0.5 修正:
//  1. [TypeError] 修正了 `performHealthCheck` 中因未穿透 Shadow DOM 导致无法找到 `status-indicator` 组件的错误。
//  2. [SyntaxError] 修正了 `getCurlGuide` 中因模板字符串多层转义不当导致的客户端语法错误。
//
// =================================================================================
// --- [第一部分: 核心配置 (Configuration-as-Code)] ---
// 架构核心：所有关键参数在此定义，后续逻辑必须从此对象读取。
const CONFIG = {
  // 项目元数据
  PROJECT_NAME: "umint-2api",
  PROJECT_VERSION: "8.0.5",
  // 安全配置
  API_MASTER_KEY: "1", // 密钥设置为 "1"
  // 上游服务配置
  UPSTREAM_URL: "https://umint-ai.hf.space/api/b1235a8f4c2f4b33a99e8a7c87912b3d",
  // 模型映射
  // 从情报中自动识别并提取的模型列表
  MODELS: [
    "moonshotai/kimi-k2-thinking",
    "deepseek-ai/deepseek-r1-0528",
    "deepseek-ai/deepseek-r1-0528-nvidia",
    "deepseek-ai/deepseek-v3.1",
    "deepseek-ai/deepseek-v3.1-terminus",
    "google/gemini-2.5-flash-lite",
    "minimaxai/minimax-m2",
    "moonshotai/kimi-k2-instruct",
    "moonshotai/kimi-k2-instruct-0905",
    "openai/gpt-4.1-nano-2025-04-14",
    "openai/gpt-5-chat-latest",
    "openai/o4-mini-2025-04-16",
    "qwen/qwen3-coder-480b-a35b-instruct",
    "qwen/qwen3-max-thinking",
    "qwen/qwen3-next-80b-a3b-thinking",
    "zai-org/glm-4.6",
  ],
  DEFAULT_MODEL: "moonshotai/kimi-k2-thinking",
};

// --- [第二部分: Worker 入口与路由] ---
// Cloudflare Worker 的主处理函数
export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    // 根据路径分发请求到不同的处理器
    if (url.pathname === '/') {
      return handleUI(request); // 处理根路径，返回开发者驾驶舱 UI
    } else if (url.pathname.startsWith('/v1/')) {
      return handleApi(request); // 处理 API 请求
    } else {
      // 对于所有其他路径，返回 404 Not Found
      return new Response(
        JSON.stringify({
          error: {
            message: `路径未找到: ${url.pathname}`,
            type: 'invalid_request_error',
            code: 'not_found'
          }
        }), {
          status: 404,
          headers: { 'Content-Type': 'application/json; charset=utf-8' }
        }
      );
    }
  }
};

// --- [第三部分: API 代理逻辑] ---
/**
 * 处理所有 /v1/ 路径下的 API 请求
 * @param {Request} request - 传入的请求对象
 * @returns {Promise<Response>} - 返回给客户端的响应
 */
async function handleApi(request) {
  // 预检请求处理：对于 OPTIONS 方法，直接返回 CORS 头部，允许跨域访问
  if (request.method === 'OPTIONS') {
    return handleCorsPreflight();
  }

  // 认证检查：验证 Authorization 头部
  const authHeader = request.headers.get('Authorization');
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return createErrorResponse('需要 Bearer Token 认证。', 401, 'unauthorized');
  }
  const token = authHeader.substring(7);
  if (token !== CONFIG.API_MASTER_KEY) {
    return createErrorResponse('无效的 API Key。', 403, 'invalid_api_key');
  }

  const url = new URL(request.url);
  const requestId = `chatcmpl-${crypto.randomUUID()}`;

  // 根据 API 路径执行不同操作
  if (url.pathname === '/v1/models') {
    return handleModelsRequest();
  } else if (url.pathname === '/v1/chat/completions') {
    return handleChatCompletions(request, requestId);
  } else {
    return createErrorResponse(`API 路径不支持: ${url.pathname}`, 404, 'not_found');
  }
}

/**
 * 处理 CORS 预检请求
 * @returns {Response}
 */
function handleCorsPreflight() {
  return new Response(null, {
    status: 204,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  });
}

/**
 * 创建标准化的 JSON 错误响应
 * @param {string} message - 错误信息
 * @param {number} status - HTTP 状态码
 * @param {string} code - 错误代码
 * @returns {Response}
 */
function createErrorResponse(message, status, code) {
  return new Response(JSON.stringify({
    error: {
      message,
      type: 'api_error',
      code
    }
  }), {
    status,
    headers: corsHeaders({
      'Content-Type': 'application/json; charset=utf-8'
    })
  });
}

/**
 * 处理 /v1/models 请求
 * @returns {Response}
 */
function handleModelsRequest() {
  const modelsData = {
    object: 'list',
    data: CONFIG.MODELS.map(modelId => ({
      id: modelId,
      object: 'model',
      created: Math.floor(Date.now() / 1000),
      owned_by: 'umint-2api',
    })),
  };
  return new Response(JSON.stringify(modelsData), {
    headers: corsHeaders({
      'Content-Type': 'application/json; charset=utf-8'
    })
  });
}

/**
 * 处理 /v1/chat/completions 请求
 * @param {Request} request - 传入的请求对象
 * @param {string} requestId - 本次请求的唯一 ID
 * @returns {Promise<Response>}
 */
async function handleChatCompletions(request, requestId) {
  try {
    const requestData = await request.json();
    const upstreamPayload = transformRequestToUpstream(requestData);

    const upstreamResponse = await fetch(CONFIG.UPSTREAM_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': '*/*',
        'Origin': 'https://umint-ai.hf.space',
        'Referer': 'https://umint-ai.hf.space/',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36',
        'X-Request-ID': requestId, // 请求水印
      },
      body: JSON.stringify(upstreamPayload),
    });

    if (!upstreamResponse.ok) {
      const errorBody = await upstreamResponse.text();
      console.error(`上游服务错误: ${upstreamResponse.status}`, errorBody);
      return createErrorResponse(`上游服务返回错误 ${upstreamResponse.status}: ${errorBody}`, upstreamResponse.status, 'upstream_error');
    }

    // 检查是否为流式响应
    const contentType = upstreamResponse.headers.get('content-type');
    if (requestData.stream && contentType && contentType.includes('text/event-stream')) {
      // 创建转换流，将上游格式实时转换为 OpenAI 格式
      const transformStream = createUpstreamToOpenAIStream(requestId, requestData.model || CONFIG.DEFAULT_MODEL);
      const [pipedStream] = upstreamResponse.body.tee();

      return new Response(pipedStream.pipeThrough(transformStream), {
        headers: corsHeaders({
          'Content-Type': 'text/event-stream; charset=utf-8',
          'Cache-Control': 'no-cache',
          'Connection': 'keep-alive',
          'X-Worker-Trace-ID': requestId, // 响应水印
        }),
      });
    } else {
        // 处理非流式响应 (尽管此 API 主要是流式的，但作为健壮性措施)
        const responseData = await upstreamResponse.json();
        const openAIResponse = transformNonStreamResponse(responseData, requestId, requestData.model || CONFIG.DEFAULT_MODEL);
        return new Response(JSON.stringify(openAIResponse), {
            headers: corsHeaders({
                'Content-Type': 'application/json; charset=utf-8',
                'X-Worker-Trace-ID': requestId,
            }),
        });
    }

  } catch (e) {
    console.error('处理聊天请求时发生异常:', e);
    return createErrorResponse(`处理请求时发生内部错误: ${e.message}`, 500, 'internal_server_error');
  }
}

/**
 * 将 OpenAI 格式的请求体转换为上游服务所需的格式
 * @param {object} requestData - OpenAI 格式的请求数据
 * @returns {object} - 上游服务格式的载荷
 */
function transformRequestToUpstream(requestData) {
  const transformedMessages = requestData.messages.map(msg => ({
    id: `msg-${crypto.randomUUID().slice(0, 12)}`,
    role: msg.role,
    parts: [{
      type: 'text',
      text: msg.content
    }],
  }));

  return {
    tools: {},
    modelId: requestData.model || CONFIG.DEFAULT_MODEL,
    sessionId: `session_${Date.now()}_${crypto.randomUUID().slice(0, 8)}`,
    id: "DEFAULT_THREAD_ID",
    messages: transformedMessages,
    trigger: "submit-message",
  };
}

/**
 * 创建一个 TransformStream 用于将上游 SSE 流转换为 OpenAI 兼容格式
 * @param {string} requestId - 本次请求的唯一 ID
 * @param {string} model - 使用的模型名称
 * @returns {TransformStream}
 */
function createUpstreamToOpenAIStream(requestId, model) {
  const encoder = new TextEncoder();
  const decoder = new TextDecoder();
  let buffer = '';

  return new TransformStream({
    transform(chunk, controller) {
      buffer += decoder.decode(chunk, { stream: true });
      const lines = buffer.split('\n');
      buffer = lines.pop(); // 保留不完整的行

      for (const line of lines) {
        if (line.startsWith('data:')) {
          const dataStr = line.substring(5).trim();
          if (dataStr === '[DONE]') {
            // 上游的 [DONE] 信号，我们将在 flush 中发送我们自己的
            continue;
          }
          try {
            const data = JSON.parse(dataStr);
            if (data.type === 'text-delta' && typeof data.delta === 'string') {
              const openAIChunk = {
                id: requestId,
                object: 'chat.completion.chunk',
                created: Math.floor(Date.now() / 1000),
                model: model,
                choices: [{
                  index: 0,
                  delta: { content: data.delta },
                  finish_reason: null,
                }],
              };
              controller.enqueue(encoder.encode(`data: ${JSON.stringify(openAIChunk)}\n\n`));
            }
          } catch (e) {
            console.error('无法解析上游 SSE 数据块:', dataStr, e);
          }
        }
      }
    },
    flush(controller) {
      // 流结束时，发送最终的 [DONE] 块
      const finalChunk = {
        id: requestId,
        object: 'chat.completion.chunk',
        created: Math.floor(Date.now() / 1000),
        model: model,
        choices: [{
          index: 0,
          delta: {},
          finish_reason: 'stop',
        }],
      };
      controller.enqueue(encoder.encode(`data: ${JSON.stringify(finalChunk)}\n\n`));
      controller.enqueue(encoder.encode('data: [DONE]\n\n'));
    },
  });
}

/**
 * 转换非流式响应 (备用)
 */
function transformNonStreamResponse(responseData, requestId, model) {
    // 这是一个简化的实现，假设非流式响应的结构
    const content = responseData?.choices?.[0]?.message?.content || "";
    return {
        id: requestId,
        object: "chat.completion",
        created: Math.floor(Date.now() / 1000),
        model: model,
        choices: [{
            index: 0,
            message: {
                role: "assistant",
                content: content,
            },
            finish_reason: "stop",
        }],
        usage: {
            prompt_tokens: 0,
            completion_tokens: 0,
            total_tokens: 0,
        },
    };
}


/**
 * 辅助函数，为响应头添加 CORS 策略
 * @param {object} headers - 现有的响应头
 * @returns {object} - 包含 CORS 头的新对象
 */
function corsHeaders(headers = {}) {
  return {
    ...headers,
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type, Authorization',
  };
}

// --- [第四部分: 开发者驾驶舱 UI] ---
/**
 * 处理对根路径的请求，返回一个功能丰富的 HTML UI
 * @param {Request} request - 传入的请求对象
 * @returns {Response} - 包含完整 UI 的 HTML 响应
 */
function handleUI(request) {
  const origin = new URL(request.url).origin;
  // 使用模板字符串嵌入完整的 HTML, CSS, 和 JS
  const html = `<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>${CONFIG.PROJECT_NAME} - 开发者驾驶舱</title>
    <style>
      /* --- 全局样式与主题 --- */
      :root {
        --bg-color: #121212;
        --sidebar-bg: #1E1E1E;
        --main-bg: #121212;
        --border-color: #333333;
        --text-color: #E0E0E0;
        --text-secondary: #888888;
        --primary-color: #FFBF00; /* 琥珀色 */
        --primary-hover: #FFD700;
        --input-bg: #2A2A2A;
        --error-color: #CF6679;
        --success-color: #66BB6A;
        --font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, 'Helvetica Neue', Arial, sans-serif;
        --font-mono: 'Fira Code', 'Consolas', 'Monaco', monospace;
      }
      * { box-sizing: border-box; }
      body {
        font-family: var(--font-family);
        margin: 0;
        background-color: var(--bg-color);
        color: var(--text-color);
        font-size: 14px;
        display: flex;
        height: 100vh;
        overflow: hidden;
      }
      /* --- 骨架屏样式 --- */
      .skeleton {
        background-color: #2a2a2a;
        background-image: linear-gradient(90deg, #2a2a2a, #3a3a3a, #2a2a2a);
        background-size: 200% 100%;
        animation: skeleton-loading 1.5s infinite;
        border-radius: 4px;
      }
      @keyframes skeleton-loading {
        0% { background-position: 200% 0; }
        100% { background-position: -200% 0; }
      }
    </style>
</head>
<body>
    <!-- 主布局自定义元素 -->
    <main-layout></main-layout>

    <!-- 模板定义 -->
    <template id="main-layout-template">
      <style>
        .layout { display: flex; width: 100%; height: 100vh; }
        .sidebar { width: 380px; flex-shrink: 0; background-color: var(--sidebar-bg); border-right: 1px solid var(--border-color); padding: 20px; display: flex; flex-direction: column; overflow-y: auto; }
        .main-content { flex-grow: 1; display: flex; flex-direction: column; padding: 20px; overflow: hidden; }
        .header { display: flex; justify-content: space-between; align-items: center; padding-bottom: 15px; margin-bottom: 15px; border-bottom: 1px solid var(--border-color); }
        .header h1 { margin: 0; font-size: 20px; }
        .header .version { font-size: 12px; color: var(--text-secondary); margin-left: 8px; }
        .collapsible-section { margin-top: 20px; }
        .collapsible-section summary { cursor: pointer; font-weight: bold; margin-bottom: 10px; }
        @media (max-width: 768px) {
          .layout { flex-direction: column; }
          .sidebar { width: 100%; height: auto; border-right: none; border-bottom: 1px solid var(--border-color); }
        }
      </style>
      <div class="layout">
        <aside class="sidebar">
          <header class="header">
            <h1>${CONFIG.PROJECT_NAME}<span class="version">v${CONFIG.PROJECT_VERSION}</span></h1>
            <status-indicator></status-indicator>
          </header>
          <info-panel></info-panel>
          <details class="collapsible-section" open>
            <summary>⚙️ 主流客户端集成指南</summary>
            <client-guides></client-guides>
          </details>
        </aside>
        <main class="main-content">
          <live-terminal></live-terminal>
        </main>
      </div>
    </template>

    <template id="status-indicator-template">
      <style>
        .indicator { display: flex; align-items: center; gap: 8px; font-size: 12px; }
        .dot { width: 10px; height: 10px; border-radius: 50%; transition: background-color: 0.3s; }
        .dot.grey { background-color: #555; }
        .dot.yellow { background-color: #FFBF00; animation: pulse 2s infinite; }
        .dot.green { background-color: var(--success-color); }
        .dot.red { background-color: var(--error-color); }
        @keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(255, 191, 0, 0.4); } 70% { box-shadow: 0 0 0 10px rgba(255, 191, 0, 0); } 100% { box-shadow: 0 0 0 0 rgba(255, 191, 0, 0); } }
      </style>
      <div class="indicator">
        <div id="status-dot" class="dot grey"></div>
        <span id="status-text">正在初始化...</span>
      </div>
    </template>

    <template id="info-panel-template">
      <style>
        .panel { display: flex; flex-direction: column; gap: 12px; }
        .info-item { display: flex; flex-direction: column; }
        .info-item label { font-size: 12px; color: var(--text-secondary); margin-bottom: 4px; }
        .info-value { background-color: var(--input-bg); padding: 8px 12px; border-radius: 4px; font-family: var(--font-mono); font-size: 13px; color: var(--primary-color); display: flex; align-items: center; justify-content: space-between; word-break: break-all; }
        .info-value.password { -webkit-text-security: disc; }
        .info-value.visible { -webkit-text-security: none; }
        .actions { display: flex; gap: 8px; }
        .icon-btn { background: none; border: none; color: var(--text-secondary); cursor: pointer; padding: 2px; display: flex; align-items: center; }
        .icon-btn:hover { color: var(--text-color); }
        .icon-btn svg { width: 16px; height: 16px; }
        .skeleton { height: 34px; }
      </style>
      <div class="panel">
        <div class="info-item">
          <label>API 端点 (Endpoint)</label>
          <div id="api-url" class="info-value skeleton"></div>
        </div>
        <div class="info-item">
          <label>API 密钥 (Master Key)</label>
          <div id="api-key" class="info-value password skeleton"></div>
        </div>
        <div class="info-item">
          <label>默认模型 (Default Model)</label>
          <div id="default-model" class="info-value skeleton"></div>
        </div>
      </div>
    </template>

    <template id="client-guides-template">
       <style>
        .tabs { display: flex; border-bottom: 1px solid var(--border-color); }
        .tab { padding: 8px 12px; cursor: pointer; border: none; background: none; color: var(--text-secondary); }
        .tab.active { color: var(--primary-color); border-bottom: 2px solid var(--primary-color); }
        .content { padding: 15px 0; }
        pre { background-color: var(--input-bg); padding: 12px; border-radius: 4px; font-family: var(--font-mono); font-size: 12px; white-space: pre-wrap; word-break: break-all; position: relative; }
        .copy-code-btn { position: absolute; top: 8px; right: 8px; background: #444; border: 1px solid #555; color: #ccc; border-radius: 4px; cursor: pointer; }
        .copy-code-btn:hover { background: #555; }
       </style>
       <div>
         <div class="tabs"></div>
         <div class="content"></div>
       </div>
    </template>

    <template id="live-terminal-template">
      <style>
        .terminal { display: flex; flex-direction: column; height: 100%; background-color: var(--sidebar-bg); border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
        .output-window { flex-grow: 1; padding: 15px; overflow-y: auto; font-size: 14px; line-height: 1.6; }
        .output-window p { margin: 0 0 1em 0; }
        .output-window pre { background-color: #0d0d0d; padding: 1em; border-radius: 4px; white-space: pre-wrap; font-family: var(--font-mono); }
        .output-window .message { margin-bottom: 1em; }
        .output-window .message.user { color: var(--primary-color); font-weight: bold; }
        .output-window .message.assistant { color: var(--text-color); }
        .output-window .message.error { color: var(--error-color); }
        .input-area { border-top: 1px solid var(--border-color); padding: 15px; display: flex; gap: 10px; align-items: flex-end; }
        textarea { flex-grow: 1; background-color: var(--input-bg); border: 1px solid var(--border-color); border-radius: 4px; color: var(--text-color); padding: 10px; font-family: var(--font-family); font-size: 14px; resize: none; min-height: 40px; max-height: 200px; }
        .send-btn { background-color: var(--primary-color); color: #121212; border: none; border-radius: 4px; padding: 0 15px; height: 40px; font-weight: bold; cursor: pointer; display: flex; align-items: center; justify-content: center; }
        .send-btn:hover { background-color: var(--primary-hover); }
        .send-btn:disabled { background-color: #555; cursor: not-allowed; }
        .send-btn.cancel svg { width: 24px; height: 24px; }
        .send-btn svg { width: 20px; height: 20px; }
        .placeholder { color: var(--text-secondary); }
      </style>
      <div class="terminal">
        <div class="output-window">
          <p class="placeholder">实时交互终端已就绪。输入指令开始测试...</p>
        </div>
        <div class="input-area">
          <textarea id="prompt-input" rows="1" placeholder="输入您的指令..."></textarea>
          <button id="send-btn" class="send-btn" title="发送">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"><path d="M3.105 2.289a.75.75 0 00-.826.95l1.414 4.949a.75.75 0 00.95.544l3.239-1.281a.75.75 0 000-1.39L4.23 6.28a.75.75 0 00-.95-.545L1.865 3.45a.75.75 0 00.95-.826l.002-.007.002-.006zm.002 14.422a.75.75 0 00.95.826l1.415-2.28a.75.75 0 00-.545-.95l-3.239-1.28a.75.75 0 00-1.39 0l-1.28 3.239a.75.75 0 00.544.95l4.95 1.414zM12.75 8.5a.75.75 0 000 1.5h5.5a.75.75 0 000-1.5h-5.5z"/></svg>
          </button>
        </div>
      </div>
    </template>

    <script>
      // --- [第五部分: 客户端逻辑 (Developer Cockpit JS)] ---

      // --- 配置占位符 (由 Worker 动态注入) ---
      const CLIENT_CONFIG = {
          WORKER_ORIGIN: '__WORKER_ORIGIN__',
          API_MASTER_KEY: '__API_MASTER_KEY__',
          DEFAULT_MODEL: '__DEFAULT_MODEL__',
          MODEL_LIST_STRING: '__MODEL_LIST_STRING__',
          CUSTOM_MODELS_STRING: '__CUSTOM_MODELS_STRING__',
      };

      // --- 状态机 ---
      const AppState = {
        INITIALIZING: 'INITIALIZING',
        HEALTH_CHECKING: 'HEALTH_CHECKING',
        READY: 'READY',
        REQUESTING: 'REQUESTING',
        STREAMING: 'STREAMING',
        ERROR: 'ERROR',
      };
      let currentState = AppState.INITIALIZING;
      let abortController = null;

      // --- 基础组件 ---
      class BaseComponent extends HTMLElement {
        constructor(templateId) {
          super();
          this.attachShadow({ mode: 'open' });
          const template = document.getElementById(templateId);
          if (template) {
            this.shadowRoot.appendChild(template.content.cloneNode(true));
          }
        }
      }

      // --- 自定义元素定义 ---

      // 1. 主布局
      class MainLayout extends BaseComponent {
        constructor() { super('main-layout-template'); }
      }
      customElements.define('main-layout', MainLayout);

      // 2. 状态指示器
      class StatusIndicator extends BaseComponent {
        constructor() {
          super('status-indicator-template');
          this.dot = this.shadowRoot.getElementById('status-dot');
          this.text = this.shadowRoot.getElementById('status-text');
        }
        setState(state, message) {
          this.dot.className = 'dot'; // Reset
          switch (state) {
            case 'checking': this.dot.classList.add('yellow'); break;
            case 'ok': this.dot.classList.add('green'); break;
            case 'error': this.dot.classList.add('red'); break;
            default: this.dot.classList.add('grey');
          }
          this.text.textContent = message;
        }
      }
      customElements.define('status-indicator', StatusIndicator);

      // 3. 信息面板
      class InfoPanel extends BaseComponent {
        constructor() {
          super('info-panel-template');
          this.apiUrlEl = this.shadowRoot.getElementById('api-url');
          this.apiKeyEl = this.shadowRoot.getElementById('api-key');
          this.defaultModelEl = this.shadowRoot.getElementById('default-model');
        }
        connectedCallback() {
          this.render();
        }
        render() {
          const apiUrl = CLIENT_CONFIG.WORKER_ORIGIN + '/v1/chat/completions';
          const apiKey = CLIENT_CONFIG.API_MASTER_KEY;
          const defaultModel = CLIENT_CONFIG.DEFAULT_MODEL;

          this.populateField(this.apiUrlEl, apiUrl);
          this.populateField(this.apiKeyEl, apiKey, true);
          this.populateField(this.defaultModelEl, defaultModel);
        }
        populateField(element, value, isPassword = false) {
            element.classList.remove('skeleton');
            let content = '<span>' + value + '</span>' +
                '<div class="actions">' +
                    (isPassword ? '<button class="icon-btn" data-action="toggle-visibility" title="切换可见性">' +
                        '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"><path d="M10 12.5a2.5 2.5 0 100-5 2.5 2.5 0 000 5z" /><path fill-rule="evenodd" d="M.664 10.59a1.651 1.651 0 010-1.18l.88-1.473a1.65 1.65 0 012.899 0l.88 1.473a1.65 1.65 0 010 1.18l-.88 1.473a1.65 1.65 0 01-2.899 0l-.88-1.473zM18.45 10.59a1.651 1.651 0 010-1.18l.88-1.473a1.65 1.65 0 012.899 0l.88 1.473a1.65 1.65 0 010 1.18l-.88 1.473a1.65 1.65 0 01-2.899 0l-.88-1.473zM10 17a1.651 1.651 0 01-1.18 0l-1.473-.88a1.65 1.65 0 010-2.899l1.473-.88a1.651 1.651 0 011.18 0l1.473.88a1.65 1.65 0 010 2.899l-1.473.88a1.651 1.651 0 01-1.18 0z" clip-rule="evenodd" /></svg>' +
                    '</button>' : '') +
                    '<button class="icon-btn" data-action="copy" title="复制">' +
                        '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"><path d="M7 3.5A1.5 1.5 0 018.5 2h3.879a1.5 1.5 0 011.06.44l3.122 3.121A1.5 1.5 0 0117 6.621V16.5a1.5 1.5 0 01-1.5 1.5h-7A1.5 1.5 0 017 16.5v-13z" /><path d="M5 6.5A1.5 1.5 0 016.5 5h3.879a1.5 1.5 0 011.06.44l3.122 3.121A1.5 1.5 0 0115 9.621V14.5a1.5 1.5 0 01-1.5 1.5h-7A1.5 1.5 0 015 14.5v-8z" /></svg>' +
                    '</button>' +
                '</div>';
            element.innerHTML = content;
            element.querySelector('[data-action="copy"]').addEventListener('click', () => navigator.clipboard.writeText(value));
            if (isPassword) {
                element.querySelector('[data-action="toggle-visibility"]').addEventListener('click', () => element.classList.toggle('visible'));
            }
        }
      }
      customElements.define('info-panel', InfoPanel);

      // 4. 客户端集成指南
      class ClientGuides extends BaseComponent {
        constructor() {
          super('client-guides-template');
          this.tabsContainer = this.shadowRoot.querySelector('.tabs');
          this.contentContainer = this.shadowRoot.querySelector('.content');
        }
        connectedCallback() {
          const guides = {
            'cURL': this.getCurlGuide(),
            'Python': this.getPythonGuide(),
            'LobeChat': this.getLobeChatGuide(),
            'Next-Web': this.getNextWebGuide(),
          };

          Object.keys(guides).forEach((name, index) => {
            const tab = document.createElement('button');
            tab.className = 'tab';
            tab.textContent = name;
            if (index === 0) tab.classList.add('active');
            tab.addEventListener('click', () => this.switchTab(name, guides));
            this.tabsContainer.appendChild(tab);
          });
          this.switchTab(Object.keys(guides)[0], guides);
        }
        switchTab(name, guides) {
          this.tabsContainer.querySelector('.active')?.classList.remove('active');
          this.tabsContainer.querySelector('button:nth-child(' + (Object.keys(guides).indexOf(name) + 1) + ')').classList.add('active');
          this.contentContainer.innerHTML = guides[name];
          this.contentContainer.querySelector('.copy-code-btn')?.addEventListener('click', (e) => {
              const code = e.target.closest('pre').querySelector('code').innerText;
              navigator.clipboard.writeText(code);
          });
        }

        // --- 指南生成函数 (已使用模板字符串重构并修正) ---
        getCurlGuide() {
            return '<pre><button class="copy-code-btn">复制</button><code>curl --location \\'' + CLIENT_CONFIG.WORKER_ORIGIN + '/v1/chat/completions\\' \\\\ <br>--header \\'Content-Type: application/json\\' \\\\ <br>--header \\'Authorization: Bearer ' + CLIENT_CONFIG.API_MASTER_KEY + '\\' \\\\ <br>--data \\'{<br>    "model": "' + CLIENT_CONFIG.DEFAULT_MODEL + '",<br>    "messages": [<br>        {<br>            "role": "user",<br>            "content": "你好，你是什么模型？"<br>        }<br>    ],<br>    "stream": true<br>}\\'</code></pre>';
        }
        getPythonGuide() {
            return '<pre><button class="copy-code-btn">复制</button><code>import openai<br><br>client = openai.OpenAI(<br>    api_key="' + CLIENT_CONFIG.API_MASTER_KEY + '",<br>    base_url="' + CLIENT_CONFIG.WORKER_ORIGIN + '/v1"<br>)<br><br>stream = client.chat.completions.create(<br>    model="' + CLIENT_CONFIG.DEFAULT_MODEL + '",<br>    messages=[{"role": "user", "content": "你好"}],<br>    stream=True,<br>)<br><br>for chunk in stream:<br>    print(chunk.choices[0].delta.content or "", end="")</code></pre>';
        }
        getLobeChatGuide() {
            return '<p>在 LobeChat 设置中，找到 "语言模型" -> "OpenAI" 设置:</p><pre><button class="copy-code-btn">复制</button><code>API Key: ' + CLIENT_CONFIG.API_MASTER_KEY + '<br>API 地址: ' + CLIENT_CONFIG.WORKER_ORIGIN + '<br>模型列表: ' + CLIENT_CONFIG.MODEL_LIST_STRING + '</code></pre>';
        }
        getNextWebGuide() {
            return '<p>在 ChatGPT-Next-Web 部署时，设置以下环境变量:</p><pre><button class="copy-code-btn">复制</button><code>CODE=' + CLIENT_CONFIG.API_MASTER_KEY + '<br>BASE_URL=' + CLIENT_CONFIG.WORKER_ORIGIN + '<br>CUSTOM_MODELS=' + CLIENT_CONFIG.CUSTOM_MODELS_STRING + '</code></pre>';
        }
      }
      customElements.define('client-guides', ClientGuides);

      // 5. 实时终端
      class LiveTerminal extends BaseComponent {
        constructor() {
          super('live-terminal-template');
          this.outputWindow = this.shadowRoot.querySelector('.output-window');
          this.promptInput = this.shadowRoot.getElementById('prompt-input');
          this.sendBtn = this.shadowRoot.getElementById('send-btn');
          this.sendIcon = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"><path d="M3.105 2.289a.75.75 0 00-.826.95l1.414 4.949a.75.75 0 00.95.544l3.239-1.281a.75.75 0 000-1.39L4.23 6.28a.75.75 0 00-.95-.545L1.865 3.45a.75.75 0 00.95-.826l.002-.007.002-.006zm.002 14.422a.75.75 0 00.95.826l1.415-2.28a.75.75 0 00-.545-.95l-3.239-1.28a.75.75 0 00-1.39 0l-1.28 3.239a.75.75 0 00.544.95l4.95 1.414zM12.75 8.5a.75.75 0 000 1.5h5.5a.75.75 0 000-1.5h-5.5z"/></svg>';
          this.cancelIcon = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"><path d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.28 7.22a.75.75 0 00-1.06 1.06L8.94 10l-1.72 1.72a.75.75 0 101.06 1.06L10 11.06l1.72 1.72a.75.75 0 101.06-1.06L11.06 10l1.72-1.72a.75.75 0 00-1.06-1.06L10 8.94 8.28 7.22z" /></svg>';
        }
        connectedCallback() {
          this.sendBtn.addEventListener('click', () => this.handleSend());
          this.promptInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
              e.preventDefault();
              this.handleSend();
            }
          });
          this.promptInput.addEventListener('input', this.autoResize);
        }
        autoResize(event) {
            const textarea = event.target;
            textarea.style.height = 'auto';
            textarea.style.height = textarea.scrollHeight + 'px';
        }
        handleSend() {
          if (currentState === AppState.REQUESTING || currentState === AppState.STREAMING) {
            this.cancelStream();
          } else {
            this.startStream();
          }
        }
        addMessage(role, content) {
            const messageEl = document.createElement('div');
            messageEl.className = 'message ' + role;

            let safeContent = content
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;');

            const parts = safeContent.split(/(\`\`\`[\\s\\S]*?\`\`\`)/g);
            const finalHtml = parts.map((part, index) => {
                if (index % 2 === 1) { // This is a code block
                    const codeBlock = part.slice(3, -3);
                    const languageMatch = codeBlock.match(/^(\\w+)\\n/);
                    const language = languageMatch ? languageMatch[1] : '';
                    const codeContent = languageMatch ? codeBlock.substring(languageMatch[0].length) : codeBlock;
                    return '<pre><code class="language-' + language + '">' + codeContent.trim() + '</code></pre>';
                } else {
                    return part.replace(/\\n/g, '<br>');
                }
            }).join('');

            messageEl.innerHTML = finalHtml;
            this.outputWindow.appendChild(messageEl);
            this.outputWindow.scrollTop = this.outputWindow.scrollHeight;
            return messageEl;
        }
        async startStream() {
          const prompt = this.promptInput.value.trim();
          if (!prompt) return;

          setState(AppState.REQUESTING);
          this.outputWindow.innerHTML = ''; // 清空
          this.addMessage('user', prompt);
          const assistantMessageEl = this.addMessage('assistant', '▍');

          abortController = new AbortController();
          try {
            const response = await fetch(CLIENT_CONFIG.WORKER_ORIGIN + '/v1/chat/completions', {
              method: 'POST',
              headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer ' + CLIENT_CONFIG.API_MASTER_KEY,
              },
              body: JSON.stringify({
                model: CLIENT_CONFIG.DEFAULT_MODEL,
                messages: [{ role: 'user', content: prompt }],
                stream: true,
              }),
              signal: abortController.signal,
            });

            if (!response.ok) {
              const err = await response.json();
              throw new Error(err.error.message);
            }

            setState(AppState.STREAMING);
            const reader = response.body.getReader();
            const decoder = new TextDecoder();
            let fullContent = '';

            while (true) {
              const { done, value } = await reader.read();
              if (done) break;

              const chunk = decoder.decode(value);
              const lines = chunk.split('\\n').filter(line => line.startsWith('data:'));

              for (const line of lines) {
                const dataStr = line.substring(5).trim();
                if (dataStr === '[DONE]') {
                    assistantMessageEl.textContent = fullContent; // 移除光标
                    break;
                }
                try {
                  const data = JSON.parse(dataStr);
                  const delta = data.choices[0].delta.content;
                  if (delta) {
                    fullContent += delta;
                    assistantMessageEl.textContent = fullContent + '▍';
                    this.outputWindow.scrollTop = this.outputWindow.scrollHeight;
                  }
                } catch (e) {}
              }
            }
          } catch (e) {
            if (e.name !== 'AbortError') {
              this.addMessage('error', '请求失败: ' + e.message);
              setState(AppState.ERROR);
            }
          } finally {
            if (currentState !== AppState.ERROR) {
              setState(AppState.READY);
            }
          }
        }
        cancelStream() {
          if (abortController) {
            abortController.abort();
            abortController = null;
          }
          setState(AppState.READY);
        }
        updateButtonState(state) {
            if (state === AppState.REQUESTING || state === AppState.STREAMING) {
                this.sendBtn.innerHTML = this.cancelIcon;
                this.sendBtn.title = "取消";
                this.sendBtn.classList.add('cancel');
                this.sendBtn.disabled = false;
            } else {
                this.sendBtn.innerHTML = this.sendIcon;
                this.sendBtn.title = "发送";
                this.sendBtn.classList.remove('cancel');
                this.sendBtn.disabled = state !== AppState.READY;
            }
        }
      }
      customElements.define('live-terminal', LiveTerminal);

      // --- 全局状态管理与初始化 ---
      function setState(newState) {
        currentState = newState;
        const terminal = document.querySelector('live-terminal');
        if (terminal) {
            terminal.updateButtonState(newState);
        }
      }

      async function performHealthCheck() {
        const statusIndicator = document.querySelector('main-layout').shadowRoot.querySelector('status-indicator');
        statusIndicator.setState('checking', '检查上游服务...');
        try {
          const response = await fetch(CLIENT_CONFIG.WORKER_ORIGIN + '/v1/models', {
            headers: { 'Authorization': 'Bearer ' + CLIENT_CONFIG.API_MASTER_KEY }
          });
          if (response.ok) {
            statusIndicator.setState('ok', '服务运行正常');
            setState(AppState.READY);
          } else {
            const err = await response.json();
            throw new Error(err.error.message);
          }
        } catch (e) {
          statusIndicator.setState('error', '健康检查失败: ' + e.message);
          setState(AppState.ERROR);
        }
      }

      // --- 应用启动 ---
      document.addEventListener('DOMContentLoaded', () => {
        setState(AppState.INITIALIZING);
        // 确保自定义元素已定义
        customElements.whenDefined('main-layout').then(() => {
            performHealthCheck();
        });
      });

    </script>
</body>
</html>`;

  // --- 动态注入所有需要的配置到 HTML 字符串中 ---
  const finalHtml = html
    .replace(/__WORKER_ORIGIN__/g, origin)
    .replace(/__API_MASTER_KEY__/g, CONFIG.API_MASTER_KEY)
    .replace(/__DEFAULT_MODEL__/g, CONFIG.DEFAULT_MODEL)
    .replace(/__MODEL_LIST_STRING__/g, CONFIG.MODELS.join(', '))
    .replace(/__CUSTOM_MODELS_STRING__/g, CONFIG.MODELS.map(m => `+${m}`).join(','));

  return new Response(finalHtml, {
    headers: {
      'Content-Type': 'text/html; charset=utf-8',
    },
  });
}




```

## 使用效果：
<img width="2544" height="1280" alt="QQ_1762251384015" src="https://github.com/user-attachments/assets/ff9c1e78-a34b-4d24-92cd-c889bbc37b39" />








# 2025年11月4日 17:54:47（提示词1.0）：

```
角色扮演： 你是一位世界级的 Serverless 架构师与全栈开发专家，对用户体验和开发者效率有着极致的追求。你能够将任何后端 API 服务，一键转化为一个部署在 Cloudflare Workers 上的、自带交互式测试面板的“艺术品级”应用。

核心任务： 我将提供一个完整的 Python API 项目源代码。请你将其完整地、无损地迁移到一个单一的 Cloudflare Worker JavaScript 文件中。

最终交付物要求：

单一、可直接部署的 JS 文件：所有逻辑和配置都在这一个文件里。
一个“旗舰级”的交互式说明页面：当用户访问 Worker 的根路径 (/) 时，必须返回一个功能强大、信息全面的 HTML 页面。
对生成的“说明页面”的硬性要求：
动态 URL 填充：
页面中所有 API 路径都必须自动包含当前 Worker 的完整访问 URL。
“即用信息”板块：
创建一个 📋 即用信息 (Ready-to-Use Info) 板块。
清晰列出在第三方客户端中需要填写的三个核心信息：API 地址、API 密钥、默认模型名称。
所有信息都必须是直接打印的、可直接复制的值。
“完整接口路径”板块：
创建一个 🔌 完整接口路径 (Full API Endpoints) 板块。
列出所有可用的 API 接口及其请求方法（如 POST, GET）。
“开发者信息”板块 (强制要求)：
创建一个 🛠️ 开发者信息 (Developer Info) 板块，此板块必须存在。
清晰展示以下技术细节：
上游接口 (Upstream API): 打印出代码中实际请求的上游 URL。
项目模式 (Project Mode): 自动分析并标注，例如“伪流式代理 (Pseudo-Stream Proxy)”。
“在线 API 测试”板块 (核心功能)：
创建一个 🚀 在线 API 测试 (Live API Tester) 板块。
此板块必须包含一个完整的、可交互的 Web UI，允许用户直接在页面上测试 v1/chat/completions 接口。
UI 组件要求：
一个文本输入框，用于用户输入问题（Prompt）。
一个“发送”按钮。
一个结果显示区域（使用 <pre> 或类似标签），用于实时显示流式返回的 AI 回答。
功能要求：
页面内嵌的 JavaScript 必须能自动获取当前页面的 URL 和硬编码的 API Key。
点击“发送”按钮后，使用 fetch API 调用当前 Worker 自己的 /v1/chat/completions 接口。
能够正确处理并实时渲染 SSE (Server-Sent Events) 流式响应。
在发送请求时，显示“正在思考...”等加载状态。
美学与体验：
大量使用 Emoji 增强页面的可读性和趣味性。
使用清晰的板块划分和标题。
整个页面设计简洁、专业，响应式布局优先。
对生成的“Worker 代码”的硬性要求：
硬编码配置：所有配置项（API_KEY, MODEL 等）必须作为常量硬编码在代码顶部。
智能路由：fetch 函数必须能处理两种请求：
API 请求 (路径以 /v1/ 开头)：执行 API 代理逻辑。
页面请求 (路径为 /)：返回包含上述所有功能的完整 HTML 和内联 JavaScript。
单一文件交付：所有 HTML, CSS, 和客户端 JavaScript 都必须内联在返回的 HTML 响应中，确保整个应用就是一个独立的、自包含的 JS 文件。
现在，这是我的项目文件，请严格按照以上所有要求开始转换：
```

## 使用效果如下：
<img width="2529" height="1344" alt="QQ_1762250179009" src="https://github.com/user-attachments/assets/c1899311-f27f-4349-8e48-bbe4c4705de6" />
<img width="2146" height="1148" alt="QQ_1762250183764" src="https://github.com/user-attachments/assets/cac4d897-cd8c-498f-903a-9730b0d05bb4" />


