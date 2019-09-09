# Node.js
### 简介：
  Node.js是一个专注于实现高性能Web服务器优化的专家，是一个让JavaScript运行在服务器端的开发平台，它没有根目录，没有任何web容器。
  Node.js不是一种独立的语言，Node.js使用JavaScript进行编程，Node.js底层是C++，运行在JavaScript引擎上（V8）。
  它跳过了Apache、IIS等HTTP服务器，它自己不用建设在任何服务器软件之上
  有效解决服务器高性能瓶颈问题，花最小的硬件成本，追求更高的并发，更高的处理性能
### 官网：https://nodejs.org/en/
### 特点：
#### 单线程、非阻塞I/O、事件驱动机制
### 优势：
  而非阻塞模式下，一个线程永远在执行计算操作，这个线程的CPU核心利用率永远是100%，操作系统完全不再有线程创建、销毁的时间开销
### Node.js适合用来开发什么样的应用程序呢？
善于I/O，不善于计算。因为Node.js最擅长的就是任务调度，如果业务有很多的CPU计算，实际上也相当于这个计算阻塞了这个单线程，就不适合Node开发。<br>
适合与web socket配合，开发长连接的实时交互应用程序。例如：聊天室、图文直播等

## 豆瓣电影爬虫
#### 需要安装两个基本的依赖和一个文件系统模块<br>
request ：用于下载网页<br>
cheerio ：用于解析网页数据，是处理网页文档的利器
fs ：nodejs文件系统模块
#### 安装依赖命令如下 <br>
npm install request cheerio

#### 分析一下，一个网络爬虫要做的事情：
获取网页内容（http\request\superagent等）<br>
筛选网页信息（cheerio）<br>
输出或存储信息（console\fs\mongodb等）<br>


```
// 定义一个类来保存电影的信息
// 分别是  电影名 评分 引言 排名 封面图片地址
const Movie = function() {

	this.name = ''
	this.score = 0
	this.quote = ''
	this.ranking = 0
	this.coverUrl = ''
}

// 这个函数来从一个电影 div 里面读取电影信息
const movieFromDiv = function(div) {
	const movie = new Movie()
	// 使用 cheerio.load 函数来返回一个可以查询的特殊对象
	const e = cheerio.load(div)

	// 然后就可以使用 querySelector 语法来获取信息了
	// .text() 获取文本信息
	movie.name = e('.title').text()
	movie.score = e('.rating_num').text()
	movie.quote = e('.inq').text()

	const pic = e('.pic')
	movie.ranking = pic.find('em').text()
	// 元素的属性用 .attr('属性名') 确定
	movie.coverUrl = pic.find('img').attr('src')

	return movie
}

const saveMovies = function(movies) {
	// 这个函数用来把一个保存了所有电影对象的数组保存到文件中
	const fs = require('fs')
	const path = 'douban.txt'
	// 第二个参数是 null 不用管
	// 第三个参数是 缩进层次
	const s = JSON.stringify(movies, null, 2)
	fs.writeFile(path, s, function(error) {
		if(error !== null) {
			log('*** 写入文件错误', error)
		} else {
			log('--- 保存成功')
		}
	})
}
const moviesFromUrl = function(url) {
	// request 从一个 url 下载数据并调用回调函数
	request(url, function(error, response, body) {
		// 回调函数的三个参数分别是  错误, 响应, 响应数据
		// 检查请求是否成功, statusCode 200 是成功的代码
		if(error === null && response.statusCode == 200) {
			// cheerio.load 用字符串作为参数返回一个可以查询的特殊对象
			// body 就是 html 内容
			const e = cheerio.load(body)
			const movies = []
			// 查询对象的查询语法和 DOM API 中的 querySelector 一样
			const movieDivs = e('.item')
			for(let i = 0; i < movieDivs.length; i++) {
				let element = movieDivs[i]
				// 获取 div 的元素并且用 movieFromDiv 解析
				// 然后加入 movies 数组中
				const div = e(element).html()
				const m = movieFromDiv(div)
				movies.push(m)
			}
			// 保存 movies 数组到文件中
			saveMovies(movies)
			downloadCovers(movies)
		} else {
			log('ERROR 请求失败', error)
		}
	})
}

const downloadCovers = function(movies) {
    for (let i = 0; i < movies.length; i++) {
        const m = movies[i]
        const url = m.coverUrl
        const path = m.name.split('/')[0] + '.jpg'
        request(url).pipe(fs.createWriteStream(path))
	
	}
}
const __main = function() {
	// 这是主函数
	// 下载网页, 解析出电影信息, 保存到文件
	const url = 'https://movie.douban.com/top250'
	moviesFromUrl(url)
}

'''
#### 我们在爬取网页的时候尤其要注意编码格式。一旦爬取的编码格式和输出的编码格式不一致将会产生乱码。
#### 所以遇到较为早期的网站用的是非utf-8编码的将会比较头疼。
这里使用向外暴露js函数的方法，外部js文件引用其他的js函数。

'''
const saveJSON = function(path, answers) {
    // 用来把一个保存了所有电影对象的数组保存到文件中
    const fs = require('fs')
    const s = JSON.stringify(answers, null, 2)
    fs.writeFile(path, s, function(error) {
        if (error !== null) {
            console.log('*** 写入文件错误', error)
        } else {
            console.log('--- 保存成功')
        }
    })
}


// 通过 exports 制作自己的模块
exports.saveJSON = saveJSON

