# planet-niyue-provider-api-docs

“音之域”星球第三方音源能力对接文档



## 对接指引

1. 用户在使用第三方音源时，需要向其提供以下配置信息：
   - `BASE_URL`: 第三方音源的服务器API基础URL地址，用于服务端接口调用。例如：`https://example.com/niyue-provider`（注意：不要以`/`结尾）
   - `ENTRY_PATH`: 第三方音源的客户端页面入口路径，用于加载iframe进行前端通信。例如：`https://example.com/entry.html`
2. 在实际使用中，以下文档内容中所提到的相对URL路径会拼接在音源配置中的`BASE_URL`后，**务必留意**`BASE_URL`不要以`/`结尾
   <br>以服务器API的 校验资源信息 接口为例，`BASE_URL`为`https://example.com/niyue-provider`，实际请求的完整URL为`https://example.com/niyue-provider/server/res_validate`



## 权利担保与授权

1. 权利担保：音源提供方应确保其通过本开放能力提供的内容（包括但不限于音乐作品、录音制品、相关图文资料等）均已获得完整的著作权及相关邻接权授权，**特别是信息网络传播权**等重要财产性权利。音源提供方保证有权依照本协议约定进行授权和分发。
2. 授权范围：一旦音源内容通过本第三方接口提交并提供，即视为音源提供方向“一零次元”及其旗下所有产品（包括但不限于"次元·星海"等相关平台）授予一项**全球性、免版权金、不可撤回**的许可，允许一零次元通过信息网络或其他合法方式向公众提供该内容，包括但不限于复制、发行、表演及信息网络传播等权利。
3. 责任承担：音源提供方应独立承担其提供内容的版权合法性审查责任。如因提供的内容侵犯他人著作权或相关权利，导致一零次元及其关联产品产生任何争议、索赔或损失，音源提供方应负责解决并承担全部赔偿责任。
4. 版权事务联系：如有任何版权相关事宜，请联系：planet-niyue-provider at binrz.com



## 服务器API



### 接口



#### 1. 校验资源信息

POST `/server/res_validate`



##### 请求体结构

```javascript
[
	{
		type: "music",								// 资源类型, 可用参数: music (音乐), album (专辑), artist (艺术家)
		id: "",										// 资源ID
		name: "",									// 音乐名称
		album_id: "",								// 音乐专辑ID
		album_name: "",								// 音乐专辑名称
		artist_ids: [""],							// 音乐艺术家ID
		artist_names: [""]							// 音乐艺术家名称
	},
	{
		type: "album",
		id: "",										// 资源ID
        fallback_url: "",							// 资源回落URL, 向不使用当前音源API的用户提供外部页面跳转
		name: "",									// 专辑名称
        cover_url: ""								// 专辑封面图片URL
	},
	{
		type: "artist",
		id: "",										// 资源ID
        fallback_url: "",							// 资源回落URL, 向不使用当前音源API的用户提供外部页面跳转
		name: "",									// 艺术家名称
        avatar_url: ""								// 艺术家头像URL
	}
]
```



##### 响应体结构

```javascript
[
    {
        result: true, 								// 歌曲信息有效
        fallback_url?: ""							// 资源回落URL, 向不使用当前音源API的用户提供外部页面跳转链接, 只在歌曲信息有效时返回
    },
    {
        result: false, 								// 专辑信息无效
        fallback_url?: ""							// 资源回落URL, 向不使用当前音源API的用户提供外部页面跳转链接, 只在专辑信息有效时返回
        cover_thumbhash?: ""						// 专辑封面thumbhash, 只在专辑信息有效时返回
    },
    {
        result: true, 								// 艺术家信息有效
        fallback_url?: ""							// 资源回落URL, 向不使用当前音源API的用户提供外部页面跳转链接, 只在艺术家信息有效时返回
        avatar_thumbhash?: ""						// 艺术家头像thumbhash, 只在艺术家信息有效时返回
    }
]
```





## 客户端API



### 说明

音之域前端与音乐API的通讯采用iframe的postMessage机制实现



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
            album_id: "",							// 音乐专辑ID
            album_name: "",							// 音乐专辑名称
            artist_ids: [""],						// 音乐艺术家ID
            artist_names: [""]						// 音乐艺术家名称
        }
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
    page: ""										// 分页页数
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
            album_id: "",							// 音乐专辑ID
            album_name: "",							// 音乐专辑名称
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
    play_url: ""									// 音乐播放链接
}
```



### 静态资源管理



#### 缓存

次元·星海前端使用ServiceWorker实现本地缓存，为保证用户体验，前端会缓存路径中包含`/planet-niyue-provider-static/`的特定类型资源

- 图片类：`webp`, `avif`, `jpg`, `jpeg`, `gif`, `png`
- 音频类：`opus`, `webm`, `weba`, `ogg`, `mp3`, `m4a`, `flac`, `wav`



#### CORS跨域配置

第三方音源服务器需要配置 CORS 响应头，允许来自 `https://planet.binrz.com` 的跨域请求：

```
Access-Control-Allow-Origin: https://planet.binrz.com
```

如果音频/图片资源存储在单独的域名（如 CDN），该域名也需要配置相同的 CORS 响应头。

