# ERBench
## 二元任务
### 如何运行
#### binary/run_qa.py
- 描述
    
    该Python文件通过{task}_data/crafted文件夹中的数据触发数据预处理步骤并运行模型（GPT、Gemini、Llama、Mistral、Claude）。程序输出将保存在results文件夹下。

- 参数
    - task
        
        要测试的数据集
    - tasktype

        验证步骤或主要测试步骤
    - index
    
        大语言模型（LLM）API可能会因为超时或敏感内容错误等问题中止程序。用户可以将对应的索引添加到代码中以跳过该索引，并在模型参数中提供该索引以继续测试。
    - demo
        
        用于Few-shot（参数值：demo）或链式思维提示（参数值：cot）
    - rag
    
        用于检索增强生成（Wikipedia），若希望启用RAG，请将参数值设置为True
    - model
    
        要测试的模型

#### binary/error_analysis.py

此Python文件基于run_qa.py的输出日志文件返回数值分析结果。终端将显示四行输出，每行对应A、R、AR、H。每行的第一个数值为基础提示的值，第二个数值为反向提示的值。

- 参数
    - task
        
        要测试的数据集
    - demo
    
        用于Few-shot（参数值：demo）或链式思维提示（参数值：cot）
    - model
    
        要测试的模型
    - rag
    
        用于检索增强生成（Wikipedia），若希望启用RAG，请将参数值设置为True

#### binary/finetune_dataset.py
此Python文件创建GPT模型微调所需的数据集。
- 参数
    - n
    
        要使用的数据点数量
    
    若用户希望修改用于微调的数据集，请更改第313行的datasets参数。

#### finetune.ipynb
此Python文件基于finetune_dataset.py创建的数据集执行微调。

#### correctness_ver.py
此Python文件验证ERBench的正确性，并与人工分析和GPT-Judge进行比较。请注意，由于样本量较小（小于4），某些值在论文中被省略，实体解析问题则通过手动处理。

### 实验流程

#### 一般任务
```
python run_qa.py --model [MODEL] --task [TASK] --tasktype validate
python run_qa.py --model [MODEL] --task [TASK]
python error_analysis.py --model [MODEL] --task [TASK]
python correctness_ver.py --model [MODEL] --task [TASK]
```

#### 微调
```
python finetune_dataset.py --n [N]
run finetune.ipynb via ipynb kernel
python run_qa.py --model [FINETUNED_MODEL] --task [TASK] --tasktype validate
python run_qa.py --model [FINETUNED_MODEL] --task [TASK]
python error_analysis.py --model [FINETUNED_MODEL] --task [TASK]
```
流程：finetune_dataset.py -> finetune.ipynb -> run_qa.py -> error_analysis.py

### 多模态模型的图像任务（如Gemini Vision Pro）

图片链接：[多模态任务图片](https://drive.google.com/drive/folders/1WXCGCG4ZPzkV1qUjR2Z0IegzkPgSSKwl?usp=drive_link)

---

## 多选题（MC）任务

> ### 如何使用数据库生成多选题模板

(1) 基于数据库结构准备FD
    
*电影标题，发行年份* $\rightarrow$ *导演，时长，电影发行公司*

(2) 生成问题和对应的选项

- 将左侧属性值包含在问题中

    ```关于{movie title}这部于{year}年发行的电影，以下哪个选项是错误的？请提供解释。```

- 将每个右侧属性值转化为各个选项

    ```选项A：该片由{director}导演。```

    ```选项B：影片时长为{length}分钟。```

    ```选项C：该片由{movie publisher}出品。```

- 请注意，其中一个选项将使用**错误**的属性值（例如错误的导演姓名），这个选项即为该多选题的正确答案。请参考run_qa.py中的*run_qa*函数。

> ### 如何运行
#### (1) multi_choice/source/run_qa.py
- 描述

    此Python文件 (1) 预处理数据集和 (2) 运行问答/验证任务。
    
    (1) 预处理 

     使用dataset/crafted来重现论文中的结果。如果希望使用自有数据库，请定义预处理函数。

    (2) 运行任务
        
    - 问答

        使用LLMs运行主要问答任务。程序的输出（日志文件）将保存在results文件夹下。

        示例：

        ```python run_qa.py --task movie --model gpt35 --tasktype multiqa```

    - 验证
    
        使用LLMs运行验证任务。程序的输出（日志文件）将保存在dataset/validated文件夹下。

        示例：

        ```python run_qa.py –task movie –model gpt35 –tasktype validate```

- 参数
    -	task
        
        选择数据集（movie/soccer/airport/music/book）
    -	tasktype
        
        选择问答或验证任务（multiqa/validate）
    -	index
    
        实体ID以恢复代码。在代码运行失败时，指定索引以跳过之前的实体。
    -	mixed
    
        [0, 1)区间，用于设置“皆非选项”与正常选项的问题比例。例如，0.2表示20%的问题为“皆非选项”类型。
    -	random_seed
    
        随机种子以重现结果
    -	model
    
        选择模型（gpt35/gpt4/mistral/llama/gemini/claude/gemini_v/gpt_v）
    -	rag
    
        使用知识增强（RAG）运行问答。运行RAG模式前，需先运行普通类型问答。
    -	demo
    
        使用Few-shot演示运行问答。在demo模式运行前，确保dataset/demo中有演示样例。
	
#### (2) multi_choice/source/error_analysis.py
-	描述

    该Python文件需在run_qa.py运行后执行。此代码 (1) 处理验证日志文件并生成数据框（.csv），(2) 处理问答日志文件并生成数据框（.csv），(3) 根据处理后的数据框分析性能指标。

    (1) 处理验证日志文件

    确保已获得run_qa.py生成的验证日志文件。程序输出（csv文件）将保存在dataset/validated文件夹中。

    (2)	处理问答日志文件

    确保已获得run_qa.py生成的问答日志文件。程序输出（csv文件）将保存在results文件夹中。

    (3)	分析性能

    提供3种模式以分析问答输出的验证结果：(a) 所有问答对（无验证），(b) 仅考虑LLM已知的有效实体问答对，(c) 所有LLMs已知的有效实体问答对。我们将“有效实体”定义为“LLM已知的实体”。详情参见论文。

    ```
    python error_anlaysis.py --task movie --model gpt35 # 同时生成(a)、(b)、(c)类型结果
    python error_anlaysis.py --task movie --model gpt35 --only_val # 仅生成(b)类型结果
    python error_anlaysis.py --task movie --model gpt35 --only_common # 仅生成(c)类型结果
    ```

-	参数
    -   task
    
        选择数据集（movie/soccer/airport/music/book）
    -	mixed
    
        [0, 1)区间，用于设置“皆非选项”与正常选项的问题比例。例如，0.2表示20%的问题为“皆非选项”类型。
    -	model
    
        选择模型（gpt35/gpt4/mistral/llama/gemini/gemini_v）
    -	rag
    
        使用知识增强（RAG）运行问答。运行RAG模式前，需先运行普通类型问答。
    -	demo
        
        使用Few-shot演示运行问答。在demo模式运行前，确保dataset/demo中有演示样例。
    -	only_val
    
        仅保存(b)类型分析结果。
    -	only_common
    
        仅保存(c)类型分析结果。

> ### 实验流程
```
python run_qa.py --model [MODEL] --task [TASK] --tasktype validate
python run_qa.py --model [MODEL] --task [TASK]
python error_analysis.py --model [MODEL] --task [TASK]
```
