<h1>InternLM3-8b-instruct部署</h1>

OpenAI o1于2024年9月13日正式发布，作为OpenAI最新发布的最强推理模型，标志着AI行业进入了一个新时代。
o1在测试化学、物理和生物学专业知识的基准GPQA-diamond上，全面超过了人类博士专家，OpenAI宣称“通用人工智能(AGI)之路，已经没有任何阻碍”。
不同于传统的语言模型，o1在回答之前会生成一个内部的思维链。这个思路链是一个逐步推导、逐步分解问题的过程，它模拟了人类思考的方式，
使得模型能够更深入地理解问题并给出更准确的答案。虽然深度思考会略微影响模型的回答速度，但准确率却有着显著提高，这使得许多研究者争先恐后的对o1进行“解剖”。

## 环境配置依赖

环境依赖如下：
```
----------------------
 Transformer >=4.48 
 Torch == 2.3.0     
 Cuda ==  12.1  
----------------------
```

 >本文默认学习者已安装好以上 Pytorch(cuda) 环境，如未安装请自行安装。

## 准备工作

首先 `pip` 换源加速下载并安装依赖包：

```shell
# 升级pip
python -m pip install --upgrade pip
pip install vllm
pip install modelscope
pip install streamlit
```
> 考虑到部分同学配置环境可能会遇到一些问题，我们在 AutoDL 平台准备了 InternLM3-8b-Instruct 的环境镜像，点击下方链接并直接创建 AutoDL 示例即可。
> ***https://www.codewithgpu.com/i/datawhalechina/self-llm/InternLM3-self-llm***

## 模型下载

`modelscope` 是一个模型管理和下载工具，支持从魔搭 (Modelscope) 等平台快速下载模型。

这里使用 `modelscope` 中的 `snapshot_download` 函数下载模型，第一个参数为模型名称，第二个参数 `cache_dir` 为模型的下载路径，第三个参数 `revision` 为模型的版本号。

在 `/root/autodl-tmp` 路径下新建 `model_download.py` 文件并在其中粘贴以下代码，并保存文件。

```python
from modelscope import snapshot_download, AutoModel, AutoTokenizer

model_dir = snapshot_download('Shanghai_AI_Laboratory/internlm3-8b-instruct', cache_dir='./', revision='master')
```

> 注意：记得修改 cache_dir 为你的模型下载路径哦~
在终端运行 `python /root/autodl-tmp/model_download.py` 执行下载，模型大小为 18GB 左右，下载模型大概需要5-30分钟。
<img src="https://github.com/riannyway/self-llm/blob/patch-1/models/InternLM3/images/o1.png?raw=true">

## 代码实现
```
import streamlit as st
from o1-like import generate_response
import json

def main():
    st.set_page_config(page_title="Internlm3-8b-instruct", page_icon="🧠", layout="wide")
    
    st.title("internlm3-8b-instruct 实现o1-like推理链")
    
    st.markdown("""
    [开源大模型食用指南 self-llm](https://github.com/datawhalechina/self-llm)
    """)
    
    # Text input for user query
    user_query = st.text_input("Enter your query:", placeholder="e.g., How many 'R's are in the word strawberry?")
    
    if user_query:
        st.write("Generating response...")
        
        # Create empty elements to hold the generated text and total time
        response_container = st.empty()
        time_container = st.empty()
        
        # Generate and display the response
        for steps, total_thinking_time in generate_response(user_query):
            with response_container.container():
                for i, (title, content, thinking_time) in enumerate(steps):
                    # Ensure content is a string
                    if not isinstance(content, str):
                        content = json.dumps(content)
                    if title.startswith("Final Answer"):
                        st.markdown(f"### {title}")
                        if '```' in content:
                            parts = content.split('```')
                            for index, part in enumerate(parts):
                                if index % 2 == 0:
                                    st.markdown(part)
                                else:
                                    if '\n' in part:
                                        lang_line, code = part.split('\n', 1)
                                        lang = lang_line.strip()
                                    else:
                                        lang = ''
                                        code = part
                                    st.code(part, language=lang)
                        else:
                            st.markdown(content.replace('\n', '<br>'), unsafe_allow_html=True)
                    else:
                        with st.expander(title, expanded=True):
                            st.markdown(content.replace('\n', '<br>'), unsafe_allow_html=True)
            
            # Only show total time when it's available at the end
            if total_thinking_time is not None:
                time_container.markdown(f"**Total thinking time: {total_thinking_time:.2f} seconds**")

if __name__ == "__main__":
    main()
```
要注意的是，学习者们需要自行在grouq(可访问grouq cloud，可能需要科学上网)并在powershell中配置临时变量api_key，具体代码如下：
```
export GROQ_API_KEY=gsk...
```
将gsk...替换为自己的api_key即可。

## 结果展示
<img src="https://github.com/riannyway/self-llm/blob/patch-1/models/InternLM3/images/o1-like.png?raw=true">
