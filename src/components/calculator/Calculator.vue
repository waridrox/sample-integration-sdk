<template>
<div class="container flex">
        <div class="flex-1 px-2">
        <form @submit="(e) => saveProfile(e)" class="bg-white shadow-md rounded px-8 pt-6 pb-6 mb-4 h-full" method="post">
          <div class="flex items-center justify-between">
            <button v-if="StartTrackingStatus" @click="StartTrackingFunc()" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" ref="btnToggleMain" type="button">
              Start Tracking
            </button>

            <button v-else-if="BackToStartStatus" @click="BackToStart()" class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" ref="btnToggle" type="button">
              Back to Start
            </button>

            <button v-else @click="StopTrackingFunc()" class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" ref="btnToggle" type="button">
              Stop Tracking
            </button> 

            <button v-if="ShowResultStatus" @click="()=>showResults()" class="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800 " type="button">
              Show Result
            </button>


            <button v-else @click="()=>deleteData()" class="inline-block align-baseline font-bold text-sm text-red-500 hover:text-red-800 " type="button">
              Delete Data
            </button>
          </div>

            <label v-if="ShowResultStatus" class="text-center block text-gray-700 text-sm font-bold my-2" for="test-input">
              Test Form
            </label>

            <label v-else class="text-center block text-gray-700 text-sm font-bold my-2" for="test-input">
              Tracking Results
            </label>
            
            <input class="shadow appearance-none border rounded h-auto w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" height="50" ref="populate" id="test-input" type="text">

            <button v-if="ShowResultStatus" type="submit" class="w-full bg-black hover:bg-gray-700 text-white font-bold my-4 py-2 px-4 rounded focus:outline-none focus:shadow-outline">
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
import { ref, onMounted } from "vue";
import "./calculator.css";
import "./../button/Button.vue";

import jQuery from "jquery";
const $ = jQuery;
window.$ = $;

var kinetic;

export default {
  data () {
      return {
        BackToStartStatus: false,
        ShowResultStatus: true,
        StartTrackingStatus: true,
        sampleResult: '',
        stopTrackingwasClicked: false,
      }
  },

  mounted: function() {
    this.handler()
  },

  methods: {
    reportAction(action, checkResp, allowTransaction) {
        var inputData = {
            profileCode: localStorage.getItem("profileCode"),
            action: action,
            refId: checkResp.refId,
            type: checkResp.data.type ? checkResp.data.type : 'gesture'
        };

        kinetic.reportAction(inputData, function (error, outputData) {
            if (error) {
                console.log(JSON.stringify(error));
            }

            console.log('reportAction outputData: ' + JSON.stringify(outputData));

            // If pin input is true or score > threshold then proceed
            if (allowTransaction) {

                // Do below after report action call response
                var selectedTransactionType = $("#transactionType :selected").text();
                localStorage.setItem("transType", selectedTransactionType);
                var amount = $("#amount").val();
                localStorage.setItem("amount", amount);

                if (config.disableChallenge == true) {
                    var transRefId = localStorage.getItem("transRefId");
                    console.log(transRefId);
                } else {
                    // Challenge required.
                    window.location.href = "challenge.html";
                }
            } else {
                // Deny
                window.location.href = "transaction-fail.html";
            }

        });
    },

    deleteData() {
      localStorage.setItem('records', null);
      this.sampleResult = localStorage.getItem('records')
      this.$refs.populate.value = this.sampleResult;
      console.log('Data deleted successfully!')
    },
    makeTransferId () {
      var text = "";
      var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-";

      for (var i = 0; i < 37; i++) {
          text += possible.charAt(Math.floor(Math.random() * possible.length));
      }

      return text;
    },

    BackToStart() {
      this.$refs.populate.value = '';
      this.BackToStartStatus = false;
      this.StartTrackingStatus = true;
      this.ShowResultStatus = true;
    },


    handler() {
      kinetic = new window.ZFS.KineticTracker(options);
      kinetic.init();
    },
    StartTrackingFunc() {
      $('#tracking-area').toggleClass('border-2')

      kinetic.trackStart()
      console.log('tracking started !')

      this.StartTrackingStatus = false
    },

    loginProfile(userName, text, callback) {

    if (text == null) {
        text = userName
    }

    var userData = {
        name: userName,
        uCode: userName
    };

    console.log(userData)
    kinetic.getProfile(userData, async function (error, profileData) {
                  console.log(profileData)

        if (error) {
            console.log(error);
            callback((error.data.errors[0].message));
        } else {
            console.log(profileData)
            localStorage.setItem("profileCode", profileData.data.profileCode);
            localStorage.setItem("userName", userName);

            console.log('all set successfully')

            document.getElementById('test-input').disabled = true;
        }
    });
},

    async saveProfile(e) {
      e.preventDefault();

      var userName = $('#test-input').val()
      console.log("Username: ", userName)

        if (userName != "") {

            this.loginProfile(userName, userName, function (error, response) {
                if (error) {
                    console.log(error);
                    alert(error);
                } else {

                    // Added inverse logic for confidence
                    var confidence = (100 - parseFloat(response.responseData.data.confidence));

                    if (response.responseData.data.score > config.loginThreshold) {
                        alert('Authentication Success. (Score = ' + response.responseData.data.score + ' and Confidence = ' + confidence + ')');
                        window.login(response.userName, function (error, response) {
                            if (error) {
                                console.log(error);
                                alert(error);
                            } else {
                              console.log(response)
                                window.location.href = "transaction.html";
                            }
                        });
                    } else {
                        alert('Authentication Failed. (Score = ' + response.responseData.data.score + ' and Confidence = ' + confidence + ')');
                        //window.location.href = "index.html";
                        // Ask for PIN input
                        var getPin = prompt("Please enter your PIN", "");

                        if (getPin == null || getPin == "") {
                            // PIN cancelled
                            window.location.href = "index.html";
                        } else {

                            // PIN entered
                            if (getPin == config.defaultPin) {
                                // PIN is correct
                                window.login(response.userName, function (error, response) {
                                    if (error) {
                                        console.log(error);
                                        alert(error);
                                    } else {
                                      console.log(response)
                                        window.location.href = "transaction.html";
                                    }
                                });
                            } else {
                                // PIN is wrong
                                window.location.href = "index.html";
                            }
                        }
                    }
                }
            });
        }
    },

    // Make transaction reference ID
    makeTransRefId() {
    var text = "";
    var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-";

    for (var i = 0; i < 37; i++) {
        text += possible.charAt(Math.floor(Math.random() * possible.length));
    }

    return text;
    },

    // Make transaction func
    makeTransaction() {
        var userName = localStorage.getItem("userName");
        var profileCode = localStorage.getItem("profileCode");
        var tempMethods = this

        if (profileCode == "" || userName == "") {
            localStorage.removeItem("authToken");
            localStorage.removeItem("userName");
            localStorage.removeItem("profileCode");
            alert("Your session has expired. Please login again");
            window.location.href = "index.html";
        } else {
            kinetic.trackStop(function (trackData) {
                var transRefId = tempMethods.makeTransRefId()
                var body = {
                    gestureInfo: trackData,
                    profileCode: profileCode,
                    transRefId: transRefId
                };

                console.log('trackData ' + trackData)

                kinetic.checkGesture(body, function (error, gestureData) {
                    if (error) {
                        alert(JSON.stringify(error));
                    } else {
                        localStorage.setItem("transRefId", gestureData.refId);
                        localStorage.setItem("appRefId", gestureData.data.reqRefId);

                        var score = gestureData.data.score;

                        // Score greater than thresh. value
                        if (score >= config.scoreThreshold) {
                            tempMethods.reportAction('allow', gestureData, true);
                            alert("Your mouse score is good: " + score);

                        } else {
                            // Score less than thres. value

                            // Ask for PIN input
                            var getPin = prompt("Your mouse score is not good " + score + "\nPlease enter your PIN", "");

                            if (getPin == null || getPin == "") {
                                // PIN cancelled
                                tempMethods.reportAction('deny', gestureData, false);
                            } else {

                                // PIN entered
                                if (getPin == config.defaultPin) {
                                    // PIN is correct
                                    tempMethods.reportAction('allow', gestureData, true);
                                } else {
                                    // PIN is wrong
                                    tempMethods.reportAction('deny', gestureData, false);
                                }
                            }
                        }
                    }
                });

                localStorage.setItem('records',JSON.stringify(trackData))
            });
            this.StartTrackingStatus = true
        }
    },
    
    async StopTrackingFunc() {
      $('#tracking-area').toggleClass('border-2')

      this.makeTransaction()

      this.stopTrackingwasClicked = true;
      this.StartTrackingStatus = false
      this.BackToStartStatus = true

      console.log('Tracking Stopped successfully!')
    },

    //Show results
    async showResults() {

      if (this.stopTrackingwasClicked == false) {
          window.alert('Stop the tracking first')
      }
      else {
        this.sampleResult = localStorage.getItem('records');

        this.$refs.populate.value = this.sampleResult;
        console.log(this.sampleResult);

        console.log(this.$refs.populate.value);
        console.log(options)
        this.ShowResultStatus = false;
      }
    }
  },

  //Calculator functionality
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
