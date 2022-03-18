### MIT6.824之MapReduce实现

这篇文章主要是大致分析一下MapReduce的实现，具体代码见Github

首先，依照MIT的lab已经给出了大致代码。

我们的实现主要分为两个部分，master.go 和 worker.go。另外还定义了用于RPC的结构rpc.go

```go
// Master 结构体
type Master struct {
	// Your definitions here.
	AllFilesName        map[string]int	  //记录文件名
	MapTaskNumCount     int
	NReduce             int               // 记录需要生成的Reduce task数量
	InterFIlename       [][]string        // intermediate file
	MapFinished         bool
	ReduceTaskStatus    map[int]int      // 记录Reduce状态
	ReduceFinished      bool              // 记录是否完成reduce，完成后任务结束
	RWLock              *sync.RWMutex
}

//两个chan用来存放map任务 和 Reduce任务
var maptasks chan string          // chan for map task
var reducetasks chan int          // chan for reduce task
```



```go
//master.go

//前期准备工作

func MakeMaster(files []string, nReduce int) *Master {} //初始化Master结构体，开启server服务，会调用master.server()

func (m *Master) server() {} //注册RPC，调用generateTask()产生任务，等待worker发来的任务请求

func (m *Master) generateTask() {} //将Master结构体中AllFileName所有文件加载入 maptasks, 再处理完成后，再加载reduce任务进入reducetasks


// 接受远程调用，为worker分配任务

func (m *Master) MyCallHandler(args *MyArgs, reply *MyReply) error {} //负责接受worker发来的请求，有三种请求MsgForTask，
//一是请求任务，请求map任务以及请求reduce任务，根据不同的请求作不同的处理。需要设置reply为相应的值。并且运行一个守护协程timerForWorker()
//二是请求结束map任务MsgForFinishMap，用来通知master map任务结束。设置AllFileName[filename]为Finished状态
//三是请求结束reduce任务MsgForFinishReduce。同样需要设置ReduceTaskStatus[index]为Finished状态


func (m *Master)timerForWorker(taskType, identify string){} //设置定时器为10秒，10秒内完成了任务，则设置相应的任务为Finished
//否则重新将任务设置为未完成状态并加入相应的任务队列
```

```gp
//worker.go

func Worker(mapf func(string, string) []KeyValue, 
            reducef func(string, []string) string) {} //主函数，死循环，不断地调用CallForTask()函数请求任务，并根据master分配的
												  //任务类型调用mapInWorker(), 或者reduceInWorker()


func CallForTask(msgType int,msgCnt string) MyReply {} // 调用RPC，返回请求到的任务


func mapInWorker(reply *MyReply,mapf func(string, string) []KeyValue) {} 
//调用mapf做map任务， 并调用WriteToJSONFile()写入JSON文件中，																   
//调用SendInterFiles，表明已处理到中间状态																  
//并调用CallForTask(MsgForFinishMap, reply.Filename)表明结束该任务

func reduceInWorker(reply *MyReply, reducef func(string, []string) string) {} 
//与mapInWorker同理
//统计的原理是按照key排序，然后遍历统计

func SendInterFiles(msgType int, msgCnt string, nReduceType int) MyReply {}
//发送中间文件到master表明位置
```

