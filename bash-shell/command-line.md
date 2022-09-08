---
# SED

- Thay thế đoạn văn bản 'unix' thành 'linux' trong tệp 

```
sed 's/unix/linux/' test.txt 
```
- thay thế dòng thứ n trong tệp văn bản ( sử dụng /1 /2 /n để thay thế lần xuất hiện của đoạn văn bản )

```
sed 's/unix/linux/2' test.txt
```
- thay thế tất cả các lần xuất hiện của đoạn văn bản trong tệp
```
sed 's/unix/linux/g' geekfile.txt
```
- Thay thế từ lần xuất hiện thứ n thành tất cả các lần xuất hiện trong một dòng ( /1 /2 /n )
```
sed 's/unix/linux/3g' geekfile.txt
```
- thay thế ở 1 dòng nhất định
```
sed '3 s/unix/linux/' geekfile.txt
```
- xóa dòng  tùy chọn trong tệp

```
sed '5d' filename.txt 
```
- 
