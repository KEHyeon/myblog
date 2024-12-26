# fmt.Sprintf
```go
 result := fmt.Sprintf("Name: %s, Age: %d, Height: %.2f meters", name, age, height)
```
`fmt.Sprintf` 함수를 이용하면 이렇게 문자열 포맷팅을 이용해 쉽게 문자열을 생성 할 수 있다.
하지만 그냥 위와 같은 단순 문자열 이어 붙이기를 하는데 `fmt.Sprintf` 남용해버리면 어떤 문제가 벌어질까?
회사 서비스를 `pprof`를 이용하여 프로파일링 했을때 예상치 못하게 `fmt.Sprintf` 함수에 의해 많은 자원이 사용되는것을 볼 수 있었다.
그럼 이제 문자열을 그냥 더하는 여러 방식의 각 성능을 벤치마킹 해보자.

```go
import (
	"fmt"
	"testing"
)

func doNothing(str string) {
	return
}

// + 연산자를 이용한 문자열 이어붙이기
func BenchmarkStringPlus(b *testing.B) {
	for i := 0; i < b.N; i++ {
		for j := 0; j < 1000; j++ {
			doNothing("hello" + "world")
		}
	}
}

// fmt.Sprintf를 이용한 문자열 이어붙이기
func BenchmarkSprintf(b *testing.B) {
	for i := 0; i < b.N; i++ {
		for j := 0; j < 1000; j++ {
			doNothing(fmt.Sprintf("%s%s ", "hello", "world"))
		}
	}
}

```
이렇게 각 코드에 대한 벤치마크 코드를 짜고 실행시켜 보았다.
| Benchmark            | Iterations | Time per Operation (ns/op) |
|----------------------|------------|----------------------------|
| BenchmarkStringPlus-8 | 3,584,295  | 321.5                      |
| BenchmarkSprintf-8    | 22,858     | 52,406                     |


### 벤치마크 분석

#### **BenchmarkStringPlus-8: + 연산자를 이용한 문자열 이어붙이기**
- **3584295**: 벤치마크가 1초 동안 **3,584,295번** 실행되었습니다.
- **321.5 ns/op**: 각 벤치마크 반복마다 **321.5 나노초**가 소요되었습니다.

#### **BenchmarkSprintf-8: fmt.Sprintf를 이용한 문자열 이어붙이기**
- **22858**: 벤치마크가 1초 동안 **22,858번** 실행되었습니다.
- **52406 ns/op**: 각 벤치마크 반복마다 **52,406 나노초**가 소요되었습니다.

### 성능 분석

`+` 연산자와 `fmt.Sprintf`를 이용한 문자열 이어붙이기 성능 차이를 보면 다음과 같은 엄청난 차이를 확인할 수 있습니다:

- **`+` 연산자**는 **3,584,295번** 실행되었으며, 각 실행에 **321.5 나노초**가 소요되었습니다.
- 반면 **`fmt.Sprintf`**는 **22,858번** 실행되었으며, 각 실행에 **52,406 나노초**가 소요되었습니다.

**결론**: `fmt.Sprintf`는 **`+` 연산자**에 비해 약 **163배 느리**다는 사실을 확인할 수 있습니다. (`52406 / 321.5`)

### 성능이 중요한 경우, fmt.Sprintf보다는 + 연산자를 사용하거나 strings.Builder와 같은 최적화된 방법을 사용하는 것이 좋습니다.