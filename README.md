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
