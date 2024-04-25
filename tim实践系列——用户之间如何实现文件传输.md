> 通讯应用中，用户之间文件传输的实现方式有许多，通常会考虑P2P，减少服务器的负载并加速传输过程，但是P2P不是百分百能成功，内网穿透可能会失败；因此，一般也要考虑用服务器中转。本文主要讲服务器中转传输，基于IM实现的方式。
> 通讯应用中用户间文件传输并非必须依赖IM实现，但IM的基础功能天然可以用于实现文件互传，因此用IM实现文件传输或许会更为便捷。一般，文件传输是用户双方在线收发文件。类似微信实现离线发送文件则需要在服务器提供暂存文件的磁盘空间，由用户上传文件到服务器后，通知对方从服务器下载文件。
> 流数据传输是TIM的基础功能，用于实现在线传输文件非常简便，而且效率非常高，传输速度限制一般会出现在带宽上。
> 订阅[《tim实践系列》](https://github.com/donnie4w/Tim-Practical-Article)文章，了解如何使用tim

##### 本文实现文件传输主要调用 [tim](https://tlnet.top/tim) 接口实现

----------

#### 文件传输在TIM中的应用实例：webtim的文件传输功能
![https://tlnet.top/f/1703861232_9923.jpg](https://tlnet.top/f/1703861232_9923.jpg)

![https://tlnet.top/f/1703861242_18609.jpg](https://tlnet.top/f/1703861242_18609.jpg)

![https://tlnet.top/f/1703870693_31151.jpg](https://tlnet.top/f/1703870693_31151.jpg)

这是在线实时文件传输一种应用方式，不占用服务器磁盘空间，由服务器中转文件数据流。文件会被切分为若干部分有序的数据块，进行分批次传输，接收方会多次接收数据块直至数据完整后合并成文件并存盘。

实际上文件传输有许多细节，如错误处理，重传机制，网络中断，数据完整性校验，这些不是本文的重点，所以商业产品在真正实现该功能时，应当考虑进去，这里则不讲述这些问题的处理。

#### TIM提供了有两种个接口，可以用于实现文件数据的传输：

* StreamToUser   基于TimMessage
* PushStream      基于TimStream

**说明：**

1. 使用StreamToUser 接口与发普通信息没有区别，等同于将文件的二进制数据如同普通信息发送给接收方，这个过程可以按照策略将大文件切分为若干小数据块分批次发送。StreamToUser 发送的是在线数据，因此不会被持久化，也没有离线数据，这个是与普通信息的区别。
2. 使用PushStream 接口并非向某个用户发送数据，而是向虚拟房间推流，接收数据的终端通过订阅虚拟房间来接收数据流。这是与StreamToUser流程上的区别。PushStream本质是一种流数据分发的实现，通过不重复的虚拟房间账号，保证流数据准确的推送与接收。这个实现更适用于大量的推流与大量的订阅流数据的场景，如现场视频直播等。


#### 具体文件传输的实现，以webtim的实现代码为例：

**webtim采用tim推流的接口实现文件传输**

1. 发送者通过tim接口通知对方接收文件，

该通知包含的数据：文件名，文件大小，接收数据的虚拟房间账号。

    let s = { "name": files[0].name, "size": files[0].size, "vnode": jsonres.n }
    tc.StreamToUser(recvnode, JSON.stringify(s), 0, 11);

2.接收者确定是否接收该文件：

    if (confirm(s, "是否接收")) {
      recvfileMap.set(jf.vnode, new fileBean(jf.name, jf.size, tm.fromTid.node));
      bigDataStreamEvent.set(jf.vnode, 2)
      tc.VirtualroomSubBinary(jf.vnode)
      showsendToast();
      sleep(1000).then(() => tc.StreamToUser(node, jf.vnode, 0, 13))
      } else {
       tc.StreamToUser(node, jf.vnode, 0, 12);
     }

3.收到确认接收的通知后，开始发送文件数据流：

    function sendfilestart(vnode) {
	let sfb = sendfileMap.get(vnode)
	readfile(sfb.file, (data) => {
		let size = data.length;
		let a = Math.floor(size / HKB);
		showsendToast()
		let t = document.getElementById("sendfileshow");
		let rateid = document.getElementById("rateid");
		let bt = '<button class="btn btn-light" onclick="cancelsendfile('' + vnode + '',true)">取消发送</button>'
		if (a > 0) {
			rateid.innerHTML = bt
			let i = 0
			sendfileLoop(vnode, data, 0, a, (d) => {
				i = i + d
				let r = i / size
				if (r > 1) {
					r = 1
				}
				t.innerHTML = "<h6 style='font-size:smaller;'>" + sfb.file.name + "已经发送:" + Math.floor(100 * r) + "%</h6>";
			})
		} else {
			tc.PushBigDataStream(vnode, gzip(data))
			sleep(100).then(() => tc.PushBigDataStream(vnode, new Uint8Array([0])));
			t.innerHTML = "<h6 style='font-size:smaller;'>" + sfb.file.name + "已经发送:100%</h6>";
			sendfileMap.delete(vnode)
		}
	});
    }


4.接收方接收数据流直至完毕后保存文件：

	const recvfileEvent = (data, node) => {
	if (data.length == 1 && data[0] == 0) {
		let fn = recvfileMap.get(node)
		if (!isEmpty(fn)) {
			let t = mergeUint8Arrays(fn.body);
			saveFile(t, fn.filename)
			recvfileMap.delete(node);
			showsendToast();
			document.getElementById("sendfileshow").innerHTML = "<h6 style='font-size:smaller;'>[" + fn.filename + "]已经接收完毕</h6>"
		}
	} else {
		let fn = recvfileMap.get(node)
		let bt = '<button class="btn btn-light" onclick="cancelrecvfile('' + node + '')">取消接收</button>'
		let rateid = document.getElementById("rateid")
		if (rateid.innerHTML != bt) {
			rateid.innerHTML = bt
		}
		if (!isEmpty(fn)) {
			fn.add(ungzip(data))
			let r = fn.getsize / fn.size;
			document.getElementById("sendfileshow").innerHTML = "<h6 style='font-size:smaller;'>" + fn.filename + "已经接收:" + Math.floor(100 * r) + "%</h6>";
		}
	}
    }

这样就完成了webtim文件传输的全过程

说明：webtim的文件传输只是为了展示tim的流传输接口的用法而开发的简单传文件功能，没有实现断点续传，重传等共功能。
