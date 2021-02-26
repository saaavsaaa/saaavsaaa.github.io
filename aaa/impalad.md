impalad 启动   
impalad-main.cc
```
int ImpaladMain(int argc, char** argv) {
  InitCommonRuntime(argc, argv, true);      // be/src/common/init.cc CPU、硬盘、内存等信息，检查CPU是否符合要求，设置默认主机名，可以被覆盖
  
```
