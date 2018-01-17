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


(code[0] & 0xff) << 24)   
