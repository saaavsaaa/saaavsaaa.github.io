目前遇到的问题：

docker run -it -v $PWD:/work <CONTAINER> /work/train.py     
starting container process caused : permission denied".     
由于train.py没有执行权限，只要把要挂进去的文件chmod +x 就好了     
