 记录使用Kaldi过程中遇到的错误：
  
kaldi/egs/multi_cn ：     
`  
--- Preparing pronunciations for OOV words ...
Traceback (most recent call last):
  File "/export1/kaldi/tools/sequitur-g2p/bin/g2p.py", line 4, in <module>
    __import__('pkg_resources').run_script('sequitur-g2p==1.0.1668.6', 'g2p.py')
  File "/home/manager/anaconda3/lib/python3.7/site-packages/pkg_resources/__init__.py", line 667, in run_script
    self.require(requires)[0].run_script(script_name, ns)
  File "/home/manager/anaconda3/lib/python3.7/site-packages/pkg_resources/__init__.py", line 1463, in run_script
    exec(code, namespace, namespace)
  File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/EGG-INFO/scripts/g2p.py", line 304, in <module>
    tool.run(main, options, args)
  File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/tool.py", line 63, in run
    status = runMain(main, options, args)
  File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/tool.py", line 99, in runMain
    status = main(options, args)
  File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/EGG-INFO/scripts/g2p.py", line 230, in main
    model = SequiturTool.procureModel(options, loadSample, log=log_stdout)
  File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/SequiturTool.py", line 217, in procureModel
    return tool.procureModel()
  File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/SequiturTool.py", line 163, in procureModel
    model = pickle.load(open(self.options.modelFile, 'rb'), encoding='latin1')
EOFError: Ran out of input
`     
查了下 cat local/prepare_dict.sh，由于服务器环境原因 wget http://sourceforge.net/projects/kaldi/files/sequitur-model4 -O conf/g2p_model 下载不下来，但是conf/g2p_model文件存在了，我在本地下载了上传上去的     
