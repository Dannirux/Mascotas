<template>
  <div >
<v-row>
  <v-col cols="12" sm="12" md="6">
  <v-card flat outlined>
    <v-card-text>
      <video id="video" playsinline autoplay style="width: 1px;"></video>
      <v-btn class="primary mb-2" block id="cambiar-camara" @click="cambiarCamara()"><v-icon>mdi-camera-party-mode</v-icon>Cambiar camara</v-btn>
      <canvas id="canvas" width="400" height="400" style="max-width: 100%;"></canvas>
      <canvas v-show="false" id="otrocanvas" width="150" height="150" ></canvas>
      <p class="text-h5 text-center black--text">
        CNN AD - {{ pedictionCnnAd }}
      </p>
    </v-card-text>
  </v-card>
  </v-col>
  <v-col cols="12" sm="12" md="2">
    <v-card flat outlined>
      <v-card-title>Denso</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pediction }}
        </p>
      </v-card-text>
    </v-card>
    <v-card v-if="!$vuetify.breakpoint.smAndDown" flat outlined class="mt-2">
      <v-card-title>CNN</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pedictionCnn }}
        </p>
      </v-card-text>
    </v-card>
    <v-card flat outlined class="mt-2">
      <v-card-title>CNN_Dropout</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pedictionCnnDrop }}
        </p>
      </v-card-text>
    </v-card>
  </v-col>
  <v-col cols="12" xs="12" md="2">
    <v-card v-if="!$vuetify.breakpoint.smAndDown" flat outlined>
      <v-card-title>Denso_AD</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pedictionDensoAd }}
        </p>
      </v-card-text>
    </v-card>
    <v-card flat outlined class="mt-2">
      <v-card-title>CNN_AD</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pedictionCnnAd }}
        </p>
      </v-card-text>
    </v-card>
    <v-card flat outlined class="mt-2">
      <v-card-title>CNN_Dropout_AD</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pedictionCnnDropAd }}
        </p>
      </v-card-text>
    </v-card>
  </v-col>
  <v-col cols="12" xs="12" md="2">
    <v-card v-if="!$vuetify.breakpoint.smAndDown" flat outlined>
      <v-card-title>AlexNet</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pedictionAlex }}
        </p>
      </v-card-text>
    </v-card>
    <v-card flat outlined class="mt-2">
      <v-card-title>AlexNet_AD</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pedictionAlexAd }}
        </p>
      </v-card-text>
    </v-card>
    <!-- <v-card flat outlined class="mt-2">
      <v-card-title>AlexNet_Dropout_AD</v-card-title>
      <v-card-text>
        <p class="text-h3 text-center black--text">
          {{ pedictionAlexDropAd }}
        </p>
      </v-card-text>
    </v-card> -->
  </v-col>
</v-row>
    <v-overlay :value="overlay">
      <v-progress-circular
        indeterminate
        size="64"
      ></v-progress-circular>
    </v-overlay>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'
export default {
  name: 'IndexPage',
  data () {
    return {
      overlay: false,
      value: [
        1000,
        213,
      ],
      // prediccion
      facingMode: 'user',
      tamano: 400,
      currentStream: undefined,
      video: undefined,
      canvas: undefined,
      otrocanvas: undefined,
      ctx: undefined,
      pediction: '-',
      pedictionCnn: '-',
      pedictionCnnDrop: '-',
      pedictionDensoAd: '-',
      pedictionCnnAd: '-',
      pedictionCnnDropAd: '-',
      pedictionAlex: '-',
      pedictionAlexAd: '-',
      pedictionAlexDropAd: '-',
      percentage: 0,
      modelo: null,
      modeloCnn: null,
      modeloCnnDropOut: null,
      modeloDensoAd: null,
      modeloCnnAd: null,
      modeloCnnDropOutAd: null,
      modeloAlex: null,
      modeloAlexAD: null,
      modeloAlexDropOutAd: null,
    }
  },
  async mounted () {
    await this.initModel()
    this.mostrarCamara()
  },
  methods: {
    async initModel () {
      console.log("Cargando modelo...");
      this.overlay = true
      this.modelo = await tf.loadLayersModel("/denso/model.json");
      if (!this.$vuetify.breakpoint.smAndDown) {
        this.modeloCnnDropOut = await tf.loadLayersModel("/cnn2/model.json");
        this.modeloCnn = await tf.loadLayersModel("/cnn/model.json");
        this.modeloDensoAd = await tf.loadLayersModel("/densoad/model.json");
      }
      this.modeloCnnAd = await tf.loadLayersModel("/cnnad/model.json");
      this.modeloCnnDropOutAd = await tf.loadLayersModel("/cnn2ad/model.json");
      this.modeloAlex = await tf.loadLayersModel("/AlexNetCNN/model.json");
      this.modeloAlexAD = await tf.loadLayersModel("/AlexNetCNNAD/model.json");
      //this.modeloAlexDropOutAd = await tf.loadLayersModel("/AlexNetCNNDropAD/model.json");
      //this.modeloCnnDropOutAd = await tf.loadLayersModel("/cnn2ad/model.json");
      this.overlay = false
      console.log("Modelo cargado");
    },
    procesarCamara() {
      this.ctx.drawImage(this.video, 0, 0,400, 400, 0, 0, 400, 400);
      setTimeout(this.procesarCamara, 20);
    },
    mostrarCamara() {
      this.video = document.querySelector("#video")
      this.canvas = document.querySelector("#canvas")
      this.otrocanvas = document.querySelector("#otrocanvas")
      this.ctx = this.canvas.getContext("2d")
      const opciones = {
        audio: false,
        video: {
          width: 400, height: 400
        }
      }

      if (navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia(opciones)
          .then(localMediaStream => {
            this.currentStream = localMediaStream
            this.video.srcObject = localMediaStream;
            this.procesarCamara()
            this.video.play();
            this.predecir()
          })
          .catch(function(err) {
            alert("No se pudo utilizar la camara :(");
            alert(err);
          })
      } else {
        alert("No existe la funcion getUserMedia");
      }
    },
    predecir() {
      if (this.modelo != null) {
        this.resamplesingle(this.canvas, 100, 100, this.otrocanvas);
      }
        //Hacer la predicción
        const ctx2 = this.otrocanvas.getContext("2d");
        const imgData = ctx2.getImageData(0,0, 100, 100);

        let arr = [];
        let arr100 = [];

        for (let p=0; p < imgData.data.length; p+= 4) {
          const rojo = imgData.data[p] / 255;
          const verde = imgData.data[p+1] / 255;
          const azul = imgData.data[p+2] / 255;

          const gris = (rojo+verde+azul)/3;

          arr100.push([gris]);
          if (arr100.length === 100) {
            arr.push(arr100);
            arr100 = [];
          }
        }

      arr = [arr];

      const tensor = tf.tensor4d(arr);
      const resultDenso = this.modelo.predict(tensor).dataSync();
      resultDenso <= .5 ? this.pediction = "Gato" : this.pediction = "Perro"
      if (!this.$vuetify.breakpoint.smAndDown) {
        const resultCnnDrop = this.modeloCnnDropOut.predict(tensor).dataSync();
        resultCnnDrop <= .5 ? this.pedictionCnnDrop = "Gato" : this.pedictionCnnDrop = "Perro"
        const resultDensoAd = this.modeloDensoAd.predict(tensor).dataSync();
        resultDensoAd <= .5 ? this.pedictionDensoAd = "Gato" : this.pedictionDensoAd = "Perro"
        const resultCnn = this.modeloCnn.predict(tensor).dataSync();
        resultCnn <= .5 ? this.pedictionCnn = "Gato" : this.pedictionCnn = "Perro"
      }
      const resultCnnAd = this.modeloCnnAd.predict(tensor).dataSync();
      resultCnnAd <= .5 ? this.pedictionCnnAd = "Gato" : this.pedictionCnnAd = "Perro"
      const resultCnnDropAd = this.modeloCnnDropOutAd.predict(tensor).dataSync();
      resultCnnDropAd <= .5 ? this.pedictionCnnDropAd = "Gato" : this.pedictionCnnDropAd = "Perro"
      const resultAlex = this.modeloAlex.predict(tensor).dataSync();
      resultAlex <= .5 ? this.pedictionAlex = "Gato" : this.pedictionAlex = "Perro"
      const resultAlexAD = this.modeloAlexAD.predict(tensor).dataSync();
      resultAlexAD <= .5 ? this.pedictionAlexAd = "Gato" : this.pedictionAlexAd = "Perro"
      // const resultAlexDropAD = this.modeloAlexDropOutAd.predict(tensor).dataSync();
      // resultAlexDropAD <= .5 ? this.pedictionAlexDropAd = "Gato" : this.pedictionAlexDropAd = "Perro"

      //this.percentage = resultado
      setTimeout(this.predecir, 150)
    },
    cambiarCamara() {
      if (this.currentStream) {
        this.currentStream.getTracks().forEach(track => {
          track.stop();
        });
      }
      this.facingMode = this.facingMode === "user" ? "environment" : "user";
      const opciones = {
        audio: false,
        video: {
          facingMode: this.facingMode, width: this.tamano, height: this.tamano
        }
      };
      navigator.mediaDevices.getUserMedia(opciones)
        .then(stream => {
          this.currentStream = stream;
          this.video.srcObject = stream;
        })
        .catch(function(err) {
          console.log("Oops, hubo un error", err);
        })
    },
    resamplesingle(canvas, width, height, resize_canvas) {
      const width_source = canvas.width;
      const height_source = canvas.height;
      width = Math.round(width);
      height = Math.round(height);
      const ratio_w = width_source / width;
      const ratio_h = height_source / height;
      const ratio_w_half = Math.ceil(ratio_w / 2);
      const ratio_h_half = Math.ceil(ratio_h / 2);

      const ctx = canvas.getContext("2d");
      const ctx2 = resize_canvas.getContext("2d");
      const img = ctx.getImageData(0, 0, width_source, height_source);
      const img2 = ctx2.createImageData(width, height);
      const data = img.data;
      const data2 = img2.data;

      for (let j = 0; j < height; j++) {
        for (let i = 0; i < width; i++) {
          const x2 = (i + j * width) * 4;
          let weight = 0;
          let weights = 0;
          let weights_alpha = 0;
          let gx_r = 0;
          let gx_g = 0;
          let gx_b = 0;
          let gx_a = 0;
          let center_y = (j + 0.5) * ratio_h;
          let yy_start = Math.floor(j * ratio_h);
          let yy_stop = Math.ceil((j + 1) * ratio_h);
          for (let yy = yy_start; yy < yy_stop; yy++) {
            let dy = Math.abs(center_y - (yy + 0.5)) / ratio_h_half;
            let center_x = (i + 0.5) * ratio_w;
            let w0 = dy * dy; //pre-calc part of w
            let xx_start = Math.floor(i * ratio_w);
            let xx_stop = Math.ceil((i + 1) * ratio_w);
            for (let xx = xx_start; xx < xx_stop; xx++) {
              let dx = Math.abs(center_x - (xx + 0.5)) / ratio_w_half;
              let w = Math.sqrt(w0 + dx * dx);
              if (w >= 1) {
                //pixel too far
                continue;
              }
              //hermite filter
              weight = 2 * w * w * w - 3 * w * w + 1;
              let pos_x = 4 * (xx + yy * width_source);
              //alpha
              gx_a += weight * data[pos_x + 3];
              weights_alpha += weight
              if (data[pos_x + 3] < 255)
                weight = weight * data[pos_x + 3] / 250;
              gx_r += weight * data[pos_x];
              gx_g += weight * data[pos_x + 1];
              gx_b += weight * data[pos_x + 2];
              weights += weight;
            }
          }
          data2[x2] = gx_r / weights;
          data2[x2 + 1] = gx_g / weights;
          data2[x2 + 2] = gx_b / weights;
          data2[x2 + 3] = gx_a / weights_alpha;
        }
      }
      ctx2.putImageData(img2, 0, 0);
    }
  },
}
</script>
