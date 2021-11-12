<template>
    <div>
      <h2>
        <img class="img-thumbnail" alt="1000x1628" src="https://ethereum.org/static/6b935ac0e6194247347855dc3d328e83/34ca5/eth-diamond-black.png"
          data-holder-rendered="true" style="width: 80px; height: 130px;">
          <br>
        <label>Ethereum Calculator</label>
      </h2>
      <div>
        <button class="sessions" @click="saveSession">Save Session</button>
        <button class="sessions" @click="loadSession">Load Session</button>
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
            <button class="space" @click="addGPU">Add GPU</button>
            <button class="space" @click="removeGPU">Remove GPU</button>
          </div>
        </div> 
        <br>
      </div>  
      <div class="buttons"> 
        <button id="current-gpu" @click="displayGPUs">Display Current GPUs</button>
        <template v-if="display">
          <div  v-for="currGpu in gpusList" :key="currGpu.name" id="GPU-List">
            <span>Name: {{currGpu.name}} Hash: {{currGpu.hash}} Power: {{currGpu.power}} Quantity: {{currGpu.quantity}}</span>
          </div>
        </template>
      </div>
      <div class="buttons">
        <button type="button" class="btn btn-secondary" id="calculate" @click="calculate" :disabled=!compute>Calculate</button>
      </div>
      <template v-if="clickedCalculated && compute">
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
      datePaidOff: null,
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
    }
  },

  created: function() {
    this.getEthereumPrice();
    this.getProfitability();
    
    let self = this;
      fetch('http://localhost:8080/user')
      .then(response => response.json())
      .then(data => {
        let myData = JSON.parse(data)
        self.user = myData;
        self.gpus = self.user.All_Gpu_Dict;
      });
  },

  methods: {
    saveSession: function() {
      let save = ""

      //getting a saved session
      fetch('http://localhost:8080/save')
      .then(response => response.json())
      .then(data => {
        console.log("save", save)
        save = data
        console.log(save)
        let link = document.createElement('a');
        link.download = 'ethereum.json';
        let blob = new Blob([save], {type: 'text/plain'});
        link.href = window.URL.createObjectURL(blob);
        link.click(); 
      });
    },

    loadSession: function() {
        //send in a file 
        //returns a new user class
        //update user text fields and reset calculation
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
      fetch('http://localhost:8080/gpu-update?name=' + self.gpuName, {
        method : 'PUT'
      })
      .then(response => response.json())
      .then(data => {
        self.gpuName = data;

        fetch('http://localhost:8080/add-gpu?quantity=' + self.gpuQuantity, {
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
      fetch('http://localhost:8080/gpu-update?name=' + self.gpuName, {
        method : 'PUT'
      })
      .then(response => response.json())
      .then(data => {
        self.gpuName = data;

        fetch('http://localhost:8080/remove-gpu?quantity=' + self.gpuQuantity, {
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
      this.clickedCalculated = true;
      this.getEthereumPrice();
      this.getProfitability();

      let self = this;
      //setting amount mined
      fetch('http://localhost:8080/amount-mined?mined=' + self.currentlyMined, {
        method : 'PUT',
      })
      .then(response => response.json())
      .then(data => {
        self.user.ethereum = data;
        self.currentlyOwn = data;
      });

      //setting rig price
      fetch('http://localhost:8080/rig?rig=' + self.rigPrice, {
        method : 'PUT',
      })
      .then(response => response.json())
      .then(data => {
        self.user.total_cost = data;
      });
      
      //setting tax rate
      fetch('http://localhost:8080/tax?tax=' + self.taxBracket, {
        method : 'PUT',
      })
      .then(response => response.json())
      .then(data => {
        self.user.tax_rate = data;
      });

      //setting power
      fetch('http://localhost:8080/power?power=' + self.powerRate, {
        method : 'PUT',
      })
      .then(response => response.json())
      .then(data => {
        self.user.power_rate = data;
      });

      //getting amount to mine
      fetch('http://localhost:8080/mine')
      .then(response => response.json())
      .then(data => {
        let myData = JSON.parse(data)
        self.amountToMine = myData.toFixed(2);
      });

      //getting current hash rate
      fetch('http://localhost:8080/hashrate')
      .then(response => response.json())
      .then(data => {
        let myData = JSON.parse(data)
        self.currentHashrate = myData;
      });

      //getting revenue per day
      fetch('http://localhost:8080/revenue')
      .then(response => response.json())
      .then(data => {
        let myData = JSON.parse(data)
        self.revenuePerDay = myData.toFixed(2);
      });

      //getting profit per day
      fetch('http://localhost:8080/profit')
      .then(response => response.json())
      .then(data => {
        let myData = JSON.parse(data)
        self.profitPerDay = myData.toFixed(2);
      });

      //getting days till paid off
      fetch('http://localhost:8080/days')
      .then(response => response.json())
      .then(data => {
        let myData = JSON.parse(data);
        self.daysTillPaidOff = myData;
      });

      this.datePaidOff = 0;
      
    },
  
    reset: function() {
      this.clickedCalculated = false;
    },

    getEthereumPrice: function() {
      let self = this;
      fetch('http://localhost:8080/eth-price')
      .then(response => response.json())
      .then(data => {
        self.currPrice = data.price;
      });
    },

    getProfitability: function() {
      let self = this;
      fetch('http://localhost:8080/profitability')
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