## 常用压缩格式
- .zip .gz .bz2
- .tar.gz .tar.bz2


## .zip
- zip 压缩文件 [源文件]
- zip -r 压缩目录 [源目录]
- zip格式和windows通用
- unzip 压缩文件

## .gz
- gzip 源文件（压缩为.gz格式文件  源文件消失）
- gzip -c 源文件 > 压缩文件（压缩为.gz,保留原我呢见）
- gzip -r 源目录 （压缩目录内文件 ，但保留目录）

## .bz2
- bzip2 源文件（压缩为.bz2格式文件  源文件消失）
- bzip2 -k 源文件（压缩后保留源文件）
- bzip2命令不能压缩目录
- bunzip2 解压
- bzip2 -k 解压
