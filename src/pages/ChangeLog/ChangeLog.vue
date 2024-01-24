<template>
  <div>
    <div class="q-pa-md q-gutter-sm">
      <q-form @submit="gerar(changeLogPostModel)" class="col-12">
        <q-card class="my-card">
          <q-card-section class="row">
            <div class="text-h6">Gerador de change log - OpenAPI</div>

            <q-icon name="help_outline" class="cursor-pointer" size="25px">
              <q-popup-proxy
                transition-show="flip-up"
                transition-hide="flip-down"
              >
                <q-banner class="bg-primary text-white">
                  <template v-slot:avatar>
                    <q-icon name="help_outline" />
                  </template>
                  Sistema gerador de change log através de dois arquivos do tipo
                  *.yaml (openApi)
                  <br />

                  Obrigatório - Informar versão anterior e atual dos arquivos
                  nos campos correspondentes
                  <br />

                  Opcional - Informar template para compor escrita padrão de
                  texto para o change log gerado
                </q-banner>
              </q-popup-proxy>
            </q-icon>

            <q-input
              class="col-12"
              outlined
              v-model="changeLogPostModel.urlOld"
              label="Url versão anterior"
              :rules="[(val) => !!val || 'Campo obrigatório']"
            />
            <q-input
              class="col-12"
              outlined
              v-model="changeLogPostModel.urlCurrent"
              label="Url versão atual"
              :rules="[(val) => !!val || 'Campo obrigatório']"
            />

            <q-input
              outlined
              class="col-3 col-md-3 col-sm-6 col-xs-12"
              v-model="changeLogPostModel.templateDescription.templateAdded"
              label="Template - Adicionado"
            />
            <q-input
              outlined
              class="col-3 col-md-3 col-sm-6 col-xs-12"
              v-model="changeLogPostModel.templateDescription.templateEdited"
              label="Template - Alterado"
            />
            <q-input
              outlined
              class="col-3 col-md-3 col-sm-6 col-xs-12"
              v-model="changeLogPostModel.templateDescription.templateRemoved"
              label="Template - Removido"
            />
            <q-input
              outlined
              class="col-3 col-md-3 col-sm-6 col-xs-12"
              v-model="changeLogPostModel.templateDescription.templateRequired"
              label="Template - Mandatoriedade"
            />
          </q-card-section>

          <q-separator />

          <q-card-actions>
            <q-btn label="Gerar change log" type="submit" color="primary" />
          </q-card-actions>
        </q-card>
      </q-form>
    </div>

    <!-- changeLog List view-->
    <div class="q-pa-md col-12" v-if="endpointChangeLogList.length > 0">
      <div class="col-12">
        <q-btn
          color="blue-grey-6"
          icon-right="archive"
          label="Download CSV"
          no-caps
          @click="exportTable"
          class="col-2"
        />

        <q-btn
          color="blue-grey-7"
          icon-right="share"
          label="Compartilhar"
          no-caps
          @click="showUrlShare"
          class="col-2"
          style="margin-left: 20px"
        />

        <q-btn
          color="blue-grey-8"
          icon-right="diff"
          label="Diferenças"
          no-caps
          @click="showDiffFile"
          class="col-2"
          style="margin-left: 20px"
        />
      </div>
      <q-list bordered class="bordered col-12">
        <q-expansion-item
          expand-separator
          group="somegroup"
          switch-toggle-side
          class="shadow-1 overflow-hidden"
          v-for="item in endpointChangeLogList"
          v-bind:key="item.endpoint"
          :label="item.endpoint"
        >
          <q-card>
            <q-card-section>
              <div class="q-pa-xs">
                <q-table
                  :rows="item.changes"
                  :columns="columns"
                  ellipsis
                  row-key="name"
                  :visible-columns="visibleColumns"
                  :pagination="pagination"
                >
                  <template v-slot:top="props">
                    <q-space />

                    <div v-if="$q.screen.gt.xs" class="col">
                      <q-toggle
                        v-model="visibleColumns"
                        val="path"
                        label="Campo"
                      />
                      <q-toggle
                        v-model="visibleColumns"
                        val="description"
                        label="Descrição"
                      />
                      <q-toggle
                        v-model="visibleColumns"
                        val="changeType"
                        label="Tipo da alteração"
                      />
                      <q-toggle
                        v-model="visibleColumns"
                        val="oldValue"
                        label="Antes"
                      />
                      <q-toggle
                        v-model="visibleColumns"
                        val="currentValue"
                        label="Depois"
                      />
                    </div>
                    <q-select
                      v-else
                      v-model="visibleColumns"
                      multiple
                      borderless
                      dense
                      options-dense
                      :display-value="$q.lang.table.columns"
                      emit-value
                      map-options
                      :options="columns"
                      option-value="name"
                      style="min-width: 150px"
                    />

                    <q-btn
                      flat
                      round
                      dense
                      :icon="
                        props.inFullscreen ? 'fullscreen_exit' : 'fullscreen'
                      "
                      @click="props.toggleFullscreen"
                      class="q-ml-md"
                    />
                  </template>
                </q-table>
              </div>
            </q-card-section>
          </q-card>
        </q-expansion-item>
      </q-list>
    </div>
    <q-dialog v-model="sharerChangeLogDiolag">
      <q-card>
        <q-card-section class="row items-center q-pb-none">
          <div class="text-h6">Compartilhar</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-card-section>
          <q-btn
            :icon="'content_paste'"
            color="primary"
            @click="copyUrlToBoard"
          />
          <div class="col-12">
          {{ urlToShare }}
        </div>
        </q-card-section>
      </q-card>
    </q-dialog>
       <q-dialog v-model="diffFileDiolag" full-width   full-height>
      <q-card>
        <q-card-section class="row items-center q-pb-none">
         <code-diff
      :old-string="fileOld"
      :new-string="fileCurrent"
      file-name="test.txt"
      output-format="side-by-side"/>
        </q-card-section>

        <q-card-section>
         
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script lang="ts">
import { Loading, Notify, copyToClipboard } from "quasar";
import {CodeDiff} from 'v-code-diff'

import {
  ChangeLogPostModel,
  ChangeLogListModel,
  EndpointChangeLogListModel,
  InfoApiChangeLogModel
} from "components/models";
import ChangeLogListComponent from "components/ChangeLogListComponent.vue";
import { defineComponent,  ref } from "vue";
import { exportFile } from "quasar";
import axios from "axios";
const columns = [
  {
    name: "path",
    label: "Campo",
    field: "path",
    align: "left",
    sortable: true,
  },

  {
    name: "description",
    label: "O que foi alterado?",
    field: "description",
    align: "left",
    sortable: true,
  },
  {
    name: "changeType",
    label: "Tipo da alteração",
    field: "changeType",
    align: "left",
    sortable: true,
  },
  {
    name: "oldValue",
    label: "Antes",
    field: "oldValue",
    align: "left",
    sortable: true,
    format: (val: string, _row: any) =>
      `${val.length > 130 ? val.substring(0, 130) + "..." : val}`,
  },
  {
    name: "currentValue",
    label: "Depois",
    field: "currentValue",
    align: "left",
    sortable: true,
    format: (val: string, _row: any) =>
      `${val.length > 130 ? val.substring(0, 130) + "..." : val}`,
  },
];

function getParametersFromUrls(): any {
  const urlParams = new URLSearchParams(window.location.search);

  let parameters: any = {
    urlOld:
      "https://raw.githubusercontent.com/OpenBanking-Brasil/openapi/main/swagger-apis/accounts/1.0.3.yml",
    urlCurrent:
      "https://raw.githubusercontent.com/OpenBanking-Brasil/openapi/main/swagger-apis/accounts/2.0.0.yml",
    hasParameter: false,
  };

  if (urlParams.has("urlOld") && urlParams.has("urlCurrent")) {
    parameters.urlCurrent = urlParams.get("urlCurrent");
    parameters.urlOld = urlParams.get("urlOld");
    parameters.hasParameter = true;
  }

  return parameters;
}

function groupByEndPoint(list: ChangeLogListModel[]) {
  const map = new Map();
  list.forEach((item) => {
    const key = item.endpoint;
    const collection = map.get(key);
    if (!collection) {
      map.set(key, [item]);
    } else {
      collection.push(item);
    }
  });
  return map;
}

export default defineComponent({
  name: "ChangeLogPage",
  components: { ChangeLogListComponent, CodeDiff},
  data() {
    let endpointChangeLogList: EndpointChangeLogListModel[] = [];
    let infoApiChanges: InfoApiChangeLogModel = { changesLog : [], 
        currentApi : {apiName : "", url : "", version : ""},
        oldApi : {apiName : "", url : "", version : ""},  };
    let parameters = getParametersFromUrls();

    const changeLogPostModel: ChangeLogPostModel = {
      urlOld: parameters.urlOld,
      urlCurrent: parameters.urlCurrent,
      templateDescription: {
        templateAdded: 'Adicionado - "${field}"',
        templateEdited: 'Alterado - "${field}"',
        templateRemoved: 'Removido - "${field}"',
        templateRequired: "Mandatoriedade",
      },
    };

    if (parameters.hasParameter) {
      this.gerar(changeLogPostModel);
    }

    return {
      changeLogPostModel: changeLogPostModel,
      infoApiChanges: infoApiChanges,
      endpointChangeLogList: endpointChangeLogList,
    };
  },
  methods: {
    showLoading() {
      Loading.show();
    },
    hideLoading() {
      Loading.hide();
    },
    validations(changeLog: ChangeLogPostModel) {
      let strErros: string[] = [];
      if (!changeLog.urlOld && changeLog.urlOld.length == 0) {
        strErros.push("Url antiga é de preenchimento obrigatório");
      }

      if (!changeLog.urlOld && changeLog.urlOld.length == 0) {
        strErros.push("Url antiga é de preenchimento obrigatório");
      }
    },
    gerar(changeLog: ChangeLogPostModel) {
      this.endpointChangeLogList = [];
      this.showLoading();
      this.validations(changeLog);

      axios
        .post(process.env.API_URL || "", changeLog)
        .then((response: any) => {
          this.infoApiChanges = response.data;

          let resultGroup = groupByEndPoint(this.infoApiChanges.changesLog);

          for (let key of resultGroup.keys()) {
            this.endpointChangeLogList.push({
              endpoint: key,
              changes: resultGroup.get(key),
            });
          }
          this.createUrlShare();
        })
        .catch((error: any) => {
          console.log(error);

          Notify.create({
            type: "negative",
            message: error,
          });
        })
        .finally(() => {
          this.hideLoading();
        });
    },
    async copyUrlToBoard() {
      await copyToClipboard(this.urlToShare);
    },
    wrapCsvValue(val: string) {
      val = val.split('"').join('""');
      return `"${val}"`;
    },
    showUrlShare() {
      this.sharerChangeLogDiolag = true;
    },
    async showDiffFile(){
      const taskFileOld = axios
        .get(this.changeLogPostModel.urlOld);
      
      const taskFileCurrent = await axios
        .get(this.changeLogPostModel.urlCurrent);

      const [resultFileOld, resultFileCurrent ] = await Promise.all([taskFileOld, taskFileCurrent ]);

      this.fileOld = resultFileOld.data;
      this.fileCurrent = resultFileCurrent.data;
      this.diffFileDiolag = true;
    },
    createUrlShare() {
      const url = new URL(window.location.href);
      const urlParams = new URLSearchParams(window.location.search);
      urlParams.delete("urlOld");
      urlParams.delete("urlCurrent");

      urlParams.append("urlOld", this.changeLogPostModel.urlOld);
      urlParams.append("urlCurrent", this.changeLogPostModel.urlCurrent);

      url.search = urlParams.toString();
      this.urlToShare = url.href;
    },
    exportTable() {
      // naive encoding to csv format
      let contentArray: string[] = [];
      let content = "";
      contentArray.push(
        `endpoint;path;field;description;changeType;oldValue;currentValue`
      );
      this.infoApiChanges.changesLog.forEach((change) => {
        contentArray.push(
          `${this.wrapCsvValue(change.endpoint)};${this.wrapCsvValue(
            change.path
          )};${this.wrapCsvValue(change.field)};${this.wrapCsvValue(
            change.description
          )};${this.wrapCsvValue(change.changeType)};${this.wrapCsvValue(
            change.oldValue
          )};${this.wrapCsvValue(change.currentValue)}`
        );
      });
      content = contentArray.join("\n");
      const status = exportFile(`ChangeLog ${this.infoApiChanges.currentApi.apiName}__${this.infoApiChanges.oldApi.version}__${this.infoApiChanges.currentApi.version}.csv`, content, "text/csv");
    },
  },
  setup() {
    return {
      columns,
      sharerChangeLogDiolag: ref(false),
      diffFileDiolag: ref(false),
      urlToShare: ref(""),
      fileCurrent: ref(""),
      fileOld:ref(""),
      visibleColumns: ref([
        "path",
        "description",
        "changeType",
        "oldValue",
        "currentValue",
      ]),
      pagination: {
        rowsPerPage: 0, // current rows per page being displayed
      },
    };
  },
});
</script>
