In this blog, we'll look at how to setup custom conda environments on JupyterHub application launched with EMR and accessing them through JupyterHub UI.


- [SSH](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-ssh.html) into the master node of the EMR cluster. (Make sure to add the application JupyterHub while launching your cluster)

- Switch to jupyterhub root user:
  ```
  $ sudo docker exec -it jupyterhub /bin/bash
  ```

- Create desired virtualenv:
  ```
  root@jupyterhub:~# conda create -n myenv python=3.x -y
  ```

- (optional) Check virtualenv list:
  ```
  root@jupyterhub:~# conda info --envs
  ```

- Switch to the desired virtualenv that you have created in the first step:
  ```
  root@jupyterhub:~# source activate myenv
  ```

- Install pip, ipykernel, if they are not already installed:
  ```
  (myenv) root@jupyterhub:~# conda install pip
  (myenv) root@jupyterhub:~# pip install ipykernel
  ```

- Link your virtualenv to the kernel:
  ```
  (myenv) root@jupyterhub:~# python -m ipykernel install --user --name myenv --display-name "Python 3.x"
  ```

- Deactivate virtualenv:
  ```
  (myenv) root@jupyterhub:~# source deactivate
  ```

- If you would like to link other virtualenv's to the kernel, repeat steps 5-8


- Open [JupyterHub UI](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-jupyterhub-connect.html).


- Create a new jupyter notebook with the desired kernel(virtualenv) or change the kernel for an existing notebook inside the notebook under Kernel → Change kernel.
