<template>
  <div class="demo-page">

    <div class="demo-page__zh-cropper-wrapper" v-if="image.src !== null">
      <cropper
        class="demo-page__zh-cropper"
        ref="cropper"
        check-orientation
        :src="image.src"
        :transitions="true"
        :default-position="{ left: 0, top: 0 }"
        :default-size="{ width: 300, height: 300 }"
        @change="change"
        :minWidth="100"
        :maxWidth="9999"
        :minHeight="100"
        :maxHeight="9999"
      />

      <div class="demo-page__zh-cropper-buttons">
        <div class="demo-page__zh-cropper-button" @click="zoom(2)">
          <img :src="require('src/assets/icons/zoom-in.svg')" alt="Zoom In" />
        </div>
        <div class="demo-page__zh-cropper-button" @click="zoom(0.5)">
          <img :src="require('src/assets/icons/zoom-out.svg')" alt="Zoom Out" />
        </div>
        <div class="demo-page__zh-cropper-button" @click="flip(true, false)">
          <img
            :src="require('src/assets/icons/flip-horizontal.svg')"
            alt="Flip Horizonal"
          />
        </div>
        <div class="demo-page__zh-cropper-button" @click="flip(false, true)">
          <img
            :src="require('src/assets/icons/flip-vertical.svg')"
            alt="Flip Vertical"
          />
        </div>
        <div class="demo-page__zh-cropper-button" @click="rotate(90)">
          <img
            :src="require('src/assets/icons/rotate-clockwise.svg')"
            alt="Rotate Right"
          />
        </div>
        <div class="demo-page__zh-cropper-button" @click="rotate(-90)">
          <img
            :src="require('src/assets/icons/rotate-counter-clockwise.svg')"
            alt="Rotate Left"
          />
        </div>
        <div class="demo-page__zh-cropper-button" @click="resize(2, 2)">
          <img
            :src="require('src/assets/icons/resize.svg')"
            alt="Resize (x2)"
          />
        </div>
        <div class="demo-page__zh-cropper-button" @click="resize(1, 2)">
          <img
            :src="require('src/assets/icons/resize-vertical.svg')"
            alt="Resize height (x2)"
          />
        </div>
        <div class="demo-page__zh-cropper-button" @click="resize(2, 1)">
          <img
            :src="require('src/assets/icons/resize-horizontal.svg')"
            alt="Resize width (x2)"
          />
        </div>
        <div class="demo-page__zh-cropper-button" @click="resize(0.5, 0.5)">
          <img
            :src="require('src/assets/icons/resize-reduce.svg')"
            alt="Resize (x1/2)"
          />
        </div>
        <div class="demo-page__zh-cropper-button" @click="maximize()">
          <img :src="require('src/assets/icons/resize-maximize.svg')" />
        </div>
        <div class="demo-page__zh-cropper-button" @click="center()">
          <img :src="require('src/assets/icons/center.svg')" alt="Center" />
        </div>

        <q-space />

        <div class="demo-page__zh-cropper-button" @click="uploadCroppedImage()">
          <img :src="require('src/assets/icons/play.svg')" alt="Upload" />
        </div>
        <div
          class="demo-page__zh-cropper-button"
          @click="downloadCroppedImage()"
        >
          <img :src="require('src/assets/icons/download.svg')" alt="Download" />
        </div>
        <div class="demo-page__zh-cropper-button" @click="resetImage()">
          <img :src="require('src/assets/icons/reset.svg')" alt="Reset" />
        </div>
      </div>
    </div>

    <div class="demo-page__load-button-wrapper">
      <button class="demo-page__load-button" @click="$refs.file.click()">
        <input
          type="file"
          ref="file"
          @change="loadImage($event)"
          accept="image/*"
        />
        Load image
      </button>
    </div>

  </div>
</template>

<script>
import { Cropper } from "vue-advanced-cropper";
import { saveAs } from "file-saver";
import "vue-advanced-cropper/dist/style.css";

export default {
  components: {
    Cropper,
  },
  data() {
    return {
      image: {
        src: null,
        type: null,
        loading: false,
      },
    };
  },
  computed: {
    shouldEnableSaveCurrentCrop() {
      return true;
    },
  },
  methods: {
    // Methods handling size changes of the background image
    zoom(ratio) {
      this.$refs.cropper.zoom(ratio);
    },
    flip(x, y) {
      this.$refs.cropper.flip(x, y);
    },
    rotate(angle) {
      this.$refs.cropper.rotate(angle);
    },
    // Methods handling position changes of the cropped area
    center() {
      this.$refs.cropper.setCoordinates(({ coordinates, imageSize }) => ({
        left: imageSize.width / 2 - coordinates.width / 2,
        top: imageSize.height / 2 - coordinates.height / 2,
      }));
    },
    // Methods handling size changes of the cropped area
    resize(width = 1, height = 1) {
      let startCoordinates;
      this.$refs.cropper.setCoordinates([
        ({ coordinates, imageSize }) => {
          startCoordinates = coordinates;
          return {
            width: coordinates.width * width,
            height: coordinates.height * height,
          };
        },
        ({ coordinates, imageSize }) => ({
          left:
            startCoordinates.left +
            (startCoordinates.width - coordinates.width) / 2,
          top:
            startCoordinates.top +
            (startCoordinates.height - coordinates.height) / 2,
        }),
      ]);
    },
    updateSize({ coordinates }) {
      this.size.width = Math.round(coordinates.width);
      this.size.height = Math.round(coordinates.height);
    },
    maximize() {
      const { cropper } = this.$refs;
      const center = {
        left: cropper.coordinates.left + cropper.coordinates.width / 2,
        top: cropper.coordinates.top + cropper.coordinates.height / 2,
      };
      cropper.setCoordinates([
        ({ coordinates, imageSize }) => ({
          width: imageSize.width,
          height: imageSize.height,
        }),
        ({ coordinates, imageSize }) => ({
          left: center.left - coordinates.width / 2,
          top: center.top - coordinates.height / 2,
        }),
      ]);
    },

    // Methods handling background image changes
    loadImage(event) {
      // Reference to the DOM input element
      const { files } = event.target;
      // Ensure that you have a file before attempting to read it
      if (files && files[0]) {
        // 1. Revoke the object URL, to allow the garbage collector to destroy the uploaded before file
        this.image.loading = true;
        if (this.image.src) {
          URL.revokeObjectURL(this.image.src);
        }
        // 2. Create the blob link to the file to optimize performance:
        const blob = URL.createObjectURL(files[0]);

        // 3. The steps below are designated to determine a file mime type to use it during the
        // getting of a cropped image from the canvas. You can replace it them by the following string,
        // but the type will be derived from the extension and it can lead to an incorrect result:
        //
        // this.image = {
        //    src: blob;
        //    type: files[0].type
        // }

        // Create a new FileReader to read this image binary data
        const reader = new FileReader();
        // Define a callback function to run, when FileReader finishes its job
        reader.onload = (e) => {
          // Note: arrow function used here, so that "this.image" refers to the image of Vue component
          this.image = {
            // Set the image source (it will look like blob:http://example.com/2c5270a5-18b5-406e-a4fb-07427f5e7b94)
            src: blob,
            // Determine the image type to preserve it during the extracting the image from canvas:
            type: files[0].type,
            // Set the image ready to use
            loading: false,
          };
        };
        // Start the reader job - read file as a data url (base64 format)
        reader.readAsArrayBuffer(files[0]);
      }
    },
    resetImage() {
      this.image = {
        src: null,
        type: null,
        loading: false,
      };
    },

    // Methods handling tranfers of the cropped area
    downloadCroppedImage() {
      const { canvas } = this.$refs.cropper.getResult();
      if (canvas) {
        canvas.toBlob((blob) => {
          saveAs(blob);
        }, this.image.type);
      }
    },
    uploadCroppedImage() {
      const { canvas } = this.$refs.cropper.getResult();
      if (canvas) {
        canvas.toBlob((blob) => {
          const form = new FormData();
          form.append("file", blob);

          // TODO: use the same function of FileUploader

          // You can use axios, superagent and other libraries instead here
          fetch("http://example.com/upload/", {
            method: "POST",
            body: form,
          });
          // Perhaps you should add the setting appropriate file format here
        }, "image/jpeg");
      }
    },

    crop() {
      const { canvas } = this.$refs.cropper.getResult();
      canvas.toBlob((blob) => {
        // Do something with blob: upload to a server, download and etc.
      }, this.image.type);
    },
    // Methods handling events emitted from the cropper
    change({ coordinates, canvas }) {
      console.log("Coordinates was changed", coordinates, canvas);
    },
    error() {
      console.log("There is error during image loading");
      this.image.loading = false;
    },
    ready() {
      console.log("Image is successfully loaded");
      this.image.loading = false;
    },
  },
  unmounted() {
    // Revoke the object URL, to allow the garbage collector to destroy the uploaded before file
    if (this.image.src) {
      URL.revokeObjectURL(this.image.src);
    }
  },
};
</script>

<style lang="scss">
.demo-page {
  margin-top: 20px;
  margin-bottom: 20px;
  user-select: none;
  &__zh-cropper {
    border: solid 1px #eee;
    min-height: 400px;
    max-height: 85vh;
    width: 100%;
  }
  &__zh-cropper-wrapper {
    position: relative;
    margin: 0 auto;
    width: 40%;
    background: black;
    padding: 0.5%;
  }
  &__zh-cropper-buttons {
    margin: 0 auto;
    width: 100%;
    background: black;
    margin-top: 15px;
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    align-content: flex-start;
    justify-items: flex-start;
    justify-content: flex-start;
    align-content: flex-start;
    align-self: center;
    justify-self: center;
  }
  &__zh-cropper-button {
    background: rgba(black, 0.4);
    display: inline-flex;
    align-items: center;
    justify-content: center;
    height: 42px;
    width: 42px;
    cursor: pointer;
    transition: background 0.5s;

    &:hover {
      border-width: 1px;
      border-bottom-color: white;
      border-top-color: white;
      border-style: solid;
    }
  }
  &__load-button-wrapper {
    padding-top: 20px;
    margin-top: 20px;
    position: relative;
    margin: 0 auto;
    width: 40%;
    display: flex;
  }
  &__load-button {
    color: black;
    background-color: white;
    padding: 1%;
    border: black 1px solid;
    border-radius: 2px;
    width: 100%;
  }
}
</style>
