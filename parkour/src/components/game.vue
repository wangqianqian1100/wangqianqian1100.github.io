<template>
    <div class="game">
        <game-box ref="game" :touchstart="touchstart" :touchmove="touchmove" :touchend="touchend"></game-box>

        <!-- 音乐 -->
        <!-- <music :musicurl="musicurl" ref="musicBoxer"></music> -->
        <!-- 音乐 -->

        <!-- 遮罩 -->
        <!-- <load-box v-if="loaderBox"></load-box> -->
        <!-- 遮罩 -->
    </div>
</template>
<script>
import gameBox from '@/public/components/gameBox'
import * as lib from '@/config/utils'
import * as conf from '@/config/index'
import * as api from '@/config/port/App/sketch'
import loadBox from '../../../public/components/loadBox'
import music from '../../../public/components/musicBox'
import { Toast } from 'mint-ui';
const rate = window.devicePixelRatio
export default {
    components: {
        gameBox, loadBox, music
    },
    data () {
        return {
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
            time: 60,   // 倒计时
            bgspeed:2,//背景图速度
            bgImgbgspeed:0,//大背景初始位置
            figurebgspeed:30,//人物初始化位置
            gmeterbgspeed:null,//加速小球初始化位置
            obstaclebgspeed:null,//减速度障碍物初始化位置
        }
    },
    computed: { // 从gameBox拿到的数据和方法
        canvas: { set() {}, get() {return this.$refs.game.canvas} },
        ctx: { set() {}, get() {return this.$refs.game.ctx} },
        drawArcImg: { set() {}, get() {return this.$refs.game.drawArcImg} },//用于canvas渲染圆形图片
        drawPic: { set() {}, get() {return this.$refs.game.drawPic} },//按照中心画图
        calcW: { set() {}, get() {return this.$refs.game.calcW} },//计算图片的宽
        calcH: { set() {}, get() {return this.$refs.game.calcH} },//计算图片的高
        randNum: { set() {}, get() {return this.$refs.game.randNum} }//随机整数
    },
    created() {
        // 禁止微信分享
        lib.wxConfig(['hideMenuItems']).then(() => {
            lib.stopShare()
        })
    },
    mounted() {
        // this.getData();
        this.init()
    },
    methods: {
        async getData() { // 获取数据
            const res = await lib.HTTP(`${conf.server}${api.gameBaseInfo}`, {
                aid: this.$store.state.aid,
                userToken: this.$store.state.userToken,
                vid: this.vid
            })
            if(res.code == 200) {
                this.imgs.bgImg = res.data.other_info.game_bgImg // 背景
                this.imgs.figure_one = res.data.user.figure_one // 头像
                this.imgs.rule = res.data.other_info.game_rule_simg // 规则弹框
                this.imgs.airBalloon = res.data.other_info.game_boximg // 未炸裂的
                this.imgs.airBalloonOver = res.data.other_info.game_boximg1 // 已炸裂的
                this.imgs.endBg = res.data.other_info.game_rule_eimg // 游戏结束弹窗
                this.fontColor = res.data.other_info.game_boxcolor
                this.init();
            } else {
                Toast(res.message)
            }
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
                        this.ctx.strokeStyle = this.fontColor; // '#FFB6C4'
                        this.ctx.lineWidth = 10;
                        this.ctx.font =  window.innerWidth*rate*0.06 + 'px Arial';
                        
                        timer =setInterval(this.drawBase(),1);
                    }
                }
            }
          
        },
        drawBase() { // 绘制动画
            let w = window.innerWidth*rate
            let h = window.innerHeight*rate
            this.ctx.clearRect(0, 0, w, h)
            this.ctx.save()
            
            
            // 大背景
            this.bgImgbgspeed -= this.bgspeed
            this.ctx.drawImage(this.imgs.bgImg, this.bgImgbgspeed, 0, this.calcW(this.imgs.bgImg,h), h)





            // 时间
            this.ctx.fillText(this.time.toFixed(2), w/2, w*0.192)
            // 分数
            this.ctx.fillText(this.num, w*0.85, w*0.185)
            this.ctx.restore()

            this.play()
        },
        play() { // 游戏中
            // 游戏开始, 发送时间戳
            this.makeStart();
            // 游戏
            let nowTime = new Date().getTime()
            let endTime = nowTime + this.time*1000;
            let that = this;
            let timerOut = requestAnimationFrame(function fn(){
                that.time = (endTime - new Date().getTime())/1000
                if(that.time >= 0) {
                    that.drawBase()
                    cancelAnimationFrame(timerOut)
                    timerOut = requestAnimationFrame(fn)
                } else if(that.time <= 0) { // 时间已到, 游戏结束
                    that.time = 0;
                    that.gameOver()
                    cancelAnimationFrame(timerOut)
                }
            })
        },
        gameOver() { // 游戏结束
            
        },
        back() { // 返回
            this.$router.replace(sessionStorage.sketchBackRouter);
        },
        makeStart() { // 游戏开始, 发送时间戳
            lib.HTTP(`${conf.server}${api.gameStart}?timer=${new Date().getTime()}`, {
                aid: this.$store.state.aid,
                userToken: this.$store.state.userToken,
                vid: this.vid
            })
        },
        makeEnd() { // 游戏结束, 发送时间戳和成绩
            lib.HTTP(`${conf.server}${api.gameOver}?timer=${new Date().getTime()}`, {
                aid: this.$store.state.aid,
                userToken: this.$store.state.userToken,
                vid: this.vid,
                scores: this.num
            }).then((data) => {
                if(data.code != 200) {
                    Toast(data.message);
                }
            })
        },
        touchstart(){

        },
        touchmove(){

        },
        touchend(){

        }
    }
}
</script>
<style lang="less" scoped>
.game{
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
}
</style>
