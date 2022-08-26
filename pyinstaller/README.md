# Pyinstaller 打包多文件夹的项目

Pyinstaller 是著名python打包module，软件或者小工具用python写好后，用pyinstaller 可以实现轻松打包，前期有几个同事用的基本上是很简单的单行式

```shell
"C:\path_of_packages\Scripts\pyinstaller.exe" --hidden-import plyer.platforms.win.notification --icon=xil.ico --noconsole --onefile --clean --name name_of_exe main_py_file.py
ECHO Exe wurde erfolgreich generiert.
pause
``` 
将上述代码copy到本地，新建一个 batch file，然后点击bat开始执行就完事。需要做的更改也很少，compile的时候只需要把路径改成pyinstaller 的path，要编译的main py添加到里面即可。
但是，问题就是一个项目的运行，尤其是带GUI的，肯定有一些资源文件夹，如果采用上述的方式就会导致资源文件夹不被打包进去的情况。release的时候需要把资源文件夹放在和exe一个文件夹里，用户会直接看到内部的文件和数据，可以说是非常不专业的处理方法。
```
---Folder
-----my_exe.exe
-----Images/
-----utils/
```
关于这个怎么把文件夹打包进去的问题，我前前后后在网上找了很久 stackoverflow, csdn，官方的docu我也都看了， 感觉写的不太好，最近在stackoverflow的一个帖子里找到了灵感。 首先，我们要通过pyinstaller 得到一个基础的spec file， 如果没有的话可以copy下面的空代码到本地。
```
# -*- mode: python ; coding: utf-8 -*-


block_cipher = None


a = Analysis(
    ['xxxx.py'],
    pathex=[],
    binaries=[],
    datas=[],
    hiddenimports=['plyer.platforms.win.notification'],
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[],
    win_no_prefer_redirects=False,
    win_private_assemblies=False,
    cipher=block_cipher,
    noarchive=False,
)

pyz = PYZ(a.pure, a.zipped_data, cipher=block_cipher)

exe = EXE(
    pyz,
    a.scripts,
    a.binaries,
    a.zipfiles,
    a.datas,
    [],
    name='xxxx',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=True,
    upx_exclude=[],
    runtime_tmpdir=None,
    console=False,
    disable_windowed_traceback=False,
    argv_emulation=False,
    target_arch=None,
    codesign_identity=None,
    entitlements_file=None,
    icon='xxx.ico',
)
```

下一步就是把资源最好全塞到一个文件夹('libraries')里面，我主要有两个资源文件夹，一个是图片(images/)，另一个是语言池(i18n/)，两个都给他塞到libraries 的文件夹里。在spec文件里面修改 Analysis的 datas :
```
datas=[('libraries','libraries')]
```

接下来需要在自己的project里面连接上这个 'libraries' , 因为我的项目主要分两部分，一个是GUI，另一个是业务逻辑，两者都通过一个start.py  的主函数连接起来，在主函数中添加连接函数: 
```python
# visit resource lib  
def resource_path(relative_path):  
    # check if Bundle Resource  
  if getattr(sys, 'frozen', False):  
        base_path = sys._MEIPASS  
    else:  
        base_path = os.path.abspath(".")  
    return os.path.join(base_path, relative_path)
```
主函数中引用连接函数: 
```python
if __name__ == "__main__":  
    libraries_path = resource_path('libraries')
    my_obj = gui_main.class_obj(libraries_path)
```
GUI的object 定义一个属性，剩下的子类继承这个属性，就可以完成全项目path的引用:
```python
def __init__(self,lib_dir):
    self.dir = lib_dir
```
改好后就可以用pyinstaller 进行打包了，先把pyinstaller 加入环境变量，然后在项目的地址栏输入cmd 进入命令行，并输入下面的的代码， xxxxxx.spec 为刚刚修改的spec文件: 
```
pyinstaller xxxxxx.spec
```
在project目录下方会出现两个文件夹，一个是build/ 一个是dist/, dist里就是compile好的exe。
是不是很简单呢，但是之前走了很多很多弯路，唉。
