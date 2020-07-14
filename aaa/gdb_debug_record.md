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
  
util/kaldi-table.cc   
bool ReadScriptFile(const std::string &rxfilename, ...  /export1/kaldi/egs/cvte/online/data/test/feats.scp    
util/kaldi-io-inl.h:Input::Open:OpenInternal     
kaldi-io.cc:InputType ClassifyRxfilename(const std::string &filename)  输入类型，如"-"标准输入kStandardInput、文件等   
kaldi-io.cc:bool Input::OpenInternal(const std::string &rxfilename,...  scp  这里是文件类型if (type ==  kFileInput) {    impl_ = new FileInputImpl(); return InitKaldiInputStream(impl_->Stream(), contents_binary);   

compute-cmvn-stats.cc:   
105	            std::string utt = uttlist[i];   
106	            if (!feat_reader.HasKey(utt)) {   
kaldi::RandomAccessTableReader<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::HasKey (this=0x7fffffffdbe0, key="aaa-a1") at ../util/kaldi-table-inl.h:2551
2551	bool RandomAccessTableReader<Holder>::HasKey(const std::string &key) {   


