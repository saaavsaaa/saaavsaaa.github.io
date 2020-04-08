  由于现成可以用于训练的免费音频实在有点少，我就打算把搜集到的各种不同格式的音频统一转成thchs30然后一同训练。然而aishell1快写完了就突然想起来，kaldi里本就包含预处理的脚本，例如aishell v1的aishell_data_prep.sh。好在还有其他格式的，也不算白写，我可以去写其他格式的。     
  简单记录一下aishell_data_prep.sh，local/download_and_untar.sh里是下载解压，如果已经下载好了，直接把tgz包放到指定目录就可以。剩下的部分视乎都差不多，可以参考[Kaldi的thchs30预处理](https://saaavsaaa.github.io/aaa/Kaldi_code_1.html)。     
