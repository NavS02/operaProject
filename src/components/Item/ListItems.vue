<template>
  <header class="mb-2">
    <router-link :to="createLink" custom v-slot="{ isActive, href, navigate }">
      <button class="btn btn-sm btn-primary" @click="navigate">
        <font-awesome-icon icon="fa-solid fa-plus" fixed-width />
        <span class="ms-1">New</span>
      </button>
      <button
        class="btn btn-sm btn-secondary"
        style="float: right"
        v-if="printer"
        @click="printItems()"
      >
        <font-awesome-icon icon="fa-solid fa-print" />
        <span class="ms-1">Print</span>
      </button>
    </router-link>
  </header>

  <Table class="w-100" :items="items" :fields="fields">
    <!-- dynamically assigna labels to each thead -->
    <!-- <template v-for="(field, index) in fields" :key="index" #[`head(${field.key})`] >
            {{ field.label ? field.label : field.key }}
        </template> -->

    <template #cell(actions)="{ item, field, value }">
      <div class="actions">
        <input
          class="form-check-input"
          type="checkbox"
          :id="item.id"
          :value="item.id"
          @change="onCheckPrinter(item)"
        />
        <button
          title="edit"
          class="btn btn-sm btn-light"
          @click="onEditClicked(item)"
        >
          <font-awesome-icon icon="fa-solid fa-pencil" fixed-width />
        </button>
        <button
          title="delete"
          class="btn btn-sm btn-light text-danger"
          @click="onDeleteClicked(item)"
        >
          <font-awesome-icon icon="fa-solid fa-trash" fixed-width />
        </button>
        <button
          title="Info"
          class="btn btn-sm btn-light"
          @click="onInfoClicked(item)"
        >
          <font-awesome-icon icon="fa-solid fa-eye" />
        </button>
      </div>
    </template>
  </Table>
</template>

<script>
import { ref, computed, watch } from "vue";
import { useRoute, useRouter } from "vue-router";
import { directus } from "../../API/";
import * as settings from "../../settings/";
import Table from "../common/Table/Table.vue";

export default {
  components: { Table },
  setup() {
    const route = useRoute();
    const router = useRouter();
    // infer the collection from the route
    const collection = ref("");
    const items = ref([]);
    const fields = ref([]);
    const printer = ref(false);
    const itemSelected = ref([]);

    // watch the route and update data based on the collection param
    watch(
      route,
      () => {
        collection.value = route.params?.collection;
        if (!collection.value) return;
        // retrieve the settings
        const itemSettings = settings[collection.value];
        // define the subset of fields you need to view in the table
        const collectionFields = itemSettings.tableFields();
        fields.value = collectionFields;
        fetchData();
      },
      { immediate: true, deep: true }
    );

    const createLink = computed(() => ({
      name: "createItem",
      params: { collection: collection.value },
    }));

    async function fetchData() {
      const response = await directus
        .items(collection.value)
        .readByQuery({ limit: -1 });
      const { data = [] } = response;
      items.value = data;
    }
    async function deleteItem(item) {
      const { id } = item;
      await directus.items(collection.value).deleteOne(id);
      fetchData();
    }
    function onEditClicked(item) {
      router.push({
        name: "editItem",
        params: { id: item.id, collection: collection.value },
      });
    }
    function onDeleteClicked(item) {
      const confirmed = confirm("Are you sure you want to delete this item?");
      if (confirmed) deleteItem(item);
    }

    function printItems() {
      console.log(itemSelected.value)

      router.push({
        name: "infoItems",
        params: { collection: collection.value, items:itemSelected.value  },

      });
    }
    function onInfoClicked(item) {
      router.push({
        name: "infoItem",
        params: { collection: collection.value, id: item.id },
      });
    }
    function onCheckPrinter(item) {
      let clicked = document.getElementById(item.id).checked;
      if (clicked == true) {
        printer.value = true;
        itemSelected.value.push(item);
      } else {
        const index = itemSelected.value.findIndex(
          (selectedItem) => selectedItem.id === item.id
        );
        if (index !== -1) {
          itemSelected.value.splice(index, 1);
        }
        if (itemSelected.value.length === 0) {
          printer.value = false;
        }
      }
    }

    return {
      items,
      fields,
      createLink,
      printer,
      itemSelected,
      onEditClicked,
      onDeleteClicked,
      onInfoClicked,
      onCheckPrinter,
      printItems
    };
  },
  props: {
    collection: { type: String, default: "" },
  },
};
</script>

<style scoped>
.actions {
  display: flex;
  gap: 5px;
  justify-content: flex-end;
}
</style>
