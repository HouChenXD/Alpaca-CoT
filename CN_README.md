![](./figures/宣传0.png)
# 特制自己的ChatGPT: 多接口统一的轻量级LLM-IFT平台 (**Alpaca-CoT**)

您可以选择加入我们的群聊(WeChat)，和更多的同好研究者们交流。
<img src="./figures/wechat.jpg" width = "100" height = "100" div align=right />

## News
- 4.11: 添加了`多轮对话`功能，感谢[@paulcx](https://github.com/paulcx)。
- 4.9: 已收集和统一格式化数据集 `firefly`, `instruct`, `Code Alpaca` [这里](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main).
- 4.7: 添加了`参数合并`、`本地使用`、`批量预测`、`web服务`功能，感谢[@weberrr](https://github.com/weberrr)。
- 4.4: 已收集和统一格式化数据集`FastChat`, `GPTeacher`,`Guanaco`,`HC3`,`prosocial-dialog`, `belle-chat&belle-math`, `xP3` 和 `natural-instructions`。
- 4.3: 中文的CoT数据集`CoT_CN_data.json`已上传到[这里](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main).
- 4.1: 在instinwild-CN(47k) + belle(1.5M)微调得到的Bloom7b的checkpoint已被上传到了[这里](https://huggingface.co/QingyiSi/Alpaca-CoT/tree/main).
- 4.1: instnwild（收集自推特，主要是生成、开放式QA和mind-storm types）已经被统一格式化并收集.


## 0. ChatGPT背后的技术

**LLM**: （ **Large Language Models**）指经过大规模预训练且体量较大的语言模型，一般是transformer-based模型。

**IFT**: （ **Instruction Fine-Tuning**）指令微调，指令是指用户传入的目的明确的输入文本，指令微调用以让模型学会遵循用户的指令。

**CoT**: （ **Chain-of-Thought**）指令形式的一种特殊情况，包含step-by-step的推理过程。如下图蓝色部分所示。

![cot](./figures/cot.jpg)

## 1. 定位

ChatGPT的出现验证了大型语言模型(LLM)在通用人工智能(AGI)上的潜力。基于LLaMA[1]等Large Language Models(LLMs)的instruction-tuning研究(如，Alpaca[2])大幅度加速了复现ChatGPT的进程。**Alpaca-CoT**希望在这个研究方向上做出适度的贡献，以推进LLMs的开源进程、降低LLMs研究和使用成本。

具体来说，**Alpaca-CoT**项目旨在探究如何更好地通过instruction-tuning的方式来诱导LLM具备类似ChatGPT的交互和instruction-following能力。为此，我们广泛收集了不同类型的instruction（尤其是Chain-of-Thought数据集），并基于LLaMA给出了深入细致的实证研究，以供未来工作参考。据我们所知，我们是首个将CoT拓展进Alpaca的工作，因此简称为"**Alpaca-CoT**"。


热烈欢迎您向我们提供任何未被本项目收集的instruction-tuning及各类tasks数据集（或其来源）。我们将：
- 将这些数据收录并进行统一格式化处理；
- 用这些数据集instruct fine-tune LLaMA模型（未来将集成更多LLMs），并开源其checkpoint；
- 进行广泛的实证研究以探究新收录的数据集的作用。


我们希望我们的项目能够为大型语言模型的开源过程做出适度的贡献，并降低NLP研究人员上手LLM相关研究的门槛。

## 2. 概述

近期，[LLaMA](https://arxiv.org/abs/2302.13971v1)[1]显示出惊人的zero-shot和few-shot能力，仅需较少的参数即可和GPT-3.5性能相当（LLaMA-13B显著优于GPT-3（175B），LLaMA-65B与PaLM-540MB相当），明显降低了训练、微调和使用competitive大型语言模型的成本。最近，为了提高LLaMA的instruction-following能力，[Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca)[2]利用[self-instruct](https://arxiv.org/abs/2212.10560)[3]生成的52K Englishi nstruction-finetuning数据对LLaMA进行了微调。然而，目前该方向的研究仍然面临着以下三个挑战：
- LLaMA-7b依然对计算资源有着较高的要求；
- 用于instruction finetuning的开源数据集较少；
- 缺乏各instruction类型带来的影响的实证研究，如响应中文的能力和CoT能力。

为此，我们提出了Alpaca-CoT项目，该项目结合了相关的近期前沿技术，具有以下优势：
- 1. **_仅需要较低计算资源即可高效完成对LLaMA的微调_**。`7b`,`13b`和`30b`版本的LLaMA模型均可在单卡80G A100上轻松完成训练。该优势主要来自于[low-rank adaptation (LoRA)](https://arxiv.org/pdf/2106.09685.pdf) [4], [PEFT](https://github.com/huggingface/peft)和[bitsandbytes](https://github.com/TimDettmers/bitsandbytes)等技术。我们的代码主要修改自[这里](https://github.com/tloen/alpaca-lora)。
- 2. 我们发布的模型 **_显著提升了CoT(reasoning)推理能力_**。
- 3. 我们发布的模型 **_显著提升了对中文指令的响应能力_**。
- 4. 维护了一个仍在不断扩大规模的 **_[intruction-finetuning的数据集集合](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT)_**。该集合包含了中文、英文和CoT的instruction数据。同时，我们也维护了一个训练自各种instruction数据集的模型checkpoint集合。
- 5. 集成了 **_多种LLMs并统一了调用接口_**，可通过超参轻松切换。目前包含 **LLaMA, ChatGLM**[5]和 **Bloom**[6]，后续将持续加入更多,以供研究者们轻松调用和对比不同LLMs。
- 6. 提供了 **_详尽透彻的实证学习和定性分析_**，这里的findings可能会对促进未来LLM探索有一定的参考价值。



## 3. 数据集合 (Data Collection)  


收集数据集的相对大小如下图所示:

![img](./figures/show.png)


我们参考[这里](https://github.com/yaodongC/awesome-instruction-dataset) ([@yaodongC](https://github.com/yaodongC)), 将收集到的数据集按照以下规则标注Tags：

(Lang)Lingual-Tags:
- EN: Instruction datasets in English
- CN: Instruction datasets in Chinese
- ML: [Multi-lingual] Instruction datasets in multiple languages

(Task)Task-Tags:
- MT: [Multi-task] Datasets containing multiple tasks
- TS: [Task-specific] Datasets tailored for specific tasks

(Gen)Generation-method:
- HG: [Human Generated Dataset] Datasets created by humans
- SI: [Self-Instruct] Datasets generated using self-instruct methods
- MIX: [Mixed Dataset] Dataset contains both human and machine generated data
- COL: [Collection of Dataset] Dataset made from a collection of other datasets

### 数据统计
| 数据集                                                                          | 数目     | Lang         | Task      | Gen        | 类型                              | 来源                                      | 链接                                                                                      |
| :----------------------------------------------------------------------------- | :------- | :----------- | :-------- | :----------| :------------------------------- | :---------------------------------------- | :---------------------------------------------------------------------------------------- |
| [Chain of Thought](https://github.com/google-research/FLAN)                    | 74771    | EN/CN        | MT        | HG         | CoT相关任务                       | 人在现有数据集上标注CoT                    | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/Chain-of-Thought)    |
| [GPT4all](https://github.com/nomic-ai/gpt4all)                                 | 806199   | EN           | MT        | COL        | 代码，故事，对话                   | GPT-3.5-turbo 蒸馏                        | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/GPT4all)            |
| [GPTeacher](https://github.com/teknium1/GPTeacher)                             | 29013    | EN           | MT        | SI         | 通用，角色扮演，工具指令           | GPT-4 & toolformer                        | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/GPTeacher)          |
| [Guanaco](https://huggingface.co/datasets/JosephusCheung/GuanacoDataset)       | 534610   | ML           | MT        | SI         | 多种nlp任务                       | text-davinci-003                          | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/Guanaco)            |
| [HC3](https://huggingface.co/datasets/Hello-SimpleAI/HC3)                      | 37175    | EN/CN        | TS        | MIX        | 对话评估                          | gpt-3.5 或 人工                           | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/HC3)                 |
| [alpaca](https://github.com/tatsu-lab/stanford_alpaca)                         | 52002    | EN           | MT        | SI         | 通用指令                          | text-davinci-003                          | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/alpaca)              |
| [Natural Instructions](https://github.com/allenai/natural-instructions)        | 5040134  | ML           | MT        | COL        | 多种nlp任务                       | 人工标注的数据集的收集                     | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/Natural-Instructions) |
| [belle_cn](https://huggingface.co/BelleGroup)                                  | 1079517  | CN           | TS/MT     | SI         | 通用指令，数学推理，对话           | text-davunci-003                         | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/belle_cn)             |
| [instinwild](https://github.com/XueFuzhao/InstructionWild)                     | 52191    | EN/CN        | MT        | SI         | 生成，开放域问答，头脑风暴         | text-davunci-003                         | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/instinwild)           |
| [prosocial dialog](https://huggingface.co/datasets/allenai/prosocial-dialog)   | 165681   | EN           | TS        | MIX        | 对话                              | GPT-3改写问题，人工回复                   | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/prosocial-dialog)     |
| [finance_en](https://huggingface.co/datasets/gbharti/finance-alpaca)           | 68912    | EN           | TS        | COL        | 金融领域问答                      | GPT3.5                                    | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/)                     |
| [xP3](https://huggingface.co/datasets/bigscience/xP3)                          | 78883588 | ML           | MT        | COL        | 多种nlp任务                       | 人工标注的数据集的收集                     | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/xP3)                  |
| [firefly](https://github.com/yangjianxin1/Firefly)                             | 1649398  | CN           | MT        | COL        | 23种nlp任务                       | 收集中文数据集，人工书写指令模板           | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/firefly)              |
| [instruct](https://huggingface.co/datasets/swype/instruct)                     | 888969   | EN           | MT        | COL        | GPT4All，Alpaca和开源数据集的增强  | 使用AllenAI提供的nlp增强工具               | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/instruct)             |
| [Code Alpaca](https://github.com/sahil280114/codealpaca)                       | 20022    | EN           | SI        | SI         | 代码生成，编辑，优化               | text-davinci-003                          | [下载](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main/CodeAlpaca)            |
| 进行中 |
| [FastChat](https://github.com/lm-sys/FastChat)                                 |          | EN           | MT        | MIX        | 通用指令                          | 众包收集ChatGPT与人的交互 (ShareGPT)       |                                                                                           |
| [Galpaca](https://huggingface.co/GeorgiaTechResearchInstitute/galpaca-30b)     |          |              |           |            |                                  |                                           |                                                                                            |
| [Alpaca_GPT4](https://github.com/Instruction-Tuning-with-GPT-4/GPT-4-LLM)      |          |              |           |            |                                  |                                           |                                                                                            |
| [Auto CoT](https://github.com/amazon-science/auto-cot)                         |          |              |           |            |                                  |                                           |                                                                                            |

该集合仍在不断更新和扩增中。可在以下链接下载和查看更多数据细节：https://github.com/PhoebusSi/alpaca-CoT/tree/main/data



### 下载
你可以在[这里](https://huggingface.co/datasets/QingyiSi/Alpaca-CoT/tree/main)下载所有我们已经统一格式后的formatted数据。然后，将下载到的文件全部放到[data](https://github.com/PhoebusSi/alpaca-CoT/tree/main/data) folder。

你可以在[这里](https://huggingface.co/QingyiSi/Alpaca-CoT/tree/main)下载训练自各种类型instruction数据的所有checkponts。然后，在`gernerate.py`中的`LoRA_Weights`设置成下载路径，即可直接运行模型的inference以查看模型效果。

### 数据格式
我们集合中的所有数据均已被转化成相同的格式，每个样本的格式如下：
```
[
{"instruction": instruction string,
"input": input string, # (may be empty)
"output": output string}
]
```
注意，对于CoT数据集,我们首先使用FLAN提供的[template](https://github.com/google-research/FLAN/blob/main/flan/v2/templates.py)将其从原数据转化成Chain-of-Thought的形式，之后再统一成以上格式。格式统一化的脚本可以在[这里](https://github.com/PhoebusSi/alpaca-CoT/blob/main/data/origin_cot_data/formating.py)找到。 


## 4. 多接口统一的开源平台

### 环境配置
```
pip install -r requirements.txt
```
### 模型微调
为了便于研究者们在LLM上做系统的IFT研究，我们收集了不同类型的instruction数据，集成了多种LLM，并统一了接口，可以轻松定制化想要的搭配：
- `--model_type`: 设置想要研究的LLM，目前已支持[llama, chatglm和bloom]，其中后两者的中文能力较强，后续将会集成更多的LLMs。
- `--data`: 设置用以IFT的数据类型，以灵活特制想要的指令遵循能力，如追求较强的推理能力可设置alpaca-cot，较强的中文能力可设置belle1.5m，较强的coding和故事创作能力可设置gpt4all，金融相关的响应能力可设置finance。
- `--model_name_or_path`: 与`--model_type`相对应，用来加载目标LLM的不同型号权重。如，要加载llama的13b的模型权重时可设置decapoda-research/llama-13b-hf。 

**单卡**
- for LLaMA
```
python3 uniform_finetune.py --model_type llama --model_name_or_path decapoda-research/llama-7b-hf \
    --data alpaca-belle-cot --lora_target_modules q_proj v_proj \
    --per_gpu_train_batch_size 4 --learning_rate 3e-4 --epochs 1 
    
```
- for ChatGLM
```
python3 uniform_finetune.py   --model_type chatglm --model_name_or_path THUDM/chatglm-6b \
    --data alpaca-belle-cot --lora_target_modules query_key_value \
    --lora_r 32 --lora_alpha 32 --lora_dropout 0.1 --per_gpu_train_batch_size 2 \
    --learning_rate 2e-5 --epochs 1
```
Note that `load_in_8bit` is not yet suitable for ChatGLM, so batch_size must be much smaller than others. 

- for BLOOM
```
python3 uniform_finetune.py   --model_type bloom --model_name_or_path bigscience/bloomz-7b1-mt \
    --data alpaca-belle-cot --lora_target_modules query_key_value \
    --per_gpu_train_batch_size 4 --learning_rate 3e-4 --epochs 1 
```

Note that you can also pass the local path (where the LLM weights saved) to `--model_name_or_path`. And the data type `--data` can be freely set according to your interests.

**多卡**
- for LLaMA
```
python3 -m torch.distributed.launch --nproc_per_node 4  \
    --nnodes=1 --node_rank=0 --master_addr=xxx --master_port=yyy uniform_finetune.py \
    --model_type llama --model_name_or_path decapoda-research/llama-7b-hf \
    --data alpaca-belle-cot --lora_target_modules q_proj v_proj \
    --per_gpu_train_batch_size 4 --learning_rate 3e-4 --epochs 1 
```
- for ChatGLM
```
python3 -m torch.distributed.launch --nproc_per_node 4  \
    --nnodes=1 --node_rank=0 --master_addr=xxx --master_port=yyy \
    uniform_finetune.py   --model_type chatglm --model_name_or_path THUDM/chatglm-6b \
    --data alpaca-belle-cot --lora_target_modules query_key_value \
    --lora_r 32 --lora_alpha 32 --lora_dropout 0.1 --per_gpu_train_batch_size 2 \
    --learning_rate 2e-5 --epochs 1
```
Note that `load_in_8bit` is not yet suitable for ChatGLM, so batch_size must be much smaller than others. 

- for BLOOM
```
python3 -m torch.distributed.launch --nproc_per_node 4  \
    --nnodes=1 --node_rank=0 --master_addr=xxx --master_port=yyy \
    uniform_finetune.py   --model_type bloom --model_name_or_path bigscience/bloomz-7b1-mt \
    --data alpaca-belle-cot --lora_target_modules query_key_value \
    --per_gpu_train_batch_size 4 --learning_rate 3e-4 --epochs 1  
```


### 参数合并
```
python3 merge.py --model_type llama --size 7b --lora_dir xxx --merged_dir yyy
```
### 本地使用
```
python3 server.py --model_type chatglm --lora_dir xxx
```
### 批量预测
```
python3 predict.py --model_type chatglm --data for_dict_data --lora_dir xxx --result_dir yyy
```

### web服务

```
python3 web.py --model_type chatglm --lora_dir xxx
```

注意，`saved-xxx7b` 文件夹是保存LoRA weights的路径，而LLaMA的weights则会在脚本执行期间自动从Hugging Face上下载。

## 5. Quantitative Analysis
注意：下图是截止到3.26日收集到的数据集的统计情况，仅作为motivation展示。目前已收集了更多数据集，如金融相关的指令数据集。
![data collection statistics](https://github.com/PhoebusSi/alpaca-CoT/blob/main/figures/piechart.png)
当前的instruction-finetuning数据集合主要包含以下三个部分：
- `alpaca_data_cleaned.json`: about 52K English instruction-following training samples.
- `CoT_data.json`: 9 CoT datasets involving about 75k samples. （相关的CoT数据集由FLAN[7]发布）
- `belle_data_cn.json`:  about 0.5M Chinese |instruction-following training samples. （相关的中文instruction数据由BELLE[8]发布）

### 关于CoT和Chinese Instructions的消融
"w/o CoT" and "w/o CN" 分别表示用在instruction-finetuning期间不采用CoT数据和Chinese instructions。

需要推理能力的问题上的表现
![f3](https://github.com/PhoebusSi/alpaca-CoT/blob/main/figures/图3.png)
 
需要遵循中文指令的问题上的表现
![f4](https://github.com/PhoebusSi/alpaca-CoT/blob/main/figures/图4.png)
 
在较复杂问题上的表现
![f5](https://github.com/PhoebusSi/alpaca-CoT/blob/main/figures/图5.png)

          

**In summary, the models finetuned from our complete dataset (English, Chinese, and CoT instruction data) can significantly improve model reasoning and Chinese instruction following abilities.**


### 更多能力展示

![ablation-cot](https://github.com/PhoebusSi/alpaca-CoT/blob/main/figures/图6.png)


![ablation-cot](https://github.com/PhoebusSi/alpaca-CoT/blob/main/figures/图8.png)


## 6. 未来工作
- 集成进来更多的LLMs
- 探究模型的few-shot能力
- 对不同大小的模型进行细致探究
- 在instruction-following evaluation suite上评估
- 收集更多的instruction-finetuning数据集.



## 参考文献

[1]: [LLaMA: Open and Efficient Foundation Language Models](https://arxiv.org/abs/2302.13971v1)

[2]: [Stanford Alpaca: An Instruction-following LLaMA model](https://github.com/tatsu-lab/stanford_alpaca)

[3]: [Self-Instruct: Aligning Language Model with Self Generated Instructions](https://arxiv.org/abs/2212.10560)

[4]: [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/pdf/2106.09685.pdf)

[5]: [ChatGLM: An Open Bilingual Dialogue Language Model](https://github.com/THUDM/ChatGLM-6B)

[6]: [BLOOM: A 176B-Parameter Open-Access Multilingual Language Model](https://arxiv.org/abs/2211.05100)

[7]: [FLAN: Scaling Instruction-Finetuned Language Models](https://arxiv.org/abs/2210.11416)

[8]: [BELLE: Bloom-Enhanced Large Language model Engine](https://github.com/LianjiaTech/BELLE)

[9]: [GPT4All: Training an Assistant-style Chatbot with Large Scale Data Distillation from GPT-3.5-Turbo](https://github.com/nomic-ai/gpt4all)



## Citation
Please cite the repo if you use the data collection, code, and experimental findings in this repo. 
```
@misc{alpaca-cot,
  author = {Qingyi Si, Rui Liu, Zheng Lin },
  school = {Institute of Information Engineering, Chinese Academy of Sciences, Beijing, China},
  title = {Alpaca-CoT: An Instruction Fine-Tuning Platform with Instruction Data Collection and Unified Large Lnguage Models Interface},
  year = {2023},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/PhoebusSi/alpaca-CoT}},
}
```
For data, please cite the original Stanford Alpaca, BELLE and FLAN papers as well.

For models, please cite the original LLaMA, Stanford Alpaca, Self-Instruct and LoRA papers as well.

