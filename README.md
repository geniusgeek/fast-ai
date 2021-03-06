# fast-ai  
http://course.fast.ai  

## Process to setup spot instance for the course (p2.xlarge)  
TODO Should be compiled into one script (T.B.D)  

**1) Request Spot Instance**  
AWS Console -> (Login) -> EC2 Dashboard -> Spot Requests
"Request Spot Instances"  
(only changed parameters shown - leave rest as default)  

Request type: Request  
AMI: Ubuntu Server 16.04 LTS (HVM)  
Instance type: p2.xlarge (delete c3.,...)  
Set your max price: e.g. 0.3  
(Next)  

Instance store: attach at launch  
EBS volumes / Size: 32 GiB  
Security groups: default  
(Next / Review)  

May need to change the security group settings if cannot login to instance:  
AWS Console -> (Login) -> EC2 Dashboard -> Instances  
Select instance -> Security Groups -> "default" (or which ever you are using)  
Tab "Inbound" -> Edit  

Type: SSH  
Protocol: TCP  
Port Range: 22  
Source: 0.0.0.0/0  

Type: TCP  
Protocol: TCP  
Port Range: 8888-8898  
Source: 0.0.0.0/0  

**2) Configure SSH**  
_in cygwin:_  
```
cd ~/.ssh
emacs config
```

copy / paste the HostName (Public DNS) of AWS instance  

Now it can look something like this:  
_Host aws-p2_  
_HostName ec2-35-166-166-129.us-west-2.compute.amazonaws.com_  
_User ubuntu_  
_IdentityFile "~/.ssh/aws-key.pem"_  

**3) Login**  
_in cygwin:_  
```
cd
ssh aws-p2
```

**4) Setup AWS Instance**  
_on aws-instance:_  
`git clone https://github.com/jonas-pettersson/fast-ai`  
(this is my own version of https://github.com/fastai/courses/ including own work)  

`./fast-ai/scripts/install-gpu.sh`  

```
sudo apt install python-pip
pip install --upgrade cliff
pip install kaggle-cli
sudo apt-get install unzip
```
(pip / unzip is not installed by the script. cliff needed for kaggle-cli)  

`pip install backports.shutil_get_terminal_size`  
(otherwise jupyter notebook does not work properly)  

emacs:  
```
sudo add-apt-repository -y ppa:cassou/emacs
sudo apt-get -qq update
sudo apt-get install -y emacs24-nox emacs24-el emacs24-common-non-dfsg
```

**5) Setup for Kaggle Competition**  
_on aws-instance:_  
```
cd
cd fast-ai
mkdir data
cd data
mkdir dogs-cats-redux
cd dogs-cats-redux
mkdir models
```
(this is the directory structure for dogs-cats-redux)  

```
cd
tmux
kg config -g -u "your_kaggle_username" -p "your_kaggle_password" -c "your_kaggle_competition"
```
```
cd ~/fast-ai/data/dogs-cats-redux
~/fast-ai/scripts/setup_kg.sh
```
(this is a setup script for kaggle, setting up directories, creating  
validation set, sample sets etc:  
 https://github.com/jonas-pettersson/fast-ai/blob/master/setup_kg.sh)  

**6) Transfer Files**  
need to transfer the files from your local machine using rsync, e.g.  
_in cygwin:_  
`rsync -avp --progress dogs-cats-redux-model.h5 aws-p2:~/fast-ai/data/dogs-cats-redux/models`  
This takes some time unfortunately and is the drawback of the spot-instance approach. Best is to consider carefully what is really need.  

**7) Start Working**  
_on aws-instance:_  
```
cd fast-ai
jupyter notebook
```

**8) Save Results**  
After work is done transfer the model(s) back with rsync:  
_in cygwin:_  
`rsync -avp --progress aws-p2:~/fast-ai/data/dogs-cats-redux/models/dogs-cats-redux-model.h5 .`  
May also want to save notebooks / scripts etc. to GitHub  
_on aws-instance:_  
```
git add ...
git commit -m "..."
git push origin master
```

**9) Terminate Instance**  
Make sure everything saved  
AWS Console -> (Login) -> EC2 Dashboard -> Spot Requests -> Actions ->  
cancel spot request + check box "Terminate instances"  
Check in EC2 Dashboard -> Instances that instance is terminated  

## ___setup_kg.sh___  
This is to setup the data directories for learning according to the directory structure in fast.ai-course.  
1. download files from kaggle  
2. unzip  
3. create directories  
4. move some files from training to validation directory  
5. copy some files to sample directories  

validation-size and sample-size are parameters  

_USAGE:  ./setup_kg.sh [validation-size] [sample-size]_  
