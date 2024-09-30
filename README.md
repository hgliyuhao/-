### 接口名称: `connect_websocket`

### 接口描述:

该接口用于通过WebSocket协议与远程医疗聊天服务进行通信。用户输入提问后，系统通过WebSocket发送消息并实时接收响应，直到获取完整回答。WebSocket的响应流逐渐显示给用户，直到接收到结束信号。

### 请求 URL:

- **正式环境:** `wss://medicaluat.cailian.net/api/medical_chat_with_answer`

### 请求方法:

- WebSocket (`wss://` )

### 请求参数:

| 参数名称 | 类型 | 是否必填 | 描述 |
| --- | --- | --- | --- |
| `sessionid` | `str` | 是 | 会话ID，用于标识当前会话。 |
| `userid` | `str` | 是 | 用户ID，用于标识当前请求的用户。 |
| `input_text` | `str` | 是 | 用户输入的提问或问题。 |
| `subject` | `str` | 是 | 当前医学主题，为病理、内科、外科、生物化学、生理或通用 |
| `model_type` | `str` | 是 | 当前使用的模型类型，可选为`small`或`large`， |

### 请求示例:

```json
{
    "sessionid": "abc123",
    "userid": "liyuhao",
    "input_text": "请问如何治疗感冒？",
    "subject": "内科",
    "model_type": "small"
}

```

### 响应参数:

WebSocket 响应为一个消息流，每个响应都是一部分文本，直到接收到 `end` 消息为止。

| 参数名称 | 类型 | 描述 |
| --- | --- | --- |
| `response` | `str` | 医学聊天的分段响应，每次接收部分答案。 |
| `end` | `str` | 结束信号，指示响应已完成，WebSocket 关闭连接。 |

### 响应示例:

```
正在处理中，请稍候...
翻找知识库...
生成答案...
感冒通常为病毒感染引起，可以通过多喝水、休息和服用抗感冒药物来缓解症状。
end

```

### 异常处理:

1. **WebSocket 连接异常**
    - 当连接超时或者中断时，将捕获 `websockets.exceptions.ConnectionClosedError` 异常，并在界面上提示“网络问题，请重试”。
2. **其他异常**
    - 其他任何异常情况将捕获通用 `Exception`，并提示用户“网络问题，请刷新页面后重试”，同时输出详细错误信息到控制台。

### Vue 调用示例：

```jsx
<template>
  <div>
    <input v-model="inputText" placeholder="请输入问题" />
    <button @click="sendMessage">发送</button>
    <div v-html="responseMessage"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputText: "",
      responseMessage: "",
      websocket: null,
    };
  },
  methods: {
    async sendMessage() {
      if (!this.inputText) {
        alert("问题不能为空");
        return;
      }

      const sessionid = "abc123"; // 会话ID
      const userid = "liyuhao";   // 用户ID
      const subject = this.$store.state.subject; // 主题
      const modelType = this.$store.state.selectedModelType; // 模型类型

      const wsUrl = "wss://medicaluat.cailian.net/api/medical_chat_with_answer";
      this.websocket = new WebSocket(wsUrl);

      this.websocket.onopen = () => {
        const message = {
          sessionid: sessionid,
          userid: userid,
          input_text: this.inputText,
          subject: subject,
          model_type: modelType,
        };
        this.websocket.send(JSON.stringify(message));
      };

      this.websocket.onmessage = (event) => {
        if (event.data === "end") {
          this.websocket.close();
        } else {
          this.responseMessage += event.data + "<br>";
        }
      };

      this.websocket.onerror = (error) => {
        console.error("WebSocket 连接错误: ", error);
        alert("网络问题，请重试");
      };
    },
  },
};
</script>

```

### 注意事项:

- `input_text` 不能为空，确保请求参数的有效性。
- WebSocket 的响应可能包含占位符文字，如“正在处理中”，需过滤不必要的响应内容。
