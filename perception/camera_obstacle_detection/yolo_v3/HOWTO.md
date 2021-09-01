# Updated steps to run the pipeline

Kindly follow the below steps: 

1.      export MODEL_DIR=`pwd`/perception/camera_obstacle_detection/yolo_v3/tensorflow_fp32_coco/

2.      mkdir ${MODEL_DIR}/example_pipeline/build

3.      docker run \
            -it --rm \
            -v ${MODEL_DIR}:${MODEL_DIR} -w ${MODEL_DIR} \
            -u $(id -u ${USER}):$(id -g ${USER}) \
            autoware/model-zoo-tvm-cli:1.0.0 \
                compile \
                --config ${MODEL_DIR}/definition.yaml \
                --output_path ${MODEL_DIR}/example_pipeline/build

4.      docker run \
            -it --rm \
            -v ${MODEL_DIR}:${MODEL_DIR} -w ${MODEL_DIR}/example_pipeline/build \
            -u $(id -u ${USER}):$(id -g ${USER}) \
            --entrypoint "" \
            autoware/model-zoo-tvm-cli:1.0.0 \
            bash -c "cmake .. && make -j"

5.      docker run \
            -it --rm \
            -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
            -v ${HOME}/.Xauthority:${HOME}/.Xauthority:rw \
            -e XAUTHORITY=${HOME}/.Xauthority \
            -e DISPLAY=$DISPLAY \
            --net=host \
            -v ${MODEL_DIR}:${MODEL_DIR} \
            -w ${MODEL_DIR}/example_pipeline/build \
            --entrypoint "" \
            autoware/model-zoo-tvm-cli:1.0.0 \
                bash -c "LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/ ./example_pipeline"


Kindly note that, we are able to generate the `deploy_lib.so`, `deploy_graph.json`, `deploy_param.params`, `inference_engine_tvm_config.hpp` (The issue we were facing yesterday has been fixed by updating the input shape in `definition.yaml`).

We are currently trying to run the example pipeline, gettting the below error:  
```
kishor@kishor-interplai:~/Interplai_ADS/Port_YoloV3/modelzoo$ docker run     -it --rm     -v /tmp/.X11-unix:/tmp/.X11-unix:rw     -v ${HOME}/.Xauthority:${HOME}/.Xauthority:rw     -e XAUTHORITY=${HOME}/.Xauthority     -e DISPLAY=$DISPLAY     --net=host     -v ${MODEL_DIR}:${MODEL_DIR}     -w ${MODEL_DIR}/example_pipeline/build     --entrypoint ""     autoware/model-zoo-tvm-cli:1.0.0          bash -c "LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/ ./example_pipeline"
terminate called after throwing an instance of 'dmlc::Error'
  what():  [16:33:23] ../src/runtime/graph/graph_runtime.cc:178: Check failed: data->shape[j] == data_out->shape[j] (255 vs. 76) : 
Stack trace:
  [bt] (0) /usr/local/lib/libtvm_runtime.so(+0xe7d48) [0x7fbb9e65cd48]
  [bt] (1) /usr/local/lib/libtvm_runtime.so(tvm::runtime::GraphRuntime::CopyOutputTo(int, DLTensor*)+0x624) [0x7fbb9e65ece4]
  [bt] (2) /usr/local/lib/libtvm_runtime.so(+0xea0c0) [0x7fbb9e65f0c0]
  [bt] (3) ./example_pipeline(main+0x903) [0x5641e18f6793]
  [bt] (4) /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf3) [0x7fbb9d6830b3]
  [bt] (5) ./example_pipeline(_start+0x2e) [0x5641e18f7f0e]


kishor@kishor-interplai:~/Interplai_ADS/Port_YoloV3/modelzoo$ 
```