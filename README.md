# BME-X: Brain MRI enhancement foundation model for motion correction, denoising, super-resolution, harmonization, and downstream tasks

_Yue Sun, Limei Wang, Gang Li, Weili Lin, Li Wang. A foundation model for enhancing magnetic resonance images and downstream segmentation, registration and diagnostic tasks, Nature Biomedical Engineering, 2024, in press_

### Structural magnetic resonance imaging is a vital tool for neuroimaging analyses, but the quality of MR images is often degraded by various factors, such as motion artifacts, large slice thickness, and imaging noise. These factors can cause confounding effects and pose significant challenges, especially in young children who tend to move during acquisition.

## Method
We proposed a tissue-aware reconstruction framework that can improve image quality through motion correction, super-resolution, denoising, and contrast enhancement. Our framework exhibits the capability to estimate high-field-like (7T-like) images from 3T images, handle pathological brain MRIs with multiple sclerosis or gliomas, harmonize MRIs acquired by various scanners, and can be easily extended for "end-to-end" neuroimaging analyses, such as tissue segmentation.
<img src="https://github.com/YueSun814/Img-folder/blob/main/BME-X_flowchart.png" width="100%">    

### Motion correction and super resolution:
<div align="center">
<img src="https://github.com/YueSun814/Img-folder/blob/main/BME-X_BCP_real_images.png" width="70%"> 
</div>   
In the first column, from top to bottom are T1w images with severe, moderate and minor artifacts, and two low resolution images (1.0×1.0×3.0 mm<sup>3</sup>). The corresponding enhanced images generated by different methods are shown from the second column to the last column. The resolution of the 2D sagittal and coronal slices (the corrupted images in the last two rows) is indicated in bold as 1.0×1.0×3.0 mm<sup>3</sup> and 1.0×1.0×3.0 mm<sup>3</sup>, respectively. The corresponding enhanced results are in a resolution of 0.8×0.8×0.8 mm<sup>3</sup>.

### Performance on 10,963 lifespan images from fetal to adulthood:
<div align="center">
<img src="https://github.com/YueSun814/Img-folder/blob/main/BME-X_lifespan_real_images.png" width="100%"> 
</div>   
Enhanced results of the BME-X model for 10,963 in vivo low-quality images across the whole human lifespan, collected from 19 datasets. a, Age distribution. b, Mid-late fetal: original T2w images with severe motion and noise, and the corresponding enhanced results. c, From neonatal to early childhood: low-resolution T1w images (1.0×1.0×3.0 mm<sup>3</sup>) and the corresponding enhanced results (0.8×0.8×0.8 mm<sup>3</sup>). d, From neonatal to late adulthood: T1w images and the corresponding enhanced results. e, Comparison of TCT values for the 10,963 original images and the corresponding enhanced images by BME-X. 

### Application on reconstruction 7T-like images from 3T MRIs:
<div align="center">
<img src="https://github.com/YueSun814/Img-folder/blob/main/BME-X_3T_7T.png" width="60%"> 
</div>   

### Application on harmonization across scanners (into a latent common space):
<div align="center">
<img src="https://github.com/YueSun814/Img-folder/blob/main/BME-X_harmonization.png" width="100%"> 
</div>  

### Preservation of small lesions during enhancement:
<div align="center">
<img src="https://github.com/YueSun814/Img-folder/blob/main/BME-X_spots.png" width="50%"> 
</div>  

### Bias quantification during reconstruction:
<div align="center">
<img src="https://github.com/YueSun814/Img-folder/blob/main/BME-X_MR-ART.png" width="100%"> 
</div>  

Enhancement results and the bias quantification for 280 _in vivo_ corrupted T1w images from the [MR-ART dataset](https://openneuro.org/datasets/ds004173/versions/1.0.2), generated by competing methods and the BME-X model. a, Visual comparison of the enhanced results. The first and the seventh columns show _in vivo_ corrupted images with two levels of real artifacts (i.e., HM1 and HM2) acquired from the same participant. The remaining columns show the enhanced results generated by four competing methods and the BME-X model. b, Quantitative comparison on 280 _in vivo_ corrupted images using tissue volumes (i.e., WM, GM, CSF, ventricle, and hippocampus), mean cortical thickness, as well as the corresponding difference compared with STAND. In each box plot, the midline represents the median value, and its lower and upper edges represent the first and third quartiles. The whiskers go down to the smallest value and up to the largest. The dotted line represents the mean value or a zero value. 

Effect size (Cohen’s _d_) of tissue volumes between Alzheimer’s disease and normal cognition participants from the ADNI dataset.

<div align="center">
<img src="https://github.com/YueSun814/Img-folder/blob/main/BME-X_ADNI.jpg" width="80%"> 
</div>  

## Training steps:

   In folder: ***Training_files***
   1. Download training samples (hdf5 data) from https://www.dropbox.com/scl/fo/8jrphll6vu4sbw56x9ni7/h?rlkey=nfspdxoyr0u61i1xh29dauohu&dl=0. More information about hdf5 data is avaliable at <https://www.mathworks.com/help/matlab/hdf5-files.html>.
   2. Train a caffe model:
      
    caffe train -solver solver.prototxt -gpu 0 >train.log 2>&1 &  # for local installation of Caffe
    
    dcaffe train -solver solver.prototxt -gpu 0 >train.log 2>&1 &  # for Docker installation of Caffe

   ***solver.prototxt***: set your learning rate (base_lr: 0.005), network (net: "train.prototxt"), step size (stepsize=222),  saving path (snapshot_prefix: "./"), etc.

   ***-gpu***: set GPU number.

   ***train.log***: a log file to record the model training stage.

   ***train.prototxt***: network architecture.

   This is the architecture based on Anatomy-Guided Densely-Connected U-Net (ADU-Net) in the paper of "L. Wang, G. Li, F. Shi, X. Cao, C. Lian, D. Nie, et al., "Volume-based analysis of 6-month-old infant brain MRI for autism biomarker identification and early diagnosis," in MICCAI, 2018, pp. 411-419."

## PyTorch version

   In folder ***PyTorch_version***, we implemented the same network architecture using PyTorch. You can find it in _Enhancement_model.py_. 

## Testing steps:

   In folder ***Pretrained_models***: eight pretrained models used for testing images at different ages.
   
   In folder ***Templates***: the corresponding eight images templates for histogram matching.
   
   In folder ***Testing_subjects***: a testing sample at 24 months (_test_img.???_), and the corresponding reconstruction result (_test_img-recon.nii.gz_).
   
   Test an image: 
   
   1. Performing histogram matching for testing images with provided templates (in folder ***Templates***).

    def hist_match(img, temp):
    ''' histogram matching from img to temp '''
    matcher = sitk.HistogramMatchingImageFilter()
    matcher.SetNumberOfHistogramLevels(1024)
    matcher.SetNumberOfMatchPoints(7)
    matcher.ThresholdAtMeanIntensityOn()
    res = matcher.Execute(img, temp)
    return res
    
   3. Test:
      
    python2 Reconstruction_test.py   # for local installation of Caffe
    
    dpython Reconstruction_test_docker.py  # for Docker installation of Caffe
    
## System requirements:

Ubuntu 20.04.1
    
Caffe version 1.0.0-rc3

We provided two options for installing the Caffe framework on your local PC.  
    
### 1. Docker version

!!! We have included the Caffe framework in a Docker image. 

For Docker and Nvidia-docker installtion, please refer to https://github.com/iBEAT-V2/iBEAT-V2.0-Docker#run-the-pipeline. 

#### a. Download an image. 

Please download the image from the following link: [https://www.dropbox.com/scl/fi/jgszaaldp97fktmi833je/caffe.tar?rlkey=snaxky2fz9ljn8a8mt0jz7d5q&dl=0](https://www.dropbox.com/scl/fi/ulx9a6ytdw2hpeoaahlxt/caffe.tar?rlkey=y6zwg5etyzhbtmw2l2i5rbjk2&dl=0). (The image is _caffe.tar_)

#### b. Load the image into your local PC. 

    docker load < caffe.tar 

Then you can check the image using this command "docker images".  

#### c. Add an alias to your _~/.bashrc_ file. to easily access Caffe from the Docker image. 

    alias dcaffe='nvidia-docker run --rm -u $(id -u):$(id -g) -v $(pwd):$(pwd) -w $(pwd) caffe:v2 /usr/local/caffe/caffe_rc3/build/tools/caffe'
    alias dpython='nvidia-docker run --rm -u $(id -u):$(id -g) -v $(pwd):$(pwd) -w $(pwd) caffe:v2 python'

Then, "source ~/.bashrc". 

#### d. Test Caffe  

    dcaffe

The screen will show:  
    
<img src="https://github.com/YueSun814/Img-folder/blob/main/caffe_display.jpg" width="50%">    

    dpython

No output will be displayed. 

### 2. Local installation 

To make sure of consistency with our used version (e.g., including 3d convolution, and WeightedSoftmaxWithLoss, etc.), we strongly recommend installing _Caffe_ using our released ***caffe_rc3***. The installation steps are easy to perform without compilation procedure: 
    
#### a. Download ***caffe_rc3*** and ***caffe_lib***.
    
caffe_rc3: <https://github.com/YueSun814/caffe_rc3>
    
caffe_lib: <https://github.com/YueSun814/caffe_lib>
    
#### b. Add paths of _caffe_lib_, and _caffe_rc3/python_ to your _~/.bashrc_ file. For example, if the folders are saved in the home path, then add the following commands to the _~/.bashrc_ 
   
    export LD_LIBRARY_PATH=~/caffe_lib:$LD_LIBRARY_PATH
   
    export PYTHONPATH=~/caffe_rc3/python:$PATH

Then, "source ~/.bashrc".

#### c. Test Caffe 
    
     cd caffe_rc3/build/tools
    
     ./caffe
    
Then, the screen will show:  
    
<img src="https://github.com/YueSun814/Img-folder/blob/main/caffe_display.jpg" width="50%">
    
Typical install time: few minutes.
   

    


