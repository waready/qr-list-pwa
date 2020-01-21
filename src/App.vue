<template>
  <div>
    <p class="decode-result">Last result: <b>{{ result }}</b></p>
      <hr>
      <h1>sistema de recepcion de material electorial</h1>
      <h2> {{ result }} </h2>
      <input type="text" name="" v-model="result">
      <button v-if="result != null" @click="buscar()"> Buscar</button>
        <ul v-if="ver">
          <li v-for="item in FiltroDato" :key="item.id">
            {{item.nombre}} - <span>{{item.Anforas}}</span> - (<span>{{item.Mesas}}</span>) - 
        
              <span v-if="item.giganto == true">
                [si]
              </span >
              <span v-else>
                [no]
              </span>

          </li>
        </ul>
  <hr>

    <qrcode-stream :camera="camera" @decode="onDecode" @init="onInit">
      <div v-if="validationSuccess" class="validation-success">
        This is a URL
      </div>

      <div v-if="validationFailure" class="validation-failure">
        This is NOT a URL!
      </div>

      <div v-if="validationPending" class="validation-pending">
        Long validation in progress...
      </div>
    </qrcode-stream>
  </div>
</template>

<script>
import { QrcodeStream } from 'vue-qrcode-reader'

export default {

  components: { QrcodeStream },

  data () {
    return {
      isValid: undefined,
      camera: 'auto',
      result: null,
      ver:false,
      centroCopio:[
        {id:'130927',nombre:"Glorioso San Carlos", Anforas:15, Mesas:10, giganto:true, obser:""},
        {id:'140927',nombre:"Maria Auxiliadora", Anforas:25, Mesas:15, giganto:true, obser:""},
        {id:'150927',nombre:"Gran Unidad Escolar San Carlos", Anforas:20, Mesas:15, giganto:true, obser:""},
        {id:'160927',nombre:"IEP 32", Anforas:10, Mesas:5, giganto:true, obser:""},
        {id:'170927',nombre:"70011 Mañanazo", Anforas:5, Mesas:1, giganto:false, obser:""},
      ]
    }
  },

  computed: {
    validationPending () {
      return this.isValid === undefined
        && this.camera === 'off'
    },

    validationSuccess () {
      return this.isValid === true
    },

    validationFailure () {
      return this.isValid === false
    },
    FiltroDato(){
      return this.centroCopio.filter((item)=> item.id.includes(this.result));
    }
  },

  methods: {

    onInit (promise) {
      promise
        .catch(console.error)
        .then(this.resetValidationState)
    },

    resetValidationState () {
      this.isValid = undefined
    },

    async onDecode (content) {
      this.result = content
      this.turnCameraOff()

      // pretend it's taking really long
      await this.timeout(3000)
      this.isValid = content.startsWith('http')

      // some more delay, so users have time to read the message
      await this.timeout(2000)

      this.turnCameraOn()
    },

    turnCameraOn () {
      this.camera = 'auto'
    },

    turnCameraOff () {
      this.camera = 'off'
    },

    timeout (ms) {
      return new Promise(resolve => {
        window.setTimeout(resolve, ms)
      })
    },
    buscar(){
      
      this.ver = true
    }
  }
}
</script>

<style scoped>
.validation-success,
.validation-failure,
.validation-pending {
  position: absolute;
  width: 100%;
  height: 100%;

  background-color: rgba(255, 255, 255, .8);
  text-align: center;
  font-weight: bold;
  font-size: 1.4rem;
  padding: 10px;

  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
}
.validation-success {
  color: green;
}
.validation-failure {
  color: red;
}
</style>