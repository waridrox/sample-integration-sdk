<template>

<div class="container flex">
        <div class="flex-1 px-2">
        <form @submit="(e) => saveProfile(e)" class="bg-white shadow-md rounded px-8 pt-6 pb-6 mb-4 h-full">
          <div class="flex items-center justify-between">
            <button v-if="stoptrackingtempo" @click="buttonvisible()" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" ref="btnToggleMain" type="button">
              Start Tracking
            </button>

            <button v-else-if="backtostart" @click="takemeback()" class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" ref="btnToggle" type="button">
              Back to Start
            </button>

            <button v-else @click="stoptrackingtemp()" class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" ref="btnToggle" type="button">
              Stop Tracking
            </button> 



            <button v-if="resultValue" @click="()=>stopTracking()" class="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800 " type="button">
              Show Result
            </button>


            <button v-else @click="()=>deleteData()" class="inline-block align-baseline font-bold text-sm text-red-500 hover:text-red-800 " type="button">
              Delete Data
            </button>
          </div>


            <label v-if="resultValue" class="text-center block text-gray-700 text-sm font-bold my-2" for="test-input">
              Test Form
            </label>

            <label v-else class="text-center block text-gray-700 text-sm font-bold my-2" for="test-input">
              Tracking Results
            </label>
            
            <textarea class="shadow appearance-none border rounded h-auto w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" rows="20" ref="populate" id="test-input" type="text"></textarea>

            <button v-if="resultValue" type="submit" class="w-full bg-black hover:bg-gray-700 text-white font-bold my-4 py-2 px-4 rounded focus:outline-none focus:shadow-outline">
              Submit
            </button> 

        </form>
        </div>
        
        <div class="flex-1" id='tracking-area'>

            <div class="phone">
                <div class="top">
                    <button @click="showHistory" class="history"></button>
                    <button @click="deleteValue" class="delete"></button>
                </div>

                <ul v-if="isHistory" class="historyDetails">
                    <li :key="operation.operations" v-for="operation in historyData" >
                        <small>{{operation.operations}}</small>
                        <h2>{{operation.sum}}</h2>
                    </li>
                </ul>

                <div class="middle">
                    <small v-if="selectedOperation">{{ prevNum }} {{ selectedOperation }} {{ currentNum }}</small>
                    <div>
                        <h1>{{currentNum}}</h1>
                    </div>
                </div>
                <div class="bottom">
                    <button @click="pressed('c')">c</button>
                    <button @click="pressed('()')">()</button>
                    <button @click="pressed('%')">%</button>
                    <button @click="pressed('/')">/</button>

                    <button @click="pressed('7')">7</button>
                    <button @click="pressed('8')">8</button>
                    <button @click="pressed('9')">9</button>
                    <button @click="pressed('*')">*</button>

                    <button @click="pressed('4')">4</button>
                    <button @click="pressed('5')">5</button>
                    <button @click="pressed('6')">6</button>
                    <button @click="pressed('-')">-</button>

                    <button @click="pressed('1')">1</button>
                    <button @click="pressed('2')">2</button>
                    <button @click="pressed('3')">3</button>
                    <button @click="pressed('+')">+</button>
                    
                    <button @click="negateValue">+/-</button>
                    <button @click="pressed('0')">0</button>
                    <button @click="pressed('.')">.</button>
                    <button @click="pressed('=')">=</button>
                </div>
            </div>

        </div>
</div>

</template>

<script>
import config from '../../config/trackingConfig';
import options from '../../config/trackingOptions';

// import "../../lib/kinetic-sdk-1.0.1.min.js";
import { ref, onMounted } from "vue";
import "./calculator.css";
import "./../button/Button.vue";

var datt = null;
var daaa = '';
// var user = '';

export default {
  data () {
      return {
        resultValue: true,
        variable: true,
        tracking: false,
        loading: false,
        msg: '',
        kineticTracker: null,
        data: '',
        user: '',
        profile: null,

        sampleResult: '',
        stoptrackingtempo: true,
        backtostart: false,

        stopTrackingwasClicked: false,
      }
  },

  methods: {
    deleteData() {
      localStorage.setItem('records', null);
      this.sampleResult = localStorage.getItem('records')
      this.$refs.populate.value = this.sampleResult;
      console.log('Testing123')
    },
    makeTransferId () {
      var text = "";
      var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-";

      for (var i = 0; i < 37; i++) {
          text += possible.charAt(Math.floor(Math.random() * possible.length));
      }

      return text;
    },


    reportAction (action, checkResp, allowTransaction) {
    const inputData = {
      profileCode: this.profile.profileCode,
      action: action,
      refId: checkResp.refId,
      type: checkResp.data.type ? checkResp.data.type : 'gesture'
    };

    datt.reportAction(inputData,(error,outputData)=>{
      if(error){
        return console.log(JSON.stringify(error));
      }
      console.log('reportAction output :',JSON.stringify(outputData))

      if(allowTransaction){
          console.log('Allow transaction')
      }
    })
  },
    makeTransaction() {
      if(this.profile.profileCode===null && this.profile.userName===null){
        return
      }
      datt.trackStop((trackData)=>{
        this.data = trackData
        // setData(trackData)
        this.tracking = false;
        // setTracking(false)
        console.log(trackData)
        const transferId = this.makeTransferId()
        const body = {
          gestureInfo: trackData,
          profileCode: this.profile.profileCode,
          transRefId: transferId
        }

        console.log('trackData ' + trackData)

        datt.checkGesture(body,(error,gestureData)=>{
          const score = gestureData.data.score
          console.log(gestureData)
          if(score>=config.scoreThreshold){
            this.reportAction('allow',gestureData,true)
            return alert('Your mouse is good : '+score)
          }

          const getPin = prompt('Your mouse score is not good :'+score+'\n Please enter your PIN ',"")
          if (getPin == null || getPin == "") {
              // PIN cancelled
              this.reportAction('deny', gestureData, false);
          } else {
              // PIN entered
              if (getPin == config.defaultPin) {
                  // PIN is correct
                  this.reportAction('allow', gestureData, true);
              } else {
                  // PIN is wrong
                  this.reportAction('deny', gestureData, false);
              }
          }
          })
      })
    },

    takemeback() {
      this.resultValue = true;
      this.$refs.populate.value = '';
      this.backtostart = false;
      this.stoptrackingtempo = true;
    },


    handler() {
      const kinetic = new window.ZFS.KineticTracker(options);
      kinetic.init();
      this.kineticTracker = kinetic;
      datt = kinetic;
      // setKinaticTracker(kinetic);
      console.log(kinetic);
    },
    buttonvisible() {
      this.variable = !this.variable;
      let target_element = document.getElementById("tracking-area")
      target_element.classList.toggle('border-2')

      // this.$refs.btnToggle.innerText = this.variable?'Start Tracking':'Stop Tracking';
      this.startTracking();
      this.stoptrackingtempo = false
    },
    startTracking() {
      console.log(options)
      this.handler()
      // console.log({ ...this.kineticTracker })
      // console.log((this.kineticTracker));
      datt.trackStart()
      console.log(datt);
      // this.kineticTracker.trackStart()
      this.tracking = true;
      console.log('tracking started !')
    },

    async saveProfile(e) {
      e.preventDefault();

      if(this.user==='') return;

      const userData = {
        name:this.user,
        uCode:this.user
    }
    datt.getProfile(userData,function(error,data){
        console.log(data)
        if(error){
            alert("Error !")
            return console.log(JSON.stringify(error))
        }
        console.log("successfully saved profile")
        this.profile = data.data
    })
    },
    
    async stoptrackingtemp() {
      
      // this.stoptrackingtempo = false
      let target_element = document.getElementById("tracking-area")
      target_element.classList.toggle('border-2')

      if(this.profile!==null) {
        return this.makeTransaction()
      }

      else{
                this.stopTrackingwasClicked = true;

        await datt.trackStop(async function(trackingData) {
            daaa = trackingData;
            console.log(daaa);

            console.log(trackingData)
            console.log('tracker is stopped')

            
            localStorage.setItem('records',JSON.stringify(trackingData))

        })

      }

            this.stoptrackingtempo = true
    },

    async stopTracking() {

      if (this.stopTrackingwasClicked == false) { //false
          window.alert('Stop the tracking first')
      }
      else {

            this.resultValue = false

            this.stoptrackingtempo = false
            this.backtostart = true

            try {
            const userData = {
              name:this.user==''? 'No User':this.user,
              uCode:this.user==''? 'No User':this.user
            }

            datt.getProfile(userData, function(data, error) {
            console.log(data)
            if(error){
                alert("Error !")
                console.log(JSON.stringify(error))
            }
            else{
                console.log("successfully saved profile")
            }
        })
              // this.saveProfile();
            }
            catch (e) {
              console.log(e);
            }
        // });
        // this.resultValue = false;
        this.tracking = false;
        this.sampleResult = localStorage.getItem('records');

        // this.backtostart = false

        // this.$refs.btnToggleMain.innerText = 'Back to Start'
        // if (this.$refs.populate.innerText!=='')
        this.$refs.populate.value = this.sampleResult;
        console.log(this.sampleResult);

        console.log(this.$refs.populate.value);
        console.log(options)
        // this.stopTrackingwasClicked
      }
    }
  },
  setup() {
    const operations = ["+", "-", "*", "/", "%", "()"];
    const numbers = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "0", "."];
    const currentNum = ref("");
    const prevNum = ref("");
    const selectedOperation = ref("");
    const total = ref('')
    const isHistory = ref(false);
    const pressed = (value) => {
      if (value === "=" || value === "Enter") {
          historyData.push({
              operations: `${prevNum.value} ${selectedOperation.value} ${currentNum.value}`,
            sum: eval(`${prevNum.value} ${selectedOperation.value} ${currentNum.value}`)
        })
          calculate();
          console.log(historyData);

      }
      else if (value === "%") percentage();
      else if (value === "c") clear();
      else if (operations.includes(value)) applyOperation(value);
      else if (numbers.includes(value)) appendNumber(value);
    }
    const appendNumber = (value) => {
      currentNum.value = currentNum.value + value;
    }
    const applyOperation = (value) => {
      calculate();
      prevNum.value = currentNum.value;
      currentNum.value = "";
      selectedOperation.value = value;
    }
    const calculate = () => {
      if (selectedOperation.value === "*") multiply();
      if (selectedOperation.value === "()") multiply();
      else if (selectedOperation.value === "/") divide();
      else if (selectedOperation.value === "%") percentage();
      else if (selectedOperation.value === "-") subtract();
      else if (selectedOperation.value === "+") sum();
      prevNum.value = "";
      selectedOperation.value = "";
    }
    const multiply = () => {
      currentNum.value = prevNum.value * currentNum.value;
      total.value = currentNum.value;
    }
    const divide = () => {
      currentNum.value = prevNum.value / currentNum.value;
      total.value = currentNum.value;

    }
    const percentage = () => {
      currentNum.value = currentNum.value / 100;
      total.value = currentNum.value;

    }
    const subtract = () => {
      currentNum.value = prevNum.value - currentNum.value;
      total.value = currentNum.value;

    }
    const sum = () => {
      currentNum.value = +prevNum.value + +currentNum.value;
      total.value = currentNum.value;

    }
    const clear = () => {
      currentNum.value = ""
      prevNum.value = ""
      selectedOperation.value = ""
      currentNum.value = ""
    }

    const deleteValue = () => {
      if(currentNum.value.length !== 0 && currentNum.value !== '0') {
        currentNum.value = currentNum.value.slice(0, -1)
      }
    }   

    const negateValue = () => {
        if(currentNum.value  === '0') {
            return currentNum.value  = '-0'
        }
        if(currentNum.value === '-') {
        return currentNum.value = '0'
        }
        if(currentNum.value === '0.') {
            return currentNum.value = `-${currentNum.value}`
        }
        if(currentNum.value === '-0.') {
            return currentNum.value = '0.'
        }
        currentNum.value = `${currentNum.value * -1}` 
    }

    let historyData = [];
    
    const showHistory = () => {
        isHistory.value = !isHistory.value;
    }

    const handleKeydown = (e) => {
        pressed(e.key);
    }
    onMounted(async () => {
      try {
        window.addEventListener('keydown', handleKeydown)
      }
      catch(e) {
        console.log(e);
      }
    });

    return { currentNum, pressed, selectedOperation, prevNum, negateValue, deleteValue, total, historyData, isHistory, showHistory };
  },
};
</script>

<style scoped>
@import "./calculator.css";
</style>