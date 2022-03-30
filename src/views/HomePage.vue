<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Bluetooth Test</ion-title>
        <ion-buttons slot="end">
          <ion-button v-if="!state.scanning" @click="scan">Scan</ion-button>
          <ion-button v-if="state.scanning" @click="stop">Stop</ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <div id="container">
        <ion-list>
          <ion-item button v-for="device in state.devices" v-bind:key="device.deviceId" @click="connect(device)">
            <ion-label>{{device.deviceId}} {{device.name}}</ion-label>
            <ion-spinner v-if="device.state===1"/>
            <span v-if="device.state===2">✓</span>
          </ion-item>
        </ion-list>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonButtons, IonList, IonItem, IonLabel, alertController, IonSpinner } from '@ionic/vue';
import { defineComponent, reactive } from 'vue';
import { BleClient, BleDevice } from '@capacitor-community/bluetooth-le';

enum DeviceState {
  unconnected = 0,
  connecting = 1,
  connected = 2
}

interface MyBleDevice extends BleDevice {
  state: DeviceState
}

interface State {
  scanning:boolean
  devices: { [deviceId:string]: MyBleDevice }
}

export default defineComponent({
  name: 'HomePage',
  setup() {

    let state = reactive<State>({scanning:false, devices:{}})
    
    return {
      state
    }
  },
  async created() {
    await BleClient.initialize({ androidNeverForLocation: true });
  },
  methods: {
    async scan() {

      await BleClient.requestLEScan({services: [],},
        (result) => {
          this.state.devices[result.device.deviceId] = {state:DeviceState.unconnected, ...result.device}
        }
      );
      this.state.devices = {}
      this.state.scanning = true

      setTimeout(this.stop, 5000);

    },
    async stop() {
      await BleClient.stopLEScan()
      this.state.scanning = false
    },
    async connect(device:BleDevice) {
      try {
        this.state.devices[device.deviceId].state = DeviceState.connecting
        await BleClient.connect(device.deviceId, this.onDisconnected)
        this.state.devices[device.deviceId].state = DeviceState.connected
        this.showSuccess(device)
      } catch( error ) {
        this.state.devices[device.deviceId].state = DeviceState.unconnected
        this.showError(error as Error)
      }
    },
    onDisconnected(deviceId:string) {
      this.state.devices[deviceId].state = DeviceState.unconnected
    },
    async read(device:BleDevice) {
      let service = ''
      let charateristic = ''
      let data = await BleClient.read(device.deviceId, service, charateristic)
      
    },
    async showSuccess(device:BleDevice) {
        const alert = await alertController
          .create({
            header: 'Connected',
            message: `Successfully conntected to device ${device.deviceId}`,
            buttons: ['OK'],
          });
        await alert.present();        
    },
    async showError(error:Error) {
        const alert = await alertController
          .create({
            header: 'Error',
            message: error.message,
            buttons: ['OK'],
          });
        await alert.present();        
    }

  },
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButton,
    IonButtons,
    IonList,
    IonItem,
    IonLabel,
    IonSpinner
  }
});
</script>

<style scoped>

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
