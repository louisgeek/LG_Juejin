# Android ASR 自动语音识别
- Automatic Speech Recognition 自动语音识别将声音转化为文字



## Kaldi
- Kaldi 是一个广泛使用的开源语音识别引擎，支持深度学习，基于 C++
- 通常需要训练模型
- 应用场景：学术研究、大型企业级应用

## Vosk
- Vosk 是基于 Kaldi 的轻量化语音识别库，提供了更简单的 API，特别适合于快速开发和部署，离线实时应用，目标是 “开箱即用”。
- 提供内置预训练模型，适合快速集成中文语音识别，无需训练模型
- 轻量化：在 Kaldi 基础上简化了解码流程，针对移动端优化内存和速度   
- 离线：支持离线语音识别
- 实时处理：提供流式语音识别，能够实时处理语音数据
- 应用场景：智能家居（离线语音控制）、移动应用（方言识别）、嵌入式设备（如车载语音助手）、离线笔记

- 写在 VoskTestService 服务里，System.loadLibrary 加载 kaldi_jni 库
org.kaldi.SpeechRecognizer#addListener 添加 RecognitionListener 监听
RecognitionListener#onPartialResult	语音输入过程中实时触发，实时返回部分、可能不准确的数据，比如用于实时显示字幕、动态更新识别结果  用户说“今天天气”，可能依次返回“今”→“今天”→“今天天”→“今天天气”
RecognitionListener#onResult	语音输入结束后触发，返回完整、准确、最终结果，比如用于语音转文字、命令识别 作为后续逻辑处理的依据  最终修正后的完整文本


## Pocketsphinx