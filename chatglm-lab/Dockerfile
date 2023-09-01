FROM registry.cn-beijing.aliyuncs.com/agi/pai-base:v1

# WORKDIR 指定工作目录
WORKDIR /home/

# 业务代码
COPY openai-api.py openai-api.py
COPY requirements.txt requirements.txt
COPY torchserver-lab/ .

# 安装项目依赖的包
# RUN pip install -r requirements.txt
RUN pip install torchserve torch-model-archiver torch-workflow-archiver
RUN pip install transformers==4.27.4
RUN python load_and_save.py --model_name_or_path /home/chatglm2-6b --quantization_bit 4 --cache_dir ./model
RUN mkdir model_store
RUN cd model

## 打包文件
RUN torch-model-archiver --model-name chatglm2  --version 1.0  --serialized-file pytorch_model.bin  --handler ../chatglm_handler.py  --export-path ../model_store  --extra-files "config.json,configuration_chatglm.py,generation_config.json,modeling_chatglm.py,quantization.py,special_tokens_map.json,tokenization_chatglm.py,tokenizer_config.json,tokenizer.model"

RUN cd /home/

# RUN torchserve --start --model-store model_store --models chatglm2.mar --ts-config config.properties 


# EXPOSE 暴露 80 端口
EXPOSE 80
EXPOSE 8880

# CMD ["python3", "openai-api.py"]
