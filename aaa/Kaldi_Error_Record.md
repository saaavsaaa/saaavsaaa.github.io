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

`
/export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
queue log file is exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p shared  --open-mode=append -e exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log 2>&1
sbatch: error: invalid partition specified: shared
sbatch: error: Batch job submission failed: Invalid partition name specified
/export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
queue log file is exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  -p shared --mem-per-cpu 2G  --open-mode=append -e exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log 2>&1
sbatch: error: invalid partition specified: shared
sbatch: error: Batch job submission failed: Invalid partition name specified
/export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
queue log file is exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p shared  --open-mode=append -e exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log 2>&1
sbatch: error: invalid partition specified: shared
sbatch: error: Batch job submission failed: Invalid partition name specified
/export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
queue log file is exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p shared  --open-mode=append -e exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log 2>&1
sbatch: error: invalid partition specified: shared
sbatch: error: Batch job submission failed: Invalid partition name specified
steps/train_mono.sh --boost-silence 1.25 --nj 20 --cmd slurm.pl --mem 2G data/aishell/train data/lang exp/mono
steps/train_mono.sh: Initializing monophone system.
feat-to-dim 'ark,s,cs:apply-cmvn --utt2spk=ark:data/aishell/train/split20/1/utt2spk scp:data/aishell/train/split20/1/cmvn.scp scp:data/aishell/train/split20/1/feats.scp ark:- | add-deltas ark:- ark:- |' - 
apply-cmvn --utt2spk=ark:data/aishell/train/split20/1/utt2spk scp:data/aishell/train/split20/1/cmvn.scp scp:data/aishell/train/split20/1/feats.scp ark:- 
WARNING (apply-cmvn[5.5.683~1-6ef3f]:Open():util/kaldi-table-inl.h:106) Failed to open script file data/aishell/train/split20/1/feats.scp
add-deltas ark:- ark:- 
ERROR (apply-cmvn[5.5.683~1-6ef3f]:SequentialTableReader():util/kaldi-table-inl.h:860) Error constructing TableReader: rspecifier is scp:data/aishell/train/split20/1/feats.scp

[ Stack-Trace: ]
/export1/kaldi/src/lib/libkaldi-base.so(kaldi::MessageLogger::LogMessage() const+0xb42) [0x7fe2cb6f4682]
apply-cmvn(kaldi::MessageLogger::LogAndThrow::operator=(kaldi::MessageLogger const&)+0x21) [0x5623208b2f2f]
apply-cmvn(kaldi::SequentialTableReader<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::SequentialTableReader(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)+0xc2) [0x5623208ba540]
apply-cmvn(main+0x79b) [0x5623208b0985]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xe7) [0x7fe2cab5eb97]
apply-cmvn(_start+0x2a) [0x5623208b010a]

kaldi::KaldiFatalErrorERROR (feat-to-dim[5.5.683~1-6ef3f]:main():feat-to-dim.cc:58) Could not read any features (empty archive?)

[ Stack-Trace: ]
/export1/kaldi/src/lib/libkaldi-base.so(kaldi::MessageLogger::LogMessage() const+0xb42) [0x7f519921b682]
feat-to-dim(kaldi::MessageLogger::LogAndThrow::operator=(kaldi::MessageLogger const&)+0x21) [0x558ad959e3f1]
feat-to-dim(main+0x2e9) [0x558ad959d7e3]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xe7) [0x7f5198685b97]
feat-to-dim(_start+0x2a) [0x558ad959d41a]

kaldi::KaldiFatalErrorerror getting feature dimension
`
