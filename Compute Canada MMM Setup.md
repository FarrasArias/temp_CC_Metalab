[![N|Solid](https://metacreation.net/wp-content/themes/mama-word-v4.0.1/images/Brand-Regular-White-right-p-500.png)](https://metacreation.net/category/projects/)

# Compute Canada MMM Setup
## mmm_api installation

If you're unfamiliar with Compute Canada, make sure to check the introductory .md [here]().

> **Note:** There is currently a bug that is not allowing anybody to build the mmm_api correctly on CC. This .md will be updated once the current bug gets fixed, and we might not need to load python **3.6.3** to make it work, but for now the instructions try to copy Jeff's original environment as close as possible to try and avoid issues. When the current error gets fixed, we might change these instructions with python **3.9**.

1. First, make sure to clone the MMM_API into a folder in your CC machine.
```sh
https://<TOKENNUMBERS>@github.com/jeffreyjohnens/MMM_API.git
```
2. First we need to load python 3.6.3, so we must load nixpkgs/16.09 to the CC machine.
```sh
module load nixpkgs/16.09
module load python/3.6.3
```
3. Then we must create an environment.
```sh
module load python/3.6.3                       
virtualenv --no-download ./ENV                # ENV is the name of the environment
source ./ENV/bin/activate
pip install --no-index --upgrade pip          
pip install --no-index --upgrade setuptools

```
4. Then we must install all requirements. The requirements.txt file can be found [here]().
```sh
pip install -r mmm_api_requirements.txt
```
> **Note:** if you wish to install or reinstall any package individually, make sure to use the --no-index flag to install from the Compute Canada Wheels

5. **IMPORTANT:** Deactivate the environment once you installed all packages
```sh
deactivate
```
6. Now, in order to load the correct protobuf version, we must purge the nixpkgs module.
```sh
module --force purge
```
7. After this, we can load the protobuf module, first loading the correct environment.
```sh
module load StdEnv/2020
module load protobuf/3.12.3
```
8. Once we do this, we can activate the python3.6.3 environment. This will allow us to have a python 3.6.3 env and use the correct version of protobuf.
```sh
./ENV/bin/activate
```
9. Now, navigate into the MMM_API python folder and download pybind.
```sh
cd MMM_API/python
rm -r pybind11
git clone https://github.com/pybind/pybind11.git
```
10. Finally, we must manually (for now) link the torch path in the CMakeLists file. The path to your torch should be in your environment in a path similar to *ENV/lib/python3.6/site-packages/torch*. Open the CMakeLists_PY_TORCH_CC.txt file (with nano) and change the path in line 26 with the **global path**.
```sh
SET (TORCH_DIR "/home/USERNAME/scratch/ENV/lib/python3.6/site-packages/torch")
```
11. That's it. Go back to the main MMM_API folder and run the following (the flag is important):
```sh
bash python_build.sh --cc
```
Everything should get installed correctly in your python environment!
