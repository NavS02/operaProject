<template>
  Items:{{items}}
  <button
    class="btn btn-sm btn-primary button"
    title="Print"
    @click="(onPrintClicked)"
  >
    Print PDF
  </button>
</template>

<script>
import { ref, watch, toRefs } from "vue";
import { useRoute } from "vue-router";
import { directus } from "../../../API";
import * as settings from "../../../settings";
import html2pdf from "html2pdf.js";

export default {
  setup(props) {
    // Define some variables to use in the component
    const route = useRoute();
    const collection = ref("");
    const { items } = toRefs(props);
    const response = ref(null);

    // Watch for changes in the route object
    watch(
      route,
      () => {
        // Infer the collection from the route parameters
        collection.value = route.params?.collection;
        if (!collection.value) return;
        // Retrieve the settings for the collection
        const itemSettings = settings[collection.value];
        // Fetch data for the collection
        console.log(collection.value)
        console.log(items.value)
        // getData();
      },
      { immediate: true, deep: true }
    );

  },
  props: {
    collection: {
      type: String,
      required: true,
    },
    items: {
      type: Array,
      required: true,
    },
  },
  methods: {
    onPrintClicked() {
      var opt = {
        margin: 1,
        filename: this.id + ".pdf",
        image: { type: "png", quality: 0.5 },
        html2canvas: { scale: 2, useCORS: true },
        jsPDF: { unit: "in", format: "a4", orientation: "portrait" },
      };

      html2pdf()
        .set(opt)
        .from(element)
        .save()
        .then(() => {
          console.log("PDF Generated");
        });
    },
  },
 
};
</script>

<style>
.button {
  width: 200px;
  margin-left: 50%;
}
</style>
