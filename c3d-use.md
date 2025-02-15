title: C3D Usage Summary 
date: 2016-03-03 17:34:05
tags:
 - Deep Learning
 - Caffe
---

[C3D](https://github.com/facebook/C3D) is a deep learning tool which is modified version of BVLC [caffe](https://github.com/BVLC/caffe) to support 3D convolution and pooling. it was released by Facebook. In the field of human action recognition, C3D feature of video clip is the state-of-the-art feature. In this blog, I write some notes for using this tool in practice.  
<!--more-->
## 1. Compile C3D
 1. Clone C3D from github:
 ```
 git clone https://github.com/facebook/C3D.git
 ```

 2.	Compile it:
 ```
	cp Makefile.config.example Makefile.config
	#adapt makefile according to configuration of your machine, for example, change atblas to mkl
	make all -j 32 
 ```
	  `-j 32` means use 32 cores to compile it in parallel.
	 
	When something goes wrong, search the Internet, find a solution and update your makefile.    
 3.	Test whether the compilation is finished correctly.  
 ```
#assume you are at C3D root directory right now  
cd examples/c3d_feature_extraction  
#download trained sport1m caffemodel from net  
sh c3d_sport1m_feature_extraction_frm.sh  
 ```
 		If the command above runs correctly, your compilation is successful!  

## 2. Use C3D to extract feature of UCF101 video dataset
 1. Get the dataset and write list files
 	First download UCF101 dataset from <http://crcv.ucf.edu/data/UCF101.php>, and then write list files.
  
 	Since C3D can read video clips and frames of videos, you can write input list file and output list file in both way.
  
 	For frame format, you need  to transform video to frames firstly, I use ffmpeg to do this job:
  
 ```
 	ffmpeg -i "/path/to/video" "/path/to/frm/dir/%06d.jpg"
 ```
 	The input list file contains the paths to video clips or frames, the output list file contains the path where to save the features.
  
 	The format of input list file is like this:  
  
 	`<string_path> <starting_frame> <label>`  
   
 	for example, `/home/yunfeng/dataset/ucf101/ucf101_frm/YoYo/v_YoYo_g23_c01/ 1 100`  
 	**NOTE: for video clip, `starting_frame` starts from 0, but for frames, it starts from 1.**  
   
 	the format of output list file is like this:  
 	`<output_folder>`  
 	for example, `/output/c3d/YoYo//v_YoYo_g23_c01/000001`  

 2. Create output directory YOURSElF
 	**NOTE: C3D does not create directories in output list file, you must create them yourself.**  
   
 	There is a simple way: since we have path to target directory in output list file, we can use it:  
  
 ```
 	#assume you are at C3D root directory right now
 	1. cd examples/c3d_feature_extraction
 	2. cp prototxt/output_list_prefix.txt create_dir.sh
 	3. vi create_dir.sh
 ```
 	In vi, do two steps to change diretories to shell commands:
 ```
 	:0,$s/\/\d\+/\//g # remove the number at the end of each line.
 	:0,$s/output/mkdir -p output/g # add `mkdir -p` command at the head of each line.
 ```
 	then run the command to create directories:
 ```
 	sh create_dir.sh
 ```
 3.	Use tools to extract features
 	After finishing first step, you need to write a prototxt file. Fortunately, there is a example file in the directory: `prototxt/c3d_sport1m_feature_extractor_frm.prototxt`, you can adapt it to have right access to input list file.  
   
 	We use tool named `extract_image_features.bin` in `build/tools` directory to extract features, the usage of it is 
   
 ```
 extract_image_features.bin <feature_extractor_prototxt_file> 
<c3d_pre_trained_model> <gpu_id> <mini_batch_size> 
<number_of_mini_batches> <output_prefix_file> <feature_name1>
<feature_name2> ... 
 ```
 	We can use command below to extract feature:
    
 ```
	GLOG_logtosterr=1 ../../build/tools/extract_image_features.bin
	prototxt/c3d_sport1m_feature_extractor_frm.prototxt 
	conv3d_deepnetA_sport1m_iter_1900000 0 50 1 
	prototxt/output_list_prefix.txt fc7­1 fc6­1 prob 
 ```
 After extraction of feature, we can use matlab code in `script` subdirectory of `example/c3d_feature_extraction` to do further job. There are two matlab files in `script`, `read_binary_blob.m`, `read_binary_blob_preserve_shape.m`. There are used to transform features into binary blob data, We can use these two functions for further analysis of features.

## 3. Train 3D convolution neural network
Since C3D is a fork of [Caffe](http://github.com/bvlc/caffe.git), which is a fast open framework for deep learning, We can use C3D to train deep networks. You can train from scratch or fine-tune C3D on your own dataset.
 1.	Train from scratch

 	C3D offers some useful shell scripts to simplify our job, we can read the scripts and adapt it according our tasks.
   
 	**NOTE: you must adapt the shell scripts to ensure that every parameter is correct for your task!**
   
	the schedule of training from scratch is(assuming we are training on ucf101):
   
 ```
 	#assume you are at C3D root directory right now
 	1. cd examples/c3d_train_ucf101
 	2. sh create_volume_means.sh # compute the mean file of your dataset
 	3. sh train_ucf101.sh # train from scratch 
 	4. test_ucf101.sh # test the accuracy of training
 ```
 2. Fine-Tune on C3D
   
 	First you have to download pretrained model, then you can do fine-tuning like this:
   
 ```
 	#assume you are at C3D root directory right now
 	1. cd examples/c3d_finetuning
 	2. sh ucf101_finetuning.sh
 	3. sh ucf101_testing.sh
 ```


## 4.Classify C3D feature using SVM


After extracting features for batchs in each video using C3D tools, in orde to use SVM to classify the videos, we must get a descriptor for each video. We average the c3d features for each video, i.e., sum up those 4096 dimension's data and calculate the mean of them. I use matlab code below to do this job(using offered funtion `read_binary_blob`):
    
```matlab
function []= read_ucf101_c3d_feat(output_list_relative)
% Read c3d features (fc6) for videos in ucf101 dataset.
% For each video, average all its features and get a video descriptor.

    % rather than fileread, importdata save each line separetely.
    dir_list = importdata(output_list_relative);

    dim_feat = 4096;

    for i = 1 : size(dir_list, 1)
        dir_str = char(dir_list(i));
        feat_files = dir([dir_str, '/*.fc6-1']);
        num_feat = length(feat_files);
        feat = zeros(num_feat, dim_feat);
        for j = 1 : num_feat
            feat_path = strcat(dir_str, '/', feat_files(j).name);
            [~, feat(j,:)] = read_binary_blob(feat_path);
        end
        avg_feat = mean(feat, 1);
        avg_feat_double = double(avg_feat);
        fID = fopen(strcat(dir_str, '/c3d.fc6'), 'w');
		% libsvm requires that input data must be double
        fwrite(fID, avg_feat_double, 'double');
        fclose(fID);
    end
end
```
   
The input parameter is the file each line is a relative path to each video frames from the location of script. for example:
   
```
../output/c3d/ApplyEyeMakeup/v_ApplyEyeMakeup_g01_c01
```
   
It takes about ten minutes to run this script.  


When use libsvm to classify video features, there are two phases: training and testing. The declaration of training and testing functions are: 
   
```matlab
	model = libsvmtrain(train_label_vector, train_data_matrix, options);
	[label, accuracy, prob] = libsvmpredict(test_label_vector, test_data_matrix, model, options);
```
   
`train_label_vector` is a m by 1 vector, each element is a double value. `train_data_matrix` is a m by n matrix, each row is the data of one video. It is similar for predicting function.  

In order to construct input data in right way, I write several  wrapper functions for training and testing, which is more convenient to running:

```matlab
% create_svm_input_data.m

function [data_matrix] = create_svm_input_data(output_list_train)
% read the c3d feature(fc6) for each video, construct libsvm format data.

    dim_feat = 4096;
    dir_list = importdata(output_list_train);
    num_train_video = size(dir_list, 1);
    data_matrix = zeros(num_train_video, dim_feat);


    for i = 1 : num_train_video
        feat_path = strcat(char(dir_list(i)), '/c3d.fc6');
        fid = fopen(feat_path, 'r');
        data = fread(fid, 'double');
        fclose(fid);

        normed_data = data / norm(data);
        data_matrix(i, :) = normed_data;
    end

end

```
The `output_list_train` is the file contains relative path to each video directory. like:
```bash
../output/c3d/YoYo/v_YoYo_g07_c02
```
And then there is the file to train svm:
```matlab
%% train_ucf101.m

function [model] = train_ucf101(label_file_path, data_file_path, varargin)
    label_int = load(label_file_path);
    label_double = double(label_int);
    data = create_svm_input_data(data_file_path);
    model = libsvmtrain(label_double, data, varargin{:});
end
```
The `label_file_path` is the complete path to the file contains all training labels, including the file name, for example, `label_file_path` can be:
```bash
/data/foo/training_label.txt
```
And each line in `training_label.txt` contains only one label, for example:
```bash
# training_label.txt
0
0
0
0
1
1
1
1
```

There is the matlab file to test svm:

```matlab
%% test_ucf101.m

function [label, accuracy, predict_prob] = test_ucf101(test_label_path, test_data_path, model, varargin)
    label_int = load(test_label_path);
    label_double = double(label_int);
    label_size = size(label_double)
    data = create_svm_input_data(test_data_path);
    data_size = size(data)
    [label, accuracy, predict_prob] = libsvmpredict(label_double, data, model, varargin{:});
end
```

Note we can use `varagin` to pass parameters from a wrapper function to an internal function.


## 5.Reference
 1. [C3D User Guide](https://docs.google.com/document/d/1-QqZ3JHd76JfimY4QKqOojcEaf5g3JS0lNh-FHTxLag/edit).
 2. [LIBSVM](https://www.csie.ntu.edu.tw/~cjlin/libsvm/).

