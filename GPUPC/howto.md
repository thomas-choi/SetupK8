# Setup a PC with NVIDIA's GPU to run Ubuntu

### 1. Setup PC to run Ubuntu 
 I have two PC with 3090 and 3060, 30 GB memory and 1TB drive. Both of them installed with Ubuntu 20.04 LTS.  
 
### 2. Setup Lambda Stack 
 The Lambda Stack builds Docker containers with Nvidia on Ubuntu. [Here is the documentation for the installation of the Stack.](https://lambdalabs.com/blog/set-up-a-tensorflow-gpu-docker-container-using-lambda-stack-dockerfile) The containers have already included those import Machine Learning packages such as TensorFlow, PyTorch, caffe, etc. Run the follow commands:
 ~~~ Shell
 LAMBDA_REPO=$(mktemp) && \
wget -O${LAMBDA_REPO} https://lambdalabs.com/static/misc/lambda-stack-repo.deb && \
sudo dpkg -i ${LAMBDA_REPO} && rm -f ${LAMBDA_REPO} && \
sudo apt-get update && sudo apt-get install -y lambda-stack-cuda
sudo reboot
 ~~~

 ### 3. Install Docker and Nvidia toolkit
 ~~~ Shell
 sudo apt-get install docker.io nvidia-container-toolkit
 ~~~

