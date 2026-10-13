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

为防止服务器API被滥用，音之域后端服务器在请求音源服务器API时，会使用私钥对请求时间戳及关键参数生成签名，并放入`X-NIYUE-PROVIDER-SIGNATURE`请求头。

```
X-NIYUE-PROVIDER-SIGNATURE: RSA-SHA256(timestamp(Unix秒级时间戳) + ":" + 请求方IPv4地址)
```



音源服务器应该通过以下公钥验证签名，以确保请求真实来源于音之域服务器且未被篡改。

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCcLyPCagiAt4v4OnmwDp5hRl2Jji1WUQB/qhavky13pxVMln4Vo1PT+PpQYlKCF3jag4XhSJPKn6XmDvjPqB3XodSsdLU6Zvrom/UhR4qq2JAXRTmGv45LDjpDv08YwZfiTU3+nDb74Xe8+y9+TdmREeIpgbhubScLknD0FTQT5vmRI/919Llt56ArXD7VrIA7+vJzRuRP3g+PLnsrBrMT07C5qfijSm69e1cK6v5MPOmG/cFiKyRGTkxW/j5JF4Q17B1RRLyVKxndui3WTF18h774vqXVDAR53K96osz9kdZconZJIF1FETnPI8HoOa1EoMWrOMWM66Ypr9ae6i/J 
```



### 接口



#### 1. 校验资源信息

POST `/server/res_validate`

音之域后端用它获取资源的权威元数据。用户收藏或播放某首歌后，后端会在后台批量调用本接口：先校验歌曲，再用歌曲响应里的 `album_id` / `artist_ids` 校验专辑和歌手。



##### 调用约定

- 单次请求最多 50 条，音源必须支持这个规模
- 同一音源同一时刻最多 1 个请求在途，两次请求之间至少间隔 2 秒
- 音源应在 30 秒内响应；超时后，后端会自动减小下一次请求的条数。来不及处理的条目可以返回 `unavailable`，先把已完成的部分返回
- 音源可以返回 HTTP 429 并带上 `Retry-After` 响应头（秒），要求后端暂停一段时间再请求
- 请求体为空数组 `[]` 时应返回空数组 `[]`，后端添加音源时用它做连通性测试



##### 请求体结构

```javascript
[
	{
		type: "music",								// 资源类型, 可用参数: music (音乐), album (专辑), artist (艺术家)
		id: ""										// 资源ID
	},
	{
		type: "album",
		id: ""
	}
]
```



##### 响应体结构

响应为数组，**每一条都必须回显请求里的 `type` 与 `id`**。后端按 `(type, id)` 匹配结果，与顺序无关；响应里找不到的条目按 `unavailable` 处理。

```javascript
[
	{
		type: "music",
		id: "",
		result: true,								// 资源有效
		name: "",									// 音乐名称, result 为 true 时必须返回
		album_id?: "",								// 音乐专辑ID, 只在有专辑实体时返回; 返回了它, 后端会接着校验这张专辑
		album_name?: "",							// 音乐专辑名称, 没有专辑ID时也可以单独返回
		artist_ids?: [""],							// 音乐艺术家ID, 与 artist_names 按下标一一对应
		artist_names?: [""],						// 音乐艺术家名称
		duration_ms: 0,								// 音乐时长（毫秒）, 正整数, result 为 true 时必须返回
		fallback_url?: ""							// 资源回落URL, 向不使用当前音源的用户提供外部页面跳转链接
	},
	{
		type: "album",
		id: "",
		result: true,
		name: "",									// 专辑名称, result 为 true 时必须返回
		cover_url?: "",								// 专辑封面图片URL
		cover_thumbhash?: "",						// 专辑封面thumbhash
		fallback_url?: ""
	},
	{
		type: "artist",
		id: "",
		result: true,
		name: "",									// 艺术家名称, result 为 true 时必须返回
		avatar_url?: "",							// 艺术家头像URL
		avatar_thumbhash?: "",						// 艺术家头像thumbhash
		fallback_url?: ""
	},
	{
		type: "artist",
		id: "INVALID_ID",
		result: false,								// 资源无效
		reason?: "not_found"						// 原因, 可用参数: not_found (确认不存在), unavailable (这次查不了, 如上游出错、超时)
	}
]
```

- 除 `result` 为 true 时的 `name` 与 music 的 `duration_ms` 以外，其余字段都是可选的。不返回的字段，后端会继续使用前端提供的信息展示
- 拿不到时长的 music 请返回 `{ type, id, result: false, reason: "unavailable" }`，不要返回 `0` 或省略
- `artist_ids` 与 `artist_names` 的条数必须相同；对不上时只返回 `artist_names`
- 无效资源**不要返回 `null`**，请返回 `{ type, id, result: false, reason }`。没给 `reason` 按 `not_found` 处理。上游临时出错请返回 `unavailable`：后端会稍后重试，且不把这次计为失败
- `cover_url` / `avatar_url` 必须是 `https`（开发调试除外），并且图片服务器需要按下文「CORS跨域配置」返回跨域响应头，否则前端无法显示



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
    "pong": true,									// 确认响应, true为成功, false为失败
    "capabilities"?: [""]							// 支持的可选接口, 可选, 取值见下
}
```

`capabilities` 声明音源支持哪些可选接口，前端只对声明了的接口显示入口（如专辑名、歌手名的链接）。不返回或为空数组时，前端当作都不支持。目前的可选接口：

| 取值 | 接口 |
|---|---|
| `album_detail` | 7. 专辑详情 |
| `artist_detail` | 8. 歌手详情 |
| `artist_songs` | 9. 歌手歌曲 |



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
            artist_names: [""],						// 音乐艺术家名称
            duration_ms: 0,							// 音乐时长（毫秒）, 正整数, 必须返回
            hash: ""								// 艺术家元数据校验值
        },
        null										// 匹配失败
    ],
}
```

拿不到时长的结果按匹配失败返回 `null`。



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
            artist_names: [""],						// 音乐艺术家名称
            duration_ms: 0							// 音乐时长（毫秒）, 正整数, 必须返回
        }
    ],
    page: {											// 分页
        more: true									// 还有更多数据
    }
}
```

拿不到时长的结果请直接从 `results` 里去掉，`page.more` 仍按上游实际情况返回。



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
    duration_ms: 0,									// 音乐时长（毫秒），正整数，必须返回，前端用它判定有效播放
    lrc: ""											// lrc格式歌词
}
```

拿不到时长时请按获取失败返回（`result: false`），不要返回 `0` 或省略。



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



#### 7. 专辑详情
可选接口，需要在 `ping` 的 `capabilities` 里声明 `album_detail`。用户打开专辑页时请求，分页返回专辑内的曲目



**请求**

type: `album_detail`

data: 

```javascript
{
    album_id: "",									// 专辑ID, 来自音乐结果里的 album_id
    count: 20,										// 分页数量
    page: 0											// 分页页数, 从0开始
}
```



**响应**

data:

```javascript
{
    album: {										// 专辑信息, 每一页都返回
        id: "",										// 专辑ID
        name: "",									// 专辑名称
        cover_url?: ""								// 专辑封面图片URL, 可选
    },
    tracks: [										// 曲目, 按专辑内顺序, 结构同「4. 搜索音乐」的结果项
        {
            id: "",
            name: "",
            album_id?: "",
            album_name?: "",
            album_cover_url?: "",
            artist_ids: [""],
            artist_names: [""],
            duration_ms: 0
        }
    ],
    page: {											// 分页
        more: true									// 还有更多曲目
    }
}
```

专辑信息不包含艺术家，艺术家只在每首曲目上。专辑不存在时按失败返回（`result: false`）。专辑存在但没有可播放的曲目时，正常返回专辑信息，`tracks` 为空数组。

拿不到时长或无法播放的曲目请从 `tracks` 里去掉，`page.more` 仍按上游实际情况返回；前端以 `page.more` 判断是否还有下一页，不以单页条数判断。



#### 8. 歌手详情
可选接口，需要在 `ping` 的 `capabilities` 里声明 `artist_detail`。用户打开歌手页时请求



**请求**

type: `artist_detail`

data: 

```javascript
{
    artist_id: ""									// 艺术家ID, 来自音乐结果里的 artist_ids
}
```



**响应**

data:

```javascript
{
    artist: {
        id: "",										// 艺术家ID
        name: "",									// 艺术家名称
        avatar_url?: ""								// 艺术家头像图片URL, 可选
    }
}
```

艺术家不存在时按失败返回（`result: false`）。



#### 9. 歌手歌曲
可选接口，需要在 `ping` 的 `capabilities` 里声明 `artist_songs`。歌手页的歌曲列表，分页返回



**请求**

type: `artist_songs`

data: 

```javascript
{
    artist_id: "",									// 艺术家ID
    count: 20,										// 分页数量
    page: 0											// 分页页数, 从0开始
}
```



**响应**

data:

```javascript
{
    results: [],									// 歌曲, 结构同「4. 搜索音乐」的结果项
    page: {											// 分页
        more: true									// 还有更多歌曲
    }
}
```

没有可展示的歌曲时返回空数组，前端此时不显示歌曲区块。分页约定同「7. 专辑详情」。



### 静态资源管理



#### 缓存

次元·星海前端已实现本地缓存，为保证用户体验，请确保服务器支持Range请求，前端会缓存路径中包含`/planet-niyue-provider-static/`的以下类型资源

- 图片类：`webp`, `avif`, `jpg`, `jpeg`, `gif`, `png`
- 音频类：`webm(opus编码)`, `mp3`, `m4a`, `flac`, `wav`, `aac`



#### CORS跨域配置

第三方音源服务器需要配置 CORS 响应头，允许来自 `https://planet.binrz.com` 的跨域请求：

```
Access-Control-Allow-Origin: https://planet.binrz.com
Access-Control-Expose-Headers: Content-Range,Accept-Ranges,Content-Length
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: cross-origin
```

如果音频/图片资源存储在单独的域名（如 CDN），该域名也需要配置相同的 CORS 响应头。

