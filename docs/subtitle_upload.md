# 字幕上传指南

## 简介

本文档将详细介绍如何使用 bilibili-api 库上传字幕。

## 字幕上传功能

### Video.submit_subtitle()

上传视频字幕

**语法**：
```python
async def submit_subtitle(lan: str, data: dict, submit: bool, sign: bool, page_index: int | None = None, cid: int | None = None) -> dict:
```

**参数**：
| 参数名 | 类型 | 说明 |
|--------|------|------|
| lan | str | 字幕语言代码，参考 [https://s1.hdslb.com/bfs/subtitle/subtitle_lan.json](https://s1.hdslb.com/bfs/subtitle/subtitle_lan.json) |
| data | dict | 字幕数据，格式见下方说明 |
| submit | bool | 是否提交，不提交为草稿 |
| sign | bool | 是否署名 |
| page_index | int | 分 P 索引，与 cid 二选一 |
| cid | int | 分 P ID，与 page_index 二选一 |

**返回值**：
`dict` - API 调用返回结果

### Episode.submit_subtitle()

上传番剧剧集字幕

**语法**：
```python
async def submit_subtitle(lan: str, data: dict, submit: bool, sign: bool) -> dict:
```

**参数**：
| 参数名 | 类型 | 说明 |
|--------|------|------|
| lan | str | 字幕语言代码，参考 [https://s1.hdslb.com/bfs/subtitle/subtitle_lan.json](https://s1.hdslb.com/bfs/subtitle/subtitle_lan.json) |
| data | dict | 字幕数据，格式见下方说明 |
| submit | bool | 是否提交，不提交为草稿 |
| sign | bool | 是否署名 |

**返回值**：
`dict` - API 调用返回结果

## 字幕数据格式

### 基本格式
```json
{
  "font_size": 0.4,
  "font_color": "#FFFFFF",
  "background_alpha": 0.5,
  "background_color": "#9C27B0",
  "Stroke": "none",
  "body": [
    {
      "from": 0.0,
      "to": 3.0,
      "location": 2,
      "content": "这是第一行字幕"
    },
    {
      "from": 3.0,
      "to": 6.0,
      "location": 2,
      "content": "这是第二行字幕"
    }
  ]
}
```

### 字段说明
| 字段名 | 类型 | 说明 |
|--------|------|------|
| font_size | float | 字体大小，默认 0.4 |
| font_color | str | 字体颜色，默认 "#FFFFFF" |
| background_alpha | float | 背景不透明度，默认 0.5 |
| background_color | str | 背景颜色，默认 "#9C27B0" |
| Stroke | str | 描边效果，默认 "none" |
| body | list | 字幕内容列表 |
| body[].from | float | 字幕开始时间（秒） |
| body[].to | float | 字幕结束时间（秒） |
| body[].location | int | 字幕位置，默认 2 |
| body[].content | str | 字幕内容 |

## 字幕语言代码

字幕语言代码可通过 [https://s1.hdslb.com/bfs/subtitle/subtitle_lan.json](https://s1.hdslb.com/bfs/subtitle/subtitle_lan.json) 获取。

**常见语言代码**：
- 中文（简体）：zh-CN
- 中文（繁体）：zh-TW
- 英语：en
- 日语：ja
- 韩语：ko

## 使用示例

### 上传视频字幕
```python
import asyncio
from bilibili_api import Video, Credential

# 初始化 Credential
credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")

# 初始化 Video 对象
video = Video(bvid="BV1xx411c7mz", credential=credential)

async def main():
    # 字幕数据
    subtitle_data = {
        "font_size": 0.4,
        "font_color": "#FFFFFF",
        "background_alpha": 0.5,
        "background_color": "#9C27B0",
        "Stroke": "none",
        "body": [
            {
                "from": 0.0,
                "to": 3.0,
                "location": 2,
                "content": "欢迎观看本视频"
            },
            {
                "from": 3.0,
                "to": 6.0,
                "location": 2,
                "content": "这是第二行字幕"
            }
        ]
    }
    
    # 上传字幕作为草稿
    result = await video.submit_subtitle(
        lan="zh-CN",
        data=subtitle_data,
        submit=False,
        sign=True,
        page_index=0
    )
    print("上传结果：", result)

asyncio.run(main())
```

### 上传番剧字幕
```python
import asyncio
from bilibili_api import Episode, Credential

# 初始化 Credential
credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")

# 初始化 Episode 对象
episode = Episode(bvid="BV1xx411c7mz", episode_id=12345, credential=credential)

async def main():
    # 字幕数据
    subtitle_data = {
        "font_size": 0.4,
        "font_color": "#FFFFFF",
        "background_alpha": 0.5,
        "background_color": "#9C27B0",
        "Stroke": "none",
        "body": [
            {
                "from": 0.0,
                "to": 3.0,
                "location": 2,
                "content": "欢迎观看本番剧"
            }
        ]
    }
    
    # 上传字幕并提交
    result = await episode.submit_subtitle(
        lan="zh-CN",
        data=subtitle_data,
        submit=True,
        sign=True
    )
    print("上传结果：", result)

asyncio.run(main())
```

### 从 SRT 文件转换为字幕数据格式
```python
import re
import asyncio
from bilibili_api import Video, Credential

def srt_to_subtitle_data(srt_content):
    """
    将 SRT 格式字幕转换为 bilibili-api 所需的 JSON 格式
    
    Args:
        srt_content (str): SRT 格式的字幕内容
        
    Returns:
        dict: bilibili-api 所需的字幕数据格式
    """
    # 正则表达式匹配 SRT 格式
    pattern = r'(\d+)\n(\d{2}:\d{2}:\d{2},\d{3}) --> (\d{2}:\d{2}:\d{2},\d{3})\n(.+?)(?=\n\d+\n|\Z)'
    matches = re.findall(pattern, srt_content, re.DOTALL)
    
    body = []
    for match in matches:
        start_time = match[1]
        end_time = match[2]
        content = match[3].strip()
        
        # 转换时间格式：HH:MM:SS,mmm -> 秒数
        def time_to_seconds(time_str):
            h, m, s_ms = time_str.split(':')
            s, ms = s_ms.split(',')
            return int(h) * 3600 + int(m) * 60 + int(s) + int(ms) / 1000
        
        from_time = time_to_seconds(start_time)
        to_time = time_to_seconds(end_time)
        
        body.append({
            "from": from_time,
            "to": to_time,
            "location": 2,
            "content": content
        })
    
    return {
        "font_size": 0.4,
        "font_color": "#FFFFFF",
        "background_alpha": 0.5,
        "background_color": "#9C27B0",
        "Stroke": "none",
        "body": body
    }

async def upload_srt_as_subtitle():
    # 初始化 Credential
    credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")
    
    # 初始化 Video 对象
    video = Video(bvid="BV1xx411c7mz", credential=credential)
    
    # 读取 SRT 文件
    with open("subtitle.srt", "r", encoding="utf-8") as f:
        srt_content = f.read()
    
    # 转换为 bilibili-api 所需的格式
    subtitle_data = srt_to_subtitle_data(srt_content)
    
    # 上传字幕
    result = await video.submit_subtitle(
        lan="zh-CN",
        data=subtitle_data,
        submit=False,  # 先保存为草稿
        sign=True,
        page_index=0
    )
    print("上传结果：", result)

asyncio.run(upload_srt_as_subtitle())
```

## 注意事项

1. **认证要求**：上传字幕需要登录凭据（sessdata 和 bili_jct）
2. **语言代码**：确保使用正确的语言代码，否则会上传失败
3. **草稿与提交**：submit 参数为 False 时，字幕会保存在草稿箱中；为 True 时，会直接提交
4. **署名**：sign 参数为 True 时，会显示上传者的用户名
5. **分 P 视频**：需要指定 page_index 或 cid

## 常见问题

### Q: 为什么上传失败？
A: 可能的原因有：
- 未提供正确的认证信息
- 使用了错误的语言代码
- 字幕数据格式不正确
- 网络问题

### Q: 如何获取字幕语言代码？
A: 可通过访问 [https://s1.hdslb.com/bfs/subtitle/subtitle_lan.json](https://s1.hdslb.com/bfs/subtitle/subtitle_lan.json) 获取所有支持的语言代码。

### Q: 如何查看草稿箱中的字幕？
A: 登录 Bilibili 网站，在视频管理中心查看字幕草稿。

### Q: 如何处理字幕中的特殊字符？
A: 确保字幕内容使用 UTF-8 编码，特殊字符如引号、反斜杠等需要正确转义。

### Q: 可以一次上传多个字幕吗？
A: 目前 API 一次只能上传一个字幕文件，需要多次调用 submit_subtitle 方法上传不同语言的字幕。

## 相关功能

### 获取字幕信息
使用 `get_subtitle()` 方法可以获取视频或番剧的字幕信息：
```python
# 获取视频字幕
subtitle_info = await video.get_subtitle()

# 获取番剧字幕
subtitle_info = await episode.get_subtitle()
```

### 生成 ASS 字幕文件
使用 `make_ass_file_subtitle()` 方法可以生成 ASS 格式的字幕文件：
```python
from bilibili_api.ass import make_ass_file_subtitle

# 生成 ASS 字幕文件
await make_ass_file_subtitle(
    obj=video,
    page_index=0,
    out="subtitle.ass",
    lan_code="zh-CN",
    credential=credential
)
```