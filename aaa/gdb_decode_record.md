find /.../kaldi/ -name "nnet3-latgen-faster"

/export1/kaldi/src/nnet3bin/

export PATH=.

gdb ./nnet3-latgen-faster

(gdb) set environment KALDI_ROOT=/export1/kaldi

b nnet3-latgen-faster.cc:91

r --frame-subsampling-factor=3 --frames-per-chunk=50 --extra-left-context=0 --extra-right-context=0 --extra-left-context-initial=-1 --extra-right-context-final=-1 --minimize=false --max-active=7000 --min-active=200 --beam=15.0 --lattice-beam=8.0 --acoustic-scale=1.0 --allow-partial=true --word-symbol-table=/export1/kaldi/egs/cvte/online/exp/chain/tdnn/graph/words.txt /export1/kaldi/egs/cvte/online/exp/chain/tdnn/final.mdl /export1/kaldi/egs/cvte/online/exp/chain/tdnn/graph/HCLG.fst "ark,s,cs:/export1/kaldi/src/featbin/apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/test/split1/1/cmvn.scp scp:/export1/kaldi/egs/cvte/online/data/test/split1/1/feats.scp ark:- |" "ark:| /export1/kaldi/src/latbin/lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"

(gdb) p model_in_filename     
$1 = "/export1/kaldi/egs/cvte/online/exp/chain/tdnn/final.mdl"     
(gdb) p fst_in_str     
$2 = "/export1/kaldi/egs/cvte/online/exp/chain/tdnn/graph/HCLG.fst"     
(gdb) p feature_rspecifier     
$3 = "ark,s,cs:/export1/kaldi/src/featbin/apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/test/"...      
(gdb) p lattice_wspecifier     
$4 = "ark:| /export1/kaldi/src/latbin/lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"     
(gdb) p words_wspecifier     
$5 = ""     
(gdb) p alignment_wspecifier     
$6 = ""     
