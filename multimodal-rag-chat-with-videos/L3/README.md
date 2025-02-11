# L3: 视频预处理用于多模态RAG (Preprocessing Videos for Multimodal RAG)

## 课程目标

本课程介绍如何预处理视频数据用于多模态RAG系统，包括:
1. 视频数据的获取和处理
2. 视频帧的提取和保存
3. 视频内容的转录处理
4. 元数据的生成和管理

## 一、环境配置

### 1.1 依赖库
```python
from pathlib import Path
import os
import cv2
import webvtt
import whisper
from moviepy.editor import VideoFileClip
from PIL import Image
import base64
```

### 1.2 工具函数
- str2time: 时间字符串转换
- maintain_aspect_ratio_resize: 保持宽高比的图像缩放

## 二、视频数据处理

### 2.1 视频获取
```python
# 从YouTube下载视频
vid1_url = "https://www.youtube.com/watch?v=7Hcg-rLYwdM"
vid1_filepath = download_video(vid1_url, vid1_dir)

# 从其他源下载视频
vid2_url = "https://multimedia-commons.s3-us-west-2.amazonaws.com/..."
vid2_filepath = urlretrieve(vid2_url, osp.join(vid2_dir, vid2_name))[0]
```

### 2.2 视频处理场景

1. 有字幕的视频:
   - 使用webvtt处理字幕文件
   - 提取关键帧和对应文本
   - 生成帧-文本对

2. 无字幕但有语音的视频:
   - 使用Whisper进行语音识别
   - 生成时间戳对齐的转录文本
   - 提取对应帧和文本

3. 无语音的视频:
   - 使用LVLM生成图像描述
   - 按固定间隔提取帧
   - 为每帧生成描述文本

## 三、帧提取与元数据生成

### 3.1 帧提取策略
- 有字幕/转录: 根据时间戳提取中间帧
- 无语音: 按固定帧率提取(如每秒1帧)
- 保持图像宽高比缩放到统一高度

### 3.2 元数据结构
```python
metadata = {
    'extracted_frame_path': img_fpath,  # 帧图像路径
    'transcript': text,                 # 对应文本
    'video_segment_id': idx,           # 片段ID
    'video_path': path_to_video,       # 源视频路径
    'mid_time_ms': mid_time_ms,        # 时间戳
}
```

## 四、具体实现

### 4.1 有字幕视频处理
```python
def extract_and_save_frames_and_metadata(
        path_to_video, 
        path_to_transcript, 
        path_to_save_extracted_frames,
        path_to_save_metadatas):
    # 加载视频和字幕
    video = cv2.VideoCapture(path_to_video)
    trans = webvtt.read(path_to_transcript)
    
    # 处理每个字幕片段
    for idx, transcript in enumerate(trans):
        # 提取中间帧和文本
        # 保存帧和元数据
```

### 4.2 使用Whisper处理
```python
def transcribe_video(video_path):
    # 加载Whisper模型
    model = whisper.load_model("base")
    # 生成转录文本
    result = model.transcribe(video_path)
    return result
```

### 4.3 使用LVLM处理
```python
def extract_and_save_frames_and_metadata_with_fps(
        path_to_video,  
        path_to_save_extracted_frames,
        path_to_save_metadatas,
        num_of_extracted_frames_per_second=1):
    # 按固定帧率提取帧
    # 使用LVLM生成描述
    # 保存帧和元数据
```

## 五、注意事项

1. 视频处理:
   - 确保视频格式支持
   - 处理视频读取错误
   - 合理设置帧提取间隔

2. 字幕/转录:
   - 处理多语言情况
   - 时间戳对齐
   - 文本清理和格式化

3. 性能考虑:
   - 大文件处理策略
   - 并行处理可能性
   - 存储空间管理

## 六、扩展应用

1. 可用于:
   - 视频检索系统
   - 内容推荐
   - 视频摘要生成
   - 多模态问答

2. 优化方向:
   - 关键帧选择算法
   - 文本摘要生成
   - 多模态特征融合

## 七、总结

本课程展示了:
1. 不同场景下的视频预处理方法
2. 帧提取和元数据生成的技术实现
3. 多种工具和模型的集成使用

这些预处理技术为构建多模态RAG系统提供了基础数据支持。 