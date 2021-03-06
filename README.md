## PVANet-FACE: PVANET for face detection

### Introduction
Training a face detection model using PVANet.

![face detection 1](imgs/0.jpg?raw=true "Face detection for women")
![face detection 2](imgs/1.jpg?raw=true "Face detection for men")

The dataset used for training is [WIDERFACE](http://mmlab.ie.cuhk.edu.hk/projects/WIDERFace/)

This repository contains source files of face detection using the PVANet. It is developed based on the awesome [pva-faster-rcnn](https://github.com/sanghoon/pva-faster-rcnn) repository.


### Requirement
1. Nivida CUDA 8.0
2. Nvidia CUDNN 6
3. Python 2


### Installation
1. Clone this repository
    ```Shell
    # Make sure to clone with --recursive
    git clone --recursive https://github.com/twmht/pva-faster-rcnn.git 
    ```

2. We'll call the directory that you cloned as `FRCN_ROOT`. Build the Cython modules
    ```Shell
    cd $FRCN_ROOT/lib
    make
    ```

3. Build Caffe and pycaffe
    ```Shell
    cd $FRCN_ROOT/caffe-fast-rcnn
    # Now follow the Caffe installation instructions here:
    #   http://caffe.berkeleyvision.org/installation.html
    # For your Makefile.config:
    #   Uncomment `WITH_PYTHON_LAYER := 1`

    cp Makefile.config.example Makefile.config
    make -j8 && make pycaffe
    ```

### Training the face detection model
1. Download all available models (including pre-trained and compressed models)
    ```Shell
    cd $FRCN_ROOT
    ./models/pvanet/download_all_models.sh
    ```

2. Download [WIDERFace](http://mmlab.ie.cuhk.edu.hk/projects/WIDERFace/) for training.

   I use [python-widerface](https://pypi.python.org/pypi/python-widerface/0.1.1) and [cute-format](https://pypi.python.org/pypi/cute_format) to pack all the images of WIDERFace into the custom-defined imdb, where the format of imdb is different from VOC format.

   please look `tools/convert_wider_to_imdb.py` for detail.

   to run `tools/convert_wider_to_imdb.py`,  update path to WIDERFace

   for example,

   ```python
   # arg1: path to split (where the label file is)
   # arg2: path to images
   # arg3: path to label file name
    wider_train = WIDER('/opt/WiderFace/wider_face_split',
                  '/opt/WiderFace/WIDER_train/images',
                  'wider_face_train.mat')

    cw = CuteWriter('wider-imdb')

    run(wider_train, cw)
   ```

   this will generate a db named `wider-imdb`, and put `wider-imdb` into `data/widerface/`


3.  Training PVANet
    ```Shell
    cd $FRCN_ROOT
    tools/train_net.py --gpu 0 --solver models/pvanet/example_train/solver.prototxt --weights models/pvanet/pretrained/pva9.1_pretrained_no_fc6.caffemodel --iters 100000 --cfg models/pvanet/cfgs/train.yml --imdb wider
    ```

### How to run the demo

1. Download [pretrained model](https://drive.google.com/open?id=0B18-oWPEXrIWTE00alJRYTA5cW8)

2. Run the `tools/demo.py`
    ```Shell
    cd $FRCN_ROOT
    ./tools/demo.py --net output/faster_rcnn_pvanet/wider/pvanet_frcnn_iter_100000.caffemodel --def models/pvanet/pva9.1/faster_rcnn_train_test_21cls.pt --cfg models/pvanet/cfgs/submit_1019.yml --gpu 0
    ```

### Compression

If you want to compress your model, please look at `tools/gen_merged_model.py`. As compared to sanghoon's implementation (https://github.com/sanghoon/pva-faster-rcnn/blob/master/tools/gen_merged_model.py), I add the function to remove redundant power layers.
