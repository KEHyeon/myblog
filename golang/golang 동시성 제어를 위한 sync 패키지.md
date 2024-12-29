<h2 id="bkmrk-go%2Fsync-%ED%8C%A8%ED%82%A4%EC%A7%80">go/sync 패키지</h2>
<p id="bkmrk-%EA%B3%A0%EB%A3%A8%ED%8B%B4%EC%9D%98-%EC%8B%A4%ED%96%89-%ED%9D%90%EB%A6%84%EC%9D%84-%EC%A0%9C%EC%96%B4%ED%95%98%EB%8A%94-%EB%8F%99%EA%B8%B0%ED%99%94"><strong><span style="background-color: rgb(45, 194, 107);">고루틴</span>의 <span style="background-color: rgb(45, 194, 107);">실행 흐름</span>을 제어하는 동기화 객체이다.</strong></p>
<h3 id="bkmrk-%EB%AE%A4%ED%85%8D%EC%8A%A4"><strong>뮤텍스</strong></h3>
<ol id="bkmrk-sync.mutex-func-%28m-%2A">
<li style="list-style-type: none;">
<ol>
<li><strong>sync.Mutex</strong></li>
<li><strong>func (m *Mutext) Lock()</strong></li>
<li><strong>func (m *Mutex) Unlock()</strong></li>
</ol>
</li>
</ol>
<pre id="bkmrk-package-main-import-"><code class="language-go">package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	runtime.GOMAXPROCS(runtime.NumCPU())

	var data = []int{}

	go func() {
		for i := 0; i &lt; 1000; i++ {
			data = append(data, 1)
			runtime.Gosched()
		}
	}()

	go func() {
		for i := 0; i &lt; 1000; i++ {
			data = append(data, 1)

			runtime.Gosched()
		}
	}()

	time.Sleep(2 * time.Second)

	fmt.Println(len(data))
}
</code></pre>
<p id="bkmrk-%EC%8B%A4%ED%96%89%EC%9D%84-%ED%95%B4%EB%B3%B4%EB%A9%B4-%EC%B6%9C%EB%A0%A5-%EA%B2%B0%EA%B3%BC%EA%B0%80-2000-">실행을 해보면 출력 결과가 2000 미만으로 불규칙 적이다. 두 고루틴이 경합을 벌이면서 동시에 data에 접근했기 때문에 append 함수가 정확하게 처리되지 않은 상황이다. 이런 상황을 경쟁 조건(Race condition) 이라고 한다.</p>
<p id="bkmrk-%ED%95%98%EC%A7%80%EB%A7%8C-data-%EC%8A%AC%EB%9D%BC%EC%9D%B4%EC%8A%A4%EB%A5%BC-%EB%AE%A4%ED%85%8D%EC%8A%A4%EB%A1%9C-">하지만 data 슬라이스를 뮤텍스로 보호하면 고정된 값 2000이 출력된다.</p>
<pre id="bkmrk-mutex.lock%28%29-data-%3D-"><code class="language-go">mutex.Lock()
data = append(data, 1)
mutex.Unlock()</code></pre>
<p id="bkmrk-%C2%A0"><br></p>
<h3 id="bkmrk-%EC%9D%BD%EA%B3%A0%2C-%EC%93%B0%EA%B8%B0-%EB%AE%A4%ED%85%8D%EC%8A%A4"><strong>읽고, 쓰기 뮤텍스</strong></h3>
<ul id="bkmrk-%EC%9D%BD%EA%B8%B0-%EB%9D%BD%28read-lock%29%C2%A0-%3A-%EC%9D%BD">
<li class="null">읽기 락(Read Lock)&nbsp; : 읽기 락끼리는 서로를 막지 않습니다. 하지만 읽기 시도 중에 값이 바뀌면 안 되므로 쓰기 락은 막습니다.</li>
<li class="null">쓰기 락(Write Lock) : 쓰기 시도 중에 다른 곳에서 이전 값을 읽으며 안 되고, 다른곳에서 값을 바꾸면 안 되므로 읽기, 쓰기 락 모두 막습니다.</li>
</ul>
<h5 id="bkmrk-%EC%9D%BD%EA%B8%B0%2C-%EC%93%B0%EA%B8%B0-%EB%AE%A4%ED%85%8D%EC%8A%A4-%EA%B5%AC%EC%A1%B0%EC%B2%B4%EC%99%80-%ED%95%A8%EC%88%98">읽기, 쓰기 뮤텍스 구조체와 함수</h5>
<ul id="bkmrk-sync.rwmutex-func-%28r">
<li class="null">sync.RWMutex</li>
<li class="null">func (rw *RWMutex) Lock(), func (rw *RWMutex) Unlock(): 쓰기 뮤텍스 잠금, 잠금 해제<br></li>
<li class="null">func (rw *RWMutex) RLock(), func (rw *RWMutex) RUnlick(): 읽기 뮤텍스 잠금 및 잠금 해제</li>
</ul>
<h5 id="bkmrk-%EC%9D%BD%EA%B8%B0%EC%93%B0%EA%B8%B0-%EB%AE%A4%ED%85%8D%EC%8A%A4%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%98%EC%9D%84%EB%95%8C">읽기쓰기 뮤텍스를 사용하지 않았을때</h5>
<pre id="bkmrk-package-main-import--0"><code class="language-go">package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	runtime.GOMAXPROCS(runtime.NumCPU()) // 모든 CPU 사용

	var data int = 0

	go func() {                               // 값을 쓰는 고루틴
		for i := 0; i &lt; 3; i++ {
			data += 1                         // data에 값 쓰기
			fmt.Println("write   : ", data)   // data 값을 출력
			time.Sleep(10 * time.Millisecond) // 10 밀리초 대기
		}
	}()

	go func() {                               // 값을 읽는 고루틴
		for i := 0; i &lt; 3; i++ {
			fmt.Println("read 1 : ", data)    // data 값을 출력(읽기)
			time.Sleep(1 * time.Second)       // 1초 대기
		}
	}()

	go func() {                               // 값을 읽는 고루틴
		for i := 0; i &lt; 3; i++ {
			fmt.Println("read 2 : ", data)    // data 값을 출력(읽기)
			time.Sleep(2 * time.Second)       // 2초 대기
		}
	}()

	time.Sleep(10 * time.Second)              // 10초 동안 프로그램 실행
}
</code></pre>
<p id="bkmrk-%EC%9D%B4%EB%A0%87%EA%B2%8C-%ED%95%98%EB%A9%B4-%EC%9D%BD%EA%B8%B0-%EC%8B%9C%EB%8F%84%EC%A4%91-%EA%B0%92%EC%9D%B4-%EB%B0%94%EB%80%94-">이렇게 하면 읽기 시도중 값이 바뀔 가능성이 있다. 쓰기에는 Lock() 읽기에는 RLock()을 걸면 된다.</p>
<pre id="bkmrk-package-main-import--1"><code class="language-go">package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

func main() {
	runtime.GOMAXPROCS(runtime.NumCPU()) // 모든 CPU 사용

	var data int = 0
	var rwMutex = new(sync.RWMutex)

	go func() { // 값을 쓰는 고루틴
		for i := 0; i &lt; 3; i++ {
			rwMutex.Lock()
			data += 1                         // data에 값 쓰기
			fmt.Println("write   : ", data)   // data 값을 출력
			time.Sleep(10 * time.Millisecond) // 10 밀리초 대기
			rwMutex.Unlock()
		}
	}()

	go func() { // 값을 읽는 고루틴
		for i := 0; i &lt; 3; i++ {
			rwMutex.RLock()
			fmt.Println("read 1 : ", data) // data 값을 출력(읽기)
			time.Sleep(1 * time.Second)    // 1초 대기
			rwMutex.RUnlock()
		}
	}()

	go func() { // 값을 읽는 고루틴
		for i := 0; i &lt; 3; i++ {
			rwMutex.RLock()
			fmt.Println("read 2 : ", data) // data 값을 출력(읽기)
			time.Sleep(2 * time.Second)    // 2초 대기
			rwMutex.RUnlock()
		}
	}()

	time.Sleep(10 * time.Second) // 10초 동안 프로그램 실행
}
</code></pre>
<p id="bkmrk-sync.rwmutex%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4-%EC%97%AC"><code>sync.RWMutex</code>를 사용하면 여러 고루틴이 데이터에 안전하게 접근할 수 있으며, 읽기 연산이 많은 경우에는 병렬로 처리하여 성능을 향상시킬 수 있습니다. 하지만 쓰기 잠금이 획득되면 다른 고루틴의 읽기 및 쓰기 잠금 획득을 막기 때문에, 쓰기 잠금을 최대한 짧게 유지하여 동시성을 향상시키는 것이 중요합니다.</p>
<p id="bkmrk-"><a href="https://docs.monitorapp.com/uploads/images/gallery/2024-04/GkPimage.png" target="_blank" rel="noopener"><img src="https://docs.monitorapp.com/uploads/images/gallery/2024-04/scaled-1680-/GkPimage.png" alt="image.png"></a></p>
<h3 id="bkmrk-%EC%A1%B0%EA%B1%B4-%EB%B3%80%EC%88%98-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0">조건 변수 사용하기</h3>
<p id="bkmrk-%EC%A1%B0%EA%B1%B4-%EB%B3%80%EC%88%98%EB%8A%94-%EB%8C%80%EA%B8%B0%ED%95%98%EA%B3%A0-%EC%9E%88%EB%8A%94-%EA%B0%9D%EC%B2%B4-%ED%95%98%EB%82%98">조건 변수는 대기하고 있는 객체 하나만 개우거나 여러 개를 동시에 깨울 때 사용합니다.</p>
<p id="bkmrk-sync-%ED%8C%A8%ED%82%A4%EC%A7%80%EC%97%90%EC%84%9C-%EC%A0%9C%EA%B3%B5%ED%95%98%EB%8A%94-%EC%A1%B0%EA%B1%B4-%EB%B3%80">sync 패키지에서 제공하는 조건 변수의 함수는 다음과 같스니다.</p>
<ul id="bkmrk-sync.cond-%2Afunc-newc">
<li class="null">sync.Cond</li>
<li class="null">*func NewCond(l Lcoker) Cond: 조건 변수 생성</li>
<li class="null">func (c *Cond) Wait(): 고루틴 실행을 멈추고 대기</li>
<li class="null">func (c *Cond) Signal(): 대기하고 있는 고루틴 하나만 깨움</li>
<li class="null">func (c *Cond) Broadcast(): 대기하고 있는 모든 고루틴을 깨움</li>
</ul>
<p id="bkmrk-%EB%8C%80%EA%B8%B0%ED%95%98%EA%B3%A0-%EC%9E%88%EB%8A%94-%EA%B3%A0%EB%A3%A8%ED%8B%B4%EC%9D%84-%ED%95%98%EB%82%98%EC%94%A9-%EA%B9%A8%EC%9B%8C%EB%B3%B4">대기하고 있는 고루틴을 하나씩 깨워보는 예제</p>
<pre id="bkmrk-package-main-import--2"><code class="language-go">package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	runtime.GOMAXPROCS(runtime.NumCPU()) // 모든 CPU 사용

	var mutex = new(sync.Mutex)    // 뮤텍스 생성
	var cond = sync.NewCond(mutex) // 뮤텍스를 이용하여 조건 변수 생성

	c := make(chan bool, 3) // 비동기 채널 생성

	for i := 0; i &lt; 3; i++ {
		go func(n int) {                    // 고루틴 3개 생성
			mutex.Lock()                    // 뮤텍스 잠금, cond.Wait() 보호 시작
			c &lt;- true                       // 채널 c에 true를 보냄
			fmt.Println("wait begin : ", n)
			cond.Wait()                     // 조건 변수 대기
			fmt.Println("wait end : ", n)
			mutex.Unlock()                  // 뮤텍스 잠금 해제, cond.Wait() 보호 종료

		}(i)
	}

	for i := 0; i &lt; 3; i++ {
		&lt;-c // 채널에서 값을 꺼냄, 고루틴 3개가 모두 실행될 때까지 기다림
	}

	for i := 0; i &lt; 3; i++ {
		mutex.Lock()                // 뮤텍스 잠금, cond.Signal() 보호 시작
		fmt.Println("signal : ", i)
		cond.Signal()               // 대기하고 있는 고루틴을 하나씩 깨움
		mutex.Unlock()              // 뮤텍스 잠금 해제, cond.Signal() 보고 종료
	}

	fmt.Scanln()
}
</code></pre>
<p id="bkmrk-%C2%A0-0"><br></p>
<p id="bkmrk--0"><a href="https://docs.monitorapp.com/uploads/images/gallery/2024-04/mUrLoHimage.png" target="_blank" rel="noopener"><img src="https://docs.monitorapp.com/uploads/images/gallery/2024-04/scaled-1680-/mUrLoHimage.png" alt="image.png"></a></p>
<h3 id="bkmrk-%C2%A0-%C2%A0">함수를 한 번만 실행하기</h3>
<p id="bkmrk-once-%ED%8C%A8%ED%82%A4%EC%A7%80%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4-%ED%95%A8%EC%88%98%EB%A5%BC-%ED%95%9C">Once 패키지를 사용하면 함수를 한 번만 실행할 수 있다.&nbsp;</p>
<h4 id="bkmrk-sync.once-%ED%8C%A8%ED%82%A4%EC%A7%80">Sync.Once 패키지</h4>
<ul id="bkmrk-%C2%A0sync.once-func-%28%2Aon">
<li class="null">&nbsp;sync.Once</li>
<li class="null">func (*Once) Do(f func()) : 함수를 한 번만 실행</li>
</ul>
<pre id="bkmrk-package-main-import--3"><code class="language-go">package main

import (
	"fmt"
	"runtime"
	"sync"
)

func hello() {
	fmt.Println("Hello, world!")
}

func main() {
	runtime.GOMAXPROCS(runtime.NumCPU()) // 모든 CPU 사용

	once := new(sync.Once) // Once 생성

	for i := 0; i &lt; 3; i++ {
		go func(n int) {   // 고루틴 3개 생성
			fmt.Println("goroutine : ", n)

			once.Do(hello) // 고루틴은 3개지만 hello 함수를 한 번만 실행
		}(i)
	}

	fmt.Scanln()
}
</code></pre>
<p id="bkmrk-%EC%9D%B4%EB%A0%87%EA%B2%8C-%ED%95%98%EB%A9%B4-hello%ED%95%A8%EC%88%98%EB%8A%94-%ED%95%9C-%EB%B2%88%EB%A7%8C">이렇게 하면 hello함수는 한 번만 실행된다.</p>