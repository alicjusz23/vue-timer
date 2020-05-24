<template>
    <div>
        <p> {{ curTime }}</p>
        <div id="chosenTime">
            <input 
                type="number" 
                placeholder="00"
                v-model="chosen.chosenHour" 
                min="0"
                max="24"
                :disabled="working"
            />
            <input 
                type="number" 
                placeholder="00"
                v-model="chosen.chosenMin" 
                min="0"
                max="60"
                :disabled="working"
            />
            <input 
                type="number" 
                placeholder="00"
                v-model="chosen.chosenSec" 
                min="0"
                max="60"
                :disabled="working"
            />
        </div>
        <button v-if="working" @click="pause()" >Pause</button>
        <button 
            v-else 
            @click="start()" 
            :disabled="isStartDisabled"
        >
            Start
        </button>
        <button @click="stop()" >Stop</button>
    </div>
</template>

<script>
export default {
    name: 'minutnik',
    data() {
        return {
            interval: null,
            curTime: 0,
            chosen: {
                chosenHour: null,
                chosenMin: null,
                chosenSec: null
            },
            working: false,
        }
    },
    methods: {
        start(){      
            this.working = true
            var curDate = new Date()
            curDate.setSeconds(curDate.getSeconds() + this.curTime + this.chosenTime)
            this.interval = setInterval(() => {
                this.count(curDate);
            },1000);            
            this.chosenTime = null
        },
        count(d){
            this.curTime = Math.floor((d - (new Date()))/1000)
            //make input disabled
        },
        pause(){
            this.working = false
            clearInterval(this.interval)
            //this.chosenTime = this.curTime
        },
        stop(){
            this.working = false
            clearInterval(this.interval)
            this.curTime =0
            this.chosenTime = null
        }
    },
    watch: {
        //when the time counter is finished
        curTime: function(){
            if(this.curTime<=0){
                this.working = false
                clearInterval(this.interval)
                this.curTime=0
                this.chosenTime = null
            }
        }
    },
    computed: {
        isStartDisabled(){
            if (this.chosenTime==null && this.curTime==0)
                return true
            else
                return false
        },
        chosenTime: {
            get(){
                return parseInt(this.chosen.chosenHour)*24*60 
                    + parseInt(this.chosen.chosenMin)*60 
                    + parseInt(this.chosen.chosenSec)
            },
            set(value){
                this.chosen.chosenHour = value
                this.chosen.chosenMin = value
                this.chosen.chosenSec = value
            }
        }
    }
}
</script>

<style scoped>
    :disabled {
        background-color: grey;
    }

    #chosenTime {
        display: flex;
    }

    input {
        text-align: right;
        width: 10%;
    }
    input, button {
        margin: 0.5em;

    }
</style>