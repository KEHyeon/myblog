<h3 id="bkmrk-go%EC%97%90%EC%84%9C%EB%8A%94-json-%ED%8C%A8%ED%82%A4%EC%A7%80%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C">Go에서는 json 패키지를 이용해서 json 데이터를 다루기</h3>
<p id="bkmrk-%C2%A0"><br></p>
<h3 id="bkmrk-%EB%A7%88%EC%83%AC%EB%A7%81">마샬링</h3>
<p id="bkmrk-go-%EC%9E%90%EB%A3%8C%ED%98%95%EC%9D%84-json-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%A1%9C-%EB%B3%80%EA%B2%BD">Go 자료형을 Json 데이터로 변경한다. Marshal 함수 사용</p>
<pre id="bkmrk-func-marshal%28v-inter"><code class="language-go">func Marshal(v interface{}) ([]byte, error)</code></pre>
<h3 id="bkmrk-%EC%96%B8%EB%A7%88%EC%83%AC%EB%A7%81">언마샬링</h3>
<p id="bkmrk-json%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%A5%BC-go-%EC%9E%90%EB%A3%8C%ED%98%95%EC%9C%BC%EB%A1%9C-%EB%B3%80%ED%99%98">Json데이터를 Go 자료형으로 변환합니다. Unmarshal 함수 사용</p>
<pre id="bkmrk-func-unmarshal%28data-"><code class="language-go">func Unmarshal(data []byte, v interface{}) error</code></pre>
<p id="bkmrk-%C2%A0-0"><br></p>
<h3 id="bkmrk-%EC%98%88%EC%A0%9C%EB%A5%BC-%ED%86%B5%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0">예제를 통해 알아보기</h3>
<h3 id="bkmrk-%EB%A7%88%EC%83%AC%EB%A7%81-0">마샬링</h3>
<pre id="bkmrk-type-movie-struct-%7B-"><code class="language-go">type Movie struct {
    Id    int
    Title string
    Adult bool
    Year  int
}

func main() {
    m := Movie{1, "The Dark Knight", false, 2008}
    b, err := json.Marshal(m)

    if err != nil {
        log.Fatalf("JSON marshaling failed: %s", err)
    }

    fmt.Println(string(b))
}</code></pre>
<pre id="bkmrk-%7B-%22id%22%3A-1%2C-%22title%22%3A-"><code class="language-">{
  "Id": 1,
  "Title": "The Dark Knight",
  "Adult": false,
  "Year": 2008
}</code></pre>
<p id="bkmrk-%EC%8B%A4%ED%96%89-%EA%B2%B0%EA%B3%BC%EB%A5%BC-%EB%B4%A4%EC%9D%84%EB%95%8C-movie-str">실행 결과를 봤을때 Movie struct가 json 형태로 출력되는 것을 볼 수 있다.</p>
<p id="bkmrk-%EB%A7%88%EC%83%AC%EB%A7%81-%EB%90%98%EB%8A%94-%EA%B2%B0%EA%B3%BC%EC%9D%98-json-%ED%95%84%EB%93%9C%EC%9D%98-">마샬링 되는 결과의 JSON 필드의 key 이름을 변경할 수 있다.</p>
<p id="bkmrk-%EA%B5%AC%EC%A1%B0%EC%B2%B4-%ED%95%84%EB%93%9C-%EC%84%A0%EC%96%B8%EC%97%90-%ED%83%9C%EA%B7%B8%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4-">구조체 필드 선언에 태그를 사용하면 된다.</p>
<pre id="bkmrk-type-movie-struct-%7B--0"><code class="language-go">type Movie struct {
    Id    int    `json:"id"`
    Title string `json:"title"`
    Adult bool   `json:"adult"`
    Year  int    `json:"released"`
}</code></pre>
<p id="bkmrk-omitempty-%EC%98%B5%EC%85%98%EC%9C%BC%EB%A1%9C-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%A5%BC-">omitempty 옵션으로 데이터를 제외 할 수도 있다.</p>
<pre id="bkmrk-type-movie-struct-%7B--1"><code class="language-go">type Movie struct {
    Id    int    `json:"id"`
    Title string `json:"title"`
    Adult bool   `json:"adult,omitempty"`
    Year  int    `json:"released"`
}
</code></pre>
<p id="bkmrk-%C2%A0-1"><br></p>
<h3 id="bkmrk-%EC%96%B8%EB%A7%88%EC%83%AC%EB%A7%81-0">언마샬링</h3>
<p id="bkmrk-unmarshal%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-json">Unmarshal을 이용해서 JSON 데이터를 Go의 값으로 변활할 수 있다.</p>
<pre id="bkmrk-func-main%28%29-%7B-data-%3A"><code class="language-">func main() {

    data := []byte(`
        {
          "id": 1,
          "title": "The Dark Knight",
          "adult": false,
          "released": 2008
        }
        `)

    var movie Movie
    err := json.Unmarshal(data, &amp;movie)
    if err != nil {
        log.Fatalf("JSON unmarshaling failed: %s", err)
        os.Exit(1)
    }

    fmt.Printf("%+v\n", movie)
}</code></pre>
<pre id="bkmrk-%7Bid%3A1-title%3Athe-dark"><code class="language-">{Id:1 Title:The Dark Knight Adult:false Year:2008}</code></pre>
<p id="bkmrk-%EC%8B%A4%ED%96%89-%EA%B2%B0%EA%B3%BC%EB%8A%94-%EB%8B%A4%EC%9D%8C%EA%B3%BC-%EA%B0%99%EB%8B%A4.">실행 결과는 다음과 같다.</p>
<p id="bkmrk-%C2%A0-2"><br></p>
<h3 id="bkmrk-array-json-%EB%A7%88%EC%83%AC%EB%A7%81">Array Json 마샬링</h3>
<pre id="bkmrk-func-main%28%29-%7B-m-%3A%3D-%5B"><code class="language-go">func main() {
    m := []Movie{
        {1, "The Dark Knight", false, 2008},
        {2, "The Godfather", true, 1972},
        {3, "GoodFellas", true, 1990},
    }

    b, err := json.Marshal(m)
    if err != nil {
        log.Fatalf("JSON marshaling failed: %s", err)
    }

    fmt.Println(string(b))
}</code></pre>
<pre id="bkmrk-%5B-%7B-%22id%22%3A-1%2C-%22title%22"><code class="language-go">[
  {
    "id": 1,
    "title": "The Dark Knight",
    "released": 2008
  },
  {
    "id": 2,
    "title": "The Godfather",
    "adult": true,
    "released": 1972
  },
  {
    "id": 3,
    "title": "GoodFellas",
    "adult": true,
    "released": 1990
  }
]</code></pre>
<p id="bkmrk-%EC%8B%A4%ED%96%89-%EA%B2%B0%EA%B3%BC%EB%8A%94-%EB%8B%A4%EC%9D%8C%EA%B3%BC-%EA%B0%99%EB%8B%A4.-0">실행 결과는 다음과 같다.</p>