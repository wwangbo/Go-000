学习笔记
```
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"

	"golang.org/x/sync/errgroup"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	egregious,_ := errgroup.WithContext(ctx)

	httpErr := make(chan error, 1)
	signals := make(chan os.Signal, 1)
	defer func() {
		close(httpErr)
		close(signals)
	}()

	s := http.Server{Addr: ":8088"}

	//监听http
	egregious.Go(func() error {
		go func() {
			httpErr <- s.ListenAndServe()
		}()
		select {
		case err := <-httpErr: //当监听到http协程退出,这里不需要去对signal进行处理，一旦返回error，errorgroup可以监听得到
			return err
		}
	})

	// 监听系统信号
	egregious.Go(func() error {
		signal.Notify(signals,syscall.SIGINT|syscall.SIGTERM|syscall.SIGKILL)
		<-signals     //监听到信号量时，退出本协程同时关闭http服务
		return s.Shutdown(context.TODO())
	})

	if err:= egregious.Wait();err != nil {
		cancel() //收到错误，cancel
		log.Print(err.Error())
	}
}
```
