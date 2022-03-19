<template>
  <div class="bg-gray-100">
    <div class="h-screen flex items-center justify-center">
      <div class="flex flex-col rounded-lg shadow-lg w-2/3 p-10 bg-white">
        <div class="mb-4">
          <div v-if="device" class="flex flex-col space-y-4">
            <div class="flex flex-col">
              <span class="text-xs">Connected to:</span>
              <span class="font-bold" v-text="device.productName" />
            </div>
            <div class="relative w-full appearance-none label-floating">
              <textarea
                v-model="zplCode"
                class="
                  autoexpand
                  tracking-wide
                  py-2
                  px-4
                  leading-relaxed
                  appearance-none
                  block
                  w-full
                  rounded
                  outline-none
                  bg-gray-100
                  border-gray-200
                "
                id="message"
                type="text"
                placeholder="ZPL II Code"
              ></textarea>
            </div>
          </div>
          <div v-else class="flex justify-center text-sm">
            <span>Please connect to a Zebra printer</span>
          </div>
        </div>
        <div class="flex space-x-2">
          <button
            v-if="device"
            class="bg-blue-600 text-gray-100 p-4 rounded flex-grow"
            :class="{
              'opacity-50 cursor-not-allowed': !zplCode,
              'hover:bg-blue-700': zplCode,
            }"
            @click="sendData"
            :disabled="!zplCode"
          >
            Send
          </button>
          <button
            v-else
            class="
              hover:bg-blue-700
              bg-blue-600
              text-gray-100
              p-4
              rounded
              flex-grow
            "
            @click="connectPrinter"
          >
            Connect
          </button>
          <button
            v-if="device"
            class="hover:bg-gray-300 bg-gray-200 text-gray-700 p-4 rounded"
            @click="disconnectPrinter"
          >
            Disconnect
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const VENDOR_ID = 2655;
const defaultZplCode = `^XA
^F020,20
^BC
^FD1234567890
^FS
^XZ`

export default {
  name: "IndexPage",
  data() {
    return {
      device: null,
      error: null,
      zplCode: defaultZplCode,
    };
  },
  methods: {
    async sendData() {
      if (this.device) {
        const encoder = new TextEncoder();
        const data = encoder.encode(this.zplCode);
        try {
          const res = await this.device.transferOut(1, data);
          console.log(res);
        } catch (e) {
          console.log("Device disconnected!");
          this.error = "Device disconnected";
        }
      } else {
        console.log("No device!");
        this.error = "No device found";
      }
    },
    async connectPrinter() {
      // Check if we have devices available
      let devices = await navigator.usb.getDevices();
      this.device = devices[0];
      if (devices.length === 0) {
        try {
          // Get permission from the user to use their printer
          this.device = await navigator.usb.requestDevice({
            filters: [{ vendorId: VENDOR_ID }],
          });
        } catch (e) {
          console.warn(e);
          this.error = e;
        }
      }
      if (this.device) {
        await this.device.open();
        await this.device.claimInterface(0);
      } else {
        console.log("No devices...");
        this.error = "No device found";
      }
    },
    async disconnectPrinter() {
      if (this.device) {
        await this.device.close();
        this.device = null;
      }
    },
  },
};
</script>
