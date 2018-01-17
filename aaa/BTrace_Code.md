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
