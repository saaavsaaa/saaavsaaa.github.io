nnet3-latgen-faster --frame-subsampling-factor=3 --frames-per-chunk=50 --extra-left-context=0 --extra-right-context=0 --extra-left-context-initial=-1 --extra-right-context-final=-1 --minimize=false --max-active=7000 --min-active=200 --beam=15.0 --lattice-beam=8.0 --acoustic-scale=1.0 --allow-partial=true --word-symbol-table=exp/chain/tdnn/graph/words.txt exp/chain/tdnn/final.mdl exp/chain/tdnn/graph/HCLG.fst "ark,s,cs:apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:data/test/split1/1/utt2spk scp:data/test/split1/1/cmvn.scp scp:data/test/split1/1/feats.scp ark:- |" "ark:|lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"



调试时 将kaldi/src/kaldi.mk中-O1改成”-O0 -DKALDI_PARANOID”，不改也行，就是有些地方的参数拿不到     



compute-cmvn-stats --spk2utt=ark:data/test/spk2utt scp:data/test/feats.scp ark,scp:/export1/kaldi/egs/cvte/online/work/data/cmvn_test.ark,/export1/kaldi/egs/cvte/online/work/data/cmvn_test.scp 
ASSERTION_FAILED (compute-cmvn-stats[5.5.683~2-f4d5f]:InitCmvnStats():cmvn.cc:27) Assertion failed: (dim > 0)

[ Stack-Trace: ]
/export1/kaldi/src/lib/libkaldi-base.so(kaldi::MessageLogger::LogMessage() const+0xb42) [0x7f93d8fef612]
/export1/kaldi/src/lib/libkaldi-base.so(kaldi::KaldiAssertFailure_(char const*, char const*, int, char const*)+0x6e) [0x7f93d8ff030e]
/export1/kaldi/src/lib/libkaldi-transform.so(kaldi::AccCmvnStats(kaldi::VectorBase<float> const&, float, kaldi::MatrixBase<double>*)+0) [0x7f93d96ee83e]
compute-cmvn-stats(main+0x510) [0x55d6dc918a1f]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xeb) [0x7f93d8a7de0b]
compute-cmvn-stats(_start+0x2a) [0x55d6dc91822a]


feature-common-inl.h:67  rows_out=0

int32 rows_out = NumFrames(wave.Dim(), computer_.GetFrameOptions()),cols_out = computer_.Dim();

feat/feature-window.cc:42 int32 NumFrames(int64 num_samples = 320 , const FrameExtractionOptions &opts, bool flush)

feat/feature-window.h:106 
  int32 WindowSize() const {return static_cast<int32>(samp_freq (=16000 16Hz) * 0.001 * frame_length_ms(=10));}
  
 opts.snip_edges  & num_samples (wave.Dim()) < frame_length
 
feat/feature-common-inl.h:65  int32 rows_out = NumFrames(wave.Dim(), computer_.GetFrameOptions()),

main (argc=<optimized out>, argv=<optimized out>) at compute-fbank-feats.cc:148
148	        fbank.ComputeFeatures(waveform, wave_data.SampFreq(),

waveform.Dim() = 320

87:SequentialTableReader<WaveHolder> reader(wav_rspecifier);
  
 kaldi::SequentialTableReader<kaldi::WaveHolder>::SequentialTableReader (
    this=0x7fffffffd6b0, 
    rspecifier="scp,p:/export1/kaldi/egs/cvte/online/exp/make_fbank/data/wav.1.scp") at ../util/kaldi-table-inl.h:857
857	SequentialTableReader<Holder>::SequentialTableReader(const std::string
  
91	  SequentialTableReaderScriptImpl(): state_(kUninitialized) { }

380	      SplitStringOnFirstSpace(line, &key_, &rest);
(gdb) p line
$3 = "aaa-a1 /export1/kaldi/egs/cvte/online/data/wav/aaa/a1.wav"

kaldi::SplitStringOnFirstSpace (
    str="aaa-a1 /export1/kaldi/egs/cvte/online/data/wav/aaa/a1.wav", 
    first=0x5555557a2eb0, rest=0x7fffffffcee0) at text-utils.cc:122
122	                             std::string *rest) {
(gdb) s
126	  I first_nonwhite = str.find_first_not_of(white_chars);
(gdb) p white_chars 
$4 = 0x7ffff7b9268c " \t\n\r\f\v"

kaldi::SequentialTableReaderScriptImpl<kaldi::WaveHolder>::NextScpLine (
    this=0x5555557a2e10) at ../util/kaldi-table-inl.h:381
381	      if (!key_.empty() && !rest.empty()) {
(gdb) p rest
$9 = "/export1/kaldi/egs/cvte/online/data/wav/aaa/a1.wav"

kaldi::SequentialTableReaderScriptImpl<kaldi::WaveHolder>::Next (
    this=0x5555557a2e10) at ../util/kaldi-table-inl.h:227
227	      if (opts_.permissive) {

kaldi::SequentialTableReader<kaldi::WaveHolder>::SequentialTableReader (
    this=0x7fffffffd6b0, 
    rspecifier="scp,p:/export1/kaldi/egs/cvte/online/exp/make_fbank/data/wav.1.scp") at ../util/kaldi-table-inl.h:861
861	}

main (argc=6, argv=0x7fffffffe158) at compute-fbank-feats.cc:88
88	    BaseFloatMatrixWriter kaldi_writer;  // typedef to TableWriter<something>.

92	      if (!kaldi_writer.Open(output_wspecifier))
(gdb) p output_wspecifier
$13 = "ark:/export1/kaldi/egs/cvte/online/work/data/to-input-copy.ark"

106	    for (; !reader.Done(); reader.Next()) {
(gdb) s
kaldi::SequentialTableReader<kaldi::WaveHolder>::Done (this=0x7fffffffd6b0)
    at ../util/kaldi-table-inl.h:949
949	  CheckImpl();
(gdb) s
kaldi::SequentialTableReader<kaldi::WaveHolder>::CheckImpl (
    this=0x7fffffffd6b0) at ../util/kaldi-table-inl.h:2582
2582	void SequentialTableReader<Holder>::CheckImpl() const {

main (argc=<optimized out>, argv=<optimized out>) at compute-fbank-feats.cc:106
106	    for (; !reader.Done(); reader.Next()) {
(gdb) s
107	      num_utts++;
(gdb) s
108	      std::string utt = reader.Key();
utt = "aaa-a1"
(gdb) n
109	      const WaveData &wave_data = reader.Value();
看看读取的代码
  











#调试过程


gdb ./compute-fbank-feats     

r --write-utt2dur=ark,t:/export1/kaldi/egs/cvte/online/exp/make_fbank/data/utt2dur.1 --verbose=2 --config=/export1/kaldi/egs/cvte/online/conf/fbank.conf scp,p:/export1/kaldi/egs/cvte/online/exp/make_fbank/data/wav.1.scp ark:/export1/kaldi/egs/cvte/online/work/data/to-input-copy.ark

b compute-fbank-feats.cc:87




main (argc=6, argv=0x7fffffffe158) at compute-fbank-feats.cc:88
88	    BaseFloatMatrixWriter kaldi_writer;  // typedef to TableWriter<something>.
(gdb) s
kaldi::TableWriter<kaldi::KaldiObjectHolder<kaldi::MatrixBase<float> > >::TableWriter (this=0x7fffffffd6b8) at ../util/kaldi-table.h:372
372	  TableWriter(): impl_(NULL) { }
(gdb) s
main (argc=6, argv=0x7fffffffe158) at compute-fbank-feats.cc:91
91	    if (output_format == "kaldi") {
(gdb) n
92	      if (!kaldi_writer.Open(output_wspecifier))
(gdb) s
kaldi::TableWriter<kaldi::KaldiObjectHolder<kaldi::MatrixBase<float> > >::Open
    (this=0x7fffffffd6b8, 
    wspecifier="ark:/export1/kaldi/egs/cvte/online/work/data/to-input-copy.ark") at ../util/kaldi-table-inl.h:1480
1480	bool TableWriter<Holder>::Open(const std::string &wspecifier) {
(gdb) s
1481	  if (IsOpen()) {
(gdb) s
1486	  WspecifierType wtype = ClassifyWspecifier(wspecifier, NULL, NULL, NULL);
(gdb) s
kaldi::ClassifyWspecifier (
    wspecifier="ark:/export1/kaldi/egs/cvte/online/work/data/to-input-copy.ark", archive_wxfilename=0x0, script_wxfilename=0x0, opts=0x0) at kaldi-table.cc:138
138	                                  WspecifierOptions *opts) {
























-----------------------------------------------------------------------

需要检查一下内容和创建过程:/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.1.ark:7     
/export1/kaldi/src/featbin/copy-feats     
gdb ./copy-feats     
r --compress=true --write-num-frames=ark,t:/export1/kaldi/egs/cvte/online/exp/make_fbank/data/utt2num_frames.1 ark:- ark,scp:/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.1.ark,/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.1.scp   

---------------------------------------

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

170	    if (\*d == ':') return kOffsetFileInput;  // Filename is like

std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string (this=0x5555557a4358) at kaldi-io.cc:556
556	class OffsetFileInputImpl: public InputImplBase {   
kaldi::Input::OpenInternal (this=0x5555557a3dc8, rxfilename="/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.1.ark:7", file_binary=<optimized out>, contents_binary=0x0) at kaldi-io.cc:556    
556	class OffsetFileInputImpl: public InputImplBase {

kaldi::RandomAccessTableReaderScriptImpl<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::HasKeyInternal (this=0x5555557a3dc0, key="aaa-a1", preload=true)
    at ../util/kaldi-table-inl.h:1768   
1768	          if (!input_.Open(data_rxfilename)) {   
kaldi::Input::Stream (this=0x5555557a3dc8) at kaldi-io.cc:826    
826	std::istream &Input::Stream() {   

kaldi::OffsetFileInputImpl::Stream (this=0x5555557a4350) at kaldi-io.cc:636   
636	  virtual std::istream &Stream() {   

kaldi::RandomAccessTableReaderScriptImpl<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::HasKeyInternal (this=0x5555557a3dc0, key="aaa-a1", preload=true)
    at ../util/kaldi-table-inl.h:1774   

kaldi::KaldiObjectHolder<kaldi::Matrix<float> >::Value (this=0x5555557a3e38)
    at ../util/kaldi-holder-inl.h:95   
95	  T &Value() {   

main (argc=<optimized out>, argv=<optimized out>) at compute-cmvn-stats.cc:112
112	            if (!is_init) {  


---------------------------------------------------------------------------------------------------

make_fbank.sh utt2num_frames   copy-feats ark:xxx.ark ark,t:xxx.txt    

run.pl JOB=1:1 exp/make_fbank/data/make_fbank_test.JOB.log     compute-fbank-feats  --write-utt2dur=ark,t:exp/make_fbank/data/utt2dur.JOB --verbose=2      --config=conf/fbank.conf scp,p:exp/make_fbank/data/wav.JOB.scp ark:- \|     copy-feats --compress=true --write-num-frames=ark,t:exp/make_fbank/data/utt2num_frames.JOB ark:-      ark,scp:/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.JOB.ark,/export1/kaldi/egs/cvte/online/work/data/raw_fbank_test.JOB.scp

run.pl JOB=1:1 exp/make_fbank/data/make_fbank_test.JOB.log     compute-fbank-feats  --write-utt2dur=ark,t:exp/make_fbank/data/utt2dur.JOB --verbose=2      --config=conf/fbank.conf scp,p:exp/make_fbank/data/wav.JOB.scp ark:/export1/kaldi/egs/cvte/online/work/data/to-input-copy.ark

compute-fbank-feats $vtln_opts $write_utt2dur_opt --verbose=2 \
 --config=$fbank_config scp,p:$logdir/wav.1.scp ark:/export1/kaldi/egs/cvte/online/work/data/to-input-copy.ark \
 || exit 1;     
/export1/kaldi/src/featbin/compute-fbank-feats     
r --write-utt2dur=ark,t:/export1/kaldi/egs/cvte/online/exp/make_fbank/data/utt2dur.1 --verbose=2 --config=/export1/kaldi/egs/cvte/online/conf/fbank.conf scp,p:/export1/kaldi/egs/cvte/online/exp/make_fbank/data/wav.1.scp ark:/export1/kaldi/egs/cvte/online/work/data/to-input-copy.ark

kaldi::OfflineFeatureTpl<kaldi::FbankComputer>::OfflineFeatureTpl (opts=..., 
    this=0x7fffffffd8c0) at ../feat/feature-common.h:119    
119	      feature_window_function_(computer_.GetFrameOptions()) { }
kaldi::FbankComputer::FbankComputer (this=0x7fffffffd8c0, opts=...)
    at feature-fbank.cc:26      
26	FbankComputer::FbankComputer(const FbankOptions &opts):

31	  int32 padded_window_size = opts.frame_opts.PaddedWindowSize();    
kaldi::FrameExtractionOptions::PaddedWindowSize (this=0x7fffffffd850)
    at ../feat/feature-window.h:113     
113	    return (round_to_power_of_two ? RoundUpToNearestPowerOfTwo(WindowSize()) :

kaldi::FeatureWindowFunction::FeatureWindowFunction (this=0x7fffffffd970, 
    opts=...) at feature-window.cc:110
110	  int32 frame_length = opts.WindowSize();

kaldi::Vector<float>::Resize (this=this@entry=0x7fffffffd970, 
    dim=dim@entry=400, resize_type=resize_type@entry=kaldi::kSetZero)
    at kaldi-vector.cc:190
190	void Vector<Real>::Resize(const MatrixIndexT dim, MatrixResizeType resize_type) {

221	  Init(dim);   (注:dim = 400)

kaldi::FeatureWindowFunction::FeatureWindowFunction (this=0x7fffffffd970, 
    opts=...) at feature-window.cc:113
113	  double a = M_2PI / (frame_length-1);  (: a = 0.015747331596941319)

116	    if (opts.window_type == "hanning") {

124	    } else if (opts.window_type == "povey") {  // like hamming but goes to zero at edges.
125	      window(i) = pow(0.5 - 0.5*cos(a * i_fl), 0.85);

until 134

main (argc=6, argv=0x7fffffffe158) at compute-fbank-feats.cc:81
81	    if (utt2spk_rspecifier != "" && vtln_map_rspecifier != "")

b compute-fbank-feats.cc:87

kaldi::SequentialTableReaderScriptImpl<kaldi::WaveHolder>::NextScpLine (
    this=0x5555557a2e10) at ../util/kaldi-table-inl.h:361
361	  void NextScpLine() {

392	          data_rxfilename = rest;
(gdb) p rest
$21 = "data/wav/aaa/a1.wav"

kaldi::SequentialTableReaderScriptImpl<kaldi::WaveHolder>::Next (
    this=0x5555557a2e10) at ../util/kaldi-table-inl.h:226
226	      if (Done()) return;

kaldi::SequentialTableReaderScriptImpl<kaldi::WaveHolder>::Done (
    this=0x5555557a2e10) at ../util/kaldi-table-inl.h:140
140	  virtual bool Done() const {

kaldi::SequentialTableReaderScriptImpl<kaldi::WaveHolder>::Next (
    this=0x5555557a2e10) at ../util/kaldi-table-inl.h:227
227	      if (opts_.permissive) {

230	        if (EnsureObjectLoaded()) return;  // Success.

kaldi::SequentialTableReaderScriptImpl<kaldi::WaveHolder>::EnsureObjectLoaded (
    this=0x5555557a2e10) at ../util/kaldi-table-inl.h:296
296	  bool EnsureObjectLoaded() {

305	        ans = data_input_.Open(data_rxfilename_, NULL);

kaldi::Input::Open (binary=0x0, rxfilename="data/wav/aaa/a1.wav", 
    this=0x5555557a2e68) at ../util/kaldi-io-inl.h:27
27	  return OpenInternal(rxfilename, true, binary);
(gdb) p rxfilename 
$23 = "data/wav/aaa/a1.wav"

std::basic_ifstream<char, std::char_traits<char> >::open (__mode=12, 
    __s=0x5555557aa400 "data/wav/aaa/a1.wav", this=0x5555557aa438)
    at kaldi-io.cc:386
386	    is_.open(MapOsPath(filename).c_str(),

aaa-a1 /export1/kaldi/egs/cvte/online/data/wav/aaa/a1.wav

kaldi::SequentialTableReader<kaldi::WaveHolder>::Done (this=0x7fffffffd6b0)
    at ../util/kaldi-table-inl.h:950
950	  return impl_->Done();

main (argc=<optimized out>, argv=<optimized out>) at compute-fbank-feats.cc:106
106	    for (; !reader.Done(); reader.Next()) {
107	      num_utts++;
108	      std::string utt = reader.Key();
109	      const WaveData &wave_data = reader.Value();

kaldi::SequentialTableReader<kaldi::WaveHolder>::Value (this=0x7fffffffd6b0)
    at ../util/kaldi-table-inl.h:935
935	  CheckImpl();

kaldi::SequentialTableReader<kaldi::WaveHolder>::CheckImpl (
    this=0x7fffffffd6b0) at ../util/kaldi-table-inl.h:2582
2582	void SequentialTableReader<Holder>::CheckImpl() const {

kaldi::SequentialTableReader<kaldi::WaveHolder>::Value (this=0x7fffffffd6b0)
    at ../util/kaldi-table-inl.h:936
936	  return impl_->Value();  // This may throw (if EnsureObjectLoaded() returned false you

main (argc=<optimized out>, argv=<optimized out>) at compute-fbank-feats.cc:110
110	      if (wave_data.Duration() < min_duration) {

kaldi::WaveData::Duration (this=<optimized out>) at ../feat/wave-reader.h:129
129	  BaseFloat Duration() const { return data_.NumCols() / samp_freq_; }

main (argc=<optimized out>, argv=<optimized out>) at compute-fbank-feats.cc:110
110	      if (wave_data.Duration() < min_duration) {

kaldi::MatrixBase<float>::NumRows (this=<optimized out>)
    at ../matrix/kaldi-matrix.h:64
64	  inline MatrixIndexT  NumRows() const { return num_rows_; }

main (argc=<optimized out>, argv=<optimized out>) at compute-fbank-feats.cc:115
115	      int32 num_chan = wave_data.Data().NumRows(), this_chan = channel;

145	      SubVector<BaseFloat> waveform(wave_data.Data(), this_chan);

148	        fbank.ComputeFeatures(waveform, wave_data.SampFreq(),

kaldi::OfflineFeatureTpl<kaldi::FbankComputer>::ComputeFeatures (
    this=0x7fffffffd8c0, wave=..., sample_freq=16000, vtln_warp=1, 
    output=0x7fffffffd6f0) at ../feat/feature-common-inl.h:29
29	void OfflineFeatureTpl<F>::ComputeFeatures(

kaldi::OfflineFeatureTpl<kaldi::FbankComputer>::ComputeFeatures (
    this=0x7fffffffd8c0, wave=..., sample_freq=16000, vtln_warp=1, 
    output=0x7fffffffd6f0) at ../feat/feature-common-inl.h:29
29	void OfflineFeatureTpl<F>::ComputeFeatures(

kaldi::OfflineFeatureTpl<kaldi::FbankComputer>::Compute (this=0x7fffffffd8c0, 
    wave=..., vtln_warp=1, output=0x7fffffffd6f0)
    at ../feat/feature-common-inl.h:60
60	void OfflineFeatureTpl<F>::Compute(

65	  int32 rows_out = NumFrames(wave.Dim(), computer_.GetFrameOptions()),

kaldi::FbankComputer::GetFrameOptions (this=0x7fffffffd8c0)
    at ../feat/feature-fbank.h:100
100	    return opts_.frame_opts;

kaldi::NumFrames (num_samples=320, opts=..., flush=true)
    at feature-window.cc:44
44	                bool flush) {

45	  int64 frame_shift = opts.WindowShift();     
(:frame_shift = 160,frame_length = 400)

kaldi::FrameExtractionOptions::WindowShift (this=0x7fffffffd8c0)
    at ../feat/feature-window.h:107
107	    return static_cast<int32>(samp_freq * 0.001 * frame_shift_ms);

kaldi::OfflineFeatureTpl<kaldi::FbankComputer>::Compute (this=0x7fffffffd8c0, 
    wave=..., vtln_warp=1, output=0x7fffffffd6f0)
    at ../feat/feature-common-inl.h:67
67	  if (rows_out == 0) {

rows_out = 0 需要从头查下为什么是0






