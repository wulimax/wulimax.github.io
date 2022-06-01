## 下载m3u8 的视频数据转换成mp4

```go
package main

import (
	"bufio"
	"io"
	"log"
	"net/http"
	"os"
	"os/exec"
	"strings"
)

type Myqloud struct {
	host          string // host
	m3u8          string // m3u8地址
	url_file_name string //本地保存m3u8地址文件名
	file_path     string // 本地存储工作目录
	new_name      string // 最后mp4的名称
}

// 入口方法
func main() {

	do := new(Myqloud)
	do.file_path = "E:\\coding\\tmpts\\"
	do.new_name = "xxxx.mp4"
	do.host = "https://www.xxxx.com/"
	do.m3u8 = "video.m3u8"
	do.url_file_name = "data.txt"

	get_ts_list(do)
	read_file(do)
	merge_ts(do)
	convert_mp4(do)
	del_ts(do)
}

//拉去m3u8内容(ts 列表) 然后本地存储
func get_ts_list(s *Myqloud) {
	res := download(s.host + s.m3u8)
	waitfile(s.file_path+s.url_file_name, res)

}
//读取文件 拼接url执行下载
func read_file(s *Myqloud) {
	log.Println("read file " + s.file_path + s.url_file_name + "..............")
	// 创建句柄
	fi, err := os.Open(s.file_path + s.url_file_name)
	if err != nil {
		panic(err)
	}

	// 创建 Reader
	r := bufio.NewReader(fi)

	for {
		//func (b *Reader) ReadString(delim byte) (string, error) {}
		line, err := r.ReadString('\n')
		line = strings.TrimSpace(line)
		if err != nil && err != io.EOF {
			panic(err)
		}
		if err == io.EOF {
			break
		}
		if strings.Index(line, ".ts") != -1 {
			res := download(s.host + line)
			if res.StatusCode == 200 {
				waitfile(s.file_path+line[:strings.Index(line, "?")], res)
			}

		}

	}
}

// 下载方法
func download(strFinal string) *http.Response {
	res, err := http.Get(strFinal)
	check(err)
	return res
}

//  写入方法
func waitfile(file_path string, res *http.Response) {
	f, err := os.Create(file_path)
	check(err)
	fileSize, writeErr := io.Copy(f, res.Body)
	check(writeErr)
	log.Println(file_path+" download done,", "file size(byte)=", fileSize)
}

//  ts 合并方法
func merge_ts(s *Myqloud) {
	log.Println("merge ts ................")
	cmd0 := exec.Command("cmd", "/C", "copy", "/b", s.file_path+"*.ts", s.file_path+"new.ts")
	execute_cmd(cmd0, s.file_path)
}

//  ts 转换成mp4 方法
func convert_mp4(s *Myqloud) {
	log.Println("ts convert mp4 ................")
	cmd0 := exec.Command("cmd", "/C", "ffmpeg", "-i", s.file_path+"new.ts", "-c", "copy", "-map", "0:v", "-map", "0:a", s.file_path+s.new_name)
	execute_cmd(cmd0, s.file_path)
}

func del_ts(s *Myqloud) {
	log.Println("del tmp file ................")
	cmd1 := exec.Command("cmd", "/C", "del", s.file_path+s.url_file_name)
	execute_cmd(cmd1, s.file_path)
	cmd0 := exec.Command("cmd", "/C", "del", s.file_path+"*.ts")
	execute_cmd(cmd0, s.file_path)

}

// 异常检查
func check(e error) {
	if e != nil {
		log.Println(e)
		panic(e)
	}
}

func execute_cmd(cmd0 *exec.Cmd, str string) {
	cmd0.Dir = str
	if err := cmd0.Start(); err != nil { //开始执行命令
		log.Println(err)
		return
	}
}

```

