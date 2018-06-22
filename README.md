# data_cookbook
数据处理、数据可视化。有关pandas、pyecharts、d3等，轮子代码备忘，以后直接抄。

# 数据获取

## 极简爬虫线程池，通过正规 API 申请数据

```
class CrawlThreadPool(object):
    '''
    启用最大并发线程数为5的线程池进行URL链接爬取及结果解析；
    最终通过crawl方法的complete_callback参数进行爬取解析结果回调
    '''
    def __init__(self):
        self.thread_pool = ThreadPoolExecutor(max_workers=5)

    def _request_parse_runnable(self, addr):
        try:
            param={
                'apikey':'xxxxxxx',
                'module':'account',
                'address':addr
                 }
            response= requests.get('http://api.xxxxx.io/api', params=param)
            data_json = json.loads(response.text)
            
            if data_json['message'] == 'OK':
                data = response.text
            else:
                print('    状态值:' + data_json['message'])
                data = None
        except BaseException as e:
            print(str(e))
            data = None
        return data

    def crawl(self, addr, complete_callback):
        future = self.thread_pool.submit(self._request_parse_runnable, addr)
        future.add_done_callback(complete_callback)


class OutPutThreadPool(object):
    '''
    启用最大并发线程数为5的线程池对上面爬取解析线程池结果进行并发处理存储；
    '''
    def __init__(self):
        self.thread_pool = ThreadPoolExecutor(max_workers=5)

    def _output_runnable(self, crawl_result, addr):
        try:
            if crawl_result is None:
                return
            output_filename = os.path.join(".", "data", addr + ".json")
            with open(output_filename, 'a') as output_file:
                output_file.write(crawl_result)
                output_file.write('\n')
            print('saved success!')
        except Exception as e:
            print('save file error. ->'+str(e))

    def save(self, crawl_result, ADDRESSdir):
        self.thread_pool.submit(self._output_runnable, crawl_result, ADDRESSdir)


class CrawlManager(object):
    '''
    爬虫管理类，负责管理爬取解析线程池及存储线程池
    '''
    def __init__(self, addr):
        self.crawl_pool = CrawlThreadPool()
        self.output_pool = OutPutThreadPool()
        self.addr = addr

    def _crawl_future_callback(self, crawl_url_future):
        try:
            data = crawl_url_future.result()
            self.output_pool.save(data, self.addr)
        except Exception as e:
            print('Run crawl url future thread error. '+str(e))

    def start_runner(self):
        for startblock in range(5000000, 6000000, 10000):# 块在[5000000, 6000000]之间
            self.crawl_pool.crawl(self.addr, startblock, self._crawl_future_callback)


if __name__ == '__main__':
    CrawlManager(addr).start_runner()
```

# 数据 io

## pandas 对 csv 处理
读取 csv
```
import pandas as pd
data = pd.read_csv(r"data_set/data.csv")
```
选中 data 表里的某些列
```
relation = data[['id', 'inviter', 'email']]
```
每列共多少不重复的项
```
relation.count()
```
读取 inviter 列的第 0 行
```
relation['inviter'][0]
```
判断是否是数字
```
math.isnan(relation['inviter'][0])
```

## 对 json 处理

读取 json 文件 `./data/data.json`， `json_object` 是字典，像 python 的基础数据类型一样用
```
import json
json_file = os.path.join('.', 'data', 'data.json')
    if os.path.exists(json_file):
        with open(json_file, 'r') as input_file:
            json_object = json.load(input_file)
```

保存为 json 文件，加 `indent=4` 则自动格式化缩进
```
data = {'a': 0, 'b':[[{}],[{}]] }
with open('data.json', 'a') as datajson:
    json.dump(data, datajson, indent=4)
```


# 数据清洗

## 排序
字典排序
```
itemsList = sorted(list(balances_dataSet.items()), key=lambda item:item[0])
```
# 数据可视化

## pyecharts

文档：[pyecharts图表配置](http://pyecharts.org/#/zh-cn/charts?id=bar%EF%BC%88%E6%9F%B1%E7%8A%B6%E5%9B%BE%E6%9D%A1%E5%BD%A2%E5%9B%BE%EF%BC%89)

文档里例子代码充足

## d3.js
配合 json 文件
1. 启动服务端口
```shell
python -m http.server 8000
```
2. 浏览器访问 `127.0.0.1:8000`

文档：[Gallery](https://github.com/d3/d3/wiki/Gallery)
example：[Examples](https://bl.ocks.org/mbostock)

example里例子代码充足

