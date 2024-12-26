고랭은 클로저를 형성할 수 있는 익명 함수를 지원한다.
익명함수는 함수이름을 지정하지 않고 함수를 인라인으로 작성할때 유용하다.

```go
func intSeq() func() int {
    i := 0
    return func() int {
        i++
        return i
    }
}
```

위 함수는 함수를 반환한는데 그 함수는 리턴문에 인라인으로 정의 됩니다.
반환된 함수는 변수를 닫아서 i클로저를 형성 합니다.

이제 위 함수를 이용해 다른 변수에 반환된 함수를 할당 해 봅시다.
```go
newFunc := intSeq()
```
newFunc함수는 i 값을 유지하며 업데이트 합니다.
한번 호출하며 확인해 봅시다.

```go
newFunc := intSeq()
for _ = range 3 {
	fmt.Println(newFunc())
}
newFunc = intSeq()
fmt.Println(newFunc())
```
위의 출력 값은
```
1
2
3
1
```
입니다 3번 값이 업데이트되고 새로운 함수로 할당된 후에는 다시 1로 초기화 되는것을 확인 할 수 있습니다.