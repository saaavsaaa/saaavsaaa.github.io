目前遇到的问题：

docker run -it -v $PWD:/work <CONTAINER> /work/train.py          
starting container process caused : permission denied".     
由于train.py没有执行权限，只要把要挂进去的文件chmod +x 就好了     

cannot uninstall 'numpy'，卸载不掉，强制卸载影响了别的东西，最后用了强制升级的方法解决     
sudo pip install numpy --ignore-installed numpy
