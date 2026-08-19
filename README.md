

# MGPBD: A Multigrid Accelerated Global XPBD Solver


[![Project Page](https://img.shields.io/badge/Project-Page-blue)](https://chunleili.github.io/project-page-mgpbd/)
   [![Repoducibility](https://www.replicabilitystamp.org/logo/Reproducibility-tiny.png)](http://www.replicabilitystamp.org#https-github-com-chunleili-mgpbd-2025-08-28t08-18-27-422z)


https://github.com/user-attachments/assets/389c2a28-cd70-4a2a-bd89-be09b1f1699c


https://github.com/user-attachments/assets/cf206a7a-bc3b-431c-a35f-fd8bfbff74d2

(Actually it is a Minotaur, but Houdini official example misnamed, so do I...😂)


https://github.com/user-attachments/assets/d571af63-5051-4c92-a94c-5833ed39f91f



https://github.com/user-attachments/assets/943d1391-4cb1-453d-95ad-2f6cff2ca60b

(Red: Ours; Blue: XPBD with same time budget (0.1s). The stiffer, the better.)



The code for SIGGRAPH 2025 paper "**MGPBD: A Multigrid Accelerated Global XPBD Solver**".

It is a Multigrid solver to solve the stalling issue of XPBD under high stiffness and high resolution. 

This code use a python front-end mixed with C++/CUDA back-end. C++/CUDA code is compiled to a dll with CUDA 12, and called by ctypes in python.


[Project Page](https://chunleili.github.io/project-page-mgpbd/)

[Paper(arix)](https://arxiv.org/abs/2505.13390)

[Latex source](https://github.com/user-attachments/files/22174539/siggraphconferencepapers25-131.zip)

[DOI](https://doi.org/10.1145/3721238.3730720)

[Poster](https://github.com/user-attachments/files/22174313/sig25.poster-v1.pptx)

SIGGRAPH 2025 presentation
 - recorded video [bilibili](https://www.bilibili.com/video/BV1z1bezDEtG/)  [youtube](https://youtu.be/heScPwJo4AU) 
 - [sildes](https://docs.google.com/presentation/d/1NcZVITDUfJIG9hKNJkx2TqsEaNnO3dNk/edit?usp=share_link&ouid=111038135074814190899&rtpof=true&sd=true)

[Detailed explaination video in Chinese](https://www.bilibili.com/video/BV1P18cziEt5/)

[Video (submitted originally)](https://youtu.be/mjLawWRBRj0)

My email: li_cl@foxmail.com 
Feel free to contact me for any questions.


CAUTION: The code is still not well organized and not clean. If it has bugs(highly possible), please let me know. 


# Pre-requisites
C++ end
- CUDA 12
- CMake

python end
```
pip install -r requirements.txt
```

I only tested it on **Windows 10, VS 2022, 4090**.

GPU: Any GPU above 8GB VRAM. 40 series GPU (4090,4080,4070, 4060 etc.) will be better.


# Replicating the paper's results
This is the script to replicate the experiement of Fig. 13 in the paper.

```
python replicate.py
```


The results are ply sequence: "result/latest/mesh/XXXX.ply". Also, there is a log file in the root directory will be generated(e.g. "residual_squash_bunnyBig_AMG_2025-08-29-15-23-25.txt").


# Build

```
buildcuda.bat
```

It will generate `cpp\mgcg_cuda\lib\fastmg.dll`, which is the C++ back-end.

# Run
Run this python script:

```
python engine/soft/soft3d.py -use_json=1 -json_path='data/scene/bunny_squash/bunny_squash.json'
```


The expected results of bunny squash case (Fig.13 in paper) will generated in `result/latest`.  You can aslo specify the output directory by `-out_dir=xxx`. The output meshes are in ply format: 0001.ply, 0002.ply, etc. 

There is also a log file (e.g. "residual_squash_bunnyBig_AMG_2025-08-29-15-23-25.txt") generated in the root directory, which contains the residual history (Fig.17 in paper).

Change the values in data/scene/bunny_squash/bunny_squash.json to change the parameters. For example, 
```
"solver_type": "XPBD",
"maxiter":10000,
```

and re-run the script 
```
python engine/soft/soft3d.py -use_json=1 -json_path='data/scene/bunny_squash/bunny_squash.json'
```

This will give you the XPBD results showed in Fig.17.


# Specify the parameters
Two ways:
1. Just use python argparse, see `common_args.py`. It is a bit messy but I have to support it because of backward compatibility. 
2. Use a json file. `-use_json=1  -json_path=xxx`. I recommend this way. You can easily change the parameters in json file without changing the code because python can dynamically add new attributes.

# Experiments
For cleanness and smaller repo size, I delete most of models, but only keep the bunny squash and armadillo collision cases. If you want to test all experiments in the paper, please let me know (propose a issue).

Just change "solver_type": "AMG", to "XPBD" and set "maxiter" to 100000. You will see that the XPBD is much harder to converge than AMG under high stiffness (Specify by "mu", the 2nd lame parameter) high resolution. This is due to the stalling issue of XPBD(See paper, the last picture).


# Q&A

## CUDA version problem
In Windows, you should specify the dll directory manually.  I use CUDA 12.5, so cuda_dir is set to "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.5/bin". If you use other version, please change it.

Related code(in the common_args.py and init_extlib.py).
```
parser.add_argument("-cuda_dir", type=str, default="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.5/bin")
...
os.add_dll_directory(cuda_dir)
```

## How to add your own models
The models are in tetgen format(.ele and .node). If you use Houdini, there is a [hip](https://github.com/chunleili/houdini_output_tet) to generate the tetrahedron mesh. 

## How to debug
The python-end is easy to debug. 

For C++ end, I recommond to use `python c++ debugger`， which is a VSCode extension. 

## Pylance report Invalid TypeForm

It is becuase the pylance does not support the `taichi` data types, which is not a big trouble but just annoying. Adding the following code to your `.vscode/settings.json` to stop the warning.
```
"python.analysis.diagnosticSeverityOverrides": {
        "reportInvalidTypeForm": false
    }
```

## CUDA ARCHITECTURES error
In some cases, you should manually set CMAKE_CUDA_ARCHITECTURES based on your GPU.
Change set(CMAKE_CUDA_ARCHITECTURES 89) in cpp/mgcg_cuda/CMakeLists.txt
Run cmake from cpp/mgcg_cuda in **cmd**(Powershell does not work for `set(CMAKE_CUDA_ARCHITECTURES 89).`)

## Data to reproduce the muscle experiments
I put them to the [release page](https://github.com/chunleili/mgpbd/releases/tag/data).


## Note

For cleanness, I delete most of messy codes in this repo. So some experiments may be missing. I put the old repo here: https://github.com/chunleili/tiPBD/tree/amg. 

If you have any questions, please feel free to propose a issue.
