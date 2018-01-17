因为-0的符号没有意义,所以符号位是1，其余都是0的补码可以用来表示-127-1，因为负数的补码符号位为1，其余位为该数绝对值的原码按位取反，然后加1

读入class文件
        File f = new File(path);
        InputStream is = new FileInputStream(f)

        while ((read = is.read(buffer)) > 0) {
            byte[] newresult = new byte[result.length + read];
            System.arraycopy(result, 0, newresult, 0, result.length);
            System.arraycopy(buffer, 0, newresult, result.length, read);
            result = newresult;
        }
        
创建BTraceProbe 
BTraceProbe bcn = factory.createProbe(traceData);


int mgc = ((code[0] & 0xff) << 24) | ((code[1] & 0xff) << 16) | ((code[2] & 0xff) << 8) | ((code[3] & 0xff));

magic

在class文件开头的四个字节， 存放着class文件的魔数， 这个魔数是class文件的标志，他是一个固定的值： 0XCAFEBABE 。 也就是说他是判断一个文件是不是class格式的文件的标准， 如果开头四个字节不是0XCAFEBABE， 那么就说明它不是class文件， 不能被JVM识别。

code[0]的低8位用于保存这个数，计算机中都是用的补码，因为计算机只能做加减运算，为了在运算中保持符号位的正确性所以使用的是补码。而这条语句一旦开始执行code[0]就会被表示为int类型，变为32位的，如果不做处理那么高的24位会补1导致补码值不正确，而先做&0xff运算，原来是0的还是0，是1的还是1，就保证了数据的正确性。
