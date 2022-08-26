# Net use 的大小坑	

公司很多数据都使用windows局域网分享盘(net use)，我们公司用的是O  盘， 使用LAN Crypt，可以在文件右下角看到一个绿色或黄色的小钥匙，数据库会实时在这个O盘里写入一个json数据，储存台架的状态和使用情况，如果我们要使用这个json处理一些东西，会收到Permission Error 的报错，所以需要解决的问题有两个:
1. 刚刚开机的时候，这个O盘的状态是"Unavailable，只有鼠标点进O盘后，状态才会变正常，所以每次用户必须手动点击O盘
2. 因为业务逻辑需求，O盘里的json需要被读取，尽管确认有读写权，但是不知道为什么有时候（10次会有一两次）会出现permission error，导致程序读写的数据不具有实时性，而且这个error确认是无法消除的。

这段时间一直在思考为什么会有O盘的问题
最开始以为是别人打开了就不能再open，但是测试了一下就是随机的，有时候可以有的时候就是permission error，看网上说可以把网盘的copy到本地再进行读写操作，测试了一下下面的代码: 

``` python
import shutil  
  
target_dir = r"C:\Users\xxxxxx\AppData\Local\our_project_name\data.json"  
src_dir = r"O:\our_orga_name\MePro\our_project_name\data.json"  
shutil.copy(src_dir, target_dir)  
  
with open(target_dir) as data:  
    print("success")
```
Stackoverflow 上面
 发现还是有时出现permission error 我特么真的无语了，加了异常处理之后完成了
```python
def open_data_from_net(self, target_dir):  
 # This function will process (read & write) data in net use properly  
 # due to the unstable properties of net drive(O), 
 # Python Intepreter will encounter Error # The Exception(Error) here is --> [Errno 13] Permission denied:  
    state = 1  
    while state:  
        try:  
            with open(target_dir,'rb') as data:  
                loaded_data = json.load(data)  
                state = 0  
                return loaded_data  
        except:  
            continue
```
