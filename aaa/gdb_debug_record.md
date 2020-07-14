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

kaldi::RandomAccessTableReaderScriptImpl<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::HasKeyInternal (this=0x5555557a3dc0, key="aaa-a1", preload=false)
    at ../util/kaldi-table-inl.h:1703   
1703	  virtual bool HasKeyInternal(const std::string &key, bool preload) {

main (argc=<optimized out>, argv=<optimized out>) at compute-cmvn-stats.cc:111
111	            const Matrix<BaseFloat> &feats = feat_reader.Value(utt);

kaldi::RandomAccessTableReader<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::Value (key="aaa-a1", this=0x7fffffffdbe0) at ../util/kaldi-table-inl.h:2562
2562	  CheckImpl();

kaldi::RandomAccessTableReader<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::CheckImpl (this=0x7fffffffdbe0) at ../util/kaldi-table-inl.h:2590
2590	void RandomAccessTableReader<Holder>::CheckImpl() const {

kaldi::RandomAccessTableReader<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::Value (key="aaa-a1", this=0x7fffffffdbe0) at ../util/kaldi-table-inl.h:2563
2563	  return impl_->Value(key);

kaldi::RandomAccessTableReaderScriptImpl<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::Value (this=0x5555557a3dc0, key="aaa-a1")
    at ../util/kaldi-table-inl.h:1679    
1679	  virtual const T&  Value(const std::string &key) {

/util/kaldi-table-inl.h:1703

kaldi::RandomAccessTableReaderScriptImpl<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::HasKeyInternal (this=0x5555557a3dc0, key="aaa-a1", preload=true)
    at ../util/kaldi-table-inl.h:1703    
1703	  virtual bool HasKeyInternal(const std::string &key, bool preload) {

1731	        if (script_[key_pos].second[script_[key_pos].second.size()-1] == ']') {     
script_ = std::vector of length 1, capacity 1 = {{first = "aaa-a1", second = "/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.1.ark:7"}}     
data_rxfilename = "/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.1.ark:7"

kaldi::Input::Open (binary=0x0, rxfilename="/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.1.ark:7", this=0x5555557a3dc8) at ../util/kaldi-io-inl.h:27
27	  return OpenInternal(rxfilename, true, binary);




