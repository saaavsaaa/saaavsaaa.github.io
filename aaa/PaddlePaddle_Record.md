目前遇到的问题：

docker run -it -v $PWD:/work <CONTAINER> /work/train.py          
starting container process caused : permission denied".     
由于train.py没有执行权限，只要把要挂进去的文件chmod +x 就好了     

cannot uninstall 'numpy'，卸载不掉，强制卸载影响了别的东西，最后用了强制升级的方法解决     
sudo pip install numpy --ignore-installed numpy

exec user process caused "exec format error" (参考https://www.tomczhen.com/2018/05/13/cross-platform-build-docker-image/)
docker run -it -v /root/book/01.fit_a_line:/work de2dd6b870dc python /work/train.py
