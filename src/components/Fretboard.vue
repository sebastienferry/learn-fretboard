<script setup>
import { ref } from 'vue'

let counter=0;
const playing = ref(false);
const difficulty = ref(1);
const speed = ref(15000);
const challenge = ref('')
const strings = ref([
    { id: 1, x: -100, y: 0,   empty: false, current: "" },
    { id: 2, x: -100, y: 30,  empty: false, current: "" },
    { id: 3, x: -100, y: 60,  empty: false, current: "" },
    { id: 4, x: -100, y: 90,  empty: false, current: "" },
    { id: 5, x: -100, y: 120, empty: false, current: "" },
    { id: 6, x: -100, y: 150, empty: false, current: "" },
])
const notes = [
    { n: ["C"] },   //1
    { n: ["C#", "Db"] },  //2
    { n: ["D"] },   //3
    { n: ["D#", "Eb"] },  //4
    { n: ["E"] },   //5
    { n: ["F"] },   //6
    { n: ["F#", "Gb"] },  //7
    { n: ["G"] },   //8
    { n: ["G#", "Ab"] },  //9
    { n: ["A"] },   //10
    { n: ["A#", "Bb"] },  //11
    { n: ["B"] }    //12 
]

function __debug(str){
    if (false) {
        console.log(str);
    }    
}

function getNoteName(id) {
    if(notes[id-1].n.length > 1)
    {
        return notes[id-1].n[counter%2];
    }
    return notes[id-1].n[0];    
}

class Tuning{
    constructor(tuning) {
        this.tuning = tuning;
    }

    getFret(string, noteId) {
        var tuning = this.tuning[string - 1];        
        __debug("Tuning for string " + string + ": " + tuning + ", noteId: " + noteId);
        
        var fret = 12;

        if(noteId > tuning){
            fret = noteId - tuning;
        } else if (noteId < tuning) {
            fret = 12 - (tuning - noteId);
        }

        __debug("Fret: " + fret);
        return fret;
    }    
}

// E, A, D, S, B, E
const StandardTuning = new Tuning([5, 12, 8, 3, 10, 5])

function moveRandom() {
    let noteId = Math.floor(Math.random() * 12) + 1;
    __debug("Random id:" + noteId);
    let noteName = getNoteName(noteId);
    __debug("Random name:" + noteName);    
    note.value = noteName;

    let fret = StandardTuning.getFret(6, noteId);
    moveToFret(6, fret);
}

function setCurrent(s, c) {
    strings.value[s-1].current = c;
}

function moveToFret(s, f) {
    strings.value[s-1].empty = f == 12;    
    strings.value[s-1].x = 50*f;
}

function moveToString(s) {
    strings.value[0].y = 30*s - 30;
}

let timeoutId = -1;

function playGame(play) {
    if(play) {
        playing.value = true;
        gameLoop();
    } else {
        if(timeoutId > 0) {
            clearTimeout(timeoutId);
        }
        playing.value = false;        
    }    
}

function resetBoard(){
    for(var i=0;i<6;i++){
        strings.value[i].x = -200;
        strings.value[i].empty = false;
    }
}

function gameLoop() {
    if(playing.value)
    {
        // challenge
        counter++;
        let noteId = Math.floor(Math.random() * 12) + 1;
        let noteName = getNoteName(noteId);
        challenge.value = noteName;
        resetBoard();

        // let thing
        timeoutId = setTimeout(() => {            

            // display result and start over
            for(var i=6; i >= (6 - (difficulty.value - 1)); i--) {
                let fret = StandardTuning.getFret(i, noteId);
                setCurrent(i, challenge.value);
                moveToFret(i, fret);
            }

            challenge.value = "";
            
            // start over aver 5 secs
            timeoutId = setTimeout(() => {
                if(playing.value) {
                    gameLoop();
                }
            }, 5000);

        }, speed.value);
    }    
}

</script>

<template>
    {{  speed }}
    <div class="outer">
        <div class="inner center">        
            <span class="challenge">{{ challenge }}</span><br />        
            Difficulty:
            <select name="difficulty" id="difficulty" @change="(e) => { difficulty = e.target.value; }">
                <option value="1">1</option>
                <option value="2">2</option>
                <option value="3">3</option>
                <option value="4">4</option>
                <option value="5">5</option>
                <option value="6">6</option>            
            </select>
            Speed:
            <select name="speed" id="speed" @change="(e) => { speed = parseInt(e.target.value); }">
                <option value="15000">15 secs</option>
                <option value="12000">12 secs</option>
                <option value="10000">10 secs</option>
                <option value="8000">8 secs</option>
                <option value="5000">5 secs</option>
            </select>
            <button v-if="!playing" @click="() => { playGame(true); }">Start</button>
            <button v-if="playing" @click="() => { playGame(false); }">Stop</button><br />
        </div>
        <div class="inner">
            <svg height="200" width="700">
                <g transform="translate(50, 25)">
                    <line x1="0" x2="600" y1="0"   y2="0" style="stroke:rgb(0,0,0);stroke-width:1" />
                    <line x1="0" x2="600" y1="30"  y2="30" style="stroke:rgb(0,0,0);stroke-width:1.3" />
                    <line x1="0" x2="600" y1="60"  y2="60" style="stroke:rgb(0,0,0);stroke-width:1.5" />
                    <line x1="0" x2="600" y1="90"  y2="90" style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="0" x2="600" y1="120" y2="120" style="stroke:rgb(0,0,0);stroke-width:2.5" />
                    <line x1="0" x2="600" y1="150" y2="150" style="stroke:rgb(0,0,0);stroke-width:3" />
                </g>
                
                <g transform="translate(50, 25)">
                    <line x1="0"   x2="0"    y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:4" />
                    <line x1="50"  x2="50"   y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="100" x2="100"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="150" x2="150"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="200" x2="200"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="250" x2="250"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="300" x2="300"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="300" x2="300"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="350" x2="350"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="400" x2="400"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="450" x2="450"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="500" x2="500"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="550" x2="550"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                    <line x1="600" x2="600"  y1="0" y2="150"  style="stroke:rgb(0,0,0);stroke-width:2" />
                </g>
                
                <g transform="translate(50, 25)">
                    <circle cx="125" cy="75" r="6" stroke="gray" stroke-width="1" fill="rgb(200,200,200)" />
                    <circle cx="225" cy="75" r="6" stroke="gray" stroke-width="1" fill="rgb(200,200,200)" />
                    <circle cx="325" cy="75" r="6" stroke="gray" stroke-width="1" fill="rgb(200,200,200)" />
                    <circle cx="425" cy="75" r="6" stroke="gray" stroke-width="1" fill="rgb(200,200,200)" />
                    <circle cx="575" cy="45" r="6" stroke="gray" stroke-width="1" fill="rgb(200,200,200)" />
                    <circle cx="575" cy="105" r="6" stroke="gray" stroke-width="1" fill="rgb(200,200,200)" />
                </g>

                <g transform="translate(25, 25)">
                    <circle :cx="strings[5].x" :cy="strings[5].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text :x="strings[5].x - 5" :y="strings[5].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[5].current }}</text>
                    <circle v-if="strings[5].empty" cx="0" :cy="strings[5].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text v-if="strings[5].empty" x="-5" :y="strings[5].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[5].current }}</text>
                </g>

                <g transform="translate(25, 25)">
                    <circle :cx="strings[4].x" :cy="strings[4].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text :x="strings[4].x - 5" :y="strings[4].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[4].current }}</text>
                    <circle v-if="strings[4].empty" cx="0" :cy="strings[4].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text v-if="strings[4].empty" x="-5" :y="strings[4].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[4].current }}</text>
                </g>

                <g transform="translate(25, 25)">
                    <circle :cx="strings[3].x" :cy="strings[3].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text :x="strings[3].x - 5" :y="strings[3].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[3].current }}</text>
                    <circle v-if="strings[3].empty" cx="0" :cy="strings[3].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text v-if="strings[3].empty" x="-5" :y="strings[3].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[3].current }}</text>
                </g>

                <g transform="translate(25, 25)">
                    <circle :cx="strings[2].x" :cy="strings[2].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text :x="strings[2].x - 5" :y="strings[2].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[2].current }}</text>
                    <circle v-if="strings[2].empty" cx="0" :cy="strings[2].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text v-if="strings[2].empty" x="-5" :y="strings[2].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[2].current }}</text>
                </g>

                <g transform="translate(25, 25)">
                    <circle :cx="strings[1].x" :cy="strings[1].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text :x="strings[1].x - 5" :y="strings[1].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[1].current }}</text>
                    <circle v-if="strings[1].empty" cx="0" :cy="strings[1].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text v-if="strings[1].empty" x="-5" :y="strings[1].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[1].current }}</text>
                </g>

                <g transform="translate(25, 25)">
                    <circle :cx="strings[0].x" :cy="strings[0].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text :x="strings[0].x - 5" :y="strings[0].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[0].current }}</text>
                    <circle v-if="strings[0].empty" cx="0" :cy="strings[0].y" r="12" stroke-width="0" fill="rgb(255,0,0)" />
                    <text v-if="strings[0].empty" x="-5" :y="strings[0].y + 5" fill="rgb(255,255,255)" style="font-weight: bold;">{{ strings[0].current }}</text>
                </g>
            </svg>
        </div>
    </div>
</template>

<style>
.center {
    text-align: center;
}

.outer {
    width: 700px;
    height: 300px;
    margin-left: auto;
    margin-right: auto;
}

.inner {
  position: relative;
}

.challenge {    
    font-size: 48px;
}
</style>