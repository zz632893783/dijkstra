<template>
    <div class="home">
        <canvas class="home" ref="canvas" v-bind:width="width * blockSize" v-bind:height="height * blockSize" v-on:mousedown="mousedownFunc($event)" v-on:contextmenu="contextmenu($event)"></canvas>
        <div class="controls">
            <el-button class="btn" type="primary" v-on:click="randomCreateObstacle">生成障碍物</el-button>
        </div>
    </div>
</template>

<script>
// @ is an alias to /src

export default {
    name: 'Home',
    components: {},
    data: function () {
        return {
            // 横向有多少网个网格
            width: 30,
            // 纵向有多少网个网格
            height: 20,
            // 每个网格的尺寸
            blockSize: 20,
            // canvas 上下文对象
            ctx: null,
            // 可移动点的列表
            pointList: [],
            // 障碍物列表
            obstacleList: [],
            // 起始点位
            startDot: {
                x: 0.5,
                y: 0.5
            },
            // 结束点位
            endDot: {
                x: 0.5,
                y: 0.5
            },
            // 当前选中点
            currentPoint: null,
            times: 0,
            pathPointList: [],
            progress: 0
        }
    },
    methods: {
        init: function () {
            this.ctx = this.$refs.canvas.getContext('2d')
            // 创建点位
            this.createPoint()
            // 计算相邻点位
            this.computeNearPoint()
            // 计算轨迹
            // this.computeTrajectory()
        },
        // 绘制函数
        draw: function () {
            this.ctx.clearRect(0, 0, this.width * this.blockSize, this.height * this.blockSize)
            this.drawGrid()
            this.drawObstacle()
            this.drawTrajectory()
            this.drawDot()
        },
        // 绘制障碍物
        drawObstacle: function () {
            this.ctx.fillStyle = 'rgba(50, 50, 50, 1)'
            this.obstacleList.forEach(item => {
                this.ctx.beginPath()
                this.ctx.fillRect(item.x * this.blockSize, item.y * this.blockSize, this.blockSize, this.blockSize)
                this.ctx.closePath()
            })
        },
        // 绘制网格
        drawGrid: function () {
            this.ctx.lineWidth = 1
            this.ctx.strokeStyle = 'rgba(0, 0, 0, 1)'
            for (let i = 0; i <= this.width; i++) {
                this.ctx.beginPath()
                this.ctx.moveTo(i * this.blockSize, 0)
                this.ctx.lineTo(i * this.blockSize, this.height * this.blockSize)
                this.ctx.stroke()
                this.ctx.closePath()
            }
            for (let i = 0; i <= this.height; i++) {
                this.ctx.beginPath()
                this.ctx.moveTo(0, i * this.blockSize)
                this.ctx.lineTo(this.width * this.blockSize, i * this.blockSize)
                this.ctx.stroke()
                this.ctx.closePath()
            }
        },
        // 创建点位
        createPoint: function () {
            this.pointList = []
            for (let i = 0.5; i < this.width + 0.5; i++) {
                for (let j = 0.5; j < this.height + 0.5; j++) {
                    if (this.obstacleList.find(item => item.x + 0.5 === i && item.y + 0.5 === j)) {
                        continue
                    }
                    // if (this.obstacleList.find(item => item.x + 1 === i && item.y === j)) {
                    //     continue
                    // }
                    // if (this.obstacleList.find(item => item.x === i && item.y + 1 === j)) {
                    //     continue
                    // }
                    // if (this.obstacleList.find(item => item.x + 1 === i && item.y + 1 === j)) {
                    //     continue
                    // }
                    this.pointList.push({
                        x: i,
                        y: j,
                        fromPoint: null,
                        enable: false,
                        distance: Infinity
                    })
                }
            }
        },
        // 计算每个点位相邻位置
        computeNearPoint: function () {
            this.pointList.forEach(currentPoint => {
                // 每个点相邻点集合
                const nearPoints = []
                // 上
                const topPoint = this.pointList.find(point => point.x === currentPoint.x && point.y === currentPoint.y - 1)
                topPoint && nearPoints.push(topPoint)
                // 下
                const bottomPoint = this.pointList.find(point => point.x === currentPoint.x && point.y === currentPoint.y + 1)
                bottomPoint && nearPoints.push(bottomPoint)
                // 左
                const leftPoint = this.pointList.find(point => point.y === currentPoint.y && point.x === currentPoint.x - 1)
                leftPoint && nearPoints.push(leftPoint)
                // 右
                const rightPoint = this.pointList.find(point => point.y === currentPoint.y && point.x === currentPoint.x + 1)
                rightPoint && nearPoints.push(rightPoint)
                // // 上左
                // const topLeftPoint = this.pointList.find(point => point.x === currentPoint.x - 1 && point.y === currentPoint.y - 1)
                // topLeftPoint && nearPoints.push(topLeftPoint)
                // // 上右
                // const topRightPoint = this.pointList.find(point => point.x === currentPoint.x + 1 && point.y === currentPoint.y - 1)
                // topRightPoint && nearPoints.push(topRightPoint)
                // // 下左
                // const bottomLeftPoint = this.pointList.find(point => point.x === currentPoint.x - 1 && point.y === currentPoint.y + 1)
                // bottomLeftPoint && nearPoints.push(bottomLeftPoint)
                // // 下右
                // const bottomRightPoint = this.pointList.find(point => point.x === currentPoint.x + 1 && point.y === currentPoint.y + 1)
                // bottomRightPoint && nearPoints.push(bottomRightPoint)
                currentPoint.nearPoints = nearPoints
            })
        },
        // 绘制起始点位
        drawDot: function () {
            if (this.pathPointList.length >= 2) {
                // console.log(1)
                const x = (this.pathPointList[0].x + (this.pathPointList[1].x - this.pathPointList[0].x) * this.progress) * this.blockSize
                const y = (this.pathPointList[0].y + (this.pathPointList[1].y - this.pathPointList[0].y) * this.progress) * this.blockSize
                this.ctx.beginPath()
                this.ctx.arc(x, y, this.blockSize / 4, 0, 2 * Math.PI)
                this.ctx.closePath()
                this.ctx.fillStyle = 'green'
                this.ctx.fill()
                this.progress = this.progress + 0.125
                if (this.progress >= 1) {
                    this.progress = 0
                    this.pathPointList.shift()
                }
            } else {
                this.ctx.beginPath()
                this.ctx.arc(this.endDot.x * this.blockSize, this.endDot.y * this.blockSize, this.blockSize / 4, 0, 2 * Math.PI)
                this.ctx.closePath()
                this.ctx.fillStyle = 'green'
                this.ctx.fill()
            }
        },
        // 计算两个点位之间的距离
        computePointDistance: function (prevPoint, nextPoint) {
            return Math.pow(Math.pow(prevPoint.x * this.blockSize - nextPoint.x * this.blockSize, 2) + Math.pow(prevPoint.y * this.blockSize - nextPoint.y * this.blockSize, 2), 1 / 2)
        },
        // 计算轨迹
        computeTrajectory: function () {
            if (!this.pointList.filter(item => !item.enable).length) {
                return false
            }
            // 如果当前选中点为空的话，就默认是轨迹刚开始的阶段
            if (!this.currentPoint) {
                this.currentPoint = this.pointList.find(item => item.x === this.startDot.x && item.y === this.startDot.y)
                this.currentPoint.distance = 0
                this.fromPoint = null
            }
            this.currentPoint.enable = true
            // 从相邻点中筛选出，还没有经过的点
            const nearPoints = this.currentPoint.nearPoints.filter(item => !item.enable)
            nearPoints.forEach(point => {
                let distance = this.computePointDistance(this.currentPoint, point)
                distance = this.currentPoint.distance + distance
                if (distance < point.distance) {
                    point.distance = distance
                    point.fromPoint = this.currentPoint
                }
            })
            const points = this.pointList.filter(item => !item.enable)
            if (points.length) {
                const min = Math.min(...points.map(item => item.distance))
                this.currentPoint = points.find(item => item.distance === min)
                this.computeTrajectory()
            }
        },
        computePath: function () {
            if (this.currentPoint) {
                this.pathPointList.push(this.currentPoint)
            }
            if (this.currentPoint.fromPoint) {
                this.currentPoint = this.currentPoint.fromPoint
                this.ctx.lineTo(this.currentPoint.x * this.blockSize, this.currentPoint.y * this.blockSize)
                this.computePath()
            }
        },
        // 绘制轨迹
        drawTrajectory: function () {
            if (this.pathPointList.length >= 2) {
                this.ctx.beginPath()
                const startX = (this.pathPointList[0].x + (this.pathPointList[1].x - this.pathPointList[0].x) * this.progress) * this.blockSize
                const startY = (this.pathPointList[0].y + (this.pathPointList[1].y - this.pathPointList[0].y) * this.progress) * this.blockSize
                this.ctx.moveTo(startX, startY)
                // this.ctx.moveTo(this.pathPointList[1].x * this.blockSize, this.pathPointList[1].y * this.blockSize)
                for (let i = 1; i < this.pathPointList.length; i++) {
                    this.ctx.lineTo(this.pathPointList[i].x * this.blockSize, this.pathPointList[i].y * this.blockSize)
                }
                this.ctx.strokeStyle = 'red'
                this.ctx.lineWidth = 2
                this.ctx.stroke()
            }
        },
        // clickFunc: function (event) {
        //     const x = Math.floor(event.offsetX / this.blockSize)
        //     const y = Math.floor(event.offsetY / this.blockSize)
        //     if (!this.obstacleList.find(item => item.x === x && item.y === y)) {
        //         this.obstacleList.push({ x, y })
        //         // 创建点位
        //         this.createPoint()
        //         // 计算相邻点位
        //         this.computeNearPoint()
        //         // 计算轨迹
        //         this.computeTrajectory()
        //         this.draw()
        //     }
        // },
        mousedownFunc: function (event) {
            // 左键1 滚轮2 右键3
            if (event.which === 3) {
                const x = Math.floor(event.offsetX / this.blockSize)
                const y = Math.floor(event.offsetY / this.blockSize)
                if (this.obstacleList.find(item => item.x === x && item.y === y)) {
                    return false
                }
                this.currentPoint = null
                if (this.pathPointList.length) {
                    this.startDot.x = this.pathPointList[0].x
                    this.startDot.y = this.pathPointList[0].y
                }
                this.endDot.x = x + 0.5
                this.endDot.y = y + 0.5
                // 创建点位
                this.createPoint()
                // 计算相邻点位
                this.computeNearPoint()
                // 计算轨迹
                this.computeTrajectory()
                this.pathPointList = []
                this.currentPoint = this.pointList.find(item => item.x === this.endDot.x && item.y === this.endDot.y)
                this.computePath()
                this.pathPointList.reverse()
            }
        },
        contextmenu: function (event) {
            event.preventDefault()
        },
        randomCreateObstacle: function () {
            this.ctx.clearRect(0, 0, this.width * this.blockSize, this.height * this.blockSize)
            this.endDot.x = this.startDot.x
            this.endDot.y = this.startDot.y
            this.obstacleList = []
            for (let i = 0; i < 150; i++) {
                const x = Math.floor(Math.random() * this.width)
                const y = Math.floor(Math.random() * this.height)
                if (x + 0.5 === this.startDot.x && y + 0.5 === this.startDot.y) {
                    continue
                }
                if (x + 0.5 === this.endDot.x && y + 0.5 === this.endDot.y) {
                    continue
                }
                if (this.obstacleList.find(item => item.x === x && item.y === y)) {
                    continue
                }
                this.obstacleList.push({ x, y })
            }
            this.drawGrid()
            this.drawDot()
            this.drawObstacle()
        },
        setAnimation: function () {
            window.cancelAnimationFrame(window.animation)
            window.animation = window.requestAnimationFrame(this.setAnimation)
            this.draw()
        }
    },
    mounted: function () {
        this.init()
        // this.draw()
        this.setAnimation()
    }
}
</script>
<style lang="stylus" scoped>
.home {
    background-color: rgba(0, 0, 0, 0.1);
    font-size: 0;
    position: relative;
    display: inline-block;
    .controls {
    }
    // .btn {
    //     position: absolute;
    //     top: 0;
    //     right: 0;
    //     transform: translate(100%, 0);
    // }
}
</style>
