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
这样才会在用户本地生成相应的物理文件来存储数据。

###跨目录访问
跨目录暂时没用到，userDate可通过window.name处理，webStorage可通过Communication APIs处理，这里不再细说。

###存储耗时
200K测试数据，webStorage仅2ms左右，userData每片64K，200K分四片，需要2000ms。

###关于数据大小和字符长度的关系
``` javascript
<meta charset="utf-8">
```
页面charset为utf-8，而utf-8又是基于Unicode产生的，所以测试数据一个字母为16位、2个字节。故字符串长度×2即为数据体积。