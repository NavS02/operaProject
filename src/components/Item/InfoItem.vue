<template>
  <div id="element">
    <div class="container" style="page-break-before: always">
      <div class="row"></div>
      <h4>{{ collection }}</h4>
      <table class="table table-bordered table-condensed">
        <tbody>
          <tr v-for="(value, field) in response" :key="field">
            <td>
              <h6>
                <strong>{{ field }}</strong>
              </h6>
              <span v-if="!Array.isArray(value)">{{ value }}</span>
              <ul v-else>
                <li v-for="(item, index) in value" :key="index">
                  <ul>
                    <li v-for="(itemValue, itemField) in item" :key="itemField">
                      {{ itemField }}: {{ itemValue }}
                    </li>
                    <hr />
                  </ul>
                </li>
              </ul>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
  <button
    class="btn btn-sm btn-primary button"
    title="Print"
    @click="onPrintClicked()"
  >
    Print PDF
  </button>
</template>

<script>
import { ref, watch, toRefs } from "vue";
import { useRoute, useRouter } from "vue-router";
import { directus } from "../../API";
import * as settings from "../../settings/";
import html2pdf from "html2pdf.js";

export default {
  setup(props, context) {
    const route = useRoute();
    const router = useRouter();

    const collection = ref("");
    const item = ref("");

    const { id } = toRefs(props);
    const response = ref(null);
    const responseArray = [];

    watch(
      route,
      () => {
        // infer the collection from the route
        collection.value = route.params?.collection;
        if (!collection.value) return;
        // retrieve the settings
        const itemSettings = settings[collection.value];
        getData();
      },
      { immediate: true, deep: true }
    );

    async function getData() {
      let arrayData;
      let originalResponse = {};
      try {
        if (collection.value == "opera") {
          response.value = await directus
            .items(collection.value)
            .readOne(id.value, {
              fields:
                "id,icona,lc,tsk,lir,nctr,nctn,roz,files,ogtd,ogtt,ogtv,ogtn,ogtp,qntn,qnts,sgti,sgtt,ldcs,piano,sala,parete,specifiche,deso,dess,desi,nsc,restauro,iscrizione,stemmi,localizzazione,roff,rofo,rofs,rofa,rofd,rofc,cronologia,autore,ambito,committenza,inventario,fotografia,fonte,mostra,acqt,acqn,acqd,acql,cdgg,cdgs,cdgi,stcc,stcs,mtc,misa,misu,misl,misp,misd,misn,miss,misg,misv,misr,mist,frm,adsp,adsm,cmpd,cmpn,fur,rvmd,rvmn,oss",
            });
        } else if (collection.value == "autore") {
          response.value = await directus
            .items(collection.value)
            .readOne(id.value, {
              fields: "id,auts,autr,autn,auta,auth,autb,autm,aat,pic",
            });
        }
 

        originalResponse = { ...response.value };

        for (let key in response.value) {
          if (Array.isArray(response.value[key])) {
            let responseArray = [];
            for (let i = 0; i < response.value[key].length; i++) {
              try {
                console.log()
                let item = await directus
                  .items(key)
                  .readOne(response.value[key][i]);
                delete item.user_created;
                delete item.date_created;
                delete item.user_updated;
                delete item.date_updated;
                responseArray.push(item);
              } catch (error) {
                responseArray.push(null);
                console.log("Content("+key+"):"+response.value[key][i]+" not found")
              }
            }
            response.value[key] = responseArray;
          }
        }
  console.log(response)
        for (let key in response.value) {
          if (!Array.isArray(response.value[key])) {
            if (response.value[key] !== originalResponse[key]) {
            }
          } else {
            for (let i = 0; i < response.value[key].length; i++) {
              if (response.value[key][i] !== originalResponse[key][i]) {
              }
            }
          }
        }
      } catch (error) {}
    }

    return { response };
  },
  props: {
    collection: {
      type: String,
      required: true,
    },
    id: {
      type: String,
      required: true,
    },
  },
  methods: {
    onPrintClicked() {
      var opt = {
        margin: 1,
        filename: this.id + ".pdf",
        image: { type: "png", quality: 0.98 },
        html2canvas: { scale: 2, useCORS: true },
        jsPDF: { unit: "in", format: "letter", orientation: "portrait" },
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
