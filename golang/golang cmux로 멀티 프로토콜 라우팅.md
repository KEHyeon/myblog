golang의 cmux패키지를 이용하여 페이로드를 기반으로 연결을 다중화 할 수 있습니다.

<https://github.com/soheilhy/cmux>

이 패키지를 이용하면 하나의 tcp 리스너에서 grpc, http, http2 등 다양한 프로토콜을 제공할 수 있습니다.

먼저 라이브러리를 사용해보며 설명을 하도록 하겠습니다.

```go
m := cmux.New(l)
```
먼제 패키지를 임포트해주고 tcp 리스너를 하나 만들고 cmux 를 생성해줍니다.

```go 
Match(...Matcher) net.Listener
```
Cmux interface에 정의 되어 있는 다음 메서드를 이용하여 리스너를 생성 할 수 있습니다.
함수를에 등록된 Matcher 순서대로 매칭하며 먼저 매칭된 리스너에 연결되게 됩니다.

Matcher 타입을 살펴보겠습니다.
```go
type Matcher func(io.Reader) bool
```
이렇게 io.Reader interface를 받은후 bool type을 리턴해주는 함수를 등록할 수있습니다. 
io.Reader를 이용하여 tcp연결후 처음 들어온 데이터를 얻고 그것을 분석하여 매칭하는 함수를 구현하며 mux에 추가하면 됩니다.
이를 이용하여 tcp위에서 동작하는 프로토콜의 matcher를 구현할 수 있습니다.

```go
grpcL := m.Match(cmux.HTTP2HeaderField("content-type", "application/grpc"))
httpL := m.Match(cmux.HTTP1Fast())
trpcL := m.Match(cmux.Any())
```
이렇게 cmux에 다양항 매처를 추가할 수 있습니다.
match함수는 muxListener 구조체를 리턴합니다.
```go
go grpcS.Serve(grpcL)
go httpS.Serve(httpL)
go trpcS.Accept(trpcL)
```

그 각 프로토콜의 serve함수에 이렇게 등록할 수 있습니다.
```go
func (l muxListener) Accept() (net.Conn, error) {
	select {
	case c, ok := <-l.connc:
		if !ok {
			return nil, ErrListenerClosed
		}
		return c, nil
	case <-l.donec:
		return nil, ErrServerClosed
	}
}
```
그후 cmux를 serve 해주면 끝입니다. 그럼 serve안에서 cmux에서 감싸준 muxListener의 accept함수를 호출하며 대기 합니다.
```
// Start serving!
m.Serve()
```

serve의 내부 구현을 간단하게 살펴 봅시다.
```go
for _, sl := range m.sls {
	for _, s := range sl.ss {
		matched := s(muc.Conn, muc.startSniffing())
		if matched {
			muc.doneSniffing()
			if m.readTimeout > noTimeout {
				_ = c.SetReadDeadline(time.Time{})
			}
			select {
			case sl.l.connc <- muc:
			case <-donec:
				_ = c.Close()
			}
			return
		}
	}
}
```
이렇게 위에서 채널로 대기중인 accept리스너중 처음 매칭된 리스너에 커넥션을 주며 serve되게 됩니다.


마지막으로 
cmux에서 제공해주는 HTTP1Fast Matcher가 어떻게 구현 되어있는지 확인해보도록 하겠습니다.

```go
var defaultHTTPMethods = []string{
	"OPTIONS",
	"GET",
	"HEAD",
	"POST",
	"PUT",
	"DELETE",
	"TRACE",
	"CONNECT",
}
func HTTP1Fast(extMethods ...string) Matcher {
	return PrefixMatcher(append(defaultHTTPMethods, extMethods...)...)
}
```
PrefixMatcher는 첫 메시지를 읽고 메서드 Prefix를 이용하여 http1 메시지인지 판단하고 http listner에 매칭시켜 줍니다.

이는 확실하지 않지만 나름 정확하고 빠른다는 장점을 가집니다.
정확한  판단을 위해 비교적 느리지만 cmux에서 제공해주는 http1 matcher를 이용할 수 있습니다.

끝!
