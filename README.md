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
## 结合另一个纯go的模拟键盘发送事件的demo参考
>- 实现监听键盘当按下ALT+Z的快捷键时候，自动发送一个Ctrl+C的组合键复制选中文本并打印剪贴板

```go

package main

import (
	"log"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/atotto/clipboard"
	"github.com/micmonay/keybd_event"
	gowinkey "github.com/pizixi/KeyBoardListen"
	"golang.org/x/net/context"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	go listenKeyboard(ctx)
	if err := run(ctx); err != nil {
		log.Printf("ERROR: %v\n", err)
		os.Exit(1)
	}
}

func listenKeyboard(ctx context.Context) {
	events := gowinkey.Listen()
	for {
		select {
		case <-ctx.Done():
			return
		case e := <-events:
			if e.State != 0 && e.PressedKeys.ContainsAll(gowinkey.VK_LMENU, gowinkey.VK_Z) {
				log.Println("INFO: Shortcut Alt+Z triggered")
				triggerCopy()
			}
		}
	}
}

func triggerCopy() {
	kb, err := keybd_event.NewKeyBonding()
	if err != nil {
		log.Printf("ERROR: Creating new key bonding: %v\n", err)
		return
	}

	kb.SetKeys(keybd_event.VK_C)
	kb.HasCTRL(true)

	time.Sleep(1 * time.Second) // Wait 1 second before copying
	log.Println("INFO: Attempting to copy")
	if err := kb.Launching(); err != nil {
		log.Printf("ERROR: Simulating key press: %v\n", err)
		return
	}

	time.Sleep(100 * time.Millisecond) // Ensure clipboard has updated
	text, err := clipboard.ReadAll()
	if err != nil {
		log.Printf("ERROR: Reading from clipboard: %v\n", err)
		return
	}
	log.Printf("INFO: Clipboard content: %s\n", text)
}

func run(ctx context.Context) error {
	signalChan := make(chan os.Signal, 1)
	signal.Notify(signalChan, os.Interrupt, syscall.SIGTERM)

	log.Println("INFO: Start capturing keyboard input")
	<-signalChan
	log.Println("INFO: Received shutdown signal")
	return nil
}


```
