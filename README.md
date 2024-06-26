# Fortran2Cpp

## Introduction
Fortran has been a widely used programming language for scientific computation since 1957. With technological advancements, modern languages like C++ have become preferable for some projects due to their greater flexibility and features. However, the lack of an accurate and comprehensive Fortran-to-C++ translation dataset means that existing large models, including GPT-4, often struggle to perform this task effectively, resulting in translations that may fail to compile or pass unit tests. Fortran2Cpp aims to address this issue.

## Model

We fine-tuned several popular pre-trained models,  including 
* WizardCoder-15B-V1.0,
* CodeLlama-13b-Instruct-hf,
* starcoder,
* starcoder2,
* Magicoder-S-DS-6.7B, and
* deepseek-coder-33b-instruct. 

After the finetuning, the deepseek-coder-33b-instruct shows the greatest improvement when checking with the CodeBLEU Score. Thus we finally use the deepseek-coder-33b-instruct as the backbone of Fortran2CPP.  

The Model is avaliable on Huggingface: [Fortran2Cpp](https://huggingface.co/Bin12345/Fortran2Cpp) 

**NOTE:** Currently, the model is only trained by using the f90 code. We are still training the model. We will continue to update the Fortran2Cpp.

## Evaluation
We compared with various models (WizardCoder-15B-V1.0, CodeLlama-13b-Instruct-hf, starcoder, Magicoder-S-DS-6.7B, deepseek-coder-33b-instruct and GPT-4) on [HPC_Fortran_CPP](https://huggingface.co/datasets/Bin12345/HPC_Fortran_CPP). And compared the CodeBLEU Score of the generated results.

The CodeBLEU Score Comparison is shown in the figure below:

![CodeBLEU Score Image](Figures/CodeBLEU.png)

### Reproduce Steps
1. Enter into Evaluation folder

```
cd Evaluation
```

2. Generate the results. Go the script `text_generation_pipline.py`. Add your own huggingface token to line 16. Modify the path where you want to store your results in line 55. Then select the model that you want to test between line 8 and line 13.

Run:
```
python text_generation_pipline.py
```

This will generate the results and compress each result to one line for the further CodeBLEU Score test.

This script does the following
* model: Bin12345/F2C-Translator // configurable to use any other models
* dataset: Bin12345/HPC_Fortran_CPP
* Translate Fortran Code in the dataset to C++ code
* Write to a log

3. Test CodeBLEU Score by using the following command

```
cd CodeBLEU
python calc_code_bleu.py --refs Fortran2Cpp/Evaluation/Groundtruth_C++.txt --hyp <path/to/your/results/txt/file> --lang cpp --params 0.25,0.25,0.25,0.25
```

* Run inference on Slurm Cluster: Your should use this script to start the inference: `sbatch <The/following/script>`

```
#!/bin/bash
#SBATCH -N 1
#SBATCH -C gpu&hbm80g
#SBATCH -G 4
#SBATCH -q regular
#SBATCH -J model_training
#SBATCH --mail-user=<Your/Email>
#SBATCH --mail-type=ALL
#SBATCH -t 00:30:00
#SBATCH -A <Your/Project>

# Load conda  
echo "loading conda..."
module load conda 
conda activate <Your/Conda/env>

# Huggingface Setting 
echo "Setting Huggingface..."
export HF_HOME=$SCRATCH/huggingface 
export HF_TOKEN=<Your/HF/ToKen>

# OpenMP settings:
echo "Setting OMP..."
export OMP_NUM_THREADS=1
export OMP_PLACES=threads
export OMP_PROC_BIND=spread

# Set CFLAGS and LDFLAGS and CUTLASS 
export CFLAGS="-I/<Your/Conda/env>/include $CFLAGS"
export LDFLAGS="-L/<Your/Conda/env>/lib $LDFLAGS"
export CUTLASS_PATH=$HOME/cutlass

# run the application:
echo "Start to run the inference..."
chmod +x <Your/inference/file/path>
srun -n 1 -c 8 --cpu_bind=cores -G 4 --gpu-bind=none  <Your/inference/file/path> > <Your/log/file/path> 2>&1 
```

## Dataset Generation 

1. Input your OpenAI Key, Output Json file path.
```
cd dataset_generation
```
Modify `line 543-550` in `engine_F2C.py` file.

2. Start the dataset generation

```
python engine_F2C.py
```

The dataset that we used is included in `F2C-Translator/data/F2C_dialogue_25K.json` file.

## Inference and Demo
The demo code is modified from [OpenCodeInterpreter](https://github.com/OpenCodeInterpreter/OpenCodeInterpreter/tree/main/demo). Appreciate for their great project!

1. Create conda and install packages
```
cd Web_demo
conda create -n demo python=3.10
conda activate demo
pip install -r requirements.txt
```

2.  Start the demo
```
python chatbot.py
```

**NOTE:** This demo will not use the interpreter function. This feature is a potential extension for this work.

## Hardware requirements

We used 6 A100 GPUs with 80GB memory for the training.  (Use Lora)

We used 2 A100 GPUs with 80GB memory for the inference. 

## Contact 
If you have any inquiries, please feel free to raise an issue or reach out to leib2765@gmail.com.

## Citation
We will complete the technical introduction paper before mid-May.

## Acknowledgments
Appreciation to Lawrence Livermore National Laboratory for their financial support of this project.
