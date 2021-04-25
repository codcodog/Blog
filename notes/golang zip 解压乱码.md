zip 包解压乱码
==============

### 场景

使用 `golang` `zip` 包解压时，文件名含有中文会出现乱码.



### 原因

因为压缩包是在 `windows` 平台压缩的，压缩软件 `winrar` 会使用本地编码方式进行压缩，所以含有中文的文件名是 `gbk` 格式的，  
而 `golang` 字符串是使用 `utf-8` 格式，从而出现乱码.


### 解决

通过标识位识别 `gbk` 编码，若是则转换为 `utf-8`
> 如果未设置通用位11，则文件名和注释使用原始的ZIP字符编码  
> 如果设置了通用位11，则文件名和注释必须使用UTF-8存储规范定义的字符编码形式支持Unicode标准版本4.1.0或更高版本

```golang
func (tc *TaskController) unzip(fileName string, dst string) (map[string][]string, int, error) {
	zf, err := zip.OpenReader(fileName)
	if err != nil {
		return nil, 0, err
	}
	defer zf.Close()

	total := 0                         // 总文件数，记录任务进度
	files := make(map[string][]string) // 文件信息缓存，sku => 文件名
	for _, file := range zf.File {
		// 注意，windows 下的压缩文件是GBK，解压可能有中文名，则需要转换为 utf-8
		if file.Flags == 0 {
			i := bytes.NewReader([]byte(file.Name))
			decoder := transform.NewReader(i, simplifiedchinese.GB18030.NewDecoder())
			content, _ := ioutil.ReadAll(decoder)
			file.Name = string(content)
		}

		path := filepath.Join(dst, file.Name)
		if file.FileInfo().IsDir() {
			continue
		}

		f, err := file.Open()
		if err != nil {
			log.Error("[file.Open] error: %v", err)
			return nil, 0, err
		}

		if err = os.MkdirAll(filepath.Dir(path), os.ModePerm); err != nil {
			f.Close()
			return nil, 0, err
		}

		fw, err := os.OpenFile(path, os.O_CREATE|os.O_RDWR|os.O_TRUNC, file.Mode())
		if err != nil {
			log.Error("[os.OpenFile] error: %v", err)
			return nil, 0, err
		}
		_, err = io.Copy(fw, f)
		if err != nil {
			f.Close()
			fw.Close()
			return nil, 0, err
		}

		f.Close()
		fw.Close()

		sku := tc.getSku(file.Name)
		if sku == "" {
			continue
		}
		files[sku] = append(files[sku], file.Name)
		total += 1
	}

	return files, total, nil
}
```

### 参考
[go 处理zip解压时乱码问题](https://studygolang.com/articles/21345)  
[Creating a zip archive with Unicode filenames using Go's archive/zip](https://stackoverflow.com/questions/30026083/creating-a-zip-archive-with-unicode-filenames-using-gos-archive-zip)
