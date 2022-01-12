<template>
  <v-app id="inspire">
    <v-main>
      <v-row align="center" justify="center" class="mt-7 mb-7">
        <!-- <v-img src="@/assets/classroom.jpg" height="800"> </v-img> -->
        <v-card elevation="2" outlined overlay img="@/assets/classroom.jpg">
          <v-card-title class="justify-center">
            <h1>Classroom Occupation</h1>
          </v-card-title>
          <v-card-subtitle class="text-center mt-1"
            >This is a means of showing if the classroom is occupied or has free
            seats</v-card-subtitle
          >
          <v-divider class="ml-16 mr-16"></v-divider>
          <v-card-actions>
            <v-row class="mx-1 my-1">
              <v-col>
                <v-img
                  :src="chairs.chair1"
                  class="mx-1 my-2"
                  contain
                  height="200"
                ></v-img>
                <v-img
                  :src="chairs.chair2"
                  class="mx-1 my-2"
                  contain
                  height="200"
                ></v-img>
                <v-img
                  :src="chairs.chair3"
                  class="mx-1 my-2"
                  contain
                  height="200"
                ></v-img>
              </v-col>
              <v-col>
                <v-img
                  :src="chairs.chair4"
                  class="mx-1 my-2"
                  contain
                  height="200"
                ></v-img>
                <v-img
                  :src="chairs.chair5"
                  class="mx-1 my-2"
                  contain
                  height="200"
                ></v-img>
                <v-img
                  :src="chairs.chair6"
                  class="mx-1 my-2"
                  contain
                  height="200"
                ></v-img>
              </v-col>
            </v-row>
          </v-card-actions>
        </v-card>
      </v-row>
    </v-main>
  </v-app>
</template>

<script>
import mqtt from "mqtt";
export default {
  data: () => ({
    chairs: {
      chair1: require("@/assets/green_chair.jpg"),
      chair2: require("@/assets/green_chair.jpg"),
      chair3: require("@/assets/green_chair.jpg"),
      chair4: require("@/assets/green_chair.jpg"),
      chair5: require("@/assets/green_chair.jpg"),
      chair6: require("@/assets/green_chair.jpg"),
    },
    greenCount:0,

    connection: {
      host: "public.mqtthq.com",
      port: 8083,
      endpoint: "/mqtt",
      clean: true, // Reserved session
      connectTimeout: 4000, // Time out
      reconnectPeriod: 4000, // Reconnection interval
      // Certification Information
    },
    subscription: {
      topic: "SensorTile",
      qos: 0,
    },
    publish: {
      topic: "topic/browser",
      qos: 0,
      payload: '{ "msg": "Hello, I am browser." }',
    },
    receiveNews: "",
    qosList: [  
      { label: 0, value: 0 },
      { label: 1, value: 1 },
      { label: 2, value: 2 },
    ],
    client: {
      connected: false,
    },
    subscribeSuccess: false,
  }),
  created() {
    this.createConnection();
    this.doSubscribe();
  },
  watch: {
    receiveNews() {
      if (this.receiveNews.includes("Occupied")) {
        let ch = this.receiveNews.substring(0, 6);
        this.chairs[ch] = require("@/assets/red_chair.jpg");
      } else if (this.receiveNews.includes("Empty")) {
        let ch = this.receiveNews.substring(0, 6);
        this.chairs[ch] = require("@/assets/green_chair.jpg");
      }
      this.receiveNews = "";

    }
  },

  methods: {
    // Create connection
    createConnection() {
      // Connect string, and specify the connection method used through protocol
      // ws unencrypted WebSocket connection
      // wss encrypted WebSocket connection
      // mqtt unencrypted TCP connection
      // mqtts encrypted TCP connection
      // wxs WeChat mini app connection
      // alis Alipay mini app connection
      const { host, port, endpoint, ...options } = this.connection;
      const connectUrl = `ws://${host}:${port}${endpoint}`;
      try {
        this.client = mqtt.connect(connectUrl, options);
      } catch (error) {
        console.log("mqtt.connect error", error);
      }
      this.client.on("connect", () => {
        console.log("Connection succeeded!");
      });
      this.client.on("error", (error) => {
        console.log("Connection failed", error);
      });
      this.client.on("message", (topic, message) => {
        this.receiveNews = this.receiveNews.concat(message);
        console.log(`Received message ${message} from topic ${topic}`);
      });
    },
    doSubscribe() {
      const { topic, qos } = this.subscription;
      this.client.subscribe(topic, { qos }, (error, res) => {
        if (error) {
          console.log("Subscribe to topics error", error);
          return;
        }
        this.subscribeSuccess = true;
        console.log("Subscribe to topics res", res);
      });
    },
  },
};
</script>
