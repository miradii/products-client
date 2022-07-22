
<script>
import { required, min, max } from "vee-validate/dist/rules";
import {
  extend,
  ValidationObserver,
  ValidationProvider,
  setInteractionMode,
} from "vee-validate";
import FormDialog from "~/components/form-dialog.vue";

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
      openAddForm: false,
      openDeleteDialog: false,
      openEditForm: false,
      name: "",
      currentCategory: { name: "", id: "" },
    };
  },
  async created() {
    const { data } = await this.$axios.get("/api/categories");
    this.categories.push(...data);
  },
  methods: {
    async handleAdd() {
      try {
        const newCategory = { name: this.currentCategory.name };
        const { data } = await this.$axios.post("/api/categories", newCategory);
        this.categories.push(data);
        this.closeForm();
        this.$store.commit("snackbar/showMessage", {
          content: "Category created",
          color: "success",
        });
      } catch (e) {
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
    async handleDelete() {
      try {
        console.log(this.currentCategory);
        const { data } = await this.$axios.delete(
          `/api/categories/${this.currentCategory.id}`
        );
        this.categories = this.categories.filter(
          (category) => category.id != data.id
        );

        this.closeForm();
        this.$store.commit("snackbar/showMessage", {
          content: "Category deleted",
          color: "success",
        });
      } catch (e) {
        this.closeForm();
        this.$store.commit("snackbar/showMessage", {
          content: "That category can not be deleted",
          color: "info",
        });
      }
    },
    async handleEdit() {
      try {
        const newCategory = { name: this.currentCategory.name };
        const { data } = await this.$axios.patch(
          `/api/categories/${this.currentCategory.id}`,
          newCategory
        );
        this.categories.find((cat) => cat.id === this.currentCategory.id).name =
          this.currentCategory.name;
        this.closeForm();
        this.$store.commit("snackbar/showMessage", {
          content: "Category updated",
          color: "success",
        });
      } catch (e) {
        this.$store.commit("snackbar/showMessage", {
          content: "failed to update category",
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
    closeForm() {
      this.openAddForm = false;
      this.openDeleteDialog = false;
      this.openEditForm = false;
      this.currentCategory.name = "";
      this.currentCategory.id = "";
    },
    openDelete(category) {
      this.currentCategory.name = category.name;
      this.currentCategory.id = category.id;
      this.openDeleteDialog = true;
    },
    openEdit(category) {
      this.currentCategory.name = category.name;
      this.currentCategory.id = category.id;
      this.openEditForm = true;
    },
  },
};
</script>


  <template>
  <v-container>
    <v-card class="pa-7" dark>
      <v-subheader large>Categories</v-subheader>
      <v-card
        v-for="category in categories "
        :key="category.name"
        color="#f9f3e5"
        class="ma-5"
        light
      >
        <v-row class="d-flex px-3" justify="space-between">
          <v-card-title class="text-subtitle-1">
            {{
            category.name
            }}
          </v-card-title>

          <v-card-actions>
            <v-btn @click="openEdit(category)" class="text-subtitle-2" color="blue accent-4" icon>
              <v-icon medium>mdi-pencil</v-icon>
            </v-btn>
            <v-btn @click="openDelete(category)" class="text-subtitle-2" color="red accent-4" icon>
              <v-icon medium>mdi-delete</v-icon>
            </v-btn>
          </v-card-actions>
        </v-row>
      </v-card>
      <v-card-actions class="pa-4">
        <v-btn @click="openAddForm = true">add</v-btn>
      </v-card-actions>
    </v-card>
    <form-dialog :dialog="openAddForm || openEditForm">
      <v-card class="pa-4" width="100%">
        <v-subheader v-if="openAddForm">Add Category</v-subheader>
        <v-subheader v-else>Edit Category</v-subheader>
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
                v-model="currentCategory.name"
                :error-messages="errors"
                name="name"
                label="Category Name"
                type="text"
                required
                outlined
              ></v-text-field>
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
            <v-btn @click.prevent="closeForm" color="red accent-3" large>Cancel</v-btn>
          </v-card-actions>
        </ValidationObserver>
      </v-card>
    </form-dialog>
    <form-dialog :dialog="openDeleteDialog">
      <v-card class="pa-4" width="100%">
        <v-subheader>Are you sure you want to delete category {{ currentCategory.name }}</v-subheader>
        <v-card-actions>
          <v-btn @click.prevent="handleDelete" large color="green accent-2" light>Yes</v-btn>
          <v-btn @click.prevent="closeForm" color="red accent-3" large>No</v-btn>
        </v-card-actions>
      </v-card>
    </form-dialog>
  </v-container>
</template>
<style>
</style>
