find /.../kaldi/ -name "nnet3-latgen-faster"

/export1/kaldi/src/nnet3bin/

export PATH=.

gdb ./nnet3-latgen-faster

(gdb) set environment KALDI_ROOT=/export1/kaldi

b nnet3-latgen-faster.cc:91

c

b nnet3-latgen-faster.cc:174

剪枝逻辑../decoder/ b lattice-faster-decoder.cc:507

info breakpoints

r --frame-subsampling-factor=3 --frames-per-chunk=50 --extra-left-context=0 --extra-right-context=0 --extra-left-context-initial=-1 --extra-right-context-final=-1 --minimize=false --max-active=7000 --min-active=200 --beam=15.0 --lattice-beam=8.0 --acoustic-scale=1.0 --allow-partial=true --word-symbol-table=/export1/kaldi/egs/cvte/online/exp/chain/tdnn/graph/words.txt /export1/kaldi/egs/cvte/online/exp/chain/tdnn/final.mdl /export1/kaldi/egs/cvte/online/exp/chain/tdnn/graph/HCLG.fst "ark,s,cs:/export1/kaldi/src/featbin/apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/test/split1/1/cmvn.scp scp:/export1/kaldi/egs/cvte/online/data/test/split1/1/feats.scp ark:- |" "ark:| /export1/kaldi/src/latbin/lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"


```

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
kaldi::LatticeFasterDecoderTpl<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > >, kaldi::decoder::StdToken>::Decode (this=this@entry=0x7fffffffd710, decodable=0x7fffffffdce0) at lattice-faster-decoder.cc:79  // src/decoder/...
79      bool LatticeFasterDecoderTpl<FST, Token>::Decode(DecodableInterface *decodable) {
80        InitDecoding();
(gdb) s
kaldi::LatticeFasterDecoderTpl<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > >, kaldi::decoder::StdToken>::InitDecoding (this=this@entry=0x7fffffffd710) at lattice-faster-decoder.cc:56
56      void LatticeFasterDecoderTpl<FST, Token>::InitDecoding() {
(gdb) s
58        DeleteElems(toks_.Clear());
(gdb) p toks_

$11 = {list_head_ = 0x0, bucket_list_tail_ = 18446744073709551615, hash_size_ = 1000,
          buckets_ = std::vector of length 1000, capacity 1000 = {{prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0,   只能加上两个大括号避免报错，反斜杠不好看:}} 
              last_elem = 0x0}, {prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0,
              last_elem = 0x0}, {prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0, last_elem = 0x0}, {prev_bucket = 0,
---Type <return> to continue, or q <return> to quit---q
last_eQuit
github 用{{ --- }} 变量，所以上面的{{ 没有 }} 就报错了，没找到转义办法

kaldi::LatticeFasterDecoderTpl<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > >, kaldi::decoder::StdToken>::InitDecoding (this=this@entry=0x7fffffffd710) at lattice-faster-decoder.cc:59
59        cost_offsets_.clear();

kaldi::LatticeFasterDecoderTpl<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > >, kaldi::decoder::StdToken>::AdvanceDecoding (this=this@entry=0x7fffffffd710, decodable=decodable@entry=0x7fffffffdce0,
    max_num_frames=max_num_frames@entry=-1) at lattice-faster-decoder.cc:580
580     void LatticeFasterDecoderTpl<FST, Token>::AdvanceDecoding(DecodableInterface *decodable,

fst::ImplToFst<fst::internal::VectorFstImpl<fst::VectorState<fst::ArcTpl<fst::TropicalWeightTpl<float> >, std::allocator<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > >, fst::MutableFst<fst::ArcTpl<fst::TropicalWeightTpl<float> > > >::Type[abi:cxx11]() const (this=0x555919f14160) at /export1/kaldi/tools/openfst-1.6.7/include/fst/fst.h:881
881       const string &Type() const override { return impl_->Type(); }
    fst::ImplToFst<fst::internal::VectorFstImpl<fst::VectorState<fst::ArcTpl<fst::TropicalWeightTpl<float> >, std::allocator<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > >, fst::MutableFst<fst::ArcTpl<fst::TropicalWeightTpl<float> > > >::Type[abi:cxx11]() const (this=0x555919f14160) at /export1/kaldi/tools/openfst-1.6.7/include/fst/fst.h:881
881       const string &Type() const override { return impl_->Type(); }

kaldi::LatticeFasterDecoderTpl<fst::Fst<fst::ArcTpl<fst::TropicalWeightTpl<float> > >, kaldi::decoder::StdToken>::AdvanceDecoding (this=this@entry=0x7fffffffd710, decodable=decodable@entry=0x7fffffffdce0,
    max_num_frames=max_num_frames@entry=-1) at lattice-faster-decoder.cc:586
586         if (fst_->Type() == "const") {
(gdb) n
591         } else if (fst_->Type() == "vector") {
(gdb) n
594           this_cast->AdvanceDecoding(decodable, max_num_frames);
(gdb) s
kaldi::LatticeFasterDecoderTpl<fst::VectorFst<fst::ArcTpl<fst::TropicalWeightTpl<float> >, fst::VectorState<fst::ArcTpl<fst::TropicalWeightTpl<float> >, std::allocator<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > >, kaldi::decoder::StdToken>::AdvanceDecoding (this=this@entry=0x7fffffffd710, decodable=decodable@entry=0x7fffffffdce0,
    max_num_frames=max_num_frames@entry=-1) at lattice-faster-decoder.cc:580
580     void LatticeFasterDecoderTpl<FST, Token>::AdvanceDecoding(DecodableInterface *decodable,

602       int32 num_frames_ready = decodable->NumFramesReady();
(gdb) s
kaldi::nnet3::DecodableAmNnetSimple::NumFramesReady (this=0x7fffffffdce0) at ../nnet3/nnet-am-decodable-simple.h:331
331         return decodable_nnet_.NumFrames();

kaldi::LatticeFasterDecoderTpl<fst::VectorFst<fst::ArcTpl<fst::TropicalWeightTpl<float> >, fst::VectorState<fst::ArcTpl<fst::TropicalWeightTpl<float> >, std::allocator<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > >, kaldi::decoder::StdToken>::NumFramesDecoded (this=0x7fffffffd710) at ../decoder/lattice-faster-decoder.h:340
340       inline int32 NumFramesDecoded() const { return active_toks_.size() - 1; }

614           PruneActiveTokens(config_.lattice_beam * config_.prune_scale);
(gdb) s
kaldi::LatticeFasterDecoderTpl<fst::VectorFst<fst::ArcTpl<fst::TropicalWeightTpl<float> >, fst::VectorState<fst::ArcTpl<fst::TropicalWeightTpl<float> >, std::allocator<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > >, kaldi::decoder::StdToken>::PruneActiveTokens (this=this@entry=0x7fffffffd710, delta=0.800000012) at lattice-faster-decoder.cc:506
506     void LatticeFasterDecoderTpl<FST, Token>::PruneActiveTokens(BaseFloat delta) {

511       for (int32 f = cur_frame_plus_one - 1; f >= 0; f--) {

kaldi::LatticeFasterDecoderTpl<fst::VectorFst<fst::ArcTpl<fst::TropicalWeightTpl<float> >, fst::VectorState<fst::ArcTpl<fst::TropicalWeightTpl<float> >, std::allocator<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > >, kaldi::decoder::StdToken>::AdvanceDecoding (this=this@entry=0x7fffffffd710, decodable=decodable@entry=0x7fffffffdce0,
    max_num_frames=max_num_frames@entry=-1) at lattice-faster-decoder.cc:616
616         BaseFloat cost_cutoff = ProcessEmitting(decodable);

kaldi::LatticeFasterDecoderTpl<fst::VectorFst<fst::ArcTpl<fst::TropicalWeightTpl<float> >, fst::VectorState<fst::ArcTpl<fst::TropicalWeightTpl<float> >, std::allocator<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > >, kaldi::decoder::StdToken>::ProcessEmitting (this=this@entry=0x7fffffffd710, decodable=decodable@entry=0x7fffffffdce0)
    at lattice-faster-decoder.cc:716
716       KALDI_ASSERT(active_toks_.size() > 0);
(gdb) n
717       int32 frame = active_toks_.size() - 1; // frame is the frame-index
(gdb) n
720       active_toks_.resize(active_toks_.size() + 1);
(gdb) n
722       Elem *final_toks = toks_.Clear(); // analogous to swapping prev_toks_ / cur_toks_
(gdb) n
725       Elem *best_elem = NULL;
(gdb) n
728       BaseFloat cur_cutoff = GetCutoff(final_toks, &tok_cnt, &adaptive_beam, &best_elem);
(gdb) s
kaldi::LatticeFasterDecoderTpl<fst::VectorFst<fst::ArcTpl<fst::TropicalWeightTpl<float> >, fst::VectorState<fst::ArcTpl<fst::TropicalWeightTpl<float> >, std::allocator<fst::ArcTpl<fst::TropicalWeightTpl<float> > > > >, kaldi::decoder::StdToken>::GetCutoff (this=this@entry=0x7fffffffd710, list_head=list_head@entry=0x555919f22050,
    tok_count=tok_count@entry=0x7fffffffcf28, adaptive_beam=adaptive_beam@entry=0x7fffffffcf18,
    best_elem=best_elem@entry=0x7fffffffcf20) at lattice-faster-decoder.cc:644
644     BaseFloat LatticeFasterDecoderTpl<FST, Token>::GetCutoff(Elem *list_head, size_t *tok_count,

```



-----------

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/gdb_decode_record.md)
