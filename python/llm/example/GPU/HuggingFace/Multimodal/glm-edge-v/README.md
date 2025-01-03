# GLM-Edge-V
In this directory, you will find examples on how you could apply IPEX-LLM INT4 optimizations on GLM-Edge-V models on [Intel GPUs](../../../README.md). For illustration purposes, we utilize the [THUDM/glm-edge-v-5b](https://huggingface.co/THUDM/glm-edge-v-5b) and [THUDM/glm-edge-v-2b](https://huggingface.co/THUDM/glm-edge-v-2b) (or [ZhipuAI/glm-edge-v-5b](https://www.modelscope.cn/models/ZhipuAI/glm-edge-v-5b) and [ZhipuAI/glm-edge-v-2b](https://www.modelscope.cn/models/ZhipuAI/glm-edge-v-2b) for ModelScope) as reference GLM-Edge-V models.

## 0. Requirements
To run these examples with IPEX-LLM on Intel GPUs, we have some recommended requirements for your machine, please refer to [here](../../../README.md#requirements) for more information.

## Example: Predict Tokens using `generate()` API
In the example [generate.py](./generate.py), we show a basic use case for a GLM-Edge-V model to predict the next N tokens using `generate()` API, with IPEX-LLM FP8 optimizations on Intel GPUs.
### 1. Install
#### 1.1 Installation on Linux
We suggest using conda to manage environment:
```bash
conda create -n llm python=3.11
conda activate llm
# below command will install intel_extension_for_pytorch==2.1.10+xpu as default
pip install --pre --upgrade ipex-llm[xpu] --extra-index-url https://pytorch-extension.intel.com/release-whl/stable/xpu/us/

# install packages required for GLM-Edge-V
pip install transformers==4.47.0
pip install accelerate==0.33.0
pip install "trl<0.12.0" 

# [optional] only needed if you would like to use ModelScope as model hub
pip install modelscope==1.11.0
```

#### 1.2 Installation on Windows
We suggest using conda to manage environment:
```bash
conda create -n llm python=3.11 libuv
conda activate llm

# below command will install intel_extension_for_pytorch==2.1.10+xpu as default
pip install --pre --upgrade ipex-llm[xpu] --extra-index-url https://pytorch-extension.intel.com/release-whl/stable/xpu/us/

# install packages required for GLM-Edge-V
pip install transformers==4.47.0
pip install accelerate==0.33.0
pip install "trl<0.12.0" 

# [optional] only needed if you would like to use ModelScope as model hub
pip install modelscope==1.11.0
```

### 2. Configures OneAPI environment variables for Linux

> [!NOTE]
> Skip this step if you are running on Windows.

This is a required step on Linux for APT or offline installed oneAPI. Skip this step for PIP-installed oneAPI.

```bash
source /opt/intel/oneapi/setvars.sh
```

### 3. Runtime Configurations
For optimal performance, it is recommended to set several environment variables. Please check out the suggestions based on your device.
#### 3.1 Configurations for Linux
<details>

<summary>For Intel Arc™ A-Series Graphics and Intel Data Center GPU Flex Series</summary>

```bash
export USE_XETLA=OFF
export SYCL_PI_LEVEL_ZERO_USE_IMMEDIATE_COMMANDLISTS=1
export SYCL_CACHE_PERSISTENT=1
```

</details>

<details>

<summary>For Intel Data Center GPU Max Series</summary>

```bash
export LD_PRELOAD=${LD_PRELOAD}:${CONDA_PREFIX}/lib/libtcmalloc.so
export SYCL_PI_LEVEL_ZERO_USE_IMMEDIATE_COMMANDLISTS=1
export SYCL_CACHE_PERSISTENT=1
export ENABLE_SDP_FUSION=1
```
> Note: Please note that `libtcmalloc.so` can be installed by `conda install -c conda-forge -y gperftools=2.10`.
</details>

<details>

<summary>For Intel iGPU</summary>

```bash
export SYCL_CACHE_PERSISTENT=1
```

</details>

#### 3.2 Configurations for Windows
<details>

<summary>For Intel iGPU and Intel Arc™ A-Series Graphics</summary>

```cmd
set SYCL_CACHE_PERSISTENT=1
```

</details>


> [!NOTE]
> For the first time that each model runs on Intel iGPU/Intel Arc™ A300-Series or Pro A60, it may take several minutes to compile.
### 4. Running examples

```bash
# for Hugging Face model hub
python ./generate.py --repo-id-or-model-path REPO_ID_OR_MODEL_PATH --prompt PROMPT --n-predict N_PREDICT --image-url-or-path IMAGE_URL_OR_PATH

# for ModelScope model hub
python ./generate.py --repo-id-or-model-path REPO_ID_OR_MODEL_PATH --prompt PROMPT --n-predict N_PREDICT --image-url-or-path IMAGE_URL_OR_PATH --modelscope
```

Arguments info:
- `--repo-id-or-model-path REPO_ID_OR_MODEL_PATH`: argument defining the **Hugging Face** (e.g. `THUDM/glm-edge-v-5b` or `THUDM/glm-edge-v-2b`) or **ModelScope** (e.g. `ZhipuAI/glm-edge-v-5b` or `ZhipuAI/glm-edge-v-2b`) repo id for the GLM-Edge-V model to be downloaded, or the path to the checkpoint folder. It is default to be `'THUDM/glm-edge-v-5b'` for **Hugging Face** or `'ZhipuAI/glm-edge-v-5b'` for **ModelScope**.
- `--image-url-or-path IMAGE_URL_OR_PATH`: argument defining the image to be infered. It is default to be `'http://farm6.staticflickr.com/5268/5602445367_3504763978_z.jpg'`.
- `--prompt PROMPT`: argument defining the prompt to be infered (with integrated prompt format for chat). It is default to be `'What is in the image?'`.
- `--n-predict N_PREDICT`: argument defining the max number of tokens to predict. It is default to be `32`.
- `--modelscope`: using **ModelScope** as model hub instead of **Hugging Face**.

#### Sample Output
#### [THUDM/glm-edge-v-2b](https://huggingface.co/THUDM/glm-edge-v-2b)

```log
Inference time: xxxx s
-------------------- Prompt --------------------
图片里有什么？
-------------------- Output --------------------
这张图片中，一个穿着粉色和白色条纹衣服的小女孩举着一只穿着粉色裙子的白色小熊玩偶。背景是砖墙
```

```log
Inference time: xxxx s
-------------------- Prompt --------------------
What is in the image?
-------------------- Output --------------------
A white teddy bear with pink accents is being held up by a person wearing a pink and white striped shirt. The bear is dressed in a
```

#### [THUDM/glm-edge-v-5b](https://huggingface.co/THUDM/glm-edge-v-5b)

```log
Inference time: xxxx s
-------------------- Prompt --------------------
图片里有什么？
-------------------- Output --------------------
这张图片展示了一个穿着红白条纹衣服的小女孩，她正在展示她手中的一个粉红色的白熊玩偶。背景是砖墙和一丛
```

```log
Inference time: xxxx s
-------------------- Prompt --------------------
What is in the image?
-------------------- Output --------------------
The image shows a young girl holding a small, white teddy bear. The girl is wearing a pink striped dress with a red bow at the neckline
```

The sample input image is (which is fetched from [COCO dataset](https://cocodataset.org/#explore?id=264959)):

<a href="http://farm6.staticflickr.com/5268/5602445367_3504763978_z.jpg"><img width=400px src="http://farm6.staticflickr.com/5268/5602445367_3504763978_z.jpg" ></a>