<template>
    <div class="center">
        <p id="curTime"> {{ current.curHour }} h  {{ current.curMin }} min  {{ current.curSec }} sec </p>
        
        <div v-if="more" 
            id="more">
            <input 
                v-model="serie.time"
                placeholder="Serie time" 
            />
            <input 
                v-model="serie.break"
                placeholder="Serie break" 
            />
            <input 
                v-model="serie.count"
                placeholder="Serie count" 
            />
        </div>
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
        <div class="divButtons">
            <div v-if="!more" id="divLess">
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
            </div>
            <div v-else id="divMore">
                <button 
                    id="pauseMore"
                    v-if="working" 
                    @click="pause()" 
                >
                    Pause
                </button>
                <button 
                    id="startMore"
                    v-else 
                    @click="startMore()"
                    :disabled="isStartDisabled"
                >
                    Start
                </button>
            </div>
            <button @click="stop()" >Stop</button>
            <button v-if="!more" @click="more=!more">More options >></button>
            <button v-else @click="more=!more">Less options &lt;&lt; </button>
        </div>
    </div>
</template>

<script>
export default {
    name: 'minutnik',
    data() {
        return {
            bigInterval: null,
            interval: null,
            //string
            chosen: {
                chosenHour: "00",
                chosenMin: "00",
                chosenSec: "10"
            },
            //integer
            current: {
                curHour: 0,
                curMin: 0,
                curSec: 0
            },
            serie: {
                time: null,
                break: null,
                count: 2
            },
            working: false,
            pausing: false,
            more: false,
            audio: {
                single:  new Audio(require('@/assets/sound.mp3')),
                double:  new Audio(require('@/assets/double_buzzer.mp3'))
            }
        }
    },
    methods: {
            // start(){  
            //         this.working = true
            //         this.pausing = false
            //         var curDate = new Date()
            //         curDate.setSeconds(curDate.getSeconds() + this.curTime + this.chosenTime)
            //         this.interval = setInterval(() => {
            //             this.count(curDate);
            //             //when the time counter is finished
            //             if(this.curTime==0){
            //                 this.stop()
            //                 this.audio.play()
            //                 this.flarePageTitle()
            //             }
            //         },1000);            
            //         this.chosenTime = 0
            // },
        start(){  
            this.working = true
            this.pausing = false
            var curDate = new Date()
            curDate.setSeconds(curDate.getSeconds() + this.curTime + this.chosenTime)
            this.interval = setInterval(() => {
                this.count(curDate);
                //when the time counter is finished
                if(this.curTime==0){
                    this.serie.count--
                    clearInterval(this.interval)
                    this.curTime = 0
                    this.audio.single.play()
                }
            },1000);        
            //this.chosenTime = 0
        },
        startMore(){
            var i=0
            while(i<this.serie.count){ 
                setTimeout(() => this.start(), 15000*i);
                i++
            }
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
            this.interval = null
            //this.curTime = 0
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
        },
        flarePageTitle(){
            let newTitle = "Time is over!"
            var titleInterval = setInterval(() => {
                var temp = document.title
                if(document.title!==newTitle){
                    document.title = newTitle
                    newTitle = temp
                }
                //when the time counter is finished
                if(window.onfocus){
                    clearInterval(titleInterval)
                }
            },2000);
        },
        changePageTitle() {
            window.visible = function () { 
                document.title = "vue-aplikacja"
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
        },
        'serie.count': function(){
            if ((parseInt(this.serie.count)==0)){
                this.working = this.pausing = false
                clearInterval(this.interval)
                this.curTime = this.chosenTime = 0
                this.interval = null
                this.audio.double.play()
                this.flarePageTitle()
            }
        }
    },
    mounted() {
        this.changePageTitle()
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
    @font-face {
        font-family: "digital-font";
        src: url("../assets/digital-font/digital-7.ttf");
    }
    div {
        text-align: center;
    }
    .divButtons {
        display: flex;
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
        width: 60%
    }
    #chosenTime, #curTime{
        font-family: "digital-font";
        font-size: 35px;
    }
    #chosenTime > input {
        text-align: right;
        width: 50px;
        padding: 0.2em;
        border: none;
        border-bottom: 2px solid #a279c8;
        /*color violet: a279c8 
        green: 6dcc85
        yellow: f3d75c
        */
    }
    input:focus {
        outline: none;
    }
    input, button {
        margin: 0.5em 0.3em 0.5em 0.3em;

    }
    /* button {
        background-color: #f3d75c;
    } */
    .break {
        padding: 8px 0 8px 0;
        margin: 0.5em 0 0.5em 0;
    }
</style>