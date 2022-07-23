<script>
export default {
  props: ["dialog", "product"],
  middleware: "auth",
  data() {
    return {
      dialogController: false,
      file: null,
      disableUpload: false,
      imgContainerKey: -1000,
      mainImage: "",
      images: [],
      key: 0,
      rules: [
        (value) =>
          !value ||
          value.size < 2000000 ||
          "the image should be less than 2mbs",
      ],
    };
  },
  watch: {
    dialog: {
      immediate: true,
      handler() {
        this.dialogController = this.dialog;
        this.images = [...this.product.Image];

        // findind the main image if it exists
        const m = this.product.Image.filter((img) => img.preview);
        if (m.length > 0) {
          this.mainImage = m[0].id;
        }
      },
    },
  },

  computed: {},
  methods: {
    handleCancel() {
      this.$emit("cancel");
    },
    async handleYes() {
      const mainImgConfig = { id: this.mainImage, product_id: this.product.id };

      if (this.mainImage !== "")
        await this.$axios.put("/api/images/main", mainImgConfig);
      this.$emit("close");
    },
    fileChangeHandler() {},
    async uploadImg() {
      this.disableUpload = true;
      try {
        const uploadConfig = await this.$axios.get(`/api/images/url/`);
        const { url, key } = uploadConfig.data;
        const uploader = this.$axios.create();
        delete uploader.defaults.headers.common.Authorization;

        await uploader.put(url, this.file, {
          headers: {
            "Content-Type": "image/jpeg",
          },
          onProgress: (progressEvent) => {
            console.log((progressEvent.loaded / progressEvent.total) * 100);
          },
        });
        const newImgUrl = url.split("?")[0];
        const newImage = {
          key: key,
          url: newImgUrl,
          product_id: this.product.id,
          preview: "string",
        };
        const { data: uploadedImage } = await this.$axios.post(
          "/api/images/",
          newImage
        );
        this.images.push(uploadedImage);
      } catch (err) {
        console.log(err);
      }
      this.disableUpload = false;
      this.file = null;
    },
    async deleteImg(img) {
      try {
        console.log(img.id);
        await this.$axios.delete(`/api/images/${img.id}`);
        this.images = this.images.filter((i) => i.id !== img.id);
        if (this.mainImage === img.id) {
          this.mainImage = "";
        }
      } catch (err) {
        console.log(err);
      }
    },
    setMainImageLocaly(id) {
      this.mainImage = id;
      console.log(id);
    },
  },
};
</script>


<template>
  <v-row justify="center">
    <v-dialog v-model="dialogController" fullscreen persistent max-width="60vw">
      <v-card>
        <v-card-title class="text-h5">‌Manage Product Photos</v-card-title>

        <v-file-input
          :rules="rules"
          v-model="file"
          :disabled="disableUpload"
          :loading="disableUpload"
          class="ma-6"
          accept="image/png, image/jpeg, image/bmp"
          placeholder="select an image"
          prepend-icon="mdi-camera"
          max-width="50vw"
          label="product photo"
          @change="fileChangeHandler($event)"
        ></v-file-input>

        <v-container :key="imgContainerKey" justify="center">
          <v-row dense>
            <v-col v-for="img in images" :key="img.id" :cols="3">
              <v-card
                @click="setMainImageLocaly(img.id)"
                :color="(mainImage===img.id)? 'green' : 'dark'"
              >
                <v-img
                  :src="img.url"
                  class="white--text align-end"
                  gradient="to bottom, rgba(0,0,0,.1), rgba(0,0,0,.5)"
                  height="200px"
                ></v-img>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="red" icon @click.stop="deleteImg(img)">
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </v-row>
        </v-container>

        <v-card-actions>
          <v-btn
            :disabled="!file || disableUpload"
            :loading="disableUpload"
            color="green darken-1"
            @click="uploadImg"
          >
            <v-icon>mdi-upload</v-icon>
          </v-btn>
          <v-btn color="green darken-1" text :disabled="disableUpload" @click="handleYes">Done</v-btn>
          <v-btn color="green darken-1" text :disabled="disableUpload" @click="handleCancel">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>
<style>
</style>
