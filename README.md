pl-matrixmultiply_moc_x86_64
===============

- [pl-matrixmultiply_moc_x86_64](#pl_matrixmultiply_moc_x86_64)
  * [Description](#description)
  * [Usage](#usage)
    + [Requirements](#requirements)
    + [Docker run on x86_64](#docker-run-on-x86_64)
    + [Docker run on PowerPC](#docker-run-on-powerpc)
  * [Example](#example)
  * [Research and Development References](#research-and-development-references)
    + [Workflow](#workflow)
  * [Troubleshoot](#troubleshoot)
  * [Related Links](#related-links)
    + [Docker Images](#docker-images)
  * [Contributor](#contributor)



  
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

What this plugin simply does is, when assigned with the COE parameter (`-c`), it will generate a list of COE values, for example, `-c 32,32,128` will generate four parameters of COE as: `[32, 64, 96, 128]`, and it is not necessary to assign the end value as exactly divisible by gap value. (which means you can go with `-c 32,32,100`.

For each COE value in this list, it will generate square matrix with size to be `(COE x TPB) ^2`, where `TPB` value is determined as 32 in the program, and therefore you have different sizes of matrix to do multiplication.

The program will record this running time along with start time, end time and matrix size, and generate a `.csv` file in your output folder, that's what you can use for benchmarking purpose.



Troubleshoot
-------

Make sure your `/out` directory has corresponding authorization, you can try:

```
mkdir in out && chmod 777 out                                       
```


Related Links
----------
https://medium.com/datathings/benchmarking-blas-libraries-b57fb1c6dc7

### Docker Images
PowerPC: https://hub.docker.com/r/fnndsc/pl-matrixmultiply_moc_ppc64

x86_64: https://hub.docker.com/r/fnndsc/pl-matrixmultiply_moc_x86_64



Contributor
---
Haoyang Wang  <haoyangw@bu.edu>

Kefan Zhang <kefan29@bu.edu>

Feel free to reach out us if you have any question. :)
