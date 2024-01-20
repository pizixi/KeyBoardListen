# KeyBoardListen

实时全局监听键盘、鼠标事件，可以用来做快捷键

## 用法

```go
package main

import (
	"fmt"
	"log"

	gowinkey "github.com/pizixi/KeyBoardListen"

)

func main() {

	events := gowinkey.Listen()
	for e := range events {
		if e.State != 0 {
			// log.Println("触发:", e.VirtualKey)
			var pressKeys string
			for key := range e.PressedKeys {
				// fmt.Printf("Key: %s\n", key)
				pressKeys += fmt.Sprintf("[%v] ", key)
			}
			if e.PressedKeys.ContainsAll(gowinkey.VK_LMENU, gowinkey.VK_Z) {
				log.Println("快捷键alt+z被触发:", pressKeys)
        // ...
			}
		}
	}

}

````
## 另外一个纯go的模拟键盘发送事件的demo参考
>- 每隔五秒自动按ctrl+c键然后打印出剪切板内容

```go

//go:build windows
// +build windows

package main

import (
	"fmt"
	"log"
	"os"
	"os/signal"
	"time"

	"github.com/atotto/clipboard"
	"github.com/micmonay/keybd_event"
)

func main() {
	log.SetFlags(0)
	log.SetPrefix("error: ")

	if err := run(); err != nil {
		log.Fatal(err)
	}
}

func run() error {
	// 设置Ctrl+C的keybd_event
	kb, err := keybd_event.NewKeyBonding()
	if err != nil {
		return err
	}

	// 可能需要根据系统更改key event
	kb.SetKeys(keybd_event.VK_C)
	kb.HasCTRL(true)

	signalChan := make(chan os.Signal, 1)
	signal.Notify(signalChan, os.Interrupt)

	fmt.Println("start capturing keyboard input")

	ticker := time.NewTicker(5 * time.Second)
	defer ticker.Stop()

	for {
		select {
		case <-ticker.C:
			// 模拟Ctrl+C（复制）键的按下
			if err := kb.Launching(); err != nil {
				log.Printf("Error simulating key press: %v\n", err)
				continue
			}
			// 等待一段时间以确保剪贴板已更新
			time.Sleep(100 * time.Millisecond)
			text, err := clipboard.ReadAll()
			if err != nil {
				log.Printf("Error reading from clipboard: %v\n", err)
				continue
			}
			fmt.Printf("Clipboard content: %s\n", text)
		case <-signalChan:
			fmt.Println("Received shutdown signal")
			return nil
		}
	}
}

```
