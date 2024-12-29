golang에서 gin이라는 프레임워크를 이용하면 쉽게 api 서버를 구성할 수 있습니다.

먼저 다음 명령어를 이용하여 gin을 설치합니다. 
```
go get -u github.com/gin-gonic/gin
```

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	r.Run()
}
```
이렇게 간단하게 /ping path에 get 요청을 보냈을때 json을 응답하는 api를 구성했습니다.

먼저 게시판 repository를 구성해보도록 하겠습니다.
```go
type Post struct {
	Id      int    `json:"id"`
	Title   string `json:"title"`
	Content string `json:"content"`
}
type PostRepository interface {
	Save(Post) error
	Update(Post) error
	Delete(id int) error
	FindAll() Post
	FindById(id int) (Post, error)
}
```
이런식으로 간단한 post 구조체와 repository의 interface룰 정의해봤습니다.
이제 postRepository 구현체를 만들어보도록 하겠습니다.
```go
type PostRepositoryImpl struct {
	PostMap   map[int]*Post
	repoMutex sync.Mutex
	lastIndex int
}

func (p *PostRepositoryImpl) Save(post Post) error {
	p.repoMutex.Lock()
	defer p.repoMutex.Unlock()
	p.PostMap[p.lastIndex+1] = &post
	p.lastIndex += 1
	return nil
}
func (p *PostRepositoryImpl) Update(post Post) error {
	p.repoMutex.Lock()
	defer p.repoMutex.Unlock()
	if _, ok := p.PostMap[post.Id]; !ok {
		return fmt.Errorf("Post Id : %d Not Found", post.Id)
	}
	p.PostMap[post.Id] = &post
	return nil
}
func (p *PostRepositoryImpl) Delete(id int) error {
	p.repoMutex.Lock()
	defer p.repoMutex.Unlock()
	if _, ok := p.PostMap[id]; !ok {
		return fmt.Errorf("Post Id : %d Not Found", id)
	}
	delete(p.PostMap, id)
	return nil
}
func (p *PostRepositoryImpl) FindAll() []Post {
	p.repoMutex.Lock()
	defer p.repoMutex.Unlock()
	ret := make([]Post, len(p.PostMap))
	idx := 0
	for _, p := range p.PostMap {
		ret[idx] = *p
		idx++
	}
	return ret
}
func (p *PostRepositoryImpl) FindById(id int) (Post, error) {
	p.repoMutex.Lock()
	defer p.repoMutex.Unlock()

	if post, ok := p.PostMap[id]; !ok {
		return Post{}, fmt.Errorf("Post Id : %d Not Found", post.Id)
	} else {
		return *post, nil
	}
}
```
이렇게 map을 이용하여 간단한 postRepository를 구현했습니다.
이제 다음 장에서는 postRepository의 테스트 코드를 작성하도록 하겠습니다.