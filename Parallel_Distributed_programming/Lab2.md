# Lab 2
使用如下来登录UPPMAX:
```
ssh -Y junjie@rackham.uppmax.uu.se
```
使用如下来上传本地文件到UPPMAX:
```
scp lab2.tar.gz junjie@rackham.uppmax.uu.se:/home/junjie
```
解压指令:
```
gunzip lab2.tar.gz 
tar xvf lab2.tar
```
加载相应模块:
```
module load gcc/10.2.0 openmpi/4.1.0
module load ddt
```
使用Infiniband服务器计算加速技术:
```
echo "OMPI_MCA_btl_openib_allow_ib=1" >> $HOME/.bashrc
```
