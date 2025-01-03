# C++ Example of running LLM on Intel NPU using IPEX-LLM
In this directory, you will find a C++ example on how to run LLM models on Intel NPUs using IPEX-LLM (leveraging *Intel NPU Acceleration Library*). See the table blow for verified models.

## Verified Models

| Model      | Model Link                                                    |
|------------|----------------------------------------------------------------|
| Qwen2 | [Qwen/Qwen2-7B-Instruct](https://huggingface.co/Qwen/Qwen2-7B-Instruct), [Qwen/Qwen2-1.5B-Instruct](https://huggingface.co/Qwen/Qwen2-1.5B-Instruct) |
| Qwen2.5 | [Qwen/Qwen2.5-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct), [Qwen/Qwen2.5-3B-Instruct](https://huggingface.co/Qwen/Qwen2.5-3B-Instruct) |
| Llama2 | [meta-llama/Llama-2-7b-chat-hf](https://huggingface.co/meta-llama/Llama-2-7b-chat-hf) |
| Llama3 | [meta-llama/Meta-Llama-3-8B-Instruct](https://huggingface.co/meta-llama/Meta-Llama-3-8B-Instruct) |
| MiniCPM | [openbmb/MiniCPM-1B-sft-bf16](https://huggingface.co/openbmb/MiniCPM-1B-sft-bf16), [openbmb/MiniCPM-2B-sft-bf16](https://huggingface.co/openbmb/MiniCPM-2B-sft-bf16) |
| Llama3.2 | [meta-llama/Llama-3.2-1B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct), [meta-llama/Llama-3.2-3B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct) |

Please refer to [Quick Start](../../../../../../../docs/mddocs/Quickstart/npu_quickstart.md#c-api) for details about verified platforms.

## 0. Prerequisites
For `ipex-llm` NPU support, please refer to [Quick Start](../../../../../../../docs/mddocs/Quickstart/npu_quickstart.md#install-prerequisites) for details about the required preparations.

## 1. Install & Runtime Configurations
### 1.1 Installation on Windows
We suggest using conda to manage environment:
```cmd
conda create -n llm python=3.11
conda activate llm

:: install ipex-llm with 'npu' option
pip install --pre --upgrade ipex-llm[npu]

:: [optional] for Llama-3.2-1B-Instruct & Llama-3.2-3B-Instruct
pip install transformers==4.45.0 accelerate==0.33.0
```

Please refer to [Quick Start](../../../../../../../docs/mddocs/Quickstart/npu_quickstart.md#install-prerequisites) for more details about `ipex-llm` installation on Intel NPU.

### 1.2 Runtime Configurations
Please refer to [Quick Start](../../../../../../../docs/mddocs/Quickstart/npu_quickstart.md#runtime-configurations) for environment variables setting based on your device.

## 2. Convert Model
We provide a [convert script](convert.py) under current directory, by running it, you can obtain the whole weights and configuration files which are required to run C++ example.

```cmd
:: to convert Qwen2.5-7B-Instruct
python convert.py --repo-id-or-model-path Qwen/Qwen2.5-7B-Instruct --save-directory <converted_model_path>

:: to convert Qwen2-1.5B-Instruct
python convert.py --repo-id-or-model-path Qwen/Qwen2-1.5B-Instruct --save-directory <converted_model_path>

:: to convert Qwen2.5-3B-Instruct
python convert.py --repo-id-or-model-path Qwen/Qwen2.5-3B-Instruct --save-directory <converted_model_path> --low_bit "sym_int8"

:: to convert Llama-2-7b-chat-hf
python convert.py --repo-id-or-model-path meta-llama/Llama-2-7b-chat-hf --save-directory <converted_model_path>

:: to convert Meta-Llama-3-8B-Instruct
python convert.py --repo-id-or-model-path meta-llama/Meta-Llama-3-8B-Instruct --save-directory <converted_model_path>

:: to convert MiniCPM-1B-sft-bf16
python convert.py --repo-id-or-model-path openbmb/MiniCPM-1B-sft-bf16 --save-directory <converted_model_path>

:: to convert MiniCPM-2B-sft-bf16
python convert.py --repo-id-or-model-path openbmb/MiniCPM-2B-sft-bf16 --save-directory <converted_model_path>

:: to convert Llama-3.2-1B-Instruct
python convert.py --repo-id-or-model-path meta-llama/Llama-3.2-1B-Instruct --save-directory <converted_model_path>

:: to convert Llama-3.2-3B-Instruct
python convert.py --repo-id-or-model-path meta-llama/Llama-3.2-3B-Instruct --save-directory <converted_model_path>
```

Arguments info:
- `--repo-id-or-model-path REPO_ID_OR_MODEL_PATH`: argument defining the huggingface repo id for the model (e.g. `Qwen/Qwen2.5-7B-Instruct`) to be downloaded, or the path to the huggingface checkpoint folder.
- `--save-directory SAVE_DIRECTORY`: argument defining the path to save converted model. If it is a non-existing path, the original pretrained model specified by `REPO_ID_OR_MODEL_PATH` will be loaded, and the converted model will be saved into `SAVE_DIRECTORY`.
- `--max-context-len MAX_CONTEXT_LEN`: Defines the maximum sequence length for both input and output tokens. It is default to be `1024`.
- `--max-prompt-len MAX_PROMPT_LEN`: Defines the maximum number of tokens that the input prompt can contain. It is default to be `960`.
- `--low_bit LOW_BIT`: Defines the low bit precision to quantize the model. It is default to be `sym_int4`.
- `--disable-transpose-value-cache`: Disable the optimization of transposing value cache.

## 3. Build C++ Example `llm-npu-cli`

You can run below cmake script in cmd to build `llm-npu-cli`, don't forget to replace below conda env dir with your own path.

```cmd
:: under current directory
:: please replace below conda env dir with your own path
set CONDA_ENV_DIR=C:\Users\arda\miniforge3\envs\llm\Lib\site-packages
mkdir build
cd build
cmake ..
cmake --build . --config Release -j
cd Release
```

## 4. Run `llm-npu-cli`

With built `llm-npu-cli`, you can run the example with specified paramaters. For example,

```cmd
# Run simple text completion
llm-npu-cli.exe -m <converted_model_path> -n 64 "AI是什么?"

# Run in conversation mode
llm-npu-cli.exe -m <converted_model_path> -cnv
```

Arguments info:
- `-m` : argument defining the path of saved converted model.
- `-cnv` : argument to enable conversation mode.
- `-n` : argument defining how many tokens will be generated.
- Last argument is your input prompt.

### 5. Sample Output
#### [`Qwen/Qwen2.5-7B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct)
##### Text Completion
```cmd
Input:
<|im_start|>system
You are a helpful assistant.<|im_end|>
<|im_start|>user
AI是什么?<|im_end|>
<|im_start|>assistant


Prefill 22 tokens cost xxxx ms.

Decode 63 tokens cost xxxx ms (avg xx.xx ms each token).
Output:
AI是"人工智能"的缩写，它是一门研究计算机如何能够完成与人类智能相关任务的学科，包括学习、推理、自我修正等能力。简而言之，人工智能就是让计算机模拟或执行人类智能行为的理论、技术和方法。

它涵盖了机器学习、深度学习、自然
```

##### Conversation
```cmd
User:你好
Assistant:你好！很高兴能为你提供帮助。有什么问题或需要聊天可以找我哦。
User:AI是什么?
Assistant: AI代表的是"Artificial Intelligence"，中文翻译为人工智能。它是指由计算机或信息技术实现的智能行为。广义的人工智能可以指任何表现出智能行为的计算机或软件系统。狭义的人工智能则指的是模拟、学习、推理、理解自然语言以及自我生成的人工智能系统。

简而言之，人工智能是一种利用计算机和机器来模仿、模拟或扩展人类智能的技术或系统。它包括机器学习、深度学习、自然语言处理等多个子领域。
User:exit
```

### Troubleshooting

#### Program crash with Chinese prompt
If you run CPP examples on Windows and find that your program raises below error when accepting Chinese prompts, you can search for `region` in the Windows search bar and go to `Region->Administrative->Change System locale..`, tick `Beta: Use Unicode UTF-8 for worldwide language support` option and then restart your computer.
```log
thread '<unnamed>' panicked at src\lib.rs:151:91:
called `Result::unwrap()` on an `Err` value: Utf8Error { valid_up_to: 77, error_len: Some(1) }
```

For detailed instructions on how to do this, see [this issue](https://github.com/intel-analytics/ipex-llm/issues/10989#issuecomment-2105598660).