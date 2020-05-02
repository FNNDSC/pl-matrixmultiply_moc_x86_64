pl-matrixmultiply_moc_x86_64
===============


  
Description
------------
```

                                 _        _                      _ _   _       _       
                                | |      (_)                    | | | (_)     | |      
                 _ __ ___   __ _| |_ _ __ ___  ___ __ ___  _   _| | |_ _ _ __ | |_   _ 
                | '_ ` _ \ / _` | __| '__| \ \/ / '_ ` _ \| | | | | __| | '_ \| | | | |
                | | | | | | (_| | |_| |  | |>  <| | | | | | |_| | | |_| | |_) | | |_| |
                |_| |_| |_|\__,_|\__|_|  |_/_/\_\_| |_| |_|\__,_|_|\__|_| .__/|_|\__, |
                                                                        | |       __/ |
                                                                        |_|      |___/ 
```

This is a benchmarking plugin for ChRIS platform of Boston Children's Hospital ([What is ChRIS?](https://www.bu.edu/rhcollab/projects/radiology/))on both x86_64 and PowerPC MOC using matrix multiplication.
| Plugin Info  | Content | Description |
|:------------:|:-----------------:|:-------------------------------------------------------------:|
| Input | matrix parameters | COE value to indicate matrix size, given by command line |
| Output | one .csv file | Contains matrix size and running time for multiplication task |


Usage
-------------------
### Requirements
Your host computer should be a linux os and installed CUDA 10.1 && nvidia container.



### Docker run on x86_64

First pull docker image to local environment:
```
docker pull fnndsc/pl-matrixmultiply_moc_x86_64
```
Then you can run it with parameters:
``` {.sourceCode .bash}
docker run --runtime=nvidia                                         \
            -e NVIDIA_VISIBLE_DEVICES=1                             \
            -v $(pwd)/in:/incoming -v $(pwd)/out:/outgoing          \
            fnndsc/pl-matrixmultiply                                \
            matmultiply.py                                          \
            -c 32,32,128                                            \
            /incoming /outgoing
```
Parameters and meaning below in the table:

| **docker run parameters for x86_64** |  |  |
|:--------------------------------:|:-----------------------------------------------------------------------------------------------------------:|:--------------------------------------------------------:|
| **parameters** | **function** | **example** |
| --runtime=nvidia | tells the docker to use the nvidia docker | --runtime=nvidia |
| -e | specifies the visible graphic device id | -e NVIDIA_VISIBLE_DEVICES=1 |
| -v | specify input and outgoing folder, check docker [volume bind](https://docs.docker.com/storage/bind-mounts/) | -v $(pwd)/in:/incoming -v $(pwd)/out:/outgoing |
| image_name | specify docker image name | fnndsc/pl-matrixmultiply |
| script_name | specify script file to run | matmultiply.py                                           |
| -c | specify COE value to define matrix size, format like: start_value, gap_value, end_value | -c 32,32,128 |

### Docker run on PowerPC
First pull docker image to local environment:
```
docker pull fnndsc/pl-matrixmultiply_moc_ppc64
```
Then you can run it with parameters:
``` {.sourceCode .bash}
docker run  --security-opt label=type:nvidia_container_t            \
            -v $(pwd):/incoming:z -v $(pwd)/out:/outgoing:z         \
            fnndsc/pl-matrixmultiply_moc_ppc64                      \
            matmultiply.py                                          \
            -c 32,32,128                                            \
            /incoming /outgoing
```
Parameters and meaning below in the table:


| **docker run parameters for PowerPC** |  |  |
|:--------------------------------------------:|:-----------------------------------------------------------------------------------------------------------:|:-----------------------------------------------:|
| **parameters** | **function** | **example** |
| --security-opt label=type:nvidia_container_t | tells the docker to use the nvidia docker | --security-opt label=type:nvidia_container_t |
| -v | specify input and outgoing folder, check docker [volume bind](https://docs.docker.com/storage/bind-mounts/) | -v $(pwd):/incoming:z -v $(pwd)/out:/outgoing:z |
| image_name | specify docker image name | fnndsc/pl-matrixmultiply_moc_ppc64 |
| script_name | specify script file to run | matmultiply.py |
| -c | specify COE value to define matrix size, format like: start_value, gap_value, end_value | -c 32,32,128 |

**Build Instructions**

For ppc64le image, we cannot use the automatic build on docker hub. We have to build this conatiner locally and push it into docker hub.

```bash
cd path/to/this/repo
docker login docker.io -u [your docker.io username]
docker build -f Dockerfile -t docker.io/fnndsc/pl-matrixmultiply_moc_ppc64 
docker push "docker.io/fnndsc/pl-matrixmultiply_moc_ppc64"

```
Example
-------

**x86_64**
``` {.sourceCode .bash}
docker run --runtime=nvidia                                         \
            -e NVIDIA_VISIBLE_DEVICES=1                             \
            -v $(pwd)/in:/incoming -v $(pwd)/out:/outgoing          \
            fnndsc/pl-matrixmultiply                                \
            matmultiply.py                                          \
            -c 32,32,128                                            \
            /incoming /outgoing
```

**PowerPC**
``` {.sourceCode .bash}
docker run  --security-opt label=type:nvidia_container_t            \
            -v $(pwd):/incoming:z -v $(pwd)/out:/outgoing:z         \
            fnndsc/pl-matrixmultiply_moc_ppc64                      \
            matmultiply.py                                          \
            -c 32,32,128                                            \
            /incoming /outgoing
```

Research and Development References
-------------

### Workflow

```
This graph show the workflow of the original python script (provided by nVidia). One of the difference in our scripts is that we use a file instead of the web camera as graph source. Therefore, this container have to use the ffmpeg to decode the video file. Also, since there is no graphic interface in the server/moc/openshift, we removed the realtime progress showing codes and replace it by saving the output to an output file (output.avi) in the `outgoing` directiory. This means ffmpeg is essentical.

There is another output file called `FramePerSecondRecord.csv`. This file contains the benchmarking results of the plugin. The output should be like this:
| maximum_fps  | minimum_fps | average_fps |
|:------------:|:-----------:|:-----------:|
|                                                                                                                            250.0                                                                                                                          |                                                                     142.86                                                                    |                                                                                                                      239.92                                                                                                                    |

If you wanna more research details of this project, [check this tutorial.](https://medium.com/better-programming/real-time-object-detection-on-gpus-in-10-minutes-6e8c9b857bb3)


(If you run it multiple times , the newest result will be added to the last line of file.)

(Results from a `ppc64le` machine)

This shows the information about the inference time for every frame. We think it shows the data bus latency from cpu/main memory to the GPU.
```

### Benchmarking result
```
On `ppc64le` machine, the typical inference time for each frame is about 4 ms. However in `x86_64` machine, we got about 6~7 ms inference time for every frame. We think the differnece is significant (powerpc is about 40% faster than `x86_64`).
```

Troubleshoot
-------

Related Links
----------



### Docker Images
PowerPC: https://hub.docker.com/r/fnndsc/pl-matrixmultiply_moc_ppc64

x86_64: https://hub.docker.com/r/fnndsc/pl-matrixmultiply_moc_x86_64



Contributor
---
Haoyang Wang  <haoyangw@bu.edu>

Kefan Zhang <kefan29@bu.edu>

Feel free to reach out us if you have any question. :)
