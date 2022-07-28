<template>
  <div >
<v-row justify="center" align="center">
  <v-col cols="12" sm="12" md="6">
  <video id="video" playsinline autoplay style="width: 1px;"></video>
  <v-btn class="btn btn-primary mb-2" id="cambiar-camara" @click="cambiarCamara()">Cambiar camara</v-btn>
  <canvas id="canvas" width="400" height="400" style="max-width: 100%;"></canvas>
  <canvas id="otrocanvas" width="150" height="150" ></canvas>
    <p>{{ pediction }} - {{ percentage }}</p>
  </v-col>
</v-row>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'
export default {
  name: 'IndexPage',
  data () {
    return {
      facingMode: 'user',
      tamano: 400,
      currentStream: undefined,
      modelo: null,
      video: undefined,
      canvas: undefined,
      otrocanvas: undefined,
      ctx: undefined,
      pediction: '',
      percentage: 0
    }
  },
  mounted () {
    this.initModel()
    this.mostrarCamara()
  },
  methods: {
    async initModel () {
      console.log("Cargando modelo...");
      this.modelo = await tf.loadLayersModel("/model.json");
      console.log("Modelo cargado");
    },
    procesarCamara() {
      this.ctx.drawImage(this.video, 0, 0,400, 400, 0, 0, 400, 400);
      setTimeout(this.procesarCamara, 20);
    },
    mostrarCamara() {
      console.log('document.querySelector(".video")', document.querySelector("#video"))
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
            console.log(localMediaStream);
            this.currentStream = localMediaStream
            this.video.srcObject = localMediaStream;
            this.procesarCamara()
            this.video.play();
            this.predecir()
          })
          .catch(function(err) {
            alert("No se pudo utilizar la camara :(");
            console.log(err);
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
          if (arr100.length == 100) {
            arr.push(arr100);
            arr100 = [];
          }
        }

        arr = [arr];

        const tensor = tf.tensor4d(arr);
        const resultado = this.modelo.predict(tensor).dataSync();
        if (resultado <= .3) {
          this.pediction = "Gato";
        } else if (resultado >= .7) {
          this.pediction = "Perro";
        } else {
          this.pediction = "Indefinido";
        }
      this.percentage = resultado
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
      console.log('resample', canvas)
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
