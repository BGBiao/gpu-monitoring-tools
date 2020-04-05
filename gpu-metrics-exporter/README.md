# GPU Metrics Exporter

基于[pod-gpu-metrics-exporter](../pod-gpu-metrics-exporter)改造的，采集GPU主机上的gpu metrics信息，并暴露到`http://host:port/gpu/metrics`端点上


### 官方依赖

* NVIDIA Tesla drivers = R384+ (download from [NVIDIA Driver Downloads page](http://www.nvidia.com/drivers))
* nvidia-docker version > 2.0 (see how to [install](https://github.com/NVIDIA/nvidia-docker) and it's [prerequisites](https://github.com/nvidia/nvidia-docker/wiki/Installation-\(version-2.0\)#prerequisites))
* Set the [default runtime](https://github.com/NVIDIA/nvidia-container-runtime#daemon-configuration-file) to nvidia


```
# 验证依赖
$ nvidia-smi  -L
GPU 0: Tesla P100-PCIE-16GB (UUID: GPU-b91e30ac-fe77-e236-11ea-078bc2d1f226)

$ nvidia-docker version
NVIDIA Docker: 2.0.3
...
...

# 修改运行时
$ type nvidia-container-runtime
nvidia-container-runtime is /usr/bin/nvidia-container-runtime

$ sudo tee /etc/docker/daemon.json <<EOF
{
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
EOF
$ sudo pkill -SIGHUP dockerd

```

#### 部署到GPU主机上

```sh
# 构建gpu-metrics-exporter镜像
$ cd gpu-metrics-exporter
$ make

# 确定dcgm-exporter是运行的
$ docker run -d --runtime=nvidia --rm --name=nvidia-dcgm-exporter bgbiao/dcgm-exporter

$ docker run -d --privileged --rm -p 9400:9400  --volumes-from nvidia-dcgm-exporter:ro bgbiao/gpu-metrics-exporter

# 检查gpu暴露出来的基础信息
$ curl -s localhost:9400/gpu/metrics

# Sample output
dcgm_ecc_dbe_aggregate_total{gpu="0",uuid="GPU-b91e30ac-fe77-e236-11ea-078bc2d1f226"} 0
# HELP dcgm_retired_pages_sbe Total number of retired pages due to single-bit errors.
# TYPE dcgm_retired_pages_sbe counter
dcgm_retired_pages_sbe{gpu="0",uuid="GPU-b91e30ac-fe77-e236-11ea-078bc2d1f226"} 0
# HELP dcgm_retired_pages_dbe Total number of retired pages due to double-bit errors.
# TYPE dcgm_retired_pages_dbe counter
dcgm_retired_pages_dbe{gpu="0",uuid="GPU-b91e30ac-fe77-e236-11ea-078bc2d1f226"} 0
# HELP dcgm_retired_pages_pending Total number of pages pending retirement.
# TYPE dcgm_retired_pages_pending counter
dcgm_retired_pages_pending{gpu="0",uuid="GPU-b91e30ac-fe77-e236-11ea-078bc2d1f226"} 0

```

