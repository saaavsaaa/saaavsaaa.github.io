  这一篇主要是 kalid 的 thchs30 示例的数据预处理部分。之前没写完的线性代数系列的可能要拖一段时间了，搞不好要拖到疫情结束了。因为上一篇写完之后，就上班了。虽然我没离开北京，但是我租的公寓因为我春节不在，不让我进。我工作时晚上就只好住在公司，晚上的精力和环境都不太适合思考数学证明，而周末回家要处理的事儿很多，只好先拖一拖了。只要公寓能进了，还是会继续写的。    
  我现在主要工作是在做语音识别，选型定了kalid，目前只用了 thchs30 。用训练出的模型测试同事的普通话，准确率大概在30%出头。只不过实际识别客服的录音惨不忍睹，另外能发现有不少是谐音问题。于是就开始进行优化工作了，一方面收集语料，一方面开始研究kalid，方便后续自制语料的使用等。    
  基本的内容，网上都很全面，我都不写了，我就把我看的代码做个流水账的描述，由于刚开始看，所以只有一小部分：    

-----

  起点：/kaldi-ali/kaldi-trunk/egs/thchs30/s5/run.sh

-----

  首先是 cmd.sh，没什么好说的。本地调代码比较方便，暂时也没有很多语料，也没有很多机器，于是run.pl。    
  path.sh：引用了必须的目录，之后要用的代码主要都在这里了。tools/python:${PATH}、egs/thchs30/s5/utils、tools/openfst/bin...、tools/openfst/bin:$PWD:$PATH，其中PATH= src下的十几个目录。 

-----

  接着，kaldi-trunk/egs/thchs30/s5/local/thchs-30_data_prep.sh生成了必须的文件，主要逻辑：
  
  corpus_dir参数指定了data_thchs30语料所在的目录。     
  语料中音频文件名的命名方式如A1_2.wav，A1是说话人speaker，A1_2是A1说的编号为2的话utternace。     
  
    for x in train dev test; do  # train dev test是音频文件所在的三个目录，分别的作用就是它们各自的名字     
    
        for nn in `find  $corpus_dir/$x -name "*.wav" | sort -u | xargs -I {} basename {} .wav`; do     # 所有wav文件的basename
        
          spkid=`echo $nn | awk -F"_" '{print "" $1}'`     # 文件名以"_"分隔，取第一项。例如：A1_1 -> A1     
          
          spk_char=`echo $spkid | sed 's/\([A-Z]\).*/\1/'`     # sed s/ / / 替换 \1第一个匹配字符。例如：A1 -> A     
          
          spk_num=`echo $spkid | sed 's/[A-Z]\([0-9]\)/\1/'`     # 第一项中的数字。例如：A1 -> 1     
          
          spkid=$(printf '%s%.2d' "$spk_char" "$spk_num")      # 第一项中如果数字是一位的变成两位。例如：A1 -> A01     
          
          utt_num=`echo $nn | awk -F"_" '{print $2}'`      # 文件名以"_"分隔的第二项。例如：A1_2 -> 2     
          
          uttid=$(printf '%s%.2d_%.3d' "$spk_char" "$spk_num" "$utt_num")    # 第二项变3位。例如：A1_2 -> A01_002     
          
          echo $uttid $corpus_dir/$x/$nn.wav >> wav.scp   # 数位变换后的文件名uttid与真实文件全路径对应写入wav.scp。例如：A11_000 /home/.../A11_0.wav     
          
          echo $uttid $spkid >> utt2spk       # 数位变换后的文件名uttid与新第一项对应spkid写入utt2spk。例如：A11_000 A11     
          
          echo $uttid `sed -n 1p $corpus_dir/data/$nn.wav.trn` >> word.txt    # 数位变换后的文件名uttid与对应汉字写入word.txt，data/*trn第一行是汉字     

          echo $uttid `sed -n 3p $corpus_dir/data/$nn.wav.trn` >> phone.txt   # 数位变换后的文件名uttid与对应音素写入phone.txt 第三行是音素     

        done # 一个wav文件的处理结束     
        
        # 创建text文件，对text、wav.scp、utt2spk、phone.txt内容排序：    
        cp word.txt text     
        sort wav.scp -o wav.scp    
        sort utt2spk -o utt2spk   
        sort text -o text   
        sort phone.txt -o phone.txt   

    done # train dev test三个目录    

    # perl语言以前没有认真接触过，不过好在语法都差不多，个别符号百度一下就好    
    utils/utt2spk_to_spk2utt.pl data/train/utt2spk > data/train/spk2utt       # 下一块里 解释   
    utils/utt2spk_to_spk2utt.pl data/dev/utt2spk > data/dev/spk2utt   
    utils/utt2spk_to_spk2utt.pl data/test/utt2spk > data/test/spk2utt    

    cd data/test_phone && rm text &&  cp phone.txt text      # test_phone用于测试训练出的模型，对比识别结果和真实语句  

-----

text:   
A02_000 绿 是 阳春 烟 景 大块 文章 的 底色 四月 的 林 峦 更是 绿 得 鲜活 秀媚 诗意 盎然   
A02_001 他 仅 凭 腰部 的 力量 在 泳道 上下 翻腾 蛹 动 蛇行 状 如 海豚 一直 以 一头 的 优势 领先   

wav.scp:   
A02_000 /home/ubuntu/github/DeepSpeechRecognition/data_thchs30/data_thchs30/train/A2_0.wav   
A02_001 /home/ubuntu/github/DeepSpeechRecognition/data_thchs30/data_thchs30/train/A2_1.wav   

utt2spk:   
A02_000 A02   
A02_001 A02   

spk2utt:   
A02 A02_033 A02_058 A02_085 A02_092 A02_103 A02_108 A02_116 A02_119 A02_167 A02_206   
A04 A04_004 A04_010 A04_026 A04_048 A04_052 A04_074 A04_125 A04_140 A04_157 A04_182 A04_197 A04_204 A04_209 A04_210 A04_235 A04_240 A04_243   

phone.txt:因素集   
A02_033 vv v3 uu uang2 uu ui3 s iy4 p ing2 sh ix4 h uo4 ch e1 s iy1 j i1 d u4 d a4 p ing2 vv ve4 vv v4 j ie2 zh uang1 x ie4 g ong1 l iu2 ch un1 sh an1 ii i4 t ong2 zh ui1 g an3 t ao2 p ao3 d e5 aa an4 f an4   
A02_058 t e4 b ie2 sh ix4 k ua4 sh eng3 q v1 d ian4 uu uang3 ch ao1 j i4 h ua4 ii iong4 d ian4 b u4 j in3 s un3 h ai4 z iy4 j i3 ii ie3 s un3 h ai4 b ie2 r en2 s un3 h ai4 d ian4 uu uang3 s un3 h ai4 g uo2 j ia1   

word.txt:   
A11_101 七十 年代 末 我 外出 求学 母亲 叮咛 我 吃饭 要 细嚼慢咽 学习 要 深 钻 细 研   
A11_119 陈云 同志 同时 要求 干部 认真学习 业务 精通 业务 向 一切 业务 内 行人 学习   

-----

  utils/utt2spk_to_spk2utt.pl: 聚合为一个说话人对应其所有说的话，A02 A02_000 A02_001 A02_002 A02_003 A02_004...     
  
    while(<>){                   # <> 句柄 默认为stdin      

        @A = split(" ", $_);     # 这里$_就是<>传入的    
        
        ($u,$s) = @A;           # 数组变量以字符 @ 开头
        
        # 每个说话者只加入一次，把spkid加入到list中，并将加入标记设为1     
        if(!$seen_spk{$s}) {     
            $seen_spk{$s} = 1;     
            push @spklist, $s;     
        }     
        push (@{$spk_hash{$s}}, "$u");   # $spk_hash{$s}是一个ARRAY，同一个speaker的utternace(发言)加入到一个ARRAY里     
    }     
    # 循环所有speaker
    foreach $s (@spklist) {     
        $l = join(' ',@{$spk_hash{$s}}); #将ARRAY中的uttid用空格连接成字符串     
        print "$s $l\n";     
    }     

-----

  回到 run.sh

    # 分别在 train dev test中创建mfcc目录,-p创建多级
    # mkdir -p data/mfcc && cp -R data/train data/mfcc && cp -R data/dev data/mfcc && cp -R data/test data/mfcc && cp -R data/test_phone data/mfcc || exit 1;
    
    下面就是提取 mfcc (梅尔频率倒谱系数Mel Frequency Cepstrum Coefficient)特征了，不过我还没写完。     
    进行的部分在（为了在公司时候用单独申请的账号）：
    https://github.com/co-develop-drv/kaldi/blob/master/egs/thchs30/s5/     
    https://github.com/co-develop-drv/kaldi/blob/master/egs/thchs30/s5/compiled_steps/make_mfcc.sh(未完)     
    https://github.com/co-develop-drv/kaldi/blob/master/egs/thchs30/s5/compiled_utils/validate_data_dir.sh     

-----
