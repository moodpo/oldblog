# 解决Ubuntu更新源Hash Sum mismatch错误

- date: 2013-04-13
- category: Work
- tags: ubuntu, source

----

有时候在更新Ubuntu时会发生如下错误：

````
Fetched 9,718 kB in 36s (268 kB/s)                                             
W: Failed to fetch bzip2:/var/lib/apt/lists/partial/mirrors.163.com_ubuntu_dists_precise_restricted_source_Sources  Hash Sum mismatch

W: Failed to fetch gzip:/var/lib/apt/lists/partial/mirrors.163.com_ubuntu_dists_precise-updates_restricted_source_Sources  Hash Sum mismatch

E: Some index files failed to download. They have been ignored, or old ones used instead.
````    

这是由于一些文件被墙的原因导致更新源服务器中的文件不完整，此时可以用Goagent代理去下载：

执行命令： 

````
sudo apt-get -o Acquire::http::proxy="http://127.0.0.1:8087/" update
````

<a><img src="/media/2013/04/3587804018.png" alt="Hash Sum mismatch Error.png" /></a>