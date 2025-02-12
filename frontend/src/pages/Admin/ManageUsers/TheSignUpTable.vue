<template>
  <v-card outlined class="mt-n1">
    <ConfirmationDialog
      ref="deleteTokenDialog"
      :title="$t('user.confirm-link-deletion')"
      :message="
        $t('user.are-you-sure-you-want-to-delete-the-link', {
          link: activeName,
        })
      "
      :icon="$globals.icons.alert"
      @confirm="deleteToken"
      :width="450"
      @close="closeDelete"
    />
    <v-toolbar flat>
      <v-icon large color="accent" class="mr-1">
        {{ $globals.icons.externalLink }}
      </v-icon>
      <v-toolbar-title class="headine">
        {{ $t("signup.sign-up-links") }}
      </v-toolbar-title>

      <v-spacer> </v-spacer>
      <v-dialog v-model="dialog" max-width="500">
        <template v-slot:activator="{ on, attrs }">
          <v-btn small color="success" dark v-bind="attrs" v-on="on">
            {{ $t("user.create-link") }}
          </v-btn>
        </template>
        <v-card>
          <v-app-bar dark dense color="primary">
            <v-icon left>
              {{ $globals.icons.user }}
            </v-icon>

            <v-toolbar-title class="headline">
              {{ $t("user.create-link") }}
            </v-toolbar-title>

            <v-spacer></v-spacer>
          </v-app-bar>
          <v-form ref="newUser" @submit.prevent="save">
            <v-card-text>
              <v-text-field
                v-model="editedItem.name"
                :label="$t('user.link-name')"
                :rules="[existsRule]"
                validate-on-blur
              ></v-text-field>
              <v-checkbox v-model="editedItem.admin" :label="$t('user.admin')"></v-checkbox>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="grey" text @click="close">
                {{ $t("general.cancel") }}
              </v-btn>
              <v-btn color="primary" type="submit">
                {{ $t("general.save") }}
              </v-btn>
            </v-card-actions>
          </v-form>
        </v-card>
      </v-dialog>
    </v-toolbar>
    <v-divider></v-divider>

    <v-card-text>
      <v-data-table :headers="headers" :items="links" sort-by="calories">
        <template v-slot:item.token="{ item }">
          {{ `${baseURL}/sign-up/${item.token}` }}
          <TheCopyButton :copy-text="`${baseURL}/sign-up/${item.token}`" />
        </template>
        <template v-slot:item.admin="{ item }">
          <v-btn small :color="item.admin ? 'success' : 'error'" text>
            <v-icon small left>
              {{ $globals.icons.admin }}
            </v-icon>
            {{ item.admin ? $t("general.yes") : $t("general.no") }}
          </v-btn>
        </template>
        <template v-slot:item.actions="{ item }">
          <v-btn class="mr-1" small color="error" @click="deleteItem(item)">
            <v-icon small left>
              {{ $globals.icons.delete }}
            </v-icon>
            {{ $t("general.delete") }}
          </v-btn>
        </template>
      </v-data-table>
    </v-card-text>
  </v-card>
</template>

<script>
import TheCopyButton from "@/components/UI/Buttons/TheCopyButton";
import ConfirmationDialog from "@/components/UI/Dialogs/ConfirmationDialog";
import { api } from "@/api";
import { validators } from "@/mixins/validators";
export default {
  components: { ConfirmationDialog, TheCopyButton },
  mixins: [validators],
  data() {
    return {
      dialog: false,
      activeId: null,
      activeName: null,
      headers: [
        {
          text: this.$t("user.link-id"),
          align: "start",
          sortable: false,
          value: "id",
        },
        { text: this.$t("general.name"), value: "name" },
        { text: this.$t("general.token"), value: "token" },
        { text: this.$t("user.admin"), value: "admin", align: "center" },
        { text: "", value: "actions", sortable: false, align: "center" },
      ],
      links: [],
      editedIndex: -1,
      editedItem: {
        name: "",
        admin: false,
        token: "",
        id: 0,
      },
      defaultItem: {
        name: "",
        token: "",
        admin: false,
        id: 0,
      },
    };
  },

  computed: {
    baseURL() {
      return window.location.origin;
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    updateClipboard(newClip) {
      navigator.clipboard.writeText(newClip).then(
        () => {
          console.log("Copied", newClip);
        },
        () => {
          console.log("Copy Failed", newClip);
        }
      );
    },
    async initialize() {
      this.links = await api.signUps.getAll();
    },

    async deleteToken() {
      if (await api.signUps.deleteToken(this.activeId)) {
        this.initialize();
      }
    },

    editItem(item) {
      this.editedIndex = this.links.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.activeId = item.token;
      this.activeName = item.name;
      this.editedIndex = this.links.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.$refs.deleteTokenDialog.open();
    },

    deleteItemConfirm() {
      this.links.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    async save() {
      if (this.editedIndex > -1) {
        api.links.update(this.editedItem);
        this.close();
      } else if (this.$refs.newUser.validate()) {
        api.signUps.createToken({
          name: this.editedItem.name,
          admin: this.editedItem.admin,
        });
        this.close();
      }
      await this.initialize();
    },
  },
};
</script>

<style></style>
