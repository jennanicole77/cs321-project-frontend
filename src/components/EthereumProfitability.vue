<template>
    <div>
      <h2>
        <img class="img-thumbnail" alt="1000x1628" src="https://ethereum.org/static/6b935ac0e6194247347855dc3d328e83/34ca5/eth-diamond-black.png"
          data-holder-rendered="true" style="width: 80px; height: 130px;">
          <br>
        <label>Ethereum Calculator</label>
      </h2>
      <div>
        <button class="btn btn-secondary sessions" @click="saveSession">Save Session</button>
        <label class="btn btn-secondary sessions" for="upload">Load Session</label>
        <input id="upload" type="file" class="custom-file-input" @change="loadSession" ref="fileInput">
      </div>
      <br>
      <div class="grid">  
        <div>
          <label>
            How much ETH have you mined so far?<span class="required">*</span>
          </label>
          <input class="textFields" v-model="currentlyMined" v-on:keyup="reset" type="number"/>
        </div>
        <div>
          <label>
            What was the total price of your rig?<span class="required">*</span>
          </label>
          <input class="textFields" v-model="rigPrice" v-on:keyup="reset" type="number"/>
        </div>
        <div>
          <label>
            What is your federal Income Tax Bracket<span class="required">*</span>
          </label>
          <input class="textFields" v-model="taxBracket" v-on:keyup="reset" type="number"/>
        </div>
        <div>
          <label>
            What is the power rate at your location?<span class="required">*</span>
          </label>
          <input class="textFields" v-model="powerRate" v-on:keyup="reset" type="number"/>
        </div>
        <br>
        <div> 
          <label>Choose a GPU</label>
          <select id="GPU-Name" v-model="gpuName">
            <option v-for="gpu in gpus" :key="gpu">{{gpu}}</option>
          </select>
          <label> Quantity </label>
          <input v-model="gpuQuantity" type="number"/>
          <div class="gpus-buttons"> 
            <button class="btn btn-secondary space" @click="addGPU">Add GPU</button>
            <button class="btn btn-secondary space" @click="removeGPU">Remove GPU</button>
          </div>
        </div> 
        <br>
      </div>  
      <div class="buttons"> 
        <button class="btn btn-secondary" id="current-gpu" @click="displayGPUs">Display Current GPUs</button>
        <template v-if="display">
          <div  v-for="currGpu in gpusList" :key="currGpu.name" id="GPU-List">
            <span>Name: {{currGpu.name}} Hash: {{currGpu.hash}} Power: {{currGpu.power}} Quantity: {{currGpu.quantity}}</span>
          </div>
        </template>
      </div>
      <div class="buttons">
        <button type="button" class="btn btn-secondary" id="calculate" @click="calculate" :disabled=!compute>Calculate</button>
      </div>
      <br>
      <div v-if="!complete && clickedCalculated && compute" class="spinner-border text-secondary" role="status">
        <span class="sr-only">Loading...</span>
      </div>
      <template v-if="clickedCalculated && compute && complete">
        <div class="buttons">
          <label class="eth-price">Calculated with 1 ETH = <span class="bold">${{currPrice}}</span> and profitablility per 100Mhs = <span class="bold">${{currProfitability}}</span></label>
        </div>
        <br>
        <div class="grid"> 
          <div> 
            <label>You currently Own:&nbsp;</label>
            <span>${{currentlyMined}}.</span>
          </div>
          <div> 
            <label>Your total system price was:&nbsp;</label>
            <span>${{rigPrice}}.</span> 
          </div>
          <div> 
            <label>You need to mine:&nbsp;</label>
            <span>${{amountToMine}} to reach ROI.</span> 
          </div>
          <div> 
            <label>Your current hashrate is:&nbsp;</label>
            <span>{{currentHashrate}} Mh/s.</span> 
          </div>
          <div> 
            <label>Revenue per day is estimated at:&nbsp;</label>
            <span>${{revenuePerDay}}</span> 
          </div>
          <div> 
            <label>Profit per day is estimated at:&nbsp;</label>
            <span>${{profitPerDay}}.</span>
          </div>
          <div> 
            <label>At current prices your Rig will be paid off in {{daysTillPaidOff}}</label>
          </div>
          <br>
          <div>
            <label id="calculationNote"><span>*</span>These calculations have been adjusted for Federal Taxes and Power Consumption.</label>
          </div>
        </div>  
      </template>
    </div>
</template>

<script>

export default {
  name: 'Etherem-Mining-Profitability-Calculator',

  data() {
    
    return {
      file: null,
      user: null,
      currentlyMined: null,
      rigPrice: null,
      taxBracket: null,
      powerRate: null,
      currentlyOwn: null,
      systemPrice: null,
      amountToMine: null,
      currentHashrate: null,
      revenuePerDay: null,
      profitPerDay: null,
      daysTillPaidOff: null,
      gpus: null,
      gpuQuantity: null,
      gpuName: null,
      gpusList: null,
      clickedCalculated: false,
      currPrice: null,
      currProfitability: null,
      display: false
    }
  },
  
  computed: {
    compute() {
      return this.currentlyMined && this.rigPrice && this.taxBracket && this.powerRate;
    },

    complete() {
      return this.amountToMine != null && this.currentHashrate != null && this.revenuePerDay != null
        && this.profitPerDay != null && this.daysTillPaidOff != null;
    },
  },

  created: function() {
    this.getEthereumPrice();
    this.getProfitability();
    
    let self = this;
      fetch('https://backend.saturnp15.com//user')
      .then(response => response.json())
      .then(data => {
        let myData = JSON.parse(data)
        self.user = myData;
        self.gpus = self.user.All_Gpu_Dict;
      });
  },

  methods: {
    loadSession(event) {      
      let self = this;
      const reader = new FileReader();

      reader.readAsText(event.target.files[0]);

      reader.onload = function() {
        let result = JSON.parse(reader.result);
        self.reset();
        console.log("HI")
        self.currentlyMined = result.ethereum;
        self.rigPrice = result.total_cost;
        self.taxBracket = result.tax_rate;
        self.powerRate = result.power_rate;
        self.gpusList = [];
        result.user_gpu.forEach(function(gpu) {
            self.gpusList.push({name: gpu[1], hash: gpu[3], power: gpu[5], quantity: gpu[7]})
        });
        fetch('https://backend.saturnp15.com//user')
        .then(response => response.json())
        .then(data => {
          let myData = JSON.parse(data)
          self.user = myData;
          self.gpus = self.user.All_Gpu_Dict;
          fetch('https://backend.saturnp15.com//load?load=' + btoa(reader.result), {
              method : 'PUT',
            })
            .then(response => response.json())
            .then(data => {
              console.log(data)
          });
        });
      };

      reader.onerror = function() {
        console.log(reader.error);
      };
    },

    saveSession: function() {
      let save = ""

      //getting a saved session
      fetch('https://backend.saturnp15.com//save')
      .then(response => response.json())
      .then(data => {
        save = data
        let link = document.createElement('a');
        link.download = 'ethereum.json';
        let blob = new Blob([save], {type: 'text/plain'});
        link.href = window.URL.createObjectURL(blob);
        link.click(); 
      });
    },

    displayGPUs: function() {
      if(this.display)
        this.display = false;
      else
        this.display = true;
    },

    addGPU: function() {
      this.display = false;

      //removing gpu 
      let self = this;
      fetch('https://backend.saturnp15.com//gpu-update?name=' + self.gpuName, {
        method : 'PUT'
      })
      .then(response => response.json())
      .then(data => {
        self.gpuName = data;

        fetch('https://backend.saturnp15.com//add-gpu?quantity=' + self.gpuQuantity, {
          method : 'PUT'
        })
        .then(response => response.json())
        .then(data => {
          self.gpusList = [];
          JSON.parse(data).forEach(function(gpu) {
            self.gpusList.push({name: gpu[1], hash: gpu[3], power: gpu[5], quantity: gpu[7]})
          });
        });
      }); 
    },

    removeGPU: function() {
      this.display = false;

      //removing gpu 
      let self = this;
      fetch('https://backend.saturnp15.com//gpu-update?name=' + self.gpuName, {
        method : 'PUT'
      })
      .then(response => response.json())
      .then(data => {
        self.gpuName = data;

        fetch('https://backend.saturnp15.com//remove-gpu?quantity=' + self.gpuQuantity, {
          method : 'PUT'
        })
        .then(response => response.json())
        .then(data => {
          self.gpusList = [];
          JSON.parse(data).forEach(function(gpu) {
            self.gpusList.push({name: gpu[1], hash: gpu[3], power: gpu[5], quantity: gpu[7]})
          });
        });
      }); 
    },

    calculate: function() {
      this.amountToMine = null;
      this.currentHashrate = null;
      this.revenuePerDay = null;
      this.profitPerDay = null;
      this.daysTillPaidOff = null;
      this.clickedCalculated = true;
      this.getEthereumPrice();
      this.getProfitability();

      let self = this;
      //setting amount mined
      fetch('https://backend.saturnp15.com//amount-mined?mined=' + self.currentlyMined, {
        method : 'PUT',
      })
      .then(response => response.json())
      .then(data => {
        self.user.ethereum = data;
        self.currentlyOwn = data

        //setting rig price
        fetch('https://backend.saturnp15.com//rig?rig=' + self.rigPrice, {
          method : 'PUT',
        })
        .then(response => response.json())
        .then(data => {
          self.user.total_cost = data;

          //setting tax rate
          fetch('https://backend.saturnp15.com//tax?tax=' + self.taxBracket, {
            method : 'PUT',
          })
          .then(response => response.json())
          .then(data => {
            self.user.tax_rate = data;

            //setting power
            fetch('https://backend.saturnp15.com//power?power=' + self.powerRate, {
              method : 'PUT',
            })
            .then(response => response.json())
            .then(data => {
              self.user.power_rate = data;

              //getting amount to mine
              fetch('https://backend.saturnp15.com//mine')
              .then(response => response.json())
              .then(data => {
                let myData = JSON.parse(data)
                self.amountToMine = myData.toFixed(2);
              });

              //getting current hash rate
              fetch('https://backend.saturnp15.com//hashrate')
              .then(response => response.json())
              .then(data => {
                let myData = JSON.parse(data)
                self.currentHashrate = myData;
              });

              //getting revenue per day
              fetch('https://backend.saturnp15.com//revenue')
              .then(response => response.json())
              .then(data => {
                let myData = JSON.parse(data)
                self.revenuePerDay = myData.toFixed(2);
              });

              //getting profit per day
              fetch('https://backend.saturnp15.com//profit')
              .then(response => response.json())
              .then(data => {
                let myData = JSON.parse(data)
                self.profitPerDay = myData.toFixed(2);
              });

              //getting days till paid off
              fetch('https://backend.saturnp15.com//days')
              .then(response => response.json())
              .then(data => {
                let myData = JSON.parse(data);
                self.daysTillPaidOff = myData;
              });
            });
          });
        });
      });
    },
  
    reset: function() {
      this.clickedCalculated = false;
      this.amountToMine = null;
      this.currentHashrate = null;
      this.revenuePerDay = null;
      this.profitPerDay = null;
      this.daysTillPaidOff = null;
      this.gpusList = null;
      const input = this.$refs.fileInput;
      input.value = null;
    },

    getEthereumPrice: function() {
      let self = this;
      fetch('https://backend.saturnp15.com//eth-price')
      .then(response => response.json())
      .then(data => {
        self.currPrice = data.price;
      });
    },

    getProfitability: function() {
      let self = this;
      fetch('https://backend.saturnp15.com//profitability')
      .then(response => response.json())
      .then(data => {
        self.currProfitability = data.id;
      });
    }
  }
}
</script>

<style scoped>
  .required {
    color:#CC3333;
  }

  .grid {
    text-align: left;
    margin-left: 275px;
  }
  
  .textFields {
    float: right;
    margin-right: 275px;
  }

  #GPU-Name {
    margin-right: 10px;
  }

  #calculate{
    color: white;
  }

  #current-gpu {
    margin-bottom: 10px;
  }

  #calculationNote {
    font-size:12px;
  }

  .gpus-buttons {
     float: right;
     margin-right: 248px;
  }

  .space, .sessions {
    margin-left: 10px;
    margin-bottom: 0px;
  }

  .buttons {
    text-align: center;
  } 

  .eth-price {
    font-size: 14px;
  }

  .bold {
    font-weight: 500;
  }

  

  @import'~bootstrap/dist/css/bootstrap.css'
</style>