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
          <select id="GPU-Name" name="test">
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
      </div>
      <div class="buttons"> 
        <button type="button" class="btn btn-success" id="calculate" @click="calculate" :disabled=!compute>Calculate</button>
      </div>
      <template v-if="clickedCalculated && compute">
        <div class="buttons">
          <label class="eth-price">Calculated with 1 ETH = <span class="bold">${{currPrice}}</span> and profitablility per 100Mhs = <span class="bold">${{currProfitability}}</span></label>
        </div>
        <br>
        <div class="grid"> 
          <div> 
            <label>You currently Own:&nbsp;</label>
            <span>{{currentlyOwn}}</span> 
          </div>
          <div> 
            <label>Your total system price was: </label>
            <span>${{systemPrice}}</span> 
          </div>
          <div> 
            <label>You need to mine:  </label>
            <span>{{amountToMine}}</span> 
          </div>
          <div> 
            <label>Your current hashrate is: </label>
            <span>{{currentHashrate}}</span> 
          </div>
          <div> 
            <label>Your estimated Power Consumtion is: </label>
            <span>${{powerConsumption}}</span> 
          </div>
          <div> 
            <label>Profit per day is estimated at ${{profitPerDay}}.</label>
          </div>
          <div> 
            <label>At current prices your Rig will be paid of in {{daysTillPaidOff}} days on {{datePaidOff}}</label>
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
      currentlyMined: null,
      rigPrice: null,
      taxBracket: null,
      powerRate: null,
      currentlyOwn: null,
      systemPrice: null,
      amountToMine: null,
      currentHashrate: null,
      powerConsumption: null,
      profitPerDay: null,
      daysTillPaidOff: null,
      datePaidOff: null,
      gpus: [1,2,3,4],
      gpuQuantity: null,
      clickedCalculated: false,
      currPrice: null,
      currProfitability: null
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
  },

  methods: {
    saveSession: function() {

    },

    loadSession: function() {

    },

    displayGPUs: function() {

    },

    addGPU: function() {
      //everytime added or removed update gpus for multiselect

    },

    removeGPU: function() {

    },

    calculate: function() {
      this.clickedCalculated = true;
      this.getEthereumPrice();
      this.getProfitability();
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
</style>