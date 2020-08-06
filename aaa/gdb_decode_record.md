find /.../kaldi/ -name "nnet3-latgen-faster"

/export1/kaldi/src/nnet3bin/

gdb ./nnet3-latgen-faster
r --frame-subsampling-factor=3 --frames-per-chunk=50 --extra-left-context=0 --extra-right-context=0 --extra-left-context-initial=-1 --extra-right-context-final=-1 --minimize=false --max-active=7000 --min-active=200 --beam=15.0 --lattice-beam=8.0 --acoustic-scale=1.0 --allow-partial=true --word-symbol-table=exp/chain/tdnn/graph/words.txt exp/chain/tdnn/final.mdl exp/chain/tdnn/graph/HCLG.fst "ark,s,cs:apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/cmvn.scp scp:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/feats.scp ark:- |" "ark:|lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"

86 std::string model_in_filename = po.GetArg(1),

87 fst_in_str = po.GetArg(2), (gdb) p model_in_filename $1 = "exp/chain/tdnn/final.mdl"

88 feature_rspecifier = po.GetArg(3), (gdb) p fst_in_str $2 = "exp/chain/tdnn/graph/HCLG.fst"

(gdb) p feature_rspecifier (gdb) p po.GetArg(3) $4 = "ark,s,cs:apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/cmvn.scp s"...

89 lattice_wspecifier = po.GetArg(4),

90 words_wspecifier = po.GetOptArg(5),

91 alignment_wspecifier = po.GetOptArg(6);

93 TransitionModel trans_model; (gdb) p lattice_wspecifier $5 = "ark:|lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"

(gdb) p words_wspecifier $6 = "" (gdb) p alignment_wspecifier $7 = ""
