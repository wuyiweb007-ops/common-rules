# API Specification · 看相第一印象分析 Agent

> 本文档定义 Web Agent 的 **最小可商用 API 接口规范**。
> 目标：稳定、可控、易扩展，服务于 7–14 天内的商业验证。

---

## 1. API 设计原则

1. **单任务接口**：一次请求 = 一次完整 Agent 执行
2. **无会话状态**：不依赖历史上下文
3. **结构化输入 / 输出**：便于前端与风控
4. **可降级**：异常时返回安全结果

---

## 2. 通用约定

### 2.1 Base URL

```
https://api.xxx.com
```

### 2.2 请求头

```http
Content-Type: application/json
Authorization: Bearer <optional_token>
```

---

## 3. 接口一览

| 接口         | 方法   | 说明         |
| ---------- | ---- | ---------- |
| /analyze   | POST | 单张照片第一印象分析 |
| /unlock    | POST | 解锁完整分析结果   |
| /compare   | POST | 双照片对比分析    |
| /share/:id | GET  | 获取分享内容     |
| /health    | GET  | 服务健康检查     |

---

## 4. POST /analyze

### 4.1 功能说明

* 执行 **基础第一印象分析**
* 返回部分结果（用于免费体验）

---

### 4.2 请求参数

```json
{
  "scene": "job_interview",
  "image": "base64_encoded_string"
}
```

#### 参数说明

| 字段    | 类型     | 必填 | 说明            |
| ----- | ------ | -- | ------------- |
| scene | string | 是  | 分析场景          |
| image | string | 是  | 图片 base64 编码字符串 |

---

### 4.3 响应示例（成功）

```json
{
  "analysis_id": "ana_123456",
  "scene": "job_interview",
  "summary": "在求职/面试场景下，这张照片给人的第一印象可能偏向稳重、克制，整体专业感较为明确。",
  "partial_suggestions": [
    "可以尝试轻微微笑，增强亲和感",
    "拍摄时略微前倾头部，拉近与观看者的心理距离"
  ]
}
```

---

## 5. POST /unlock

### 5.1 功能说明

* 解锁并返回 **完整分析结果**
* 默认前端已完成支付

---

### 5.2 请求参数

```json
{
  "analysis_id": "ana_123456",
  "payment_confirmed": true
}
```

#### 参数说明

| 字段                | 类型      | 必填 | 说明                    |
| ----------------- | ------- | -- | --------------------- |
| analysis_id       | string  | 是  | /analyze 返回的分析 ID     |
| payment_confirmed | boolean | 是  | 用户点击"已支付"后传 true |

---

### 5.3 响应示例

```json
{
  "analysis_id": "ana_123456",
  "share_id": "share_xyz789",
  "summary": "在求职/面试场景下，这张照片看起来可能给人一种理性、稳重但略显克制的第一印象。",
  "positive_signals": [
    "画面清晰，显得较为专业",
    "光线均匀，没有明显干扰"
  ],
  "potential_risks": [
    "表情偏中性，亲和力可能不够突出",
    "整体情绪表达较为克制"
  ],
  "suggestions": [
    "可以尝试轻微微笑，增强亲和感",
    "拍摄时略微前倾头部，拉近与观看者的心理距离",
    "注意控制拍摄距离，避免过远显得疏离",
    "确保背景简洁，突出个人形象",
    "建议在自然光下拍摄，提升整体质感"
  ]
}
```

> 解锁后自动生成 share_id 用于分享

---

## 6. POST /compare

### 6.1 功能说明

* 对两张照片进行 **场景适配度对比分析**
* 属于付费接口

---

### 6.2 请求参数

```json
{
  "scene": "dating",
  "image_a": "base64_encoded_string_a",
  "image_b": "base64_encoded_string_b"
}
```

#### 参数说明

| 字段      | 类型     | 必填 | 说明            |
| ------- | ------ | -- | ------------- |
| scene   | string | 是  | 分析场景          |
| image_a | string | 是  | 照片 A base64 编码 |
| image_b | string | 是  | 照片 B base64 编码 |

---

### 6.3 响应示例

```json
{
  "scene": "dating",
  "better_photo": "A",
  "reasons": [
    "照片 A 的表情更容易让人产生亲近感",
    "整体构图更放松，符合交友场景",
    "背景更简洁，减少视觉干扰"
  ],
  "suggestions": [
    "如果选择照片 B，可以适当增加表情变化",
    "注意避免过于正式的拍摄风格"
  ]
}
```

---

## 7. GET /share/:id

### 7.1 功能说明

* 获取分享的完整分析结果

---

### 7.2 请求参数

| 字段 | 类型     | 必填 | 说明      |
| -- | ------ | -- | ------- |
| id | string | 是  | 分享链接 ID |

---

### 7.3 响应示例

```json
{
  "share_id": "share_xyz789",
  "scene": "job_interview",
  "summary": "在求职/面试场景下，这张照片看起来可能给人一种理性、稳重但略显克制的第一印象。",
  "positive_signals": [
    "画面清晰，显得较为专业",
    "光线均匀，没有明显干扰"
  ],
  "potential_risks": [
    "表情偏中性，亲和力可能不够突出",
    "整体情绪表达较为克制"
  ],
  "suggestions": [
    "可以尝试轻微微笑，增强亲和感",
    "拍摄时略微前倾头部，拉近与观看者的心理距离",
    "注意控制拍摄距离，避免过远显得疏离",
    "确保背景简洁，突出个人形象",
    "建议在自然光下拍摄，提升整体质感"
  ]
}
```

---

## 8. GET /health

### 8.1 功能说明

* 服务可用性检测

### 8.2 响应

```json
{
  "status": "ok",
  "timestamp": 1730000000
}
```

---

## 9. 错误码规范

| code | 含义         |
| ---- | ---------- |
| 400  | 参数错误       |
| 401  | 未授权        |
| 429  | 请求过多       |
| 500  | Agent 执行失败 |

---

## 10. 错误响应示例

### 10.1 参数错误（400）

```json
{
  "error": "invalid_parameter",
  "message": "scene 必须是 job_interview、dating 或 business_profile 之一"
}
```

### 10.2 人脸检测失败（400）

```json
{
  "error": "no_face_detected",
  "message": "未检测到人脸，请重新上传"
}
```

### 10.3 分析超时（500）

```json
{
  "error": "analysis_timeout",
  "message": "分析超时，请稍后重试"
}
```

### 10.4 服务不可用（500）

```json
{
  "error": "service_unavailable",
  "message": "服务暂时不可用",
  "support_phone": "18655205007"
}
```

---

## 11. 降级策略

当发生以下情况：

* LLM 超时
* 输出违规
* 连续失败 3 次

系统应返回：

```json
{
  "error": "analysis_failed",
  "message": "当前图片信息有限，暂无法生成稳定分析结果，请联系客服",
  "support_phone": "18655205007"
}
```

---

## 12. API 版本策略

* 当前版本：v1
* 新增能力通过新接口或字段扩展
* 不破坏已有返回结构

---

**End of API Specification**
