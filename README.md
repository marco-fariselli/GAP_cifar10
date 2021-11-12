# CIFAR10 on GAP

In the cifar10_training.ipynb notebook you can find the training script. I used the google colab with GPU enabled, it can be also run on your local machine.

It produced 2 tflite files:

	- cifar10_model_fp32.tflite
	- cifar10_model_uint8.tflite

With the quantized one you can generate your project:

```
source gap_sdk/sourceme.sh
nntool cifar10_model_uint8.tflite -q
adjust
fusions --scale8
gen_project .
```

The default project can already run on uninitialized input data, the application code is in **cifar10_model_uint8.c**:

```
make clean all run platform=gvsoc
```

For clarity reason I reorganized the folder moving the model and the nntool_script into the *model* folder, changing accordingly the paths in the *common.mk* file. I also added the hwc version for completeness, it can be chosen adding **MODEL_HWC=1** in the make command.

From the training scripts you can save several sample data, I made a directory for them: *samples*

I've then changed the application code to run on real data (images from testing dataset) adding the snippet below:

```
#include "gaplib/ImgIO.h"
#define __XSTR(__s) __STR(__s)
#define __STR(__s) #__s    ImageName = __XSTR(AT_IMAGE);
...
    #if defined(MODEL_HWC)
    int Traspose2CHW = 0;
    #else
    int Traspose2CHW = 1;
    #endif
    printf("Reading image in %s\n", Traspose2CHW?"CHW":"HWC");
    if (ReadImageFromFile(ImageName, 32, 32, 3, Input_1, 32*32*3*sizeof(char), IMGIO_OUTPUT_CHAR, Traspose2CHW)) {
        printf("Failed to load image %s\n", ImageName);
        pmsis_exit(-1);
    }
    for (int i=0; i<32*32*3; i++) Input_1[i] -= 128;
...
```

This loads data from an image defined by the *AT_IMAGE* in the *Makefile*. It also already shift the [0:255] data into the [-128:127] required by the so trained model.