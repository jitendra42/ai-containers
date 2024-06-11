# Hugging Face PyTorch Distributed Training

![Version: 0.2.1](https://img.shields.io/badge/Version-0.2.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.16.0](https://img.shields.io/badge/AppVersion-1.16.0-informational?style=flat-square)

This Helm chart deploys a distributed training job using the Kubeflow PyTorchJob training operator.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| dmsuehir | <dina.s.jones@intel.com> | <https://github.com/dmsuehir> |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| deploy.env.configMapName | string | `nil` |  |
| deploy.env.enabled | bool | `false` |  |
| distributed.benchmark.coresPerInstance | int | `-1` | When benchmarking is enabled, the number of CPU cores to use per instance |
| distributed.benchmark.iterations | int | `300` | When benchmarking is enabled, the number of iterations to run performance tests |
| distributed.benchmark.numInstances | int | `1` | When benchmarking is enabled, the number of instances to use for performance testing. |
| distributed.benchmark.warmup | int | `30` | When benchmarking is enabled, the number of iterations to warmup before running performance tests. |
| distributed.doBenchmark | bool | `false` | If set to true, the Intel Neural Compressor will be used to benchmark the trained model. If the model is being quantized, the quantized model will also be benchmarked. |
| distributed.doEval | bool | `true` | If set to true, evaluation will be run with the validation split of the dataset, using the Hugging Face Transformers library. |
| distributed.doQuantize | bool | `false` | If set to true, the Intel Neural Compressor will be used to quantize the trained model. |
| distributed.doTrain | bool | `true` | If set to true, training will be run using the Hugging Face Transformers library. |
| distributed.eval.perDeviceBatchSize | int | `8` | Batch size to use for evaluation for each device. |
| distributed.eval.validationSplitPercentage | float | `0.2` | The percentage of the train set used as validation set in case there's no validation split. Set to 0.20 for a 20% validation split. |
| distributed.logLevel | string | `"info"` | The Hugging Face Transformers logging level (`debug`, `info`, `warning`, `error`, and `critical`). |
| distributed.modelNameOrPath | string | `"meta-llama/Llama-2-7b-chat-hf"` | The name or path of the pretrained model to pass to the Hugging Face transformers training arguments. |
| distributed.quantize.outputDir | string | `"/tmp/pvc-mount/output/quantized_model"` |  |
| distributed.quantize.peftModelDir | string | `"/tmp/pvc-mount/output/saved_model"` |  |
| distributed.quantize.woqAlgo | string | `"RTN"` |  |
| distributed.quantize.woqBits | int | `8` |  |
| distributed.quantize.woqGroupSize | int | `-1` |  |
| distributed.quantize.woqScheme | string | `"sym"` |  |
| distributed.script | string | `"/workspace/scripts/finetune.py"` | The script that will be executed using `torch.distributed.launch`. |
| distributed.train.bf16 | bool | `true` | Whether to use bf16 (mixed) precision instead of 32-bit. Requires hardware that supports bfloat16. |
| distributed.train.dataFile | string | `nil` | Path to a Llama formatted data file to use, if no dataset name is provided. |
| distributed.train.datasetConcatenation | bool | `true` | Whether to concatenate the sentence for more efficient training. |
| distributed.train.datasetName | string | `"medalpaca/medical_meadow_medical_flashcards"` | Name of a Hugging Face dataset to use. If no dataset name is provided, the dataFile path will be used instead. |
| distributed.train.ddpBackend | string | `"ccl"` | The backend to be used for distributed training. It is recommended to use `ccl` with the Intel Extension for PyTorch. |
| distributed.train.ddpFindUnusedParameters | bool | `false` | The `find_used_parameters` flag to pass to DistributedDataParallel. |
| distributed.train.epochs | int | `1` | Number of training epochs to perform. |
| distributed.train.gradientAccumulationSteps | int | `1` | Number of updates steps to accumulate before performing a backward/update pass. |
| distributed.train.inputColumnName | string | `nil` | Name of the column in the dataset that optionally provides context or input for the task. If no column name is provided, the "input" column is used. |
| distributed.train.instructionColumnName | string | `nil` | Name of the column in the dataset that describes the task that the model should perform. If no column name is provided, the "instruction" column is used. |
| distributed.train.learningRate | float | `0.00002` | The initial learning rate. |
| distributed.train.loggingSteps | int | `1` | Log every X updates steps. Should be an integer or a float in range `[0,1]`. If smaller than 1, will be interpreted as ratio of total training steps. |
| distributed.train.loraAlpha | int | `16` | Alpha parameter in the LoRA method. |
| distributed.train.loraDropout | float | `0.1` | Dropout parameter in the LoRA method. |
| distributed.train.loraRank | int | `8` | Rank parameter in the LoRA method. |
| distributed.train.loraTargetModules | string | `"q_proj vproj"` | Target modules for the LoRA method. |
| distributed.train.maxSteps | int | `-1` | If set to a positive number, the total number of training steps to perform. Overrides the number of training epochs. |
| distributed.train.noCuda | bool | `true` | Use CPU when set to true. |
| distributed.train.outputColumnName | string | `nil` | Name of the column in the dataset with the answer to the instruction. If no column name is provided, the "output" column is used. |
| distributed.train.outputDir | string | `"/tmp/pvc-mount/output/saved_model"` | The output directory where the model predictions and checkpoints will be written. |
| distributed.train.overwriteOutputDir | bool | `true` | Overwrite the content of the output directory. Use this to continue training if output_dir points to a checkpoint directory. |
| distributed.train.perDeviceBatchSize | int | `8` | The batch size per device. |
| distributed.train.promptWithInput | string | `"Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request."` | Prompt string for an instruction with context |
| distributed.train.promptWithoutInput | string | `"Below is an instruction that describes a task. Write a response that appropriately completes the request."` | Prompt string for an instruction without context |
| distributed.train.saveStrategy | string | `"epoch"` | The checkpoint save strategy to use (`no`, `steps`, or `epoch`). |
| distributed.train.saveTotalLimit | int | `2` | If a value is passed, will limit the total amount of checkpoints. Deletes the older checkpoints in `output_dir`. |
| distributed.train.useFastTokenizer | bool | `false` | Whether to use one of the fast tokenizer (backed by the tokenizers library) or not. |
| distributed.train.useIpex | bool | `true` | Use Intel #xtension for PyTorch when it is available. |
| distributed.train.useLora | bool | `true` | Whether or not to use LoRA. |
| distributed.workers | int | `4` | The number of worker pods to deploy. |
| elasticPolicy.maxReplicas | int | `4` | The upper limit for the number of pods that can be set by the autoscaler. Cannot be smaller than `elasticPolicy.minReplicas` or `distributed.workers`. |
| elasticPolicy.maxRestarts | int | `10` | The maximum number of restart times for pods in elastic mode. |
| elasticPolicy.minReplicas | int | `1` | The lower limit for the number of replicas to which the job can scale down. |
| elasticPolicy.rdzvBackend | string | `"c10d"` | The rendezvous backend type (c10d, etcd, or etcd-v2). |
| envVars.cclWorkerCount | int | `1` | Value for the CCL_WORKER_COUNT environment variable. Must be >1 to use the CCL DDP backend. |
| envVars.ftpProxy | string | `nil` | Set the ftp_proxy environment variable. |
| envVars.hfDatasetsCache | string | `"/tmp/pvc-mount/hf_dataset_cache"` | Path to a directory used to cache Hugging Face datasets using the HF_DATASETS_CACHE environment variable. |
| envVars.hfHome | string | `"/tmp/home"` | Sets the `HF_HOME` environment variable and is used as the volume mount location for your Hugging Face token. |
| envVars.httpProxy | string | `nil` | Set the http_proxy environment variable. |
| envVars.httpsProxy | string | `nil` | Set the https_proxy environment variable. |
| envVars.ldPreload | string | `"/usr/lib/x86_64-linux-gnu/libtcmalloc.so.4.5.9:/usr/local/lib/libiomp5.so"` | Paths set to the LD_PRELOAD environment variable. |
| envVars.logLevel | string | `"INFO"` | Value set to the LOG_LEVEL environment variable. |
| envVars.noProxy | string | `nil` | Set the no_proxy environment variable. |
| envVars.socksProxy | string | `nil` | Set the socks_proxy environment variable. |
| envVars.transformersCache | string | `"/tmp/pvc-mount/transformers_cache"` | Location for the Transformers cache (using the TRANSFORMERS_CACHE environment variable). |
| image.name | string | `"intel/ai-workflows"` | Name of the image to use for the PyTorch job. The container should include the fine tuning script and all the dependencies required to run the job. |
| image.pullPolicy | string | `"IfNotPresent"` | Determines when the kubelet will pull the image to the worker nodes. Choose from: `IfNotPresent`, `Always`, or `Never`. If updates to the image have been made, use `Always` to ensure the newest image is used. |
| image.tag | string | `"torch-2.3.0-huggingface-multinode-py3.10"` | The image tag for the container that will be used to run the PyTorch job. The container should include the fine tuning script and all the dependencies required to run the job. |
| resources.cpuLimit | string | `nil` | Optionally specify the maximum amount of CPU resources for each worker, where 1 CPU unit is equivalent to 1 physical CPU core or 1 virtual core. For more information see the [Kubernetes documentation on CPU resource units](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu). |
| resources.cpuRequest | string | `nil` | Optionally specify the amount of CPU resources requested for each worker, where 1 CPU unit is equivalent to 1 physical CPU core or 1 virtual core. For more information see the [Kubernetes documentation on CPU resource units](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu). |
| resources.memoryLimit | string | `nil` | Optionally specify the maximum amount of memory resources for each worker. For more information see the [Kubernetes documentation on memory resource units](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory). |
| resources.memoryRequest | string | `nil` | Optionally specify the amount of memory resources requested for each worker. For more information see the [Kubernetes documentation on memory resource units](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory). |
| resources.nodeSelectorLabel | string | `nil` | Optionally specify a label for the type of node that will be used for the PyTorch job workers. |
| resources.nodeSelectorValue | string | `nil` | If `resources.nodeSelectorLabel` is set, specify the value for the node selector label. |
| secret.encodedToken | string | `nil` | Hugging Face token encoded using base64. |
| securityContext.allowPrivilegeEscalation | bool | `false` | Boolean indicating if a process can gain more privileges than its parent process. |
| securityContext.fsGroup | string | `nil` | File system group ID to run as a non-root user |
| securityContext.runAsGroup | string | `nil` | Group ID to run as a non-root user |
| securityContext.runAsUser | string | `nil` | User ID to run as a non-root user |
| storage.pvcMountPath | string | `"/tmp/pvc-mount"` | The location where the persistent volume claim will be mounted in the worker pods. |
| storage.resources | string | `"50Gi"` | Specify the [capacity](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#capacity) for the persistent volume claim. |
| storage.storageClassName | string | `"nfs-client"` | Name of the storage class to use for the persistent volume claim. To list the available storage classes use: `kubectl get storageclass`. |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.13.1](https://github.com/norwoodj/helm-docs/releases/v1.13.1)