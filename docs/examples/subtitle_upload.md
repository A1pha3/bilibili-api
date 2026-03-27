# 字幕上传示例

本文档提供了使用 bilibili-api 上传字幕的详细示例代码。

## 基本示例

### 上传视频字幕

```python
import asyncio
from bilibili_api import Video, Credential

async def upload_video_subtitle():
    """
    上传视频字幕的基本示例
    """
    # 初始化 Credential
    credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")
    
    # 初始化 Video 对象
    video = Video(bvid="BV1xx411c7mz", credential=credential)
    
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
            },
            {
                "from": 6.0,
                "to": 9.0,
                "location": 2,
                "content": "这是第三行字幕"
            }
        ]
    }
    
    # 上传字幕作为草稿
    result = await video.submit_subtitle(
        lan="zh-CN",
        data=subtitle_data,
        submit=False,  # 先保存为草稿
        sign=True,
        page_index=0
    )
    print("上传结果：", result)
    
    # 如果需要提交
    # result = await video.submit_subtitle(
    #     lan="zh-CN",
    #     data=subtitle_data,
    #     submit=True,  # 直接提交
    #     sign=True,
    #     page_index=0
    # )
    # print("提交结果：", result)

asyncio.run(upload_video_subtitle())
```

### 上传番剧字幕

```python
import asyncio
from bilibili_api import Episode, Credential

async def upload_episode_subtitle():
    """
    上传番剧剧集字幕的基本示例
    """
    # 初始化 Credential
    credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")
    
    # 初始化 Episode 对象
    episode = Episode(bvid="BV1xx411c7mz", episode_id=12345, credential=credential)
    
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
            },
            {
                "from": 3.0,
                "to": 6.0,
                "location": 2,
                "content": "这是第二行字幕"
            }
        ]
    }
    
    # 上传字幕并提交
    result = await episode.submit_subtitle(
        lan="zh-CN",
        data=subtitle_data,
        submit=True,  # 直接提交
        sign=True
    )
    print("上传结果：", result)

asyncio.run(upload_episode_subtitle())
```

## 高级示例

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
    """
    从 SRT 文件上传字幕的示例
    """
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

### 批量上传多语言字幕

```python
import asyncio
from bilibili_api import Video, Credential

async def upload_multiple_subtitles():
    """
    批量上传多语言字幕的示例
    """
    # 初始化 Credential
    credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")
    
    # 初始化 Video 对象
    video = Video(bvid="BV1xx411c7mz", credential=credential)
    
    # 多语言字幕数据
    subtitles = {
        "zh-CN": {  # 中文简体
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
        },
        "en": {  # 英语
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
                    "content": "Welcome to this video"
                },
                {
                    "from": 3.0,
                    "to": 6.0,
                    "location": 2,
                    "content": "This is the second subtitle"
                }
            ]
        },
        "ja": {  # 日语
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
                    "content": "この動画へようこそ"
                },
                {
                    "from": 3.0,
                    "to": 6.0,
                    "location": 2,
                    "content": "これは二つ目の字幕です"
                }
            ]
        }
    }
    
    # 上传各种语言的字幕
    for lan, data in subtitles.items():
        try:
            result = await video.submit_subtitle(
                lan=lan,
                data=data,
                submit=False,  # 先保存为草稿
                sign=True,
                page_index=0
            )
            print(f"上传 {lan} 字幕结果：", result)
        except Exception as e:
            print(f"上传 {lan} 字幕失败：", e)

asyncio.run(upload_multiple_subtitles())
```

### 自定义字幕样式

```python
import asyncio
from bilibili_api import Video, Credential

async def upload_styled_subtitle():
    """
    上传自定义样式字幕的示例
    """
    # 初始化 Credential
    credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")
    
    # 初始化 Video 对象
    video = Video(bvid="BV1xx411c7mz", credential=credential)
    
    # 自定义样式的字幕数据
    subtitle_data = {
        "font_size": 0.5,  # 增大字体
        "font_color": "#FFFF00",  # 黄色字体
        "background_alpha": 0.8,  # 更不透明的背景
        "background_color": "#0000FF",  # 蓝色背景
        "Stroke": "1px 1px 2px #000000",  # 黑色描边
        "body": [
            {
                "from": 0.0,
                "to": 3.0,
                "location": 2,  # 屏幕底部
                "content": "这是自定义样式的字幕"
            },
            {
                "from": 3.0,
                "to": 6.0,
                "location": 1,  # 屏幕顶部
                "content": "这是屏幕顶部的字幕"
            },
            {
                "from": 6.0,
                "to": 9.0,
                "location": 0,  # 屏幕中间
                "content": "这是屏幕中间的字幕"
            }
        ]
    }
    
    # 上传自定义样式的字幕
    result = await video.submit_subtitle(
        lan="zh-CN",
        data=subtitle_data,
        submit=False,
        sign=True,
        page_index=0
    )
    print("上传结果：", result)

asyncio.run(upload_styled_subtitle())
```

## 错误处理示例

```python
import asyncio
from bilibili_api import Video, Credential
from bilibili_api.exceptions import ArgsException, NetworkException

async def upload_subtitle_with_error_handling():
    """
    带错误处理的字幕上传示例
    """
    try:
        # 初始化 Credential
        credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")
        
        # 初始化 Video 对象
        video = Video(bvid="BV1xx411c7mz", credential=credential)
        
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
                }
            ]
        }
        
        # 上传字幕
        result = await video.submit_subtitle(
            lan="zh-CN",
            data=subtitle_data,
            submit=False,
            sign=True,
            page_index=0
        )
        print("上传成功：", result)
        
    except ArgsException as e:
        print("参数错误：", e)
    except NetworkException as e:
        print("网络错误：", e)
    except Exception as e:
        print("未知错误：", e)

asyncio.run(upload_subtitle_with_error_handling())
```

## 获取字幕信息示例

```python
import asyncio
from bilibili_api import Video, Credential

async def get_subtitle_info():
    """
    获取视频字幕信息的示例
    """
    # 初始化 Credential
    credential = Credential(sessdata="你的sessdata", bili_jct="你的bili_jct")
    
    # 初始化 Video 对象
    video = Video(bvid="BV1xx411c7mz", credential=credential)
    
    # 获取字幕信息
    subtitle_info = await video.get_subtitle()
    print("字幕信息：", subtitle_info)
    
    # 如果有字幕，可以进一步处理
    if subtitle_info and "subtitles" in subtitle_info:
        for subtitle in subtitle_info["subtitles"]:
            print(f"语言代码：{subtitle['lan']}")
            print(f"语言名称：{subtitle['lan_doc']}")
            print(f"字幕URL：{subtitle['subtitle_url']}")

asyncio.run(get_subtitle_info())
```