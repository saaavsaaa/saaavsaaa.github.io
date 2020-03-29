  这一篇主要是 kalid 的 thchs30 示例的数据预处理部分。之前没写完的线性代数系列的可能要拖一段时间了，搞不好要拖到疫情结束了。因为上一篇写完之后，就上班了。虽然我没离开北京，但是我租的公寓因为我春节不在，不让我进。我工作时晚上就只好住在公司，晚上的精力和环境都不太适合思考数学证明，而周末回家要处理的事儿很多，只好先拖一拖了。只要公寓能进了，还是会继续写的。    
  我现在主要工作是在做语音识别，选型定了kalid，目前只用了 thchs30 。用训练出的模型测试同事的普通话，准确率大概在30%出头。只不过实际识别客服的录音惨不忍睹，另外能发现有不少是谐音问题。一方面收集语料，一方面开始研究kalid，方便后续自制语料的使用等。    
  基本的内容，网上都很全面，我都不写了，我就把我看的代码做个流水账的描述，由于刚开始看，所以只有一小部分：    

-----

/kaldi-ali/kaldi-trunk/egs/thchs30/s5/run.sh

-----

  首先是 cmd.sh，没什么好说的。本地调代码比较方便，暂时也没有很多语料，也没有很多机器，于是run.pl。    
  path.sh：引用了必须的目录，之后要用的代码主要都在这里了。tools/python:${PATH}、egs/thchs30/s5/utils、tools/openfst/bin...、tools/openfst/bin:$PWD:$PATH，其中PATH= src下的十几个目录。 

-----

  接着，kaldi-trunk/egs/thchs30/s5/local/thchs-30_data_prep.sh生成了必须的文件，主要逻辑：
  
  corpus_dir参数指定了data_thchs30语料所在的目录。     
  语料中音频文件名的命名方式如A1_2.wav，A1是说话人speaker，A1_2是A1说的编号为2的话utternace。     
for x in train dev test; do
    for nn in `find  $corpus_dir/$x -name "*.wav" | sort -u | xargs -I {} basename {} .wav`; do
      spkid=`echo $nn | awk -F"_" '{print "" $1}'` #文件名以"_"分隔的第一项 A1_1 -> A1
      spk_char=`echo $spkid | sed 's/\([A-Z]\).*/\1/'` #sed s/ / / 替换 \1第一个匹配字符 A1 -> A
      spk_num=`echo $spkid | sed 's/[A-Z]\([0-9]\)/\1/'` #第一项中的数字 A1 -> 1
      spkid=$(printf '%s%.2d' "$spk_char" "$spk_num") #第一项中如果数字是一位的变成两位 A1 -> A01
      utt_num=`echo $nn | awk -F"_" '{print $2}'` #文件名以"_"分隔的第二项 A1_2 -> 2
      uttid=$(printf '%s%.2d_%.3d' "$spk_char" "$spk_num" "$utt_num") # 第二项变3位 A1_1 -> A01_001
      echo $uttid $corpus_dir/$x/$nn.wav >> wav.scp #数位变换后的文件名uttid与真实文件全路径对应写入wav.scp  A11_000 /home/.../A11_0.wav
      echo $uttid $spkid >> utt2spk #数位变换后的文件名uttid与新第一项对应spkid写入utt2spk   A11_000 A11
      echo $uttid `sed -n 1p $corpus_dir/data/$nn.wav.trn` >> word.txt #数位变换后的文件名uttid与对应汉字写入word.txt，data/*trn第一行是汉字
      echo $uttid `sed -n 3p $corpus_dir/data/$nn.wav.trn` >> phone.txt #数位变换后的文件名uttid与对应音素写入phone.txt 第三行是音素
    done 
    #排序：
    cp word.txt text
    sort wav.scp -o wav.scp
    sort utt2spk -o utt2spk
    sort text -o text
    sort phone.txt -o phone.txt
done 

perl
utils/utt2spk_to_spk2utt.pl data/train/utt2spk > data/train/spk2utt
utils/utt2spk_to_spk2utt.pl data/dev/utt2spk > data/dev/spk2utt
utils/utt2spk_to_spk2utt.pl data/test/utt2spk > data/test/spk2utt

cd data/test_phone && rm text &&  cp phone.txt text #test_phone用于测试训练出的模型，对比识别结果和真实语句

-----
