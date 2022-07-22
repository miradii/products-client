<script>
import { required, min, max } from "vee-validate/dist/rules";
import {
  extend,
  ValidationObserver,
  ValidationProvider,
  setInteractionMode,
} from "vee-validate";
import FormDialog from "~/components/form-dialog.vue";
import ImageDialog from "~/components/image-dialog.vue";

import { mapMutations } from "vuex";
extend("max", {
  ...max,
  message: "{_field_} may not be more than {length} characters",
});

extend("min", {
  ...min,
  message: "{_field_} must be at least {length} characters",
});

extend("required", {
  ...required,
  message: "{_field_} can not be empty",
});

export default {
  layout: "main",
  middleware: "auth",
  components: { ValidationObserver, ValidationProvider, FormDialog },
  data() {
    return {
      categories: [],
      products: [],
      overlay: false,
      openAddForm: false,
      openDeleteDialog: false,
      openEditForm: false,
      openImageDialog: false,
      currentProduct: {
        id: "",
        name: "",
        description: "",
        price: "",
        category_id: "",
        Image: [],
      },
    };
  },
  async mounted() {
    const { data } = await this.$axios.get("/api/categories");
    this.categories.push(...data);
    const res = await this.$axios.get("/api/products");
    this.products.push(...res.data);
  },
  methods: {
    async handleAdd() {
      try {
        const { id, Image, ...newProduct } = { ...this.currentProduct };
        console.log(newProduct);

        const { data } = await this.$axios.post("/api/products", newProduct);
        this.products.push(data);
        this.closeForm();
        this.$store.commit("snackbar/showMessage", {
          content: "Product created",
          color: "success",
        });
      } catch (e) {
        this.$store.commit("snackbar/showMessage", {
          content: "Failed to create product",
          color: "error",
        });
        const errors = e.response.data.message;
        if (typeof errors == "string") {
          console.log(errors);
        } else if (errors !== undefined) {
          errors.forEach((error) => {
            if (error.includes("name")) {
              this.$refs.name.setErrors(error);
            } else if (error.includes("price")) {
              this.$refs.price.setErrors(error);
            } else if (error.includes("description")) {
              this.$refs.description.setErrors(error);
            }
          });
        }
      }
    },
    async handleDelete() {
      try {
        console.log(this.currentProduct);
        const { data } = await this.$axios.delete(
          `/api/products/${this.currentProduct.id}`
        );
        this.products = this.products.filter(
          (product) => product.id != data.id
        );

        this.closeForm();
        this.$store.commit("snackbar/showMessage", {
          content: "Product deleted",
          color: "success",
        });
      } catch (e) {
        this.closeForm();
        this.$store.commit("snackbar/showMessage", {
          content: "Unable to delete product",
          color: "info",
        });
      }
    },
    async handleEdit() {
      try {
        const { id, Image, ...editedProduct } = { ...this.currentProduct };
        const { data } = await this.$axios.patch(
          `/api/products/${this.currentProduct.id}`,
          editedProduct
        );
        let p = this.products.find(
          (prod) => prod.id === this.currentProduct.id
        );
        p.name = data.name;
        p.price = data.price;
        p.description = data.description;
        p.Image = data.Image;
        p.category_id = data.category_id;
        this.closeForm();
        this.$store.commit("snackbar/showMessage", {
          content: "Product Updated",
          color: "success",
        });
      } catch (e) {
        this.$store.commit("snackbar/showMessage", {
          content: "failed to update product",
          color: "error",
        });
        const errors = e.response.data.message;
        if (typeof errors == "string") {
          console.log(errors);
        } else if (errors !== undefined) {
          errors.forEach((error) => {
            if (error.includes("name")) {
              this.$refs.name.setErrors(error);
            }
          });
        }
      }
    },
    cancelForm() {
      this.openAddForm = false;
      this.openDeleteDialog = false;
      this.openEditForm = false;
      this.openImageDialog = false;
      this.currentProduct = {
        id: "",
        name: "",
        description: "",
        price: "",
        category_id: "",
        Image: [],
      };
    },
    closeForm() {
      this.openAddForm = false;
      this.openDeleteDialog = false;
      this.openEditForm = false;
      this.openImageDialog = false;
      this.currentProduct = {
        id: "",
        name: "",
        description: "",
        price: "",
        category_id: "",
        Image: [],
      };

      window.location.reload();
    },
    openDelete(product) {
      this.currentProduct = { ...product };
      this.openDeleteDialog = true;
    },
    openEdit(product) {
      const { Image, ...p } = { ...product };
      this.currentProduct = { ...p };
      this.openEditForm = true;
    },
    openImage(product) {
      this.currentProduct = { ...product };
      this.openImageDialog = true;
    },
    handleImgDialogClose() {
      this.openImageDialog = false;
      this.currentProduct = {
        id: "",
        name: "",
        description: "",
        price: "",
        category_id: "",
        Image: [],
      };
      window.location.reload();
    },
    getMainImage(product) {
      const mar = product.Image.filter((i) => i.preview);
      if (mar.length > 0) {
        return mar[0].url;
      }
      return "";
    },
  },
};
</script>


<template>
  <v-container>
    <v-btn @click.prevent="openAddForm = true" class="fab mx-5 mt-10" fab left top absolute>
      <v-icon>mdi-plus</v-icon>
    </v-btn>
    <v-row>
      <template
        v-for="category in categories.filter(cat => products.filter(p=>p.category_id == cat.id).length >0)"
      >
        <v-col :key="category.id" class="mt-2" cols="12">
          <strong>{{ category.name }}</strong>
        </v-col>
        <v-col
          v-for="product in products.filter(el=>el.category_id===category.id)"
          :key="`${product.id}-${category.id}`"
          cols="6"
          md="2"
        >
          <v-hover>
            <template v-slot:default="{ hover }">
              <v-card>
                <v-img
                  :src="getMainImage(product)"
                  class="white--text align-end"
                  gradient="to bottom, rgba(0,0,0,.1), rgba(0,0,0,.5)"
                  height="200px"
                >
                  <v-card-title v-text="product.name"></v-card-title>
                  <v-card-subtitle>{{ product.price }}&#x24;</v-card-subtitle>
                </v-img>

                <v-fade-transition>
                  <v-overlay v-if="hover" absolute color="#036358">
                    <v-card-text>{{ product.description }}</v-card-text>

                    <v-card-actions>
                      <v-spacer></v-spacer>

                      <v-btn @click="openEdit(product)" icon>
                        <v-icon>mdi-pencil</v-icon>
                      </v-btn>

                      <v-btn @click="openImage(product)" icon>
                        <v-icon>mdi-image-multiple</v-icon>
                      </v-btn>

                      <v-btn @click="openDelete(product)" icon>
                        <v-icon>mdi-delete</v-icon>
                      </v-btn>
                    </v-card-actions>
                  </v-overlay>
                </v-fade-transition>
              </v-card>
            </template>
          </v-hover>
        </v-col>
      </template>
    </v-row>

    <form-dialog :dialog="openAddForm || openEditForm">
      <v-card class="pa-4" width="100%">
        <v-subheader v-if="openAddForm">Add Product</v-subheader>
        <v-subheader v-else>Edit Product</v-subheader>
        <ValidationObserver ref="observer" v-slot="{ invalid }">
          <v-form ref="form">
            <validation-provider
              ref="name"
              v-slot="{ errors }"
              name="Name"
              vid="name"
              rules="required|min:5|max:40"
            >
              <v-text-field
                class="ma-2"
                v-model="currentProduct.name"
                :error-messages="errors"
                name="name"
                label="Product Name"
                type="text"
                required
                outlined
              ></v-text-field>
            </validation-provider>
            <v-select
              v-model="currentProduct.category_id"
              :items="categories"
              item-text="name"
              item-value="id"
              label="Category"
              outlined
            ></v-select>

            <validation-provider
              ref="price"
              v-slot="{ errors }"
              name="price"
              vid="price"
              rules="required"
            >
              <v-text-field
                class="ma-2"
                v-model.number="currentProduct.price"
                :error-messages="errors"
                name="price"
                prefix="$"
                label="Price"
                type="number"
                required
                outlined
              ></v-text-field>
            </validation-provider>
            <validation-provider
              ref="description"
              v-slot="{ errors }"
              name="Description"
              vid="description"
              rules="required|min:10|max:280"
            >
              <v-textarea
                class="ma-2"
                v-model="currentProduct.description"
                :error-messages="errors"
                name="description"
                label="Description"
                required
                outlined
              ></v-textarea>
            </validation-provider>
          </v-form>
          <v-card-actions>
            <v-btn
              v-if="openAddForm"
              :disabled="invalid"
              @click.prevent="handleAdd"
              large
              color="green accent-2"
              light
            >Add</v-btn>
            <v-btn
              v-if="openEditForm"
              :disabled="invalid"
              @click.prevent="handleEdit"
              large
              color="green accent-2"
              light
            >Edit</v-btn>
            <v-btn @click.prevent="cancelForm" color="red accent-3" large>Cancel</v-btn>
          </v-card-actions>
        </ValidationObserver>
      </v-card>
    </form-dialog>
    <form-dialog :dialog="openDeleteDialog">
      <v-card class="pa-4" width="100%">
        <v-subheader>Are you sure you want to delete {{ currentProduct.name }}</v-subheader>
        <v-card-actions>
          <v-btn @click.prevent="handleDelete" large color="green accent-2" light>Yes</v-btn>
          <v-btn @click.prevent="closeForm" color="red accent-3" large>No</v-btn>
        </v-card-actions>
      </v-card>
    </form-dialog>
    <ImageDialog
      :dialog="openImageDialog"
      @cancel="cancelForm"
      @close="handleImgDialogClose"
      :product="currentProduct"
    ></ImageDialog>
  </v-container>
</template>

<style>
</style>
