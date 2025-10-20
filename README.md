# planet-niyue-provider-api-docs

“音之域”星球第三方音源能力对接文档



## 对接指引

1. 用户在使用第三方音源时，需要向其提供以下配置信息：
   - `IFRAME_URL`: 第三方音源的客户端入口页面路径URL地址，用于加载iframe进行前端通信。例如：`https://example.com/entry.html`
2. 在实际使用中，以下文档内容中所提到的相对URL路径会拼接在`IFRAME_URL`的上一级URL后面
   <br>以服务器API的 校验资源信息 接口为例，`IFRAME_URL`为`https://example.com/niyue-provider/example.html`，实际请求的完整URL为`https://example.com/niyue-provider/server/res_validate`



## 权利担保与授权

1. 权利担保：音源提供方应确保其通过本开放能力提供的内容（包括但不限于音乐作品、录音制品、相关图文资料等）均已获得完整的著作权及相关邻接权授权，**特别是信息网络传播权**等重要财产性权利。音源提供方保证有权依照本协议约定进行授权和分发。
2. 授权范围：一旦音源内容通过本第三方接口提交并提供，即视为音源提供方向“一零次元”及其旗下所有产品（包括但不限于"次元·星海"等相关平台）授予一项**全球性、免版权金、不可撤回**的许可，允许一零次元通过信息网络或其他合法方式向公众提供该内容，包括但不限于复制、发行、表演及信息网络传播等权利。
3. 责任承担：音源提供方应独立承担其提供内容的版权合法性审查责任。如因提供的内容侵犯他人著作权或相关权利，导致一零次元及其关联产品产生任何争议、索赔或损失，音源提供方应负责解决并承担全部赔偿责任。
4. 版权事务联系：如有任何版权相关事宜，请联系：planet-niyue-provider at binrz.com



## 服务器API

用于音之域后端服务器与音源之间的通讯



### 鉴权

为防止服务器API被滥用，音之域后端服务器在请求音源服务器API时，会在header带上`X-NIYUE-PROVIDER-SECRET`供音源校验请求是否有真实的音之域服务器发起。Secret内容可`GET`请求`https://api.binrz.com/planet/niyue/provider_secret`获取，Secret根据请求者IP生成。



GET `https://api.binrz.com/planet/niyue/provider_secret`

```javascript
{
    "code": 0,
    "msg": "获取成功",
    "data": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
}
```



### 接口



#### 1. 校验资源信息

POST `/server/res_validate`



##### 请求体结构

```javascript
[
	{
        type: "music",								// 资源类型, 可用参数: music (音乐), album (专辑), artist (艺术家)
        id: "",										// 资源ID
	},
	{
        type: "album",
        id: "",										// 资源ID
	},
	{
        type: "artist",
        id: "",										// 资源ID
	},
	{
        type: "artist",
        id: "INVALID_ID",							// 资源ID
	}
]
```



##### 响应体结构

```javascript
[
    {
        name: "",									// 音乐名称
        album_id?: "",								// 音乐专辑ID, 只在有专辑信息时返回
        album_name?: "",							// 音乐专辑名称, 只在有专辑信息时返回
        artist_ids: [""],							// 音乐艺术家ID
        artist_names: [""],							// 音乐艺术家名称
        hash: ""									// 音乐元数据校验值
        fallback_url?: ""							// 资源回落URL, 向不使用当前音源API的用户提供外部页面跳转链接, 只在歌曲信息有效时返回
    },
    {
        result: false, 								// 专辑ID无效
        name: "",									// 专辑名称
        hash: ""									// 专辑元数据校验值
        fallback_url?: ""							// 资源回落URL, 向不使用当前音源API的用户提供外部页面跳转链接, 只在专辑信息有效时返回
        cover_thumbhash?: ""						// 专辑封面thumbhash, 只在专辑信息有效时返回
        cover_url: ""								// 专辑封面图片URL
    },
    {
        result: true, 								// 艺术家信息有效
        name: "",									// 艺术家名称
        hash: ""									// 艺术家元数据校验值
        fallback_url?: ""							// 资源回落URL, 向不使用当前音源API的用户提供外部页面跳转链接, 只在艺术家信息有效时返回
        avatar_thumbhash?: ""						// 艺术家头像thumbhash, 只在艺术家信息有效时返回
        avatar_url: ""								// 艺术家头像URL
    },
    null											// 资源无效
]
```





## 客户端API

用于音之域前端浏览器与音源之间的通讯



### 说明

音之域前端与音源的通讯采用iframe的postMessage机制实现



### 通用结构体定义



#### 请求

```javascript
{
    id: "",											// 请求ID，用于区分请求，每个请求ID对应一个响应
    type: "",										// 请求类型
    data: {}										// 请求参数, 由业务进行定义
}
```



#### 响应

```javascript
{
    req_id: "",										// 请求ID，用于区分请求，每个请求ID对应一个响应
    result: true,									// 响应结果, 成功为true, 失败为false
    data?: {},										// 响应内容, 由业务进行定义, 仅在成功时返回
	fallback?: {									// 响应失败回落, 仅在失败时返回
        type: "",									// 回落类型, 可用参数: none (无), message (提示), modal (对话框)
        
        // type: message
        message_text: "",							// 纯文本消息
        
        // type: modal
        modal_title: "",							// 对话框标题
        modal_text: "",								// 对话框文本
        modal_ok_text: "",							// 对话框确认按钮文本
        modal_ok_callback?: "",						// 对话框确认按钮回调，见接口-对话框回调, 可选
        modal_cancel_text: "",						// 对话框取消按钮文本
        modal_cancel_callback?: ""					// 对话框取消按钮回调，见接口-对话框回调, 可选
	}
}
```



### 初始化

在iframe加载完毕后，应主动向音之域前端通过postMessage发送以下消息；否则，音之域前端将认为iframe尚未加载完毕

```javascript
{
    req_id: "PROVIDER_IFRAME_READY",
    result: true,
    data: {}
}
```



### 接口



#### 1. 连通性测试

在用户添加音源API时，会进行连通性测试，只有成功响应测试请求，才能添加音源；否则，前端将提示用户添加失败



**请求**

type: `ping`

data: 

```javascript
{}
```



**响应**

data:

```javascript
{
    "pong": true									// 确认响应, true为成功, false为失败
}
```



#### 2. 对话框回调

当某次响应返回失败，并且回落类型为modal时，会把用户点击的按钮回调给音乐API（若对应选择的callback字段未定义，则不会回调）



**请求**

type: `model_response`

data: 

```javascript
{
    response: ""									// 用户点击的按钮对应callback内容
}
```



**响应**

data:

```javascript
{}
```



#### 3. 资源匹配

通过关键词，进行批量资源匹配



**请求**

type: `batch_music_match`

data: 

```javascript
{
    [
        "name": "",									// 音乐名称
        "album_name": "",							// 专辑名称
        "artist_names": "",							// 艺术家名称
        "postfix"?: ""								// 音乐标签后缀, 如: "Instrument", "Off Vocal", "Radio Edit"等, 可选
    ]
}
```



**响应**

data:

```javascript
{
    results: [										// 匹配结果
        {
            id: "",									// 资源ID
            name: "",								// 音乐名称
            album_id?: "",							// 音乐专辑ID, 只在有专辑信息时返回
            album_name?: "",						// 音乐专辑名称, 只在有专辑信息时返回
            album_cover_url?: "",					// 音乐专辑封面图片URL, 只在有专辑信息时返回
            artist_ids: [""],						// 音乐艺术家ID
            artist_names: [""]						// 音乐艺术家名称
            hash: ""								// 艺术家元数据校验值
        },
        null										// 匹配失败
    ],
}
```



#### 4. 搜索音乐
用户搜索音乐



**请求**

type: `search_music`

data: 

```javascript
{
    keyword: "",									// 搜索关键词
    count: "",										// 分页数量
    page: ""										// 分页页数, 从0开始
}
```



**响应**

data:

```javascript
{
	results: [										// 搜索结果
        {
            id: "",									// 资源ID
            name: "",								// 音乐名称
            album_id?: "",							// 音乐专辑ID, 只在有专辑信息时返回
            album_name?: "",						// 音乐专辑名称, 只在有专辑信息时返回
            album_cover_url?: "",					// 音乐专辑封面图片URL, 只在有专辑信息时返回
            artist_ids: [""],						// 音乐艺术家ID
            artist_names: [""]						// 音乐艺术家名称
        }
    ],
    page: {											// 分页
        more: true									// 还有更多数据
    }
}
```



#### 5. 资源链接获取
用户请求播放资源



**请求**

type: `get_play_url`

data: 

```javascript
{
    music_id: ""									// 音乐ID
}
```



**响应**

data:

```javascript
{
    play_url: "",									// 播放链接
    backup_play_urls?: [],							// 备用播放链接，可选，在主播放链接播放失败时，按顺序尝试
    lrc: ""											// lrc格式歌词
}
```



#### 6. LRC歌词获取
用户请求音乐歌词，如为双语歌词，应将同一句歌词的时间戳设置为相同时间，第一行是原文，第二行是翻译



**请求**

type: `get_lyric`

data: 

```javascript
{
    music_id: ""									// 音乐ID
}
```



**响应**

data:

```javascript
{
    lrc: ""											// lrc格式歌词
}
```



### 静态资源管理



#### 缓存

次元·星海前端使用ServiceWorker实现本地缓存，为保证用户体验，前端会缓存路径中包含`/planet-niyue-provider-static/`的特定类型资源

- 图片类：`webp`, `avif`, `jpg`, `jpeg`, `gif`, `png`
- 音频类：`opus(webm容器)`, `webm`, `weba`, `mp3`, `m4a`, `flac`, `wav`, `aac`, `wma`



#### CORS跨域配置

第三方音源服务器需要配置 CORS 响应头，允许来自 `https://planet.binrz.com` 的跨域请求：

```
Access-Control-Allow-Origin: https://planet.binrz.com
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: cross-origin
```

如果音频/图片资源存储在单独的域名（如 CDN），该域名也需要配置相同的 CORS 响应头。

