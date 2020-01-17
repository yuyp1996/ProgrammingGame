<template>
    <div class="h100vh" ref="mainBox">
        <div class="dog-pic">
            <img @load="dogOnLoad" ref="dogPic0" src="../../static/dog/0.png" alt="">
            <img @load="dogOnLoad" ref="dogPic1" src="../../static/dog/1.png" alt="">
            <img @load="dogOnLoad" ref="dogPic2" src="../../static/dog/2.png" alt="">
            <img @load="dogOnLoad" ref="dogPic3" src="../../static/dog/3.png" alt="">
            <img @load="dogOnLoad" ref="dogPic4" src="../../static/dog/4.png" alt="">
            <img @load="dogOnLoad" ref="dogPic5" src="../../static/dog/5.png" alt="">
            <img @load="dogOnLoad" ref="dogPic6" src="../../static/dog/6.png" alt="">
            <img @load="dogOnLoad" ref="dogPic7" src="../../static/dog/7.png" alt="">
            <img @load="dogOnLoad" ref="dogPic8" src="../../static/dog/8.png" alt="">
            <img @load="dogOnLoad" ref="kennel" src="../../static/kennel.png" alt="">
        </div>
        <div class="game-content">
            <transition name="fade">
                <div class="mask-bg" v-show="successPlay">
                    <div class="success-box">
                        <div class="success-text">小狗到家啦</div>
                        <div class="success-button">
                            <button class="button" @click="restart">重新开始</button>
                            <button class="button">下一关</button>
                        </div>
                    </div>
                </div>
            </transition>
            <div class="step-number">剩余{{stepNumber}}步</div>
            <!--结果展示区-->
            <canvas class="canvas" id="animationArea" ref="animationArea"></canvas>
            <!--编程控制区-->
            <div class="programming-box">
                <div class="programming-box-program" ref="boxProgram">
                    <p v-for="(item,index) in dogMovedOrders" :key="index">{{item}}</p>
                </div>
                <div class="programming-box-option">
                    <div class="programming-option" v-for="(item,index) in dogMoveOrders" :key="index" @click="moveOrder(item)">{{item.name}}</div>
                    <div class="programming-option" @click="restart">重置</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'HelloWorld',
        data() {
            return {
                isMove: false,
                moveElement: null, // 正在移动的选项元素
                nowPositionX: 0,
                nowPositionY: 0, // 当前移动选项的鼠标坐标
                optionWidth: 0,
                optionHeight: 0, // 当前移动选项的宽高
                dogPicLoadingNum: 0, // 存放加载完成的狗的图片数量
                dogAnimation: null, // 狗的canvas动画函数
                drawObstacle: null, // 障碍物canvas绘图函数
                drawEnd: null, // 终点canvas绘图函数
                dogMoveOrders: [
                    {
                        name: '前进100px',
                        description: '小狗前进了100像素距离',
                        content: 100
                    },{
                        name: '向左转',
                        description: '小狗向左转头',
                        content: 'turnLeft'
                    },{
                        name: '向右转',
                        description: '小狗向右转头',
                        content: 'turnRight'
                    },{
                        name: '向下转',
                        description: '小狗向下转头',
                        content: 'turnDown'
                    },{
                        name: '向上转',
                        description: '小狗向上转头',
                        content: 'turnUp'
                    },
                ], // 狗的指令列表
                dogMovedOrders: [], // 狗已完成的指令
                obstacleArray: [
                    {
                        position: [300, 0], // 左上角点的位置
                        obWidth: 100,
                        obHeight: 100, // 障碍物宽高
                        obColor: 'black', //障碍物颜色
                    },{
                        position: [200, 100],
                        obWidth: 200,
                        obHeight: 100,
                        obColor: 'black',
                    },
                ], // 障碍物数组
                originalDog: {
                    // 1步10px 
                    stepDistance: 10,
                    // 狗的速度
                    speed: 0.3,
                    // X轴上一次的转向，用来控制上下方向时的小狗的腿部位置
                    lastDirectX: 1,
                    // 控制X轴转向,正为右，负为左，0为Y轴运动
                    directX: 1,
                    // 控制Y轴转向,正为下，负为上，0为X轴运动
                    directY: 0,
                    //用来控制小狗是否停下
                    stopWalking: true,
                    // 小狗现在的位置
                    nowDogPosX: 0,
                    nowDogPosY: 0,
                    // 目的地位置
                    destinationX: 0,
                    destinationY: 0,
                }, // 小狗的初始状态
                endPosition: {
                    position: [400, 0], // 左上角点的位置
                }, // 终点位置
                successPlay: false, // 是否完成游戏
                stepNumber: 15, // 剩余步数
            }
        },
        watch: {
            dogPicLoadingNum(newVal,oldVal) { // 图片加载完成启动狗动画函数
                if (newVal === 9) {
                    this.dogAnimation.drawDogMove();
                }
            }
        },
        mounted() {
            this.canvasContent();
        },
        methods: {
            dogOnLoad() {
                this.dogPicLoadingNum++;
            },
            moveOrder(order) {
                if (this.successPlay || this.stepNumber === 0) {
                    return;
                }
                this.stepNumber--;
                if (!this.dogAnimation.dog.stopWalking) {
                    console.log('小狗正在前进');
                    return;
                }
                if (typeof order.content === 'number') {
                    this.dogAnimation.walking(order.content);
                } else {
                    if(this.dogAnimation[order.content]()) {
                        return;
                    }
                }
                if (!this.dogAnimation.dog.stopWalking
                    || typeof order.content === 'string') {
                    this.dogMovedOrders.push(order.description);
                    this.$nextTick(() => {
                        this.$refs.boxProgram.scrollTop = this.$refs.boxProgram.scrollHeight;
                    });
                }

            },

            //重置函数
            restart (){
                this.dogMovedOrders = [];
                this.dogAnimation.dog = JSON.parse(JSON.stringify(this.originalDog));
                this.successPlay = false;
                this.stepNumber = 15;
                this.dogAnimation.drawDogMove();
            },

            mouseDownEvent(e) { // 元素鼠标按下事件
                this.optionWidth = e.target.clientWidth;
                this.optionHeight = e.target.clientHeight;
                this.nowPositionX = e.pageX;
                this.nowPositionY = e.pageY;
                this.isMove = true;
                this.moveElement = e.target;
            },
            mouseMoveEvent(e) { // 元素鼠标移动事件
                if (!this.isMove || this.moveElement === null) {
                    return
                }
                const distanceX = e.pageX - this.nowPositionX;
                const distanceY = e.pageY - this.nowPositionY; // 鼠标移动的距离
                this.nowPositionX = e.pageX;
                this.nowPositionY = e.pageY; // 保存当前鼠标坐标
                this.moveElement.style.left = this.moveElement.offsetLeft + distanceX + 'px';
                this.moveElement.style.top = this.moveElement.offsetTop + distanceY + 'px';
                this.moveElement.style.zIndex = 99;
            },
            mouseUpEvent(e) { // 元素鼠标放开事件
                if (this.moveElement) {
                    this.moveElement.style.zIndex = 1;
                }
                this.isMove = false;
                this.moveElement = null;
            },

            // canvas画图代码

            canvasContent() {
                const _this = this;
                let windowScaleX = 1;
                let windowScaleY = 1;
                const canvas = {
                    init() {
                        this.ele = _this.$refs.animationArea;
                        this.resize();
                        window.addEventListener('resize', () => this.resize(), false);
                        this.ctx = this.ele.getContext('2d');
                        return this.ele;
                    },
                    onResize(callback) {
                        this.resizeCallback = callback;
                    },
                    resize() {
                        _this.$refs.mainBox ? this.ele.width = _this.$refs.mainBox.offsetWidth * 1.4
                            : _this.$nextTick(() => {
                            this.ele.width = _this.$refs.mainBox.offsetWidth * 1.4;
                        });
                        this.ele.height = window.innerHeight * 2;
                        windowScaleX = Math.floor(this.ele.width / 2 / 1332  * 10) / 10;
                        windowScaleY = Math.floor(this.ele.height / 2 / 937  * 10) / 10; // 屏幕缩小比例
                        this.ele.style.width = this.ele.width * 0.5 + 'px';
                        this.ele.style.height = this.ele.height * 0.5 + 'px';
                        this.ctx = this.ele.getContext('2d');
                        this.ctx.scale(2, 2);
                        this.resizeCallback && this.resizeCallback();
                    },
                };

                const ctx = canvas.init();

                //console.log(windowScale, ctx.width)

                class DrawObstacle {
                    constructor(canvas) {
                        this.canvas = canvas;
                        this.ctx = canvas.getContext("2d");
                    }

                    startDraw(obstacle) { // 画出相应的障碍物
                        this.ctx.save();
                        this.ctx.translate(obstacle.position[0] * windowScaleX, obstacle.position[1] * windowScaleY);
                        this.ctx.fillStyle = obstacle.obColor;
                        this.ctx.fillRect(0, 0, obstacle.obWidth * windowScaleX, obstacle.obHeight * windowScaleY);
                        this.ctx.restore();
                    }
                }

                // 终点画图函数
                class DrawEnd {
                    constructor(canvas) {
                        this.canvas = canvas;
                        this.ctx = canvas.getContext("2d");
                    }

                    startDraw() {
                        this.ctx.save();
                        let img = _this.$refs.kennel;
                        this.ctx.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight,
                            // dx = 20, dy, dwidth, dheight
                            _this.endPosition.position[0] * windowScaleX, _this.endPosition.position[1] * windowScaleY, 100 * windowScaleX, 100 * windowScaleY);
                        this.ctx.restore();
                    }
                }

                class DogAnimation {
                    constructor(canvas) {
                        // 图片目录
                        this.IMG_COUNT = 8;
                        this.canvas = canvas;
                        this.ctx = canvas.getContext("2d");
                        // 记录上一帧的时间
                        this.lastWalkingTime = Date.now();
                        // 记录当前画的图片索引
                        this.keyFrameIndex = -1;
                        // 控制狗运动的要素
                        this.dog = JSON.parse(JSON.stringify(_this.originalDog));
                        this.dogControlFun = { // 狗行动额外执行的函数
                            'start': null, // 行动开始前执行的函数
                            'halfway': null, // 行动中函数
                            'end': null // 行动结束后函数
                        };
                    }

                    turnLeft() {
                        if (this.dog.directX === -1) {
                            return true;
                        }
                        this.dog.directX = this.dog.lastDirectX = -1;
                        this.dog.directY = 0;
                        this.drawDogControl();
                        return false;
                    }

                    turnRight() {
                        if (this.dog.directX === 1) {
                            return true;
                        }
                        this.dog.directX = this.dog.lastDirectX = 1;
                        this.dog.directY = 0;
                        this.drawDogControl();
                        return false;
                    }

                    turnUp() {
                        if (this.dog.directY === -1) {
                            return true;
                        }
                        this.dog.directX = 0;
                        this.dog.directY = -1;
                        this.drawDogControl();
                        return false;
                    }

                    turnDown() {
                        if (this.dog.directY === 1) {
                            return true;
                        }
                        this.dog.directX = 0;
                        this.dog.directY = 1;
                        this.drawDogControl();
                        return false;
                    }

                    walking(distance) { // 开始行动
                        this.dog.stopWalking = false;
                        if (this.dog.directY === 0) {
                            this.dog.destinationX = this.dog.destinationX + distance * this.dog.directX;
                            if (this.dog.destinationX < 0
                                || this.dog.destinationX * windowScaleX > this.canvas.width / 2 - 46.5) { // 超过屏幕不再行走
                                this.dog.destinationX = this.dog.destinationX - distance * this.dog.directX;
                                this.dog.stopWalking = true;
                            }
                        } else {
                            this.dog.destinationY = this.dog.destinationY + distance * this.dog.directY;
                            if (this.dog.destinationY < 0
                                || this.dog.destinationY * windowScaleY > this.canvas.height / 2 - 45.5) { // 超过屏幕不再行走
                                this.dog.destinationY = this.dog.destinationY - distance * this.dog.directY;
                                this.dog.stopWalking = true;
                            }
                        }
                        _this.obstacleArray.forEach(obstacle => { // 遇到障碍物停止行走
                            if (this.dog.destinationX + 46.5 > obstacle.position[0]
                            && this.dog.destinationX + 46.5 < (obstacle.position[0] + obstacle.obWidth)
                            && this.dog.destinationY + 45.5 > obstacle.position[1]
                            && this.dog.destinationY + 45.5 < (obstacle.position[1] + obstacle.obHeight)) {
                                this.dog.directY === 0 ? this.dog.destinationX = this.dog.destinationX - distance * this.dog.directX : this.dog.destinationY = this.dog.destinationY - distance * this.dog.directY;
                                this.dog.stopWalking = true;
                            }
                        });
                        //console.log(this.dog.destinationX,this.dog.destinationY);
                        this.drawDogControl();
                    }

                    drawDogMove() {

                        // 先清掉上一次画的内容
                        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
                        if (_this.drawObstacle) {
                            for (let i = 0; i < _this.obstacleArray.length; i++) {
                                _this.drawObstacle.startDraw(_this.obstacleArray[i]);
                            }
                        }
                        _this.drawEnd && _this.drawEnd.startDraw();
                        this.ctx.save();
                        this.ctx.translate((this.dog.nowDogPosX + 46.5) * windowScaleX, (this.dog.nowDogPosY + 45.5) * windowScaleY);
                        // 控制狗的左右上下翻转
                        this.ctx.scale(this.dog.directX || -this.dog.lastDirectX, this.dog.directY || 1);
                        // 控制狗的上下旋转
                        if (this.dog.directY === 1) { // 朝下
                            this.ctx.rotate(this.dog.directY * 90 * Math.PI / 180);
                        } else if (this.dog.directY === -1) { // 朝上
                            this.ctx.rotate(-this.dog.directY * 90 * Math.PI / 180);
                        }
                        // 获取下一张图片的索引
                        let keyFrameIndex = ++this.keyFrameIndex % this.IMG_COUNT;
                        // 控制狗的走动与否
                        if (this.dog.stopWalking) {
                            keyFrameIndex = -1
                        }
                        let img = _this.$refs[`dogPic${keyFrameIndex + 1}`];
                        // img, sx, sy, swidth, sheight
                        this.ctx.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight,
                            // dx = 20, dy, dwidth, dheight
                            -46.5 * windowScaleX, -40.5 * windowScaleY, 93 * windowScaleX, 81 * windowScaleY);
                        this.ctx.restore();
                    }

                    drawDogControl() {
                        // 绘制狗的图片，每过100ms就画一张
                        let now = Date.now();
                        if (now - this.lastWalkingTime > 50) {
                            if (!this.dog.stopWalking) {
                                this.dog.nowDogPosX += this.dog.stepDistance * this.dog.directX;
                                this.dog.nowDogPosY += this.dog.stepDistance * this.dog.directY;
                            }
                            this.drawDogMove();
                            this.lastWalkingTime = now;
                            // 判断狗是否到达指定位置
                            //console.log(this.dog.nowDogPosX, this.dog.destinationX, this.dog.nowDogPosY, this.dog.destinationY)
                            if (this.dog.nowDogPosX === this.dog.destinationX
                             && this.dog.nowDogPosY === this.dog.destinationY) { // 行动结束
                                this.dog.stopWalking = true;
                                this.drawDogMove();
                            }
                        }
                        if (this.dog.nowDogPosX + 46.5 > _this.endPosition.position[0]
                            && this.dog.nowDogPosX + 46.5 < (_this.endPosition.position[0] + 100)
                            && this.dog.nowDogPosY + 45.5 > _this.endPosition.position[1]
                            && this.dog.nowDogPosY + 45.5 < (_this.endPosition.position[1] + 100)) { // 到达终点

                            _this.successPlay = true;
                        }
                        // 继续给下一帧注册一个函数
                        // let distance = (now - this.lastWalkingTime) * this.dog.speed;
                        if (!this.dog.stopWalking) {
                            window.requestAnimationFrame(() => {
                                this.drawDogControl();
                            });
                        }
                    }
                }

                this.drawObstacle = new DrawObstacle(ctx);
                this.drawEnd = new DrawEnd(ctx);
                this.dogAnimation = new DogAnimation(ctx);

                canvas.onResize(() => {
                    this.dogAnimation.drawDogMove();
                });
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
    }
    .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
        opacity: 0;
    }

    .programming-box {
        position: relative;
        box-sizing: border-box;
        width: calc(30% - 2px);
        height: 100%;
        border-left: 2px solid black;
    }

    .game-content {
        position: relative;
        display: flex;
        height: 100%;
    }

    .step-number {
        position: absolute;
        right: 32%;
        font-size: 15px;
        line-height: 30px;
    }

    .mask-bg {
        position: fixed;
        z-index: 9;
        width: 100vw;
        height: 100vh;
        background: rgba(0,0,0,.5);
    }

    .success-box {
        position: absolute;
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        margin: auto;
        width: 200px;
        height: 150px;
        background: #3cbcff;
        border-radius: 10px;
        box-shadow: inset 0 0 15px rgba(0,0,0,.5);
    }

    .success-box .success-text {
        margin-top: 50px;
        font-size: 25px;
        line-height: 30px;
        color: white;
    }

    .success-box .success-button {
        display: flex;
        justify-content: center;
        margin-top: 10px;
    }

    .success-box .success-button .button {
        margin: 0 10px;
        padding: 5px;
        border: 0 none;
        background-color: #28ffca;
        border-radius: 10px;
        color: white;
        font-size: 10px;
        line-height: 30px;
        box-shadow: inset 0 0 15px rgba(0,0,0,.5);
    }

    .programming-box-program, .programming-box-option {
        width: 100%;
        height: 50%;
        padding-top: 1px;
        overflow-y: auto;
    }

    .programming-box-program {
        background-color: antiquewhite;
    }

    .programming-box-option {
        background-color: bisque;
    }

    .programming-option {
        margin: 10px auto;
        width: 80%;
        padding: 5px;
        background-color: aqua;
        border-radius: 10px;
        color: black;
        font-size: 25px;
        line-height: 30px;
        user-select: none;
        -webkit-touch-callout: none;
        cursor: pointer;
    }

    .dog-pic {
        display: none;
    }

    .canvas {
        width: 70%;
        height: 100%;
    }
</style>
