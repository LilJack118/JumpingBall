<template>
    <canvas id="deadzone-canvas" @click.prevent></canvas>
    <fade-in mode="out-in">
      <section v-if="!isPlaying">
        <game-title></game-title>
        <how-to-play></how-to-play>
        <score-log></score-log>
      </section>

      <points-counter v-else></points-counter>
    </fade-in>

    <canvas id="ball-canvas" v-bind:class="{'fade-in':!isPlaying}"></canvas>
    <div class="canvas-overlay" @click.prevent="jumpBall"></div>

</template>
  
<script>
    import PointsCounter from '../components/game/PointsCounter.vue';
    import GameTitle from '../components/game/GameTitle.vue';
    import HowToPlay from '../components/game/HowToPlay.vue';
    import ScoreLog from '../components/game/ScoreLog.vue';
    export default{
      components:{
        'points-counter':PointsCounter,
        'game-title':GameTitle,
        'how-to-play':HowToPlay,
        'score-log':ScoreLog,
      },
      data(){
        return{
          previousXVelocity: null,
          DOMHighResTimeStampCollection: [],
          requestId: null,
          fpsRate: 0,
          fpsCheckNum: 5,
          fpsIter:0,
        }
      },
      computed:{
        isPlaying(){
          return this.$store.getters['isPlaying'];
        }
      },
      methods:{
        pressSpace(e){
          if (e.code === 'Space'){
            this.jumpBall()
          }
        },
        jumpBall(){
          if(!this.isPlaying){
            // this.isPlaying = true;
            this.$store.dispatch('startPlaying');
            // this.runAnimation();
            this.runAnimation();
          }else{
            this.$store.dispatch('resetVelocityY');
            this.$store.dispatch('resetBallTailCounter');
          }
        },
        runAnimation(timestamp){
  
          // console.log(this.$store.state['gameModule']['velocityX'])          
          if(!this.$store.getters['isGameOver']){
              requestAnimationFrame(this.runAnimation);
              this.$store.dispatch('clearRectangle')
              this.$store.dispatch('calculateBallCoordinates')
              this.$store.dispatch('displayCircle');
  
              // display deadzones
              if (this.previousXVelocity && this.$store.getters['velocityX'] != this.previousXVelocity){
                this.$store.dispatch('newDeadZones');
                this.$store.dispatch('IncrementCounter');
              }
              this.previousXVelocity = this.$store.getters['velocityX']
            }else{
              // this.resetGame()
              this.$store.dispatch('resetBallTailCounter');
              this.$store.dispatch('ballCrashAnimation');
              // reset all game 2 s after ball crash animation
              setTimeout(() => {
                this.$store.dispatch('resetCanvas');
                this.$store.dispatch('resetDeadZoneCtx');
                this.$store.dispatch('displayCircle');
                this.$store.dispatch('resetCounter');
                // reset previous velocity
                this.previousXVelocity = this.$store.getters['velocityX']
              }, 3500)
          }
        },
        setFpsAndGame(){
          this.requestId = null;
          if (!window.requestAnimationFrame) {
              window.requestAnimationFrame = window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame;
          }
          window.requestAnimationFrame(this.triggerAnimation);
        },
        triggerAnimation(DOMHighResTimeStamp){
          this.DOMHighResTimeStampCollection.unshift(DOMHighResTimeStamp);
          
          if (this.DOMHighResTimeStampCollection.length > 10) {
              let t0 = this.DOMHighResTimeStampCollection.pop();
              let fps = Math.round(1000 * 10 / (DOMHighResTimeStamp - t0));
              if (fps > this.fpsRate){
                this.fpsRate = fps
              }
              this.fpsIter ++;
          }
          
          if (this.fpsIter < this.fpsCheckNum){
            this.requestId = window.requestAnimationFrame(this.triggerAnimation);
          }else{
            window.cancelAnimationFrame(this.requestId);
            // when fps rate are determined set rest of game properties
            this.setGame();
          }
        },
        setGame(){
          this.$store.dispatch('setCanvasSize');
          // this.setInitialProperties();
          this.$store.dispatch('setInitialProperties', {fps:this.fpsRate});
          const canvasSize = this.$store.getters['getCanvasSize']
  
          const canvasWraper = document.getElementById("canvas-wraper");
          canvasWraper.style.width = canvasSize.width + 'px';
          canvasWraper.style.height = canvasSize.height + 'px';
          
          const ballCanvas = document.getElementById("ball-canvas");
          ballCanvas.width = canvasSize.width;
          ballCanvas.height = canvasSize.height;
  
          const deadZoneCanvas = document.getElementById("deadzone-canvas");
          deadZoneCanvas.width = canvasSize.width;
          deadZoneCanvas.height = canvasSize.height;
          
          const deadZoneCtx = deadZoneCanvas.getContext('2d');
          deadZoneCtx.lineWidth = 1;
          deadZoneCtx.shadowBlur = 20;
          deadZoneCtx.strokeStyle = '#cb0000'
          deadZoneCtx.fillStyle = '#ff5b5b'
          deadZoneCtx.shadowColor = "#e20000";
          this.$store.dispatch('setdeadZoneCtx', {ctx:deadZoneCtx});
  
          const ctx = ballCanvas.getContext("2d");
          ctx.fillStyle = '#e76f51';
          const circle = new Path2D();
          const ballCoordinates = this.$store.getters['getBallCoordinates'];
          circle.arc(ballCoordinates.currentX, ballCoordinates.currentY, ballCoordinates.radius, 0, 2 * Math.PI);
          ctx.fill(circle);
          this.$store.dispatch('setCtx', {ctx:ctx})
        }
      },
      mounted(){
        this.setFpsAndGame();
        // load best score and games played number
        this.$store.dispatch('loadBestScoreAndGamesPlayed');
        // add keydown event listener
        document.addEventListener('keydown', this.pressSpace)
      },
      beforeUnmount() {
        document.removeEventListener('click', this.pressSpace)
      },
    }
</script>
  
<style scoped>
    #ball-canvas,
    #deadzone-canvas{
      position: absolute;
      cursor: pointer;
      left: 0; 
      top: 0; 
    }
    #deadzone-canvas{
      z-index: 0;
    }
    #ball-canvas{
      z-index: 1;
    }

    .canvas-overlay{
      width: 100%;
      height: 100%;
      position: absolute;
      z-index:1000;
    }
    

    .fade-in{
      animation: fade-in 0.4s ease-in-out;
    }
    /* fade-in animation */
    @keyframes fade-in{
      0%{opacity:0;}
      50%{opacity:0;}
      100%{opacity:1;}
    }
</style>
  
  