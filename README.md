# 项目简介
这个项目为音乐播放器，功能有放歌，切歌（上一曲下一曲暂停）以及切歌时背景图片会发生改变，歌曲进度调整，歌曲时间和进度条的对应。
预览链接如下：
https://wuxuweilalala.github.io/musicplay1/index.html

# 项目制作
## 1.静态页面制作（包括html和css）
## 2.播放效果制作（Javascript）
## 3.音乐相关api

### 1.静态页面制作
#### html部分
  首先是一个div，class为cover，这样方便之后歌曲切换背景图片的切换。
  之后把音乐播放器包裹在div下，分为两部分上方的音乐部分（包括歌曲控制器上一曲下一曲暂停以及歌曲信息歌曲名和作者），下方的进度条和时间。
  这里面的icon我用的是www.npmjs.com的font-awesome ，进去之后搜索font-awesome，点击搜索结果的第一个。进去他的github，里面有icon的网站https://fontawesome.com ，这个是我们找图标用的。然后进入css，我用的是这个https://unpkg.com/font-awesome@4.7.0/css/font-awesome.min.css。
#### css部分
  首先进行样式重置：
  ```
      * {
        margin: 0;
        padding: 0;
    }
    body {
        height: 100vh;
    }
  ```
  给cover设置为绝对定位，+背景图片，display为block，宽高100%，background-size: cover铺满。</br>
  给musicbox绝对定位，left: 50%; top: 40%;transform: translate(-50%, -50%)居中，设置宽度width: 340px;颜色字体：color: #f06d6a;font-family: cursive;</br>
  music-panel画板，设置边框border: 1px solid #76dba3;间距padding: 20px 20px 5px 20px;阴影：box-shadow: 0px 2px 5px 0px rgba(0, 0, 0, 0.1), 0px 2px 10px 0px rgba(0, 0, 0, 0.05);背景颜色：background-color: rgba(255, 255, 255, 0.9);</br>
  icon（.music .control）：上边距：margin-top: 20px;字体和颜色：font-size: 22px;color: #ee8a87;然后左浮动：float: left；因为要靠左嘛，**用完浮动一定要清除浮动**。</br>
  .music .control .fa:icon之间的边距：margin-right: 12px;cursor: pointer;</br>
  信息（歌曲名和作者）：这个比较简单，仅仅就是左边距与字体大小
  ```
      .musicbox .info {
        margin-left: 120px;
    }
    .musicbox .info .title {
        font-size: 18px;
    }
    .musicbox .info .author {
        font-size: 13px;
    }
  ```
  接下来是进度条：进度条的设计是由porgress-total和progress-now组成的，total的颜色是.bar一样，now的颜色单独设置，这样之后js运作起来就可以看到效果了。
  ```
      .musicbox .progress .bar {
        height: 3px;
        margin-top: 5px;
        background-color: rgba(0, 0, 0, 0.2);
        cursor: pointer;
    }
    .musicbox .progress .progress-now {
        background-color: #ee8a87;
        height: 3px;
        width: 0;
        position: relative;
    }
    .musicbox .time {
        text-align: right;
    }
  ```
  最后是清除浮动:control用了左浮动。
  ```
      .musicbox:after,
    .musicbox .music:after {
        content: '';
        display: block;
        clear: both;
    }
  ```
  ### 2.播放效果制作（Javascript）
  #### Ajax获取数据
  Ajax中涉及的json数据我们最后一部分讲。
  ```
          function getMusicList(callback){
            var xhr = new XMLHttpRequest()
            xhr.open('GET', '/js/index.json', true)
            xhr.send()
            xhr.onload = function(){
                if((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304){
                callback(JSON.parse(this.responseText))
            }else {
                console.log('获取数据失败')
            }
            }           
            xhr.onerror = function(){
                console.log('网络异常')
            }

        }

  ```
  自动播放：
  ```
        var musicList = [0]
        var currentIndex = 0
        var clock
        var audio = new Audio()
        audio.autoplay = true

        getMusicList(function(list){
            musicList = list
            loadMusic(list[currentIndex])
            generateList(list)
        })
        function loadMusic(musicObj){
          console.log('begin play', musicObj)
          $('.musicbox .title').innerText = musicObj.title
          $('.musicbox .author').innerText = musicObj.author
          $('.cover').style.backgroundImage = 'url(' + musicObj.img + ')'
          audio.src = musicObj.src
        } 
  
  ```
  进度条：
  ```
          audio.ontimeupdate = function(){
            console.log(this.currentTime)
            $('.musicbox .progress-now').style.width = (this.currentTime/this.duration)*100 + '%'
        }
  
  ```
  时间：
  ```
          audio.onplay = function(){
            clock = setInterval(function(){
            var min = Math.floor(audio.currentTime/60)
            var sec = Math.floor(audio.currentTime)%60 + ''
            sec = sec.length === 2? sec : '0' + sec
            $('.musicbox .time').innerText = min + ':' + sec
            }, 1000)
        }
        audio.onpause = function(){
            clearInterval(clock)
        }
        audio.onended = function(){
            currentIndex = (++currentIndex)%musicList.length
            loadMusic(musicList[currentIndex])
        }

  ```
  点击事件：（1.上一曲；2.下一曲；3.暂停；4.进度条的点击；）
  ```
          $('.musicbox .play').onclick = function(){
            var icon = this.querySelector('.fa')
            if(icon.classList.contains('fa-play')){
            audio.play()
            }else{
            audio.pause()
            }
            icon.classList.toggle('fa-play')
            icon.classList.toggle('fa-pause')
        }
        
        $('.musicbox .forward').onclick = function(){
            currentIndex = (++currentIndex)%musicList.length
            loadMusic(musicList[currentIndex])
        }
        $('.musicbox .back').onclick = function(){
            currentIndex = (musicList.length + (--currentIndex)) %musicList.length
            loadMusic(musicList[currentIndex])
        }

        $('.musicbox .bar').onclick = function(e){
            console.log(e)
            var percent = e.offsetX / parseInt(getComputedStyle(this).width)
            audio.currentTime = audio.duration * percent
            console.log(audio.duration)
        }
 
  ```
  
  ### 3.音乐相关api
  https://xiedaimala.com/tasks/e062b939-5a41-4efd-b48e-16d74255d878/text_tutorials/3bb3afcb-afcf-42b6-a315-f932a95a53f8
  参考资料：[MDN-HTMLMediaElement](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLMediaElement)
           [MDN-Media_events](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Media_events)
  
