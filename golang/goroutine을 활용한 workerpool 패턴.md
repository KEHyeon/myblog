<p id="bkmrk-allabs-open-api%EB%A5%BC-%ED%94%84%EB%A1%9D%EC%8B%9C"><br></p>
<h3 id="bkmrk-go-worker-pool-%EC%9D%B4%EB%9E%80%3F">Go Worker Pool 이란?</h3>
<p id="bkmrk-woker-pool%EC%9D%80-%EA%B3%A0%EC%A0%95%EB%90%9C-%EC%88%98%EC%9D%98-%EC%9E%91">woker Pool은 고정된 수의 작업자를 사용하여 큐에 있는 여러 작업을 실행하여 동시성을 구현하는 패턴이다. GO 생태계에서는 고루틴을 사용하여 작업자를 생성하고 채널을 사용하여 큐를 구현한다.</p>
<p id="bkmrk-%EC%A0%95%EC%9D%98%EB%90%9C-%EC%9E%91%EC%97%85%EC%9E%90-%EC%88%98%EB%8A%94-%ED%81%90%EC%97%90%EC%84%9C-%EC%9E%91%EC%97%85%EC%9D%84-%EA%B0%80">정의된 작업자 수는 큐에서 작업을 가져와 작업을 완료하며, 작업이 완료되면 새 작업을 계속 가져와서 진행한다.</p>
<p id="bkmrk-%C2%A0"><br></p>
<h3 id="bkmrk-%ED%95%84%EC%9A%94%EC%84%B1">필요성</h3>
<ul id="bkmrk-%EC%9E%A5%EC%A0%90-%EC%9E%90%EC%9B%90-%EC%9E%AC%EC%82%AC%EC%9A%A9-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%95%88%EC%A0%95%EC%84%B1-%ED%96%A5%EC%83%81">
<li class="null">
<h4 id="bkmrk-%EC%9E%A5%EC%A0%90-0">장점</h4>
<ul>
<li class="null">
<h5 id="bkmrk-%EC%9E%90%EC%9B%90-%EC%9E%AC%EC%82%AC%EC%9A%A9-0">자원 재사용</h5>
</li>
<li class="null">
<h5 id="bkmrk-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%95%88%EC%A0%95%EC%84%B1-%ED%96%A5%EC%83%81-0">시스템 안정성 향상</h5>
</li>
<li class="null">
<h5 id="bkmrk-%EB%A6%AC%EC%86%8C%EC%8A%A4-%EA%B4%80%EB%A6%AC-%EC%9A%A9%EC%9D%B4%EC%84%B1-0">리소스 관리 용이성</h5>
</li>
<li class="null">
<h5 id="bkmrk-%EC%9E%91%EC%97%85-%EC%B2%98%EB%A6%AC-%EC%8B%9C%EA%B0%84-%EA%B0%90%EC%86%8C-0">작업 처리 시간 감소</h5>
</li>
</ul>
</li>
<li>
<h4 id="bkmrk-%EB%8B%A8%EC%A0%90-0">단점</h4>
<ul>
<li>
<h5 id="bkmrk-%EC%B4%88%EA%B8%B0-%EA%B5%AC%EC%84%B1-%EB%B3%B5%EC%9E%A1%EC%84%B1-0">초기 구성 복잡성</h5>
</li>
<li>
<h5 id="bkmrk-%EB%A6%AC%EC%86%8C%EC%8A%A4-%EB%82%AD%EB%B9%84-%EA%B0%80%EB%8A%A5%EC%84%B1-0">리소스 낭비 가능성</h5>
</li>
<li>
<h5 id="bkmrk-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%98%A4%EB%B2%84%ED%97%A4%EB%93%9C-0">동기화 오버헤드</h5>
</li>
<li>
<h5 id="bkmrk-%EB%8D%B0%EB%93%9C%EB%9D%BD-%EB%B0%8F-%EA%B2%BD%EC%9F%81-%EC%83%81%ED%83%9C-0">데드락 및 경쟁 상태</h5>
<ul>
<li>채널을 이용한 해결 방안</li>
<li>lock을 이용한 해결 방안</li>
</ul>
</li>
</ul>
</li>
</ul>
<p id="bkmrk-%C2%A0-0"><br></p>
<p id="bkmrk-%EC%BD%94%EB%93%9C-%EA%B5%AC%EC%A1%B0-%EC%84%A4%EB%AA%85">코드 구조 설명</p>
<pre id="bkmrk-type-workerpool-stru"><code class="language-go">type WorkerPool struct {
	MaxWorkers int
	JobQueue   chan Job
}</code></pre>
<p id="bkmrk-workerpool-%EA%B5%AC%EC%A1%B0%EC%B2%B4%EC%9D%B4%EB%8B%A4-max">WorkerPool 구조체이다 MaxWorkers와 Job이 들어오는 Job 채널이 선언 되어있다.</p>
<pre id="bkmrk-type-job-struct-%7B-ta"><code class="language-Go">type Job struct {
	task     func() interface{} // 작업자에 의해 처리될 작업
	resultCh chan&lt;- Result      // 작업 결과를 반환할 채널
}</code></pre>
<p id="bkmrk-job-struct%EC%9D%B4%EB%8B%A4.-%EC%9E%91%EC%97%85-%ED%95%A8%EC%88%98%EC%99%80">Job struct이다. 작업 함수와 결과 반환 채널이 들어있다. 필요에 따라 정보를 추가 할 수도 있다.</p>
<pre id="bkmrk-type-result-struct-%7B"><code class="language-go">type Result struct {
	value interface{} // 작업의 결과
}</code></pre>
<p id="bkmrk-%EC%9E%91%EC%97%85%EC%9D%98-%EA%B2%B0%EA%B3%BC-%EA%B5%AC%EC%A1%B0%EC%B2%B4%EC%9D%B4%EB%8B%A4.">작업의 결과 구조체이다.</p>
<pre id="bkmrk-func-%28wp-%2Aworkerpool"><code class="language-go">func (wp *WorkerPool) Start() {
	for i := 0; i &lt; wp.MaxWorkers; i++ {
		go func(workerID int) {
			for job := range wp.JobQueue {
				fmt.Printf("Worker %d: 작업을 시작합니다.\n", workerID)
				result := job.task() // 실제 작업을 수행하고 결과를 가져옴.
				job.resultCh &lt;- Result{value: result} // 결과 반환
				fmt.Printf("Worker %d: 작업이 완료되었습니다.\n", workerID)
			}
		}(i)
	}
}</code></pre>
<p id="bkmrk-%EC%9B%8C%EC%BB%A4%ED%92%80-%EC%8B%9C%EC%9E%91-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%9D%B4%EB%8B%A4.">워커풀 시작 메서드 이다.</p>
<p id="bkmrk-maxworkers-%EB%A7%8C%ED%81%BC-%EA%B3%A0%EB%A3%A8%ED%8B%B4%EC%9D%84-%EB%8F%8C">MaxWorkers 만큼 고루틴을 돌린다.</p>
<p id="bkmrk-%EA%B0%81%EA%B0%81%EC%9D%98-%EA%B3%A0%EB%A3%A8%ED%8B%B4%EC%9D%80-jobqueue%EC%9D%98-%EC%8B%A0">각각의 고루틴은 JobQueue의 신호를 기다리고 신호가 오면 작업을 수행하며 result가 올때까지 블로킹 된다.</p>
<pre id="bkmrk-func-%28wp-%2Aworkerpool-0"><code class="language-go">func (wp *WorkerPool) AddJob(task func() interface{}, resultCh chan&lt;- Result) {
	wp.JobQueue &lt;- Job{task: task, resultCh: resultCh}
}</code></pre>
<p id="bkmrk-%EC%9E%91%EC%97%85%ED%81%90%EC%97%90-%EC%9E%91%EC%97%85%EC%9D%84-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%9D%B4%EB%8B%A4.">작업큐에 작업을 메서드 이다.</p>
<pre id="bkmrk-%2F%2F-shutdown-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EC%9B%8C%EC%BB%A4-"><code class="language-go">// Shutdown 메서드는 워커 풀을 종료합니다.
func (wp *WorkerPool) Shutdown() {
	close(wp.JobQueue)
}
</code></pre>
<p id="bkmrk-%EC%9B%8C%EC%BB%A4%ED%92%80%EC%9D%84-%EC%A2%85%EB%A3%8C%ED%95%98%EB%8A%94-%EB%A9%94%EC%84%9C%EB%93%9C%EC%9D%B4%EB%8B%A4.">워커풀을 종료하는 메서드이다.</p>
<p id="bkmrk-jobqueue%EB%A5%BC-%EB%8B%AB%EA%B2%8C-%EB%90%98%EB%A9%B4-%EB%8C%80%EA%B8%B0%ED%95%98%EA%B3%A0">JobQueue를 닫게 되면 대기하고 있는 고루틴들이 종료가된다.</p>
<h3 id="bkmrk-%EC%A0%84%EC%B2%B4%EC%BD%94%EB%93%9C">전체코드</h3>
<pre id="bkmrk-package-main-import-"><code class="language-GO">package main

import (
	"fmt"
	"sync"
	"time"
)

// Job은 처리할 작업과 결과를 반환할 채널을 포함합니다.
type Job struct {
	task     func() interface{} // 작업자에 의해 처리될 작업
	resultCh chan&lt;- Result      // 작업 결과를 반환할 채널
}

// Result는 작업의 결과를 포함합니다.
type Result struct {
	value interface{} // 작업의 결과
}

// WorkerPool은 워커의 수와 작업 채널을 정의합니다.
type WorkerPool struct {
	MaxWorkers int
	JobQueue   chan Job
}

// NewWorkerPool 함수는 새로운 워커 풀을 생성합니다.
func NewWorkerPool(maxWorkers int) *WorkerPool {
	pool := &amp;WorkerPool{
		MaxWorkers: maxWorkers,
		JobQueue:   make(chan Job),
	}
	return pool
}

// Start 메서드는 워커 풀을 시작합니다.
func (wp *WorkerPool) Start() {
	for i := 0; i &lt; wp.MaxWorkers; i++ {
		go func(workerID int) {
			for job := range wp.JobQueue {
				fmt.Printf("Worker %d: 작업을 시작합니다.\n", workerID)
				result := job.task() // 실제 작업을 수행하고 결과를 가져옴.
				job.resultCh &lt;- Result{value: result} // 결과 반환
				fmt.Printf("Worker %d: 작업이 완료되었습니다.\n", workerID)
			}
		}(i)
	}
}

// AddJob 메서드는 워커 풀에 새로운 작업을 추가합니다.
func (wp *WorkerPool) AddJob(task func() interface{}, resultCh chan&lt;- Result) {
	wp.JobQueue &lt;- Job{task: task, resultCh: resultCh}
}

// Shutdown 메서드는 워커 풀을 종료합니다.
func (wp *WorkerPool) Shutdown() {
	close(wp.JobQueue)
}

func main() {
	pool := NewWorkerPool(5)
	pool.Start()

	var wg sync.WaitGroup
	resultCh := make(chan Result, 10) // 결과를 수신할 채널

	// 여러 작업을 워커 풀에 추가
	for i := 0; i &lt; 10; i++ {
		wg.Add(1)
		job := func() interface{} {
			defer wg.Done()
			// 실제 작업을 수행 
			// 여기서는 시뮬레이션을 위해 Sleep을 사용
			time.Sleep(2 * time.Second)
			return fmt.Sprintf("작업 결과 %d", i+1)
		}
		pool.AddJob(job, resultCh)
	}

	go func() {
		wg.Wait()
		close(resultCh)
	}()

	for result := range resultCh {
		fmt.Println("수신된 결과:", result.value)
	}

	pool.Shutdown()
	fmt.Println("모든 작업이 완료되었습니다.")
}</code></pre>
<h3 id="bkmrk-%EC%96%B4%EB%94%94%EB%8B%A4-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C%3F">어디다 사용할까?</h3>
<p id="bkmrk-proxy%EA%B0%9C%EB%B0%9C%EC%9D%84-%ED%95%98%EB%8A%94%EC%A4%91-allabs%EC%9D%84">proxy개발을 하는중 ALlabs을 백그라운드에 돌리고 곧 바로 응답을 해줘야 하는 상황이 왔다.</p>
<p id="bkmrk-%EC%9E%91%EC%97%85%EC%9D%B4-%EB%A7%A4%EC%9A%B0-%EC%98%A4%EB%9E%98%EA%B1%B8%EB%A6%AC%EA%B3%A0-%EC%A3%BC%EA%B8%B0%EC%A0%81%EC%9C%BC%EB%A1%9C-h">작업이 매우 오래걸리고 주기적으로 http요청을 보내야 하므로 무작정 고루틴을 만들면 안된다고 생각했고 워커풀 패턴을 떠올렸다.</p>
<p id="bkmrk-%C2%A0-1"></p>