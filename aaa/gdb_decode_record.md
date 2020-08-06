Skip to content
 
Search or jump to…

Pull requests
Issues
Marketplace
Explore
 
@co-develop-drv 
saaavsaaa
/
saaavsaaa.github.io
 Watch 
1
 Star 3  Fork 1
Code
Issues
Pull requests
Actions
Projects
Wiki
Security
Insights
saaavsaaa.github.io
/
aaa
/

gdb_decode_record.md
Cancel
 Edit file    Preview changes

1
find /.../kaldi/ -name "nnet3-latgen-faster"
2
​
3
/export1/kaldi/src/nnet3bin/
4
​
5
gdb ./nnet3-latgen-faster     
6
r --frame-subsampling-factor=3 --frames-per-chunk=50 --extra-left-context=0 --extra-right-context=0 --extra-left-context-initial=-1 --extra-right-context-final=-1 --minimize=false --max-active=7000 --min-active=200 --beam=15.0 --lattice-beam=8.0 --acoustic-scale=1.0 --allow-partial=true --word-symbol-table=exp/chain/tdnn/graph/words.txt exp/chain/tdnn/final.mdl exp/chain/tdnn/graph/HCLG.fst "ark,s,cs:apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/cmvn.scp scp:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/feats.scp ark:- |" "ark:|lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"
7
​
8
86          std::string model_in_filename = po.GetArg(1),
9
​
10
87              fst_in_str = po.GetArg(2),
11
(gdb) p model_in_filename
12
$1 = "exp/chain/tdnn/final.mdl"
13
​
14
88              feature_rspecifier = po.GetArg(3),
15
(gdb) p fst_in_str
16
$2 = "exp/chain/tdnn/graph/HCLG.fst"
17
​
18
(gdb) p feature_rspecifier
19
(gdb) p po.GetArg(3)
20
$4 = "ark,s,cs:apply-cmvn --norm-means=true --norm-vars=false --utt2spk=ark:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/utt2spk scp:/export1/kaldi/egs/cvte/online/data/gdb_debug/test/split1/1/cmvn.scp s"...
21
​
22
89              lattice_wspecifier = po.GetArg(4),
23
​
24
90              words_wspecifier = po.GetOptArg(5),
25
​
26
91              alignment_wspecifier = po.GetOptArg(6);
27
​
28
93          TransitionModel trans_model;
29
(gdb) p lattice_wspecifier
30
$5 = "ark:|lattice-scale --acoustic-scale=10.0 ark:- ark:/dev/null"
31
​
32
(gdb) p words_wspecifier
33
$6 = ""
34
(gdb) p alignment_wspecifier
35
$7 = ""
36
​
37
​
@co-develop-drv
Commit changes
Commit summary 
Update gdb_decode_record.md
Optional extended description

Add an optional extended description…
  Commit directly to the master branch.
  Create a new branch for this commit and start a pull request. Learn more about pull requests.
Commit changes  Cancel
© 2020 GitHub, Inc.
Terms
Privacy
Security
Status
Help
Contact GitHub
Pricing
API
Training
Blog
About
