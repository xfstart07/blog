---
title: '[Go] 计算文件MD5'
date: 2018-09-13 18:12:43
categories: Go
tags: Go
---

### Go 计算小文件


```go
import (
	"fmt"
	"crypto/md5"
	"io"
	"os"
)

func main() {
	filePath := "/Users/x/golang/src/example/Readme.md"

	file, err := os.Open(filePath)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	md5h := md5.New()
	io.Copy(md5h, file)
	fmt.Printf("%x", md5h.Sum([]byte(""))) //md5
}
```

因为文件小，所有直接将文件使用 `io.Copy` 读取进行 md5 的计算。

<!-- more -->

### 大文件计算

大文件计算的原来就是，设置文件块的读取，一次读取文件的一部分。


```go
import (
	"crypto/md5"
	"fmt"
	"io"
	"math"
	"os"
)

const filechunk = 8192 // we settle for 8KB

func main() {
	filePath := "/Users/x/Downloads/201809111536630287271_鲸唱扫码2.mkv"

	file, err := os.Open(filePath)

	if err != nil {
		panic(err)
	}
	defer file.Close()

	// calculate the file size
	info, _ := file.Stat()

	filesize := info.Size()

	blocks := uint64(math.Ceil(float64(filesize) / float64(filechunk)))

	hash := md5.New()

	for i := uint64(0); i < blocks; i++ {
		blocksize := int(math.Min(filechunk, float64(filesize-int64(i*filechunk))))
		buf := make([]byte, blocksize)

		file.Read(buf)
		io.WriteString(hash, string(buf)) // append into the hash
	}

	fmt.Printf("%s checksum is %x\n", file.Name(), hash.Sum(nil))
}
```