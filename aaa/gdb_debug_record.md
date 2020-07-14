#调试未完，临时记录过程

cd /export1/kaldi/src/featbin   
gdb ./compute-cmvn-stats   
r --spk2utt=ark:/export1/kaldi/egs/cvte/online/data/test/spk2utt scp:/export1/kaldi/egs/cvte/online/data/test/feats.scp ark,scp:/export1/kaldi/egs/cvte/online/work/data/cmvn_test.ark,/export1/kaldi/egs/cvte/online/work/data/cmvn_test.scp

b compute-cmvn-stats.cc:59

r   
p argv[3]   
br compute-cmvn-stats.cc:111   
s

kaldi/src/util/kaldi-table.cc   ClassifyWspecifier 解析表单格式   

b compute-cmvn-stats.cc:97   
RandomAccessBaseFloatMatrixReader feat_reader(rspecifier);   
util/kaldi-table-inl.h:2521   
kaldi-table.cc: 250: RspecifierType rs = ClassifyRspecifier(rspecifier, NULL, &opts);(解析表单格式)   
rs = kScriptRspecifier:case kScriptRspecifier:impl_ = new SequentialTableReaderScriptImpl<Holder>():util/kaldi-table-inl.h:template<class Holder>  class SequentialTableReaderScriptImpl    
  
  
