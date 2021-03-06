<template>
  <q-card square>
    <q-card-section>
      <q-table
        :loading="itensStore.carregando"
        title="Itens importados"
        :rows="itensStore.itensNominais"
        :columns="colunasItens"
        row-key="id"
        separator="cell"
        selection="multiple"
        :wrap-cells="true"
        no-data-label="Não foram encontrados dados."
        rows-per-page-label="Registros por página:"
        :selected-rows-label="registroPortugues"
        :filter="filtro"
        class="my-sticky-header-table"
        :pagination="paginacaoOpcoes"
        :bordered="false"
        loading-label="Carregando"
        flat
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th auto-width></q-th>
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
          </q-tr>
        </template>

        <template v-slot:top-right>
          <q-input
            borderless
            dense
            debounce="300"
            v-model="filtro"
            placeholder="Filtrar"
          >
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td>
              <q-checkbox
                left-label
                v-model="itensSelecionados"
                :val="props.row.id"
              />
            </q-td>
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              @click="props.expand = !props.expand"
            >
              {{ refatoraTexto(col.name, col.value) }}
            </q-td>
          </q-tr>
          <q-tr v-show="props.expand" :props="props">
            <q-td colspan="100%">
              <div class="text-left">
                {{ props.row.descricao }}
              </div>
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </q-card-section>
    <q-card-actions>
      <q-btn
        v-if="qtItensSelec && qtItensSelec === 1"
        dense
        color="orange"
        class="text-white"
        label="Visualizar"
        @click="verItem"
      />
    </q-card-actions>
  </q-card>
</template>

<script setup>
import { ref, reactive, watch, computed, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import { diminuiTexto, registroPortugues } from "src/helper/functions";
import { paginacaoOpcoes } from "src/helper/qtableOpcoes";
import { useItensStore } from "src/stores/itens.js";
import { useSetoresStore } from "src/stores/setores.js";
import { useDependenciasStore } from "src/stores/dependencias";
import { storeToRefs } from "pinia";

const dependenciasStore = useDependenciasStore();
const setoresStore = useSetoresStore();
const { setoresDependencias } = storeToRefs(setoresStore);
const { setoresDepsComNome } = storeToRefs(setoresStore);
const itensStore = useItensStore();
const { itensImportados } = storeToRefs(itensStore);
const itensSelecionados = ref([]);
const filtro = ref("");
const setores = ref([]);
// const { setores } = storeToRefs();
const dependencias = ref([]);
const router = useRouter();
const route = useRoute();
const inventarios = reactive([]);
// const itensInventario = ref([]);
const idInventario = ref(0);
const idItemSelecionado = ref(0);
const origemItens = ref("");
const loadingInventarios = ref(false);
const loadingItens = ref(false);

const colunasItens = reactive([
  {
    name: "id",
    align: "left",
    label: "ID",
    field: "id",
    sortable: true,
  },
  {
    name: "patrimonio",
    align: "left",
    label: "Patrimônio",
    field: "patrimonio",
    sortable: true,
  },
  {
    name: "descricao",
    align: "left",
    label: "Descrição",
    field: "descricao",
    sortable: true,
  },
  {
    name: "setor",
    align: "left",
    label: "Setor",
    field: "setor",
    sortable: true,
  },
  {
    name: "dependencia",
    align: "left",
    label: "Dependência",
    field: "dependencia",
    sortable: true,
  },
]);

const qtItensSelec = computed(() => {
  return itensSelecionados.value.length;
});

onMounted(() => {
  renderPage();
});

watch(
  () => route.path,
  (to, from) => {
    renderPage();
  }
);

watch(itensSelecionados, (nv, ov) => {
  console.log();
  if (nv.length > 1) {
    itensSelecionados.value.shift();
  }
});

function verItem() {
  if (itensSelecionados.value.length === 1) {
    router.push({
      name: "item",
      params: { idItem: itensSelecionados.value[0] },
    });
  }
}

function refatoraTexto(colNome, colValor) {
  switch (colNome) {
    default:
      return diminuiTexto(colValor);
      break;
  }
}

async function renderPage() {
  itensStore.tipoGetter = "importados";
  itensImportados.value = [];
  itensSelecionados.value = [];
  const id = route.params.idInventario || false;

  if (!id) return;

  try {
    await setoresStore.buscarSetoresDependencias(id);
    await itensStore.buscarItensImportados(id);
  } catch (error) {
  } finally {
  }
}
</script>

<style lang="sass">
.my-sticky-header-table

.q-card__section--vert
  padding: 0

.q-btn
  min-width: 5rem
</style>
