find /.../kaldi/ -name "nnet3-latgen-faster"

/export1/kaldi/src/nnet3bin/

export PATH=.

gdb ./nnet3-latgen-faster

(gdb) set environment KALDI_ROOT=/export1/kaldi

b nnet3-latgen-faster.cc:91

b nnet3-latgen-faster.cc:174

r --frame-subsampling-factor=3 --frames-per-chunk=50 --extra-left-context=0 --extra-right-context=0 --extra-left-context-initial=-1 --extra-right-context-final=-1 --minimize=false --max-active=7000 --min-active=200 --beam=15.0 --lattice-beam=8.0 --acoustic-scale=1.0 --allow-partial=true --word-symbol-table=/export1/kaldi/egs/cvte/online/exp/chain/tdnn/graph/words.txt /export1/kaldi/egs/cvte/online/exp/chain/tdnn/final.mdl /export1/kaldi/egs/cvte/online/exp/chain/tdnn/graph/HCLG.fst "ark,s,cs:/export1/kaldi/src/featbin/apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/test/split1/1/cmvn.scp scp:/export1/kaldi/egs/cvte/online/data/test/split1/1/feats.scp ark:- |" "ark:| /export1/kaldi/src/latbin/lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"

(gdb) p model_in_filename     
$1 = "/export1/kaldi/egs/cvte/online/exp/chain/tdnn/final.mdl"     
(gdb) p fst_in_str     
$2 = "/export1/kaldi/egs/cvte/online/exp/chain/tdnn/graph/HCLG.fst"     
139           Fst<StdArc> *decode_fst = fst::ReadFstKaldiGeneric(fst_in_str);

(gdb) p feature_rspecifier     
$3 = "ark,s,cs:/export1/kaldi/src/featbin/apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/test/"...      
(gdb) p lattice_wspecifier     
$4 = "ark:| /export1/kaldi/src/latbin/lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"     
(gdb) p words_wspecifier     
$5 = ""     
(gdb) p alignment_wspecifier     
$6 = ""     

kaldi::DecodeUtteranceLatticeFaster<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > (decoder=...,
    decodable=..., trans_model=..., word_syms=0x555555c36e30, utt="aaa-a100-4399000-15000", acoustic_scale=1,
    determinize=true, allow_partial=true, alignment_writer=0x7fffffffd4f0, words_writer=0x7fffffffd4e8,
    compact_lattice_writer=0x7fffffffd4d0, lattice_writer=0x7fffffffd4d8, like_ptr=0x7fffffffd508)
    at decoder-wrappers.cc:287
287     bool DecodeUtteranceLatticeFaster(
303       if (!decoder.Decode(&decodable)) {
(gdb) s
kaldi::LatticeFasterDecoderTpl<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > >, kaldi::decoder::StdToken>::Decode (this=this@entry=0x7fffffffd710, decodable=0x7fffffffdce0) at lattice-faster-decoder.cc:79
79      bool LatticeFasterDecoderTpl<FST, Token>::Decode(DecodableInterface *decodable) {
80        InitDecoding();
(gdb) s
kaldi::LatticeFasterDecoderTpl<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > >, kaldi::decoder::StdToken>::InitDecoding (this=this@entry=0x7fffffffd710) at lattice-faster-decoder.cc:56
56      void LatticeFasterDecoderTpl<FST, Token>::InitDecoding() {
(gdb) s
58        DeleteElems(toks_.Clear());
(gdb) p toks_
$11 = {list_head_ = 0x0, bucket_list_tail_ = 18446744073709551615, hash_size_ = 1000,
  buckets_ = std::vector of length 1000, capacity 1000 = {{prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0,
      last_elem = 0x0}, {prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0,
      last_elem = 0x0}, {prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0,
---Type <return> to continue, or q <return> to quit---q
last_eQuit

kaldi::LatticeFasterDecoderTpl<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > >, kaldi::decoder::StdToken>::InitDecoding (this=this@entry=0x7fffffffd710) at lattice-faster-decoder.cc:59
59        cost_offsets_.clear();

