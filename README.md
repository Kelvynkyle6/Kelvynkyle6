{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "provenance": []
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "accelerator": "GPU"
  },
  "cells": [
    {
      "cell_type": "markdown",
      "source": [
        "# Credits for bubarino giving me the huggingface import code (感谢 bubarino 给了我 huggingface 导入代码)"
      ],
      "metadata": {
        "id": "himHYZmra7ix"
      }
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "e9b7iFV3dm1f"
      },
      "source": [
        "!git clone https://github.com/RVC-Boss/GPT-SoVITS.git\n",
        "%cd GPT-SoVITS\n",
        "!apt-get update && apt-get install -y --no-install-recommends tzdata ffmpeg libsox-dev parallel aria2 git git-lfs && git lfs install\n",
        "!pip install -r requirements.txt"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# @title Download pretrained models 下载预训练模型\n",
        "!mkdir -p /content/GPT-SoVITS/GPT_SoVITS/pretrained_models\n",
        "!mkdir -p /content/GPT-SoVITS/tools/damo_asr/models\n",
        "!mkdir -p /content/GPT-SoVITS/tools/uvr5\n",
        "%cd /content/GPT-SoVITS/GPT_SoVITS/pretrained_models\n",
        "!git clone https://huggingface.co/lj1995/GPT-SoVITS\n",
        "%cd /content/GPT-SoVITS/tools/damo_asr/models\n",
        "!git clone https://www.modelscope.cn/damo/speech_paraformer-large_asr_nat-zh-cn-16k-common-vocab8404-pytorch.git\n",
        "!git clone https://www.modelscope.cn/damo/speech_fsmn_vad_zh-cn-16k-common-pytorch.git\n",
        "!git clone https://www.modelscope.cn/damo/punc_ct-transformer_zh-cn-common-vocab272727-pytorch.git\n",
        "# @title UVR5 pretrains 安装uvr5模型\n",
        "%cd /content/GPT-SoVITS/tools/uvr5\n",
        "!git clone https://huggingface.co/Delik/uvr5_weights\n",
        "!git config core.sparseCheckout true\n",
        "!mv /content/GPT-SoVITS/GPT_SoVITS/pretrained_models/GPT-SoVITS/* /content/GPT-SoVITS/GPT_SoVITS/pretrained_models/"
      ],
      "metadata": {
        "id": "0NgxXg5sjv7z",
        "cellView": "form"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "#@title Create folder models 创建文件夹模型\n",
        "import os\n",
        "base_directory = \"/content/GPT-SoVITS\"\n",
        "folder_names = [\"SoVITS_weights\", \"GPT_weights\"]\n",
        "\n",
        "for folder_name in folder_names:\n",
        "  if os.path.exists(os.path.join(base_directory, folder_name)):\n",
        "    print(f\"The folder '{folder_name}' already exists. (文件夹'{folder_name}'已经存在。)\")\n",
        "  else:\n",
        "    os.makedirs(os.path.join(base_directory, folder_name))\n",
        "    print(f\"The folder '{folder_name}' was created successfully! (文件夹'{folder_name}'已成功创建！)\")\n",
        "\n",
        "print(\"All folders have been created. (所有文件夹均已创建。)\")"
      ],
      "metadata": {
        "cellView": "form",
        "id": "cPDEH-9czOJF"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "import requests\n",
        "import zipfile\n",
        "import shutil\n",
        "import os\n",
        "\n",
        "#@title Import model 导入模型 (HuggingFace)\n",
        "hf_link = 'https://huggingface.co/modelloosrvcc/Nagisa_Shingetsu_GPT-SoVITS/resolve/main/Nagisa.zip' #@param {type: \"string\"}\n",
        "\n",
        "output_path = '/content/'\n",
        "\n",
        "response = requests.get(hf_link)\n",
        "with open(output_path + 'file.zip', 'wb') as file:\n",
        "    file.write(response.content)\n",
        "\n",
        "with zipfile.ZipFile(output_path + 'file.zip', 'r') as zip_ref:\n",
        "    zip_ref.extractall(output_path)\n",
        "\n",
        "os.remove(output_path + \"file.zip\")\n",
        "\n",
        "source_directory = output_path\n",
        "SoVITS_destination_directory = '/content/GPT-SoVITS/SoVITS_weights'\n",
        "GPT_destination_directory = '/content/GPT-SoVITS/GPT_weights'\n",
        "\n",
        "for filename in os.listdir(source_directory):\n",
        "    if filename.endswith(\".pth\"):\n",
        "        source_path = os.path.join(source_directory, filename)\n",
        "        destination_path = os.path.join(SoVITS_destination_directory, filename)\n",
        "        shutil.move(source_path, destination_path)\n",
        "\n",
        "for filename in os.listdir(source_directory):\n",
        "    if filename.endswith(\".ckpt\"):\n",
        "        source_path = os.path.join(source_directory, filename)\n",
        "        destination_path = os.path.join(GPT_destination_directory, filename)\n",
        "        shutil.move(source_path, destination_path)\n",
        "\n",
        "print(f'Model downloaded. (模型已下载。)')"
      ],
      "metadata": {
        "cellView": "form",
        "id": "vbZY-LnM0tzq"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# @title launch WebUI 启动WebUI\n",
        "!/usr/local/bin/pip install ipykernel\n",
        "!sed -i '10s/False/True/' /content/GPT-SoVITS/config.py\n",
        "%cd /content/GPT-SoVITS/\n",
        "!/usr/local/bin/python  webui.py"
      ],
      "metadata": {
        "id": "4oRGUzkrk8C7",
        "cellView": "form"
      },
      "execution_count": null,
      "outputs": []
    }
  ]
}
