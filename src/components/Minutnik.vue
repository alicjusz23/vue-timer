<template>
    <div class="center">
        <p id="curTime"> {{ current.curHour }} h  {{ current.curMin }} min  {{ current.curSec }} sec </p>
        <div id="chosenTime" class="center">
            <input
                v-model="chosen.chosenHour" 
                :disabled="working || pausing"
                @keypress="isNumber($event)"
            />
            <div class="break">:</div>
            <input 
                v-model="chosen.chosenMin" 
                :disabled="working || pausing"
                @keypress="isNumber($event)"
            />
            <div class="break">:</div>
            <input 
                v-model="chosen.chosenSec" 
                :disabled="working || pausing"
                @keypress="isNumber($event)"
            />
        </div>
        <div>
            <button 
                v-if="working" 
                @click="pause()" 
            >
                Pause
            </button>
            <button 
                v-else 
                @click="start()" 
                :disabled="isStartDisabled"
            >
                Start
            </button>
            <button @click="stop()" >Stop</button>
        </div>
    </div>
</template>

<script>
export default {
    name: 'minutnik',
    data() {
        return {
            interval: null,
            //string
            chosen: {
                chosenHour: "00",
                chosenMin: "00",
                chosenSec: "00"
            },
            //integer
            current: {
                curHour: 0,
                curMin: 0,
                curSec: 0
            },
            working: false,
            pausing: false,
            audio: new Audio(require('@/assets/sound.mp3'))
        }
    },
    methods: {
        start(){      
            this.working = true
            this.pausing = false
            var curDate = new Date()
            curDate.setSeconds(curDate.getSeconds() + this.curTime + this.chosenTime)
            this.interval = setInterval(() => {
                this.count(curDate);
                //when the time counter is finished
                if(this.curTime==0){
                    this.stop()
                    this.audio.play()
                }
            },1000);            
            this.chosenTime = 0
        },
        count(d){
            this.curTime = Math.floor((d - (new Date()))/1000)
        },
        pause(){
            this.working = false
            this.pausing = true
            clearInterval(this.interval)
        },
        stop(){
            this.working = this.pausing = false
            clearInterval(this.interval)
            this.curTime = this.chosenTime = 0
        },
        toTimeFormat(d){
            if(parseInt(d)<10)
                d = "0" + d.toString()
            return d
        },
        //check if number is inserted in inputs
        isNumber(evt){
            var charCode = evt.keyCode
            if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
                evt.preventDefault()
            } else {
                return true;
            }
        }
    },
    watch: {
        //change the text color is time is almost finished
        curTime: function() {
            if(this.curTime<4 && this.curTime>0)
                document.querySelector('#curTime').style.color = 'red';
            else
                document.querySelector('#curTime').style.color = 'black';
        }
    },
    computed: {
        isStartDisabled(){
            if (this.chosenTime==0 && this.curTime==0)
                return true
            else
                return false
        },
        chosenTime: {
            get(){
                return parseInt(this.chosen.chosenHour)*60*60 
                    + parseInt(this.chosen.chosenMin)*60 
                    + parseInt(this.chosen.chosenSec)
            },
            set(value){
                this.chosen.chosenHour = this.toTimeFormat(Math.floor(value/3600).toString())
                this.chosen.chosenMin = this.toTimeFormat(Math.floor((value - this.chosen.chosenHour*3600)/60).toString())
                this.chosen.chosenSec = this.toTimeFormat(value - this.chosen.chosenHour*3600 - this.chosen.chosenMin*60)
            }
        },
        curTime: {
            get(){
                return parseInt(this.current.curHour)*3600 
                    + parseInt(this.current.curMin)*60 
                    + parseInt(this.current.curSec)
            },
            set(value){
                this.current.curHour = Math.floor(value/3600)
                this.current.curMin = Math.floor((value - this.current.curHour*3600)/60)
                this.current.curSec = value - this.current.curHour*3600 - this.current.curMin*60
            }
        }
    }
}
</script>

<style scoped>
    div {
        text-align: center;
    }
    .center {
        margin: auto;
        width: 50%;
    }
    :disabled {
        background-color: rgb(216, 216, 216);
        border-color : rgb(165, 165, 165);
    }

    #chosenTime {
        display: flex;
        margin: auto auto 1em auto;
    }

    input {
        text-align: right;
        width: 50px;
        padding: 0.2em;
    }
    input, button {
        margin: 0.5em 0.3em 0.5em 0.3em;

    }
    .break {
        padding: 8px 0 8px 0;
        margin: 0.5em 0 0.5em 0;
    }
</style>