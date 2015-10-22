# storejsplus

## 概况
基于[store.js](https://github.com/marcuswestin/store.js)实现的跨浏览器持久化数据方案。

## 详解
webStorage（标准浏览器 + ie 8+） + userData （ie 6/7）。
由于最近项目场景恰好需要sessionStorage，故将store.js中默认的loaclStorage改为sessionStorage。具体方式可按需选择。

###webStorage
webStorage没什么好说，不过还是要留意下存储上限，不同浏览器间有差异，详见[http://dev-test.nemikor.com/web-storage/support-test/](http://dev-test.nemikor.com/web-storage/support-test/)。

###userData
userDate的存储上限却很麻烦，[详见](https://msdn.microsoft.com/en-us/library/ms531424(v=vs.85).aspx)。

<table>
  <tbody>
    <tr>
      <th> Security Zone </th>
      <th> Document Limit (KB) </th>
      <th> Domain Limit (KB) </th>
    </tr>
    <tr>
      <td> Local Machine </td>
      <td> 128 </td>
      <td> 1024 </td>
    </tr>
    <tr>
      <td> Intranet </td>
      <td> 512 </td>
      <td> 10240 </td>
    </tr>
    <tr>
      <td> Trusted Sites </td>
      <td> 128 </td>
      <td> 1024 </td>
    </tr>
    <tr>
      <td> Internet </td>
      <td> 128 </td>
      <td> 1024 </td>
    </tr>
    <tr>
      <td> Restricted </td>
      <td> 64 </td>
      <td> 640 </td>
    </tr>
  </tbody>
</table>

稳妥起见，将数据分割成最小的每片64K分别存储。仅数据分片还不行，还要为每个数据分片load一个StorageName。
``` javascript
storage.load(store.userDataStorageName)
```
这样才会在用户本地生成相应的物理文件来存储数据。注意：不要轻易删除userdata的物理文件，否则浏览器可能报错，需要通过清IE缓存（有时候还需要重启电脑，原因未知，但确实解决了问题）解决。

###跨目录访问
跨目录暂时没用到，userDate可通过window.name处理，webStorage可通过Communication APIs处理，这里不再细说。

###存储耗时
200K测试数据，webStorage仅2ms左右，userData每片64K，200K分四片，需要2000ms。

###关于数据大小和字符长度的关系
``` javascript
<meta charset="utf-8">
```
utf-8是在Unicode基础上衍生出的可变长度字符编码，一个字符可占1-4个字节。

规则：
<div>
  <p> Unicode符号范围   |        UTF-8编码方式<br>
    (十六进制)          |              （二进制）<br>
    --------------------+---------------------------------------------<br>
    0000 0000-0000 007F | 0xxxxxxx<br>
    0000 0080-0000 07FF | 110xxxxx 10xxxxxx<br>
    0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx<br>
    0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx</p>
  <p>如果一个字节的第一位是0，则这个字节单独就是一个字符；如果第一位是1，则连续有多少个1，就表示当前字符占用多少个字节。</p>
</div>

###测试
在文本文件中使用UTF-8编码。

var s = "一";
s.charCodeAt(0).toString(16)为4e00，根据上表，“一”占3字节。

utf-8有一个“头”，根据头编辑器选择合适的编码显示。例二个汉字“一二”，16进制查看为“EF BB BF E4 B8 80 E4 BA  8C ”，头3字节、一3字节、二3字节，共9字节。

