<template>
    <div class="game" >
        <canvas id="gameCanvas">
                抱歉,您的手机不支持canvas游戏!!!
        </canvas>
        <div v-show="joiningdots" class="incident" @click="followup()">

        </div>
        <!-- 音乐 -->
        <!-- <music :musicurl="musicurl" ref="musicBoxer"></music> -->
        <!-- 音乐 -->

        <!-- 遮罩 -->
        <!-- <load-box v-if="loaderBox"></load-box> -->
        <!-- 遮罩 -->
    </div>
</template>
<script>
import { clearInterval } from 'timers';
// import * as lib from '@/config/utils'
// import * as conf from '@/config/index'
// import * as api from '@/config/port/App/sketch'
// import loadBox from '../../../public/components/loadBox'
// import music from '../../../public/components/musicBox'
// import { Toast } from 'mint-ui';
const rate = window.devicePixelRatio
let w = window.innerWidth*rate
let h = window.innerHeight*rate
export default {
    data () {
        return {
            canvas: null,
            ctx: null,
            vid: sessionStorage.sketchVid, // 计分者的vid
            loaderBox: true, // 遮罩
            musicurl: '', // 音乐
            imgs: { // 图片
                bgImg: 'http://sucai.weizaojiao.cn/image/pao/gameBg.png', // 大背景
                figure_one: 'http://sucai.weizaojiao.cn/image/pao/player1.png', // 人物图1
                figure_two: 'http://sucai.weizaojiao.cn/image/pao/player2.png',   // 人物图2
                figure_three: 'http://sucai.weizaojiao.cn/image/pao/daodi.png',  // 人物图3摔倒动作图
                surprised_one: 'http://sucai.weizaojiao.cn/image/pao/xiongba1.png', // 目标图1
                surprised_two: 'http://sucai.weizaojiao.cn/image/pao/xiongba2.png', // 目标图2
                gmeter: 'http://sucai.weizaojiao.cn/image/pao/ball.png', // 加速度小球
                obstacle: 'http://sucai.weizaojiao.cn/image/pao/banana.png', // 减速度障碍物
            },
            num: 0, // 得分
            time: 30,   // 倒计时
            bgspeed:w/150,//背景图速度
            bgImgbgspeed:0,//大背景初始位置
            gmeterbgspeed:[],//加速小球位置
            obstaclebgspeed:[],//减速度障碍物位置
            surprisedspeed:{w:w*0.77, h:h*0.58,img:'http://sucai.weizaojiao.cn/image/pao/xiongba1.png',width:w*0.2,height:w*0.14},//目标图位置
            figuredspeed:{w:w*0.01,h:h*0.61,img:'http://sucai.weizaojiao.cn/image/pao/player1.png',width:w*0.2,height:w*0.2},//人物图位置
            addspeed:true,//是否添加加速小球
            adddecelerate:true,//是否添加减速障碍物
            addsurprised:true,//目标切换
            item:1,//目标切换控制
            figureaddsurprised:true,//人物切换
            peoplejump:true,//人物跳跃控制
            figureitem:1,//人物切换控制
            jumphead:true,//人物跳跃到顶部
            jumpspeed:w/80,//人物跳跃速度
            joiningdots:true,//点击跳跃控制
            displacement:true,//人物水平位移控制
            displacementspeed:w/1200,//人物水平位移速度
            suspend:false,//人物追上目标立即暂停
            falldown:true,//人物跌倒控制
            falldownteme:true,//人物跌倒时间控制
        }
    },
    computed: { // 从gameBox拿到的数据和方法
        // canvas: { set() {}, get() {return this.$refs.game.canvas} },
        // ctx: { set() {}, get() {return this.$refs.game.ctx} },
        // drawArcImg: { set() {}, get() {return this.$refs.game.drawArcImg} },//用于canvas渲染圆形图片
        // // drawPic: { set() {}, get() {return this.$refs.game.drawPic} },//按照中心画图
        // calcW: { set() {}, get() {return this.$refs.game.calcW} },//计算图片的宽
        // calcH: { set() {}, get() {return this.$refs.game.calcH} },//计算图片的高
        // randNum: { set() {}, get() {return this.$refs.game.randNum} }//随机整数
    },
    created() {
        // 禁止微信分享
        // lib.wxConfig(['hideMenuItems']).then(() => {
        //     lib.stopShare()
        // })
    },
    mounted() {
        const rate = window.devicePixelRatio;
        $('#gameCanvas').attr('height', window.innerHeight*rate);
        $('#gameCanvas').attr('width', window.innerWidth*rate);
        this.canvas = $('#gameCanvas')[0];
        this.ctx    = $('#gameCanvas')[0].getContext('2d');
        this.init()

    },
    methods: {
        async getData() { // 获取数据
            // const res = await lib.HTTP(`${conf.server}${api.gameBaseInfo}`, {
            //     aid: this.$store.state.aid,
            //     userToken: this.$store.state.userToken,
            //     vid: this.vid
            // })
            // if(res.code == 200) {
            //     this.imgs.bgImg = res.data.other_info.game_bgImg // 背景
            //     this.imgs.figure_one = res.data.user.figure_one // 头像
            //     this.imgs.rule = res.data.other_info.game_rule_simg // 规则弹框
            //     this.imgs.airBalloon = res.data.other_info.game_boximg // 未炸裂的
            //     this.imgs.airBalloonOver = res.data.other_info.game_boximg1 // 已炸裂的
            //     this.imgs.endBg = res.data.other_info.game_rule_eimg // 游戏结束弹窗
            //     this.fontColor = res.data.other_info.game_boxcolor
            //     this.init();
            // } else {
            //     Toast(res.message)
            // }
        },
        init() { // 初始化
        
            let index = 0;
            for(let key in this.imgs) {
                let img = new Image();
                img.src = this.imgs[key];
                img.onload = () => {
                    this.imgs[key] = img;
                    index++;
                    if(index == Object.keys(this.imgs).length) {
                        this.step = 1;
                        // 减少频繁ctx赋值
                        this.ctx.textAlign = 'center'
                        this.ctx.fillStyle = 'rgba(255,255,255,0.8)'
                        this.ctx.font =  window.innerWidth*rate*0.06 + 'px arial';
                        this.play()
                    }
                }
            }
          
        },
        drawBase() { // 绘制动画时间和分数
            this.ctx.clearRect(0, 0, w, h)
            this.ctx.save()
            
            // 大背景
            if(this.bgImgbgspeed <= -this.calcW(this.imgs.bgImg,h)){
                this.bgImgbgspeed = 0;
            }
            this.bgImgbgspeed -= this.bgspeed
            this.ctx.drawImage(this.imgs.bgImg, this.bgImgbgspeed, 0, this.calcW(this.imgs.bgImg,h), h)
            this.ctx.drawImage(this.imgs.bgImg, this.bgImgbgspeed+this.calcW(this.imgs.bgImg,h), 0, this.calcW(this.imgs.bgImg,h), h)

            //绘制减速障碍物
            if(this.adddecelerate){
                var that = this
                this.adddecelerate = false
                var obstaclebgspeedindex = this.randNum(w*1.5,w*2)
                var addtime = this.randNum(1500,3000)
                this.obstaclebgspeed.push(obstaclebgspeedindex)
                setTimeout(function(){
                    that.adddecelerate = true
                },addtime);
            }
            if(this.obstaclebgspeed.length){
                for(var i = 0;i<this.obstaclebgspeed.length;i++){
                    this.obstaclebgspeed[i] -= this.bgspeed
                    this.ctx.drawImage(this.imgs.obstacle, this.obstaclebgspeed[i], h*0.7, w*0.1, w*0.05)
                }
            }
        
            //绘制目标
            if(this.addsurprised){
                var that = this
                this.addsurprised = false
                if(this.figureitem>=2){
                    this.figureitem=1;
                    this.surprisedspeed.img = this.imgs.surprised_one
                }else{
                    this.figureitem++;
                    this.surprisedspeed.img = this.imgs.surprised_two
                }
                setTimeout(function(){
                    that.addsurprised = true
                },300);
            }
            if(this.surprisedspeed){
                this.ctx.drawImage(this.surprisedspeed.img, this.surprisedspeed.w, this.surprisedspeed.h, this.surprisedspeed.width, this.surprisedspeed.height)
            }

            //绘制人物切换
            if(this.figureaddsurprised&&this.peoplejump&&this.falldown){
                var that = this
                this.figureaddsurprised = false
                if(this.item>=2){
                    this.item=1;
                    this.figuredspeed.img = this.imgs.figure_one
                }else{
                    this.item++;
                    this.figuredspeed.img = this.imgs.figure_two
                }

                setTimeout(function(){
                    that.figureaddsurprised = true
                },300);
            }

            //绘制人物跳跃
            if(this.figureaddsurprised&&!this.peoplejump&&this.falldown){
                var that = this
                this.figureaddsurprised = false //切换控制
                this.figuredspeed.img = this.imgs.figure_two
                if(this.jumphead){      //跳跃置顶判断
                    this.figuredspeed.h -= this.jumpspeed   
                    if(this.figuredspeed.h<h*0.355){     //跳跃高度限制
                        this.jumphead = false         //切换到下降模式
                    }
                }else{
                    this.figuredspeed.h += this.jumpspeed 
                    if(this.figuredspeed.h>=h*0.61){  //跳跃最低限制
                        this.figuredspeed.h=h*0.61
                        this.peoplejump = true   //切换到非跳跃模式
                        this.jumphead = true    //跳跃模式
                        this.joiningdots = true  
                        this.displacementspeed = w/1200
                    }
                }
                //碰撞小球
                for(var i = 0;i<this.gmeterbgspeed.length;i++){
                    if(this.figuredspeed.h<=h*0.45&&(this.figuredspeed.w+w*0.2)>=this.gmeterbgspeed[i]&&(this.figuredspeed.w+w*0.2)<=(this.gmeterbgspeed[i]+w*0.29)){
                            this.gmeterbgspeed.splice(i,1)
                            this.num ++
                            this.displacementspeed = w/250 
                    }
                }
                setTimeout(function(){
                    that.figureaddsurprised = true
                },1);
            }

            //绘制人物碰到障碍物
            if(!this.falldown){
                this.figuredspeed.img = this.imgs.figure_three
                if(this.falldownteme){
                    var that = this 
                    setTimeout(function(){
                        that.falldownteme = true
                        that.falldown = true
                        that.joiningdots = true
                    },500);
                }
            }
            if(this.peoplejump){//跳跃模式不执行碰到障碍物false为跳跃
                for(var i = 0;i<this.obstaclebgspeed.length;i++){
                    if((this.figuredspeed.h+w*0.2)>=h*0.65&&(this.figuredspeed.w+w*0.2)>=this.obstaclebgspeed[i]&&(this.figuredspeed.w+w*0.2)<=(this.obstaclebgspeed[i]+w*0.3)){
                        if(this.falldownteme){
                            this.figuredspeed.h = h*0.61
                            this.falldown = false
                            this.joiningdots = false
                            this.peoplejump = true //碰到障碍物不执行跳跃模式
                        }
                    }
                }
            }
            
            //绘制加速小球
            if(this.addspeed){
                var that = this
                this.addspeed = false
                var gmeterbgspeedindex = this.randNum(w*1.1,w*1.5)
                var addtime = this.randNum(1000,3000)
                this.gmeterbgspeed.push(gmeterbgspeedindex)
                setTimeout(function(){
                    that.addspeed = true
                },addtime);
            }
            if(this.gmeterbgspeed.length){
                for(var i = 0;i<this.gmeterbgspeed.length;i++){
                    this.gmeterbgspeed[i] -= this.bgspeed
                    this.ctx.drawImage(this.imgs.gmeter, this.gmeterbgspeed[i], h*0.45, w*0.09, w*0.09)
                }
            }

            //绘制人物水平位移
            if(this.displacement&&this.falldown){
                var that = this
                this.displacement = false //切换控制
                this.figuredspeed.w += this.displacementspeed   //水平速度
                setTimeout(function(){
                    that.displacement = true
                },50);
            }else if(!this.falldown){
                var that = this
                this.displacement = false //切换控制
                if(this.figuredspeed.w>w*0.01){
                    this.figuredspeed.w -= w/700   //跳跃速度
                }
                setTimeout(function(){
                    that.displacement = true
                },50);
            }
            if(this.figuredspeed){
                this.ctx.drawImage(this.figuredspeed.img, this.figuredspeed.w, this.figuredspeed.h, this.figuredspeed.width, this.figuredspeed.height)
            }

            //人物碰撞目标结束游戏
            if(this.figuredspeed.w + this.surprisedspeed.width/2 >= this.surprisedspeed.w){
                this.suspend = true
            }

            // 渲染时间
            // this.ctx.lineWidth=1;//定义线条的宽度
            // this.ctx.strokeStyle="#000";//定义线条的样式
            // this.ctx.font="normal 20px Arial";    //定义由粗到细的字符。400 等同于 normal，而 700 等同于 bold。
            this.ctx.fillText("倒计时",w/2, w*0.1);
            this.ctx.fillText(this.time.toFixed(2), w/2, w*0.192)
            
            // 渲染吃球球个数
            this.ctx.fillText(this.num, w*0.85, w*0.185)
            this.ctx.restore()

            
            
        },
        followup(){    //点击跳跃
            if(this.joiningdots){
                this.joiningdots = false  //连点控制
                this.peoplejump=false  //切换到跳跃模式
            }
        },
        play() { // 游戏中
            // 游戏开始, 发送时间戳
            // this.makeStart();
            // 游戏
            let nowTime = new Date().getTime()
            let endTime = nowTime + this.time*1000;
            let that = this;
            // let timerOut = requestAnimationFrame(function fn(){ 
            //     that.time = (endTime - new Date().getTime())/1000
            //     if(that.time >= 0) {  //判断游戏进行
            //         that.drawBase()  //循环渲染
            //         timerOut = requestAnimationFrame(fn)
            //         if(that.suspend){
            //             cancelAnimationFrame(timerOut)
            //         }
            //     } else if(that.time <= 0) { // 时间已到, 游戏结束
            //         that.time = 0;
            //         that.drawBase()//最后的渲染
            //         cancelAnimationFrame(timerOut)
            //         that.gameOver()
            //     }
            // })
            let timerOuts = setInterval(() => {
                that.time = (endTime - new Date().getTime())/1000
                if(that.time >= 0) {  //判断游戏进行
                    that.drawBase()  //循环渲染
                    if(that.suspend){
                        window.clearInterval(timerOuts)
                    }
                } else if(that.time < 0) { // 时间已到, 游戏结束
                    that.time = 0;
                    that.drawBase()//最后的渲染
                    window.clearInterval(timerOuts)
                    that.gameOver()
                }
            }, 10)
        
        },
        gameOver() { // 游戏结束
            console.log("游戏结束")
        },
        back() { // 返回
            this.$router.replace(sessionStorage.sketchBackRouter);
        },
        makeStart() { // 游戏开始, 发送时间戳
            // lib.HTTP(`${conf.server}${api.gameStart}?timer=${new Date().getTime()}`, {
            //     aid: this.$store.state.aid,
            //     userToken: this.$store.state.userToken,
            //     vid: this.vid
            // })
        },
        makeEnd() { // 游戏结束, 发送时间戳和成绩
            // lib.HTTP(`${conf.server}${api.gameOver}?timer=${new Date().getTime()}`, {
            //     aid: this.$store.state.aid,
            //     userToken: this.$store.state.userToken,
            //     vid: this.vid,
            //     scores: this.num
            // }).then((data) => {
            //     if(data.code != 200) {
            //         Toast(data.message);
            //     }
            // })
        },
        /**
         * 用于canvas渲染圆形图片
         * @param {object} img 传入要渲染的图片
         * @param {Number} x x轴坐标
         * @param {Number} y y轴坐标
         * @param {Number} r 圆半径
         */
        drawArcImg(img, x, y, r) {
            this.ctx.save();
            this.ctx.translate(x, y);
            this.ctx.beginPath();
            this.ctx.arc(0, 0, r, 0, 2 * Math.PI);
            this.ctx.clip();
            this.ctx.fill();
            this.ctx.drawImage(img, -r, -r, 2*r, 2*r);
            this.ctx.closePath();
            this.ctx.restore();
        },
        /** 
         * 计算图片的宽
         * @param {any} img 图片对象
         * @param {Number} h 高
         */
        calcW(img, h) {
            return h/img.height * img.width;
        },
        /** 
         * 计算图片的高
         * @param {any} img 图片对象
         * @param {Number} w 宽
         */
        calcH(img, w) {
            return w/img.width * img.height;
        },
        /** 
         * 随机整数
         * @param {Number} min 随机到的最小数
         * @param {Number} max 随机到的最大数 
         */
        randNum(min, max) {
            if(min > max){
                var t = min;
                min = max;
                max = t;
            }
            return parseInt(Math.random()*(max - min + 1) + min);
        },
        /** 
         * 按照中心画图
         * @param {Object} img 图片对象
         * @param {Number} x x轴
         * @param {Number} y y轴
         * @param {Number} w 宽
         * @param {Number} h 高
         */
        drawPic(img, x, y, w, h) {
            this.ctx.save();
            this.ctx.translate(x, y);
            this.ctx.drawImage(img, -w/2, -h/2, w, h);
            this.ctx.restore();
        },
    }
}
</script>
<style>
.game{
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background: #fff;
    z-index: 0;
}
.incident{
    position: fixed;
    z-index: 99999;
    width: 100%;
    height: 100%;
}
#gameCanvas{
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
}
</style>
