[toc]
## CodeReview-Checklist
### 1. 前言

- codereview时要注意编程规范，参考uber的编程规范。https://github.com/uber-go/guide/blob/master/style.md


- 参考google的codereview方式：https://google.github.io/eng-practices/review/reviewer/


### 2. review重点

- **设计**： review最重要的是看总体的设计，是否有意义，是否与系统很好集成，是否过度设计
- **功能**：考察代码的功能是否正确实现，需要reviewer理解业务，站在需求的角度去看代代码功能是否实现
- **复杂性**：本次review的复杂度是否超过预期，单行代码是否过于复杂，单个函数是否过于复杂。“复杂”通常意味着该代码很难阅读，很有可能也意味着其他开发者在修改这段代码的时候引入其他bug。还有一种复杂性就是过度设计(Over-engineering)造成的，开发者会让那段代码过度通用化，超过了原本所需，或者还新增了系统目前不需要的功能。

### 3. review速度

- **review时间**：mr提交后review代码的时间一定不能太久，会拖慢开发进度，这个时间必须控制在24小时以内，最优的建议当然是越快越好
- **避免阻塞**：如果没有时间review，需要尽快通知开发者，换一个人去review代码
- **大型改动**：如果可以最好要求开发人员将大型改动拆分成更多小的提交，如果是在无法拆分，且没有足够时间review整个内容时，至少对它的整体设计写些评论，并发送回开发人员以便进行改进。直到有时间review或者把这个mr提交给其他人
- **紧急状态**： 在某些紧急状态下，可以适当放宽review标准，但是在紧急情况之后，需要再次review之前放宽标准的代码。可以参考[紧急情况](https://google.github.io/eng-practices/review/emergencies.html#what)

### 4. MR评论

- 当开发人员不同意你的建议时，思考一下谁是正确的，并解释清楚，保持礼貌。
- 如果现在无法解决review的评论的问题的话，TODO的注释并链接到刚记录下的缺陷。
- 如果有评论问题没有回复确认，需要确认清楚

### 5. 测试

- 提交一般有适当的测试，不要忘记review测试的代码
- 合并代码前需要跑通之前的所有测试case才能执行合并操作，这一步可以交给gitlabci执行
- 关键代码必须要单元测试

### 6. 文档

- 如果修改的内容影响到文档的时候，请检查是否也同时更新相关文档， 包括README、设计文件和其他参考文件。

### 7. 代码规范

#### 7.1 代码风格

##### 7.1.1 静态检查

- 所有代码都应该通过`golint`、`go vet`、`gofmt`的检查并无错误， 即提交前请执行make build

##### 7.1.2 换行

- 建议一行代码不要超过120列，超过的情况，使用合理的换行方法换行。

##### 7.1.3 import规范

- 包导入需要按照系统、本地包、第三方包的顺序。换行隔开

  ```go
  import (
	"os"
	"os/signal"
	"syscall"
  
	"gitlab.sz.sensetime.com/SenseRadar/radarservice/config"
	"gitlab.sz.sensetime.com/SenseRadar/radarservice/monitor"
	"gitlab.sz.sensetime.com/SenseRadar/radarservice/server"
	
  
	log "github.com/sirupsen/logrus"
  )
  ```

##### 7.1.4 类型断言

- 类型断言均需要判断

  ```go
  t, ok := i.(string)
  if !ok {
    // 优雅地处理错误
  }
  ```

#### 7.2 命名规范

##### 7.2.1 包名规范

- 全部小写。没有大写或下划线。

- 大多数使用命名导入的情况下，不需要重命名。

- 简短而简洁。请记住，在每个使用的地方都完整标识了该名称。

- 不用复数。例如`net/url`，而不是`net/urls`。

- 不要用“common”，“util”，“shared”或“lib”。这些是不好的，信息量不足的名称。

  

##### 7.2.2 别名规范

- 如果程序包名称与导入路径的最后一个元素不匹配，则必须使用导入别名。在所有其他情况下，除非导入之间有直接冲突，否则应避免导入别名。

  ```go
  import (
    "net/http"
  
    client "example.com/client-go"
    trace "example.com/trace/v2"
  )
  ```

##### 7.2.3 变量命名

- 避免使用内置名称

  ```go
  type Foo struct {
      // 虽然这些字段在技术上不构成阴影，但`error`或`string`字符串的重映射现在是不明确的。
      error  error
      string string
  }
  ```

- 变量名必须遵循驼峰式，首字母根据访问控制决定使用大写或小写

- 若变量类型为`bool`类型，则名称应以`Has`，`Is`，`Can`或者`Allow`开头。

  

#### 7.3 数据类型&语法

##### 7.3.1 channel 

- channel 如果不为0或者1,任何其他尺寸都必须经过严格的审查。

  ```go
  // 该数值是否合理，需要界定通道边界，竞态条件，以及逻辑上下文梳理)
  c := make(chan int, 64)
  ```


##### 7.3.2 字符串

- 字符串转义请使用原生字符串

  ```go
  //不推荐： wantError := "unknown name:\"test\""
  wantError := `unknown error:"test"`
  ```

##### 7.3.3 for 

-  for range时不要往原始slice中append元素，无限循环

  ```go
  // 错误示范
  arr := []int{1, 2, 3}
  for _, v := range arr {
  	arr = append(arr, v)
  }
  fmt.Println(arr)
  ```

- for range 取临时变量指针保存，只能取到最后一个值

  ```go
  // 错误示范
  arr := []int{1, 2, 3}
  newArr := []*int{}
  for _, v := range arr {
  	newArr = append(newArr, &v)
      //正确应该取 newArr = append(newArr, &arr[i])
  }
  for _, v := range newArr {
  	fmt.Println(*v)
  }
  ```

##### 7.3.4 make 

- 在尽可能的情况下，在初始化为`make()`提供一个容量值

  ```go
  data := make([]int, 0, size)
  for k := 0; k < size; k++{
  data = append(data, k)
  }
  
  files, _ := ioutil.ReadDir("./files")
  m := make(map[string]os.FileInfo, len(files))
  for _, f := range files {
      m[f.Name()] = f
  }
  ```
