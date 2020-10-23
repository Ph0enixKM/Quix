<template>
    <div>
        <canvas ref="bg"></canvas>
    </div>
</template>

<script>
    const SIZE_MIN = 20
    const SIZE_MAX = 100
    const COUNT = 20
    const SPEED = 0.5
    const BLUR_MIN = 0
    const BLUR_MAX = 7

    class Ball {
        constructor(context, iteration) {
            this.color = `hsl(${(iteration/COUNT) * 72 + 273}, 100%, 50%)`
            this.size = (iteration/COUNT) * (SIZE_MAX - SIZE_MIN) + SIZE_MIN
            this.blur = ((COUNT-iteration)/COUNT) * (BLUR_MAX - BLUR_MIN) + BLUR_MIN
            this.pos = [
                Math.random() * (window.innerWidth - this.size) + this.size, 
                Math.random() * (window.innerHeight - this.size) + this.size
            ]
            this.vel = [
                (Math.random() - 0.5) * SPEED + 0.1, 
                (Math.random() - 0.5) * SPEED + 0.1
            ]
            this.c = context
        }

        draw() {
            this.c.beginPath()
            this.c.fillStyle = this.color
            this.c.globalAlpha = (this.size / SIZE_MAX) * 0.9
            // this.c.filter = `blur(${this.blur}px)`
            this.c.arc(this.pos[0], this.pos[1], this.size, 0, 2*Math.PI)

            if (this.pos[0] + this.size > window.innerWidth || this.pos[0] - this.size < 0) {
                this.vel[0] = -this.vel[0]
            }
            if (this.pos[1] + this.size > window.innerHeight || this.pos[1] - this.size < 0) {
                this.vel[1] = -this.vel[1]
            }
            this.pos[0] += this.vel[0]
            this.pos[1] += this.vel[1]
            this.c.fill()
        }


    }

    export default {
        mounted() {
            let canvas = this.$refs.bg
            canvas.width = window.innerWidth
            canvas.height = window.innerHeight
            let c = canvas.getContext('2d')
            
            let balls = []
            for (let i = 0; i < COUNT; i++) {
                balls.push(new Ball(c, i))
            }

            requestAnimationFrame(function update () {
                c.clearRect(0, 0, window.innerWidth, window.innerHeight)
                for (const ball of balls) {
                    ball.draw()
                }
                requestAnimationFrame(update)
            })
        }
    }
</script>

<style scoped>
    canvas {
        position: fixed;
        top: 0;
        left: 0;
        z-index: -3;
        width: 100vw;
        height: 100vh;
    }
</style>