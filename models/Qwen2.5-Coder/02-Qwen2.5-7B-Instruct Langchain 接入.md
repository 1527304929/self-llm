```

```

# Qwen2.5-7B-Instruct Langchain ����

## ����׼��  

���Ļ����������£�

```
----------------
ubuntu 22.04
python 3.12
cuda 12.1
pytorch 2.3.0
----------------
```

> ����Ĭ��ѧϰ���Ѱ�װ������ Pytorch(cuda) ��������δ��װ�����а�װ��

pip ��Դ�������ز���װ������

```shell
# ����pip
python -m pip install --upgrade pip
# ���� pypi Դ���ٿ�İ�װ
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

pip install transformers==4.46.2
pip install modelscope==1.20.0
pip install langchain==0.3.7
pip install accelerate==1.1.1
```



## ģ������

ʹ�� `modelscope` �е� `snapshot_download` ��������ģ�ͣ���һ������Ϊģ�����ƣ����� `cache_dir` Ϊģ�͵�����·����

���½� `model_download.py` �ļ��������������������ݣ�ճ�������ǵñ����ļ�������ͼ��ʾ�������� `python model_download.py` ִ�����أ�ģ�ʹ�СΪ 16 GB������ģ�ʹ����Ҫ 12 ���ӡ�

```python  
import torch
from modelscope import snapshot_download, AutoModel, AutoTokenizer
import os
model_dir = snapshot_download('Qwen/Qwen2.5-Coder-7B-Instruct', cache_dir='/root/autodl-tmp', revision='master')
```

> ע�⣺�ǵ��޸� `cache_dir` Ϊ���ģ������·��Ŷ~

## ����׼��

Ϊ��ݹ��� `LLM` Ӧ�ã�������Ҫ���ڱ��ز���� `Qwen2_5_Coder`���Զ���һ�� `LLM` �࣬�� `Qwen2.5-Coder` ���뵽 `LangChain` ����С�����Զ��� `LLM` ��֮�󣬿�������ȫһ�µķ�ʽ���� `LangChain` �Ľӿڣ������迼�ǵײ�ģ�͵��õĲ�һ�¡�

���ڱ��ز���� `Qwen2_5_Coder` �Զ��� `LLM` �ಢ�����ӣ�����ֻ��� `LangChain.llms.base.LLM` ��̳�һ�����࣬����д���캯���� `_call` �������ɣ�

�ڵ�ǰ·���½�һ�� `LLM.py` �ļ����������������ݣ�ճ�������ǵñ����ļ���

```python
from langchain.llms.base import LLM
from typing import Any, List, Optional
from langchain.callbacks.manager import CallbackManagerForLLMRun
from transformers import AutoTokenizer, AutoModelForCausalLM, GenerationConfig, LlamaTokenizerFast
import torch

class Qwen2_5_Coder(LLM):
    # ���ڱ��� Qwen2_5-Coder �Զ��� LLM ��
    tokenizer: AutoTokenizer = None
    model: AutoModelForCausalLM = None        
    def __init__(self, mode_name_or_path :str):

        super().__init__()
        print("���ڴӱ��ؼ���ģ��...")
        self.tokenizer = AutoTokenizer.from_pretrained(mode_name_or_path, use_fast=False)
        self.model = AutoModelForCausalLM.from_pretrained(mode_name_or_path, torch_dtype=torch.bfloat16, device_map="auto")
        self.model.generation_config = GenerationConfig.from_pretrained(mode_name_or_path)
        print("��ɱ���ģ�͵ļ���")
        
    def _call(self, prompt : str, stop: Optional[List[str]] = None,
                run_manager: Optional[CallbackManagerForLLMRun] = None,
                **kwargs: Any):

        messages = [{"role": "user", "content": prompt }]
        input_ids = self.tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
        model_inputs = self.tokenizer([input_ids], return_tensors="pt").to('cuda')
        generated_ids = self.model.generate(model_inputs.input_ids, attention_mask=model_inputs['attention_mask'], max_new_tokens=512)
        generated_ids = [
            output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
        ]
        response = self.tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]        
        return response
    @property
    def _llm_type(self) -> str:
        return "Qwen2_5_Coder"
```

�������ඨ���У����Ƿֱ���д�˹��캯���� `_call` ���������ڹ��캯���������ڶ���ʵ������һ��ʼ���ر��ز���� `Qwen2_5_Coder` `ģ�ͣ��Ӷ�����ÿһ�ε��ö���Ҫ���¼���ģ�ʹ�����ʱ�������_call` ������ `LLM` ��ĺ��ĺ�����`LangChain` ����øú��������� `LLM`���ڸú����У����ǵ�����ʵ����ģ�͵� `generate` �������Ӷ�ʵ�ֶ�ģ�͵ĵ��ò����ص��ý����

��������Ŀ�У����ǽ����������װΪ `LLM.py`��������ֱ�ӴӸ��ļ��������Զ���� LLM �ࡣ

## ����

Ȼ��Ϳ�����ʹ���κ�������langchain��ģ�͹���һ��ʹ���ˡ�

> ע�⣺�ǵ��޸�ģ��·��Ϊ���·��Ŷ~

```python
from LLM import Qwen2_5_Coder
llm = Qwen2_5_Coder(mode_name_or_path = "autodl-tmp/Qwen/Qwen2___5-Coder-7B-Instruct")
print(llm.invoke("����˭"))
```

������£�
![](E:\self-llm\models\Qwen2.5-Coder\images\02-1.png)

��Ȼ��Coderģ�ͣ���ȻҪ����������д����

```python
text = llm.invoke("Ϊ����pythonдһ���򵥵Ĳ�ȭС��Ϸ��������ʤ")
print(text)
```

������£�
![](E:\self-llm\models\Qwen2.5-Coder\images\02-2.png)
������������һ����δ��룺

�ɹ����У�