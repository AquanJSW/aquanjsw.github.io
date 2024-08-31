+++
title = '不建议在Windows平台用VSCode编写C/C++/CUDA程序'
date = 2024-08-31T16:04:42+08:00
draft = false
+++

由于C/C++/CUDA编程需要配置大量环境变量，虽然MSVC工具链提供了可以一键设置环境的`vcvarsall.bat`脚本，但目前在VSCode中并没有任何方法可以通过自动调用/分析该脚本来设置编译环境，因此不建议在Windows平台使用VSCode编写C/C++/CUDA程序。

以CUDA编译环境搭建为例展示其复杂性：

1. 需要手动设置`CMakeLists.txt`
2. 需要手动设置`CUDA`的相关环境变量，例如`CUDACXX`, `CUDAToolkit_ROOT`等
3. 需要手动设置`vcvarsall.bat`所生成的环境变量，例如`INCLUDE`、`LIB`、`LIBPATH`、`PATH`等。
4. 在VSCode中配置`-DCMAKE_EXPORT_COMPILE_COMMANDS=1`为`cmake`的固定参数，以便生成`compile_commands.json`文件用于IntelliSense。
5. 如果使用`clangd`作为IntelliSense的后端，还需要手动设置`.clangd`文件内容，例如：

    ```yaml
    CompileFlags:
        Add:
        - --cuda-path=C://Program Files//NVIDIA GPU Computing Toolkit//CUDA//v12.6//
        - --cuda-gpu-arch=sm_70
    ```

## 总结

VSCode终究只是一个文本编辑器，对于C/C++/CUDA这种需要配置许多环境变量的编程语言，用更专业的IDE或许是个不错的选择，例如`Visual Studio`、`CLion`等。
